## Automated Search for Specific Files Inside S3 ZIP Archives

```bash
#!/bin/bash

# Example S3 path:
# s3://john-doe-archival/QAR/demo-qar_archive_2024/done/0000-181124-TAIL-001-QAR.zip

# Configuration
BUCKET="john-doe-archival"
PREFIX="QAR/demo-qar_archive_2024/done/"
EXTENSION="\.adr$|\.dat$"   # File extensions to search for inside ZIPs
LOGFILE="matched_files.txt"
DOWNLOAD_DIR="./downloads"

# Create or clear the log file
> "$LOGFILE"

# Create the download directory if it doesn't exist
mkdir -p "$DOWNLOAD_DIR"

# List all .zip files under the prefix
aws s3 ls "s3://$BUCKET/$PREFIX" --recursive | awk '{print $4}' | grep '\.zip$' | while read -r ZIPKEY; do
    echo "Processing: $ZIPKEY"

    # Define local path for temporary download
    LOCAL_ZIP="$DOWNLOAD_DIR/$(basename "$ZIPKEY")"

    # Download the ZIP file from S3
    aws s3 cp "s3://$BUCKET/$ZIPKEY" "$LOCAL_ZIP"

    # List contents of the ZIP and filter by extension
    unzip -l "$LOCAL_ZIP" | awk '{print $4}' | grep -E "$EXTENSION" | while read -r MATCH; do
        echo "$ZIPKEY -> $MATCH" >> "$LOGFILE"
        echo "Found: $MATCH in $ZIPKEY"
    done

    # Remove the downloaded ZIP file to save space
    rm -f "$LOCAL_ZIP"
done
```

---

## Code Explanation

1. **Configuration**  
   - Defines dummy S3 bucket (`john-doe-archival`), prefix path (`QAR/demo-qar_archive_2024/done/`), and target file extensions (`.adr` or `.dat`).  
   - Sets up a log file (`matched_files.txt`) and a local download directory (`./downloads`).

2. **Prepare environment**  
   - Clears the log file at the start so results are fresh.  
   - Ensures the download directory exists.

3. **List ZIP files in S3**  
   - Uses `aws s3 ls` to list all objects under the prefix.  
   - Filters only `.zip` files.  
   - Iterates through each ZIP file found.

4. **Download each ZIP locally**  
   - Copies the ZIP file from S3 to the local `downloads` directory.  
   - Names the local file the same as the original.

5. **Inspect ZIP contents**  
   - Runs `unzip -l` to list the contents of the ZIP file.  
   - Filters for files matching the target extensions (`.adr` or `.dat`).  
   - Logs matches to `matched_files.txt` in the format:  
     ```
     <ZIP file path> -> <matched file inside ZIP>
     ```

6. **Logging and feedback**  
   - Prints a message to the console whenever a match is found.  
   - Maintains a persistent log file for later review.

7. **Cleanup**  
   - Deletes the local ZIP file after inspection to save disk space.  
   - Ensures the script doesn’t accumulate unnecessary files.

---
