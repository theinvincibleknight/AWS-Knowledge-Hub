## Automated Workflow: Download from S3, Organize, and Upload to SFTP

```bash
#!/bin/bash

# Define variables
bucket="john-doe"
sftp_host="sftp.demo-server.com"
sftp_user="demo-user"
sftp_pass="demo-password"
remote_base="/Demo/Remote/Path"
temp_dir="./s3-temp"

# Create temporary directory
mkdir -p "$temp_dir"

# Define fleet mapping
declare -A fleet_map=(
["TAIL-001"]="B737-8"
["TAIL-002"]="B737-8"
["TAIL-003"]="B737-8200"
["TAIL-004"]="B737-8200"
)

# Loop through S3 files
aws s3 ls "s3://$bucket/prod/boeing-cfm/done/2025/02/" --recursive | grep -E '_ADR-' | awk '{print $4}' | while read file; do
  filename=$(basename "$file")

  # Download file locally
  aws s3 cp "s3://$bucket/$file" "$temp_dir/$filename"

  # Extract aircraft tail (assumes it's the first part before '_REPORT')
  tail_code=$(echo "$filename" | cut -d'_' -f1)
  fleet_type="${fleet_map[$tail_code]}"

  # Extract year and month from timestamp after QAR_
  timestamp=$(echo "$filename" | grep -oP 'QAR_\K[0-9]{14}')
  year=${timestamp:0:4}
  month=${timestamp:4:2}

  # Construct remote path
  remote_path="$remote_base/$fleet_type/$year/$month"

  # Upload via SFTP
  lftp -u "$sftp_user","$sftp_pass" sftp://$sftp_host <<EOF
put "$temp_dir/$filename" -o "$remote_path/$filename"
bye
EOF

  # Log the transfer
  echo "$(date '+%Y-%m-%d %H:%M:%S') = $file = $remote_path"  >> log_transfer.txt

  # Delete local copy
  rm "$temp_dir/$filename"
done
```

---

## Code Explanation

1. **Variable setup**  
   - Defines S3 bucket (`john-doe`), SFTP server details (`sftp.demo-server.com`, `demo-user`, `demo-password`), and a base remote path.  
   - Creates a temporary local directory (`./s3-temp`) to hold downloaded files.

2. **Fleet mapping**  
   - Uses a Bash associative array to map dummy aircraft tail numbers (`TAIL-001`, etc.) to fleet types (`B737-8`, `B737-8200`).  
   - This allows the script to categorize files by aircraft type.

3. **List and filter S3 files**  
   - Runs `aws s3 ls` on a dummy path (`prod/boeing-cfm/done/2025/02/`).  
   - Filters only files containing `_ADR-` in their names.  
   - Extracts the full key (path) of each file.

4. **Download each file**  
   - Uses `aws s3 cp` to copy the file locally into the temporary directory.  
   - Extracts the filename using `basename`.

5. **Parse metadata from filename**  
   - Extracts the aircraft tail code from the filename.  
   - Looks up the fleet type using the `fleet_map`.  
   - Extracts a timestamp (after `QAR_`) to determine year and month.

6. **Build remote path**  
   - Constructs a folder structure on the SFTP server:  
     ```
     /Demo/Remote/Path/<fleet_type>/<year>/<month>
     ```

7. **Upload via SFTP**  
   - Uses `lftp` with the dummy credentials to upload the file to the remote path.  
   - Ensures the file is placed in the correct fleet/year/month folder.

8. **Logging**  
   - Appends a line to `log_transfer.txt` with the timestamp, source file, and destination path.  
   - Useful for auditing and debugging.

9. **Cleanup**  
   - Deletes the local copy of the file after successful upload to save space.

---
👉 *List files in S3 → Download locally → Parse metadata → Upload to SFTP → Log → Clean up.*

---
