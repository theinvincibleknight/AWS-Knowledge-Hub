## Automated File Transfer: Rename and Upload S3 Objects to SFTP

```bash
#!/bin/bash

# Define variables
bucket="john-doe-archival"
sftp_host="sftp.demo-server.com"
sftp_user="demo-user"
sftp_pass="demo-password"
remote_path="/demo-data"
temp_dir="./s3-temp"

# Create temporary directory
mkdir -p "$temp_dir"

# Loop through S3 files matching DAR.DAT
aws s3 ls "s3://$bucket/QAR/demo-folder/" --recursive | grep 'DAR.DAT' | awk '{print $4}' | while read file; do
  # Extract original filename
  original_filename=$(basename "$file")

  # Generate unique name (timestamp + random suffix)
  timestamp=$(date +"%Y%m%d%H%M%S%3N")
  random_suffix=$RANDOM
  new_filename="${timestamp}_${random_suffix}_DAR.DAT"

  # Download file locally
  aws s3 cp "s3://$bucket/$file" "$temp_dir/$new_filename"

  # Upload to SFTP
  lftp -u "$sftp_user","$sftp_pass" sftp://$sftp_host <<EOF
put "$temp_dir/$new_filename" -o "$remote_path/$new_filename"
bye
EOF

  # Log original and new filename
  echo "Uploaded: $file as $new_filename" >> transfer.log.txt

  # Delete local copy
  rm "$temp_dir/$new_filename"
done
```

---

## Code Explanation

1. **Variable setup**  
   - Defines dummy S3 bucket (`john-doe-archival`), SFTP server (`sftp.demo-server.com`), credentials (`demo-user`, `demo-password`), and remote path (`/demo-data`).  
   - Creates a temporary local directory (`./s3-temp`) to hold downloaded files.

2. **List and filter S3 files**  
   - Uses `aws s3 ls` to list objects in a demo folder (`QAR/demo-folder/`).  
   - Filters only files containing `DAR.DAT` in their names.  
   - Extracts the full key (path) of each matching file.

3. **Filename handling**  
   - Extracts the original filename with `basename`.  
   - Generates a new unique filename by combining a timestamp (`YYYYMMDDHHMMSSmmm`) and a random suffix, ensuring no collisions.  
   - Appends `_DAR.DAT` to preserve the file type.

4. **Download from S3**  
   - Copies the file locally from S3 using the new filename.  
   - This avoids overwriting files if multiple objects are processed.

5. **Upload to SFTP**  
   - Uses `lftp` with dummy credentials to upload the file to the SFTP server.  
   - Places the file in the correct remote path (`/demo-data`) with the new unique filename.

6. **Logging**  
   - Appends a line to `transfer.log.txt` recording the original S3 key and the new filename used for upload.  
   - Provides traceability between source and destination.

7. **Cleanup**  
   - Deletes the local copy after successful upload to save disk space.  
   - Ensures the temporary directory doesn’t accumulate files.

---
