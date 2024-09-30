# AWS S3 Commands

1. **Copy a Folder to Your S3 Bucket:**
   ```bash
   aws s3 cp /path/of/source s3://your-bucket-name/ --recursive
   ```

2. **Sync the Local Folder to an S3 Bucket:**
   ```bash
   aws s3 sync /path/of/source s3://your-bucket-name/
   ```

3. **Exclude Certain Files:**
   To exclude certain files or directories from the sync process, use the `--exclude` option:
   ```bash
   aws s3 sync /path/of/source s3://your-bucket-name/ --exclude "*.tmp"
   ```

4. **Include Only Certain Files:**
   To include only specific files, use the `--include` option:
   ```bash
   aws s3 sync /path/of/source s3://your-bucket-name/ --include "*.jpg"
   ```

5. **Delete Files in the Destination Not Present in the Source:**
   To ensure that files not present in the source are deleted from the destination, use the `--delete` option:
   ```bash
   aws s3 sync /path/of/source s3://your-bucket-name/ --delete
   ```

6. **Get the Count of the Files in an S3 Bucket Using PowerShell:**
   ```powershell
   (aws s3 ls s3://your-bucket-name --recursive | Measure-Object -Line).Lines
   ```

7. **Get the Count of the Files in an S3 Bucket Using Linux Shell:**
   ```bash
   aws s3 ls s3://your-bucket-name --recursive | wc -l
   ```

8. **Create a Pre-signed URL for Download:**
   ```bash
   aws s3 presign s3://your-bucket-name/sub-folder/file.mp4 --expires-in 86400
   ```

9. **Create a Pre-signed URL for Upload:**
   ```bash
   aws s3 presign s3://your-bucket-name/sub-folder/file-to-upload.mp4 --expires-in 86400 --acl bucket-owner-full-control
   ```

10. **List All Objects in a Bucket:**
   ```bash
   aws s3 ls s3://your-bucket-name/ --recursive
   ```

11. **Remove a File from an S3 Bucket:**
    ```bash
    aws s3 rm s3://your-bucket-name/sub-folder/file.mp4
    ```

12. **Empty a Bucket (Delete All Objects):**
    ```bash
    aws s3 rm s3://your-bucket-name --recursive
    ```

13. **Copy a File from S3 to Local:**
    ```bash
    aws s3 cp s3://your-bucket-name/sub-folder/file.mp4 /path/to/destination/
    ```

14. **Sync an S3 Bucket to a Local Folder:**
    ```bash
    aws s3 sync s3://your-bucket-name/ /path/to/local/destination/
    ```

15. **Change Storage Class of an Object:**
    ```bash
    aws s3 cp s3://your-bucket-name/sub-folder/file.mp4 s3://your-bucket-name/sub-folder/file.mp4 --storage-class STANDARD_IA
    ```

16. **Set Bucket Policy:**
    ```bash
    aws s3api put-bucket-policy --bucket your-bucket-name --policy file://policy.json
    ```

167. **Get Bucket Location:**
   ```bash
   aws s3api get-bucket-location --bucket your-bucket-name
   ```

18. **Enable Versioning on a Bucket:**
   ```bash
   aws s3api put-bucket-versioning --bucket your-bucket-name --versioning-configuration Status=Enabled
   ```

19. **List Bucket Versions:**
   ```bash
   aws s3api list-object-versions --bucket your-bucket-name
   ```

20. **Restore a Deleted Object (if versioning is enabled):**
   ```bash
   aws s3api restore-object --bucket your-bucket-name --key sub-folder/file.mp4 --restore-request Days=1
   ```

21. **Get Object Metadata:**
   ```bash
   aws s3api head-object --bucket your-bucket-name --key sub-folder/file.mp4
   ```

22. **Copy an Object within S3:**
   ```bash
   aws s3 cp s3://your-bucket-name/source-file.mp4 s3://your-bucket-name/destination-file.mp4
   ```
   
### Notes:
- Ensure that you have the necessary permissions to perform these actions in your AWS account.
- Replace `your-bucket-name`, `/path/of/source`, and other placeholders with your actual bucket name and paths.
- The `--expires-in` parameter for pre-signed URLs is specified in seconds (e.g., `86400` seconds = 24 hours).
- The `--acl` option in the upload command specifies the access control list for the uploaded file. Adjust as necessary based on your requirements.
