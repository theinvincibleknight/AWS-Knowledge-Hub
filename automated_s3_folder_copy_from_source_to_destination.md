## Automated S3 Folder Copy from Source to Destination

```bash
#!/bin/bash

# Set the log file name
LOG_FILE="s3-copy-log.txt"

# List of folders to copy
FOLDERS=(
    "demo-eff"
    "demo-cognito"
    "demo-dfdr"
    "demo-ecom-affiliate"
    "demo-efr-comments"
    "demo-glp"
)

# Loop through each folder and copy recursively
for folder in "${FOLDERS[@]}"; do
    echo "Copying folder: $folder" >> "$LOG_FILE"
    aws s3 cp "s3://source-bucket/prod/$folder" "s3://destination-bucket/prod/$folder" \
      --recursive --acl bucket-owner-full-control >> "$LOG_FILE" 2>&1
    if [ $? -eq 0 ]; then
        echo "$(date '+%Y-%m-%d %H:%M:%S') Copy successful for folder: $folder" >> "$LOG_FILE"
    else
        echo "$(date '+%Y-%m-%d %H:%M:%S') Copy failed for folder: $folder" >> "$LOG_FILE"
    fi
done

echo "$(date '+%Y-%m-%d %H:%M:%S') = Copying process completed. Log file: $LOG_FILE"
```

---

## Code Explanation

1. **Log file setup**  
   - Defines a log file (`s3-copy-log.txt`) where all copy operations and results will be recorded.

2. **Folder list**  
   - Creates an array of folder names (`demo-eff`, `demo-cognito`, etc.).  
   - These represent subdirectories under the `prod` path in the source bucket.

3. **Copy loop**  
   - Iterates through each folder in the list.  
   - For each folder, runs `aws s3 cp` to copy all files recursively from the **source bucket** (`source-bucket`) to the **destination bucket** (`destination-bucket`).  
   - The `--acl bucket-owner-full-control` flag ensures the destination bucket owner has full access to the copied files.

4. **Error handling**  
   - Checks the exit status (`$?`) of the copy command.  
   - If successful, logs a timestamped “Copy successful” message.  
   - If failed, logs a timestamped “Copy failed” message.

5. **Completion message**  
   - After all folders are processed, writes a final timestamped line indicating the process is complete and points to the log file.

---
