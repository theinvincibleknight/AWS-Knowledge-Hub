## Automated Workflow: Download, Unzip, Upload, and Cleanup for S3 Files

```bash
#!/bin/bash

# Check if target name is provided
if [ -z "$1" ]; then
    echo "Usage: sh script.sh <TargetName>"
    exit 1
fi

TARGET="$*"
ZIP_DIR="/home/ubuntu/demo/S3/QAR/zip"
UNZIP_DIR="/home/ubuntu/demo/S3/QAR/unzip"
LOG="/home/ubuntu/demo/S3/QAR/$TARGET.txt"

# Logging function
log() {
    echo "$(date '+%Y-%m-%d %H:%M:%S') - $1"
}

log "Starting process for TARGET: $TARGET" >> "$LOG"

# Step 1: Download the zip file from S3
aws s3 cp "s3://john-doe-archival/QAR/2022/${TARGET}.zip" "$ZIP_DIR/"
log "Downloaded ${TARGET}.zip from S3" >> "$LOG"

# Step 2: Unzip the file
unzip "$ZIP_DIR/${TARGET}.zip" -d "$UNZIP_DIR/"
log "Unzipped ${TARGET}.zip to $UNZIP_DIR" >> "$LOG"

# Step 3: Upload unzipped contents to S3
aws s3 cp "$UNZIP_DIR/" "s3://john-doe-archival/QAR/2022_unzip/" --recursive
log "Uploaded unzipped contents to S3" >> "$LOG"

# Step 4: Remove the downloaded zip file
rm "$ZIP_DIR/${TARGET}.zip"
log "Removed local zip file ${TARGET}.zip" >> "$LOG"

# Step 5: Remove the extracted folder
sudo rm -rf "$UNZIP_DIR/${TARGET}/"
log "Removed extracted folder ${TARGET}/" >> "$LOG"

log "Process completed for TARGET: $TARGET" >> "$LOG"
```

---

## Code Explanation (Logic Walkthrough)

1. **Argument check**  
   - Ensures a target name is provided when running the script.  
   - If missing, prints usage instructions and exits.

2. **Variable setup**  
   - Defines dummy local directories for ZIP downloads (`zip`) and extracted files (`unzip`).  
   - Sets a log file path (`$TARGET.txt`) to record progress.  
   - Uses a dummy S3 bucket (`john-doe-archival`) for safe public reference.

3. **Logging function**  
   - A helper function `log()` writes timestamped messages to the log file.  
   - Provides traceability for each step of the process.

4. **Download ZIP from S3**  
   - Uses `aws s3 cp` to copy the target ZIP file from the bucket to the local ZIP directory.  
   - Logs the action.

5. **Unzip locally**  
   - Extracts the contents of the downloaded ZIP into the local unzip directory.  
   - Logs the extraction.

6. **Upload extracted files back to S3**  
   - Copies the unzipped contents recursively to a new S3 prefix (`2022_unzip`).  
   - Ensures all files are uploaded.  
   - Logs the upload.

7. **Cleanup**  
   - Deletes the local ZIP file to save space.  
   - Removes the extracted folder using `rm -rf` to avoid clutter.  
   - Logs both cleanup actions.

8. **Completion message**  
   - Writes a final log entry confirming the process is complete for the given target.

---

## Usage

### Prerequisites
- AWS CLI installed and configured with valid credentials (`aws configure`).  
- `unzip` utility installed on the system.  
- Proper IAM permissions to read/write from the specified S3 buckets.  

### Run Command
```bash
sh script.sh <TargetName>
```

### Example
```bash
sh script.sh TAIL-001
```

- This will download `TAIL-001.zip` from S3, unzip it locally, upload the contents back to S3 under the `2022_unzip` prefix, and then clean up local files.  
- A log file named `TAIL-001.txt` will be created under `/home/ubuntu/demo/S3/QAR/` with details of the process.  

---
