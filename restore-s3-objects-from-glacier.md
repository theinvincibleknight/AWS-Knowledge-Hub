# Restore Objects from Glacier Deep Archive and Transfer to Another AWS Account

Before transferring objects to another bucket, you must first restore the objects. After restoration, you can perform the copy or move action to another account.

## Step 1: List Objects in the S3 Bucket

First, list all the objects in the S3 bucket and save them to a text file:

```bash
aws s3 ls s3://bucket-name --recursive --human-readable | awk '{print $5}' > s3_objects.txt
```

## Step 2: Restore Objects from Glacier Deep Archive

You can use the following script to restore each object from the Glacier Deep Archive:

```bash
#!/bin/bash

# Set the log file name
LOG_FILE="restore_log.txt"

# Loop through the objects and restore from Glacier Deep Archive
while read line; do
  echo "Restoring object: $line" >> $LOG_FILE
  aws s3api restore-object --bucket boeing-sdm-backup --key "$line" --restore-request Days=1 >> $LOG_FILE 2>&1
  if [ $? -eq 0 ]; then
    echo "Restore successful for object: $line" >> $LOG_FILE
  else
    echo "Restore failed for object: $line" >> $LOG_FILE
  fi
done < s3_objects.txt

echo "Restore process completed. Log file: $LOG_FILE"
```

### Explanation of the Script

- Sets the log file name to `restore_log.txt`.
- Loops through the `s3_objects.txt` file and executes the `aws s3api restore-object` command for each object.
- Redirects both standard output and standard error to the log file.
- Checks the exit status of the previous command to determine if the restore was successful and logs the appropriate message.
- After the loop completes, it prints a message indicating that the restore process is completed along with the log file name.

## Step 3: Assign Permissions

After all the objects in the bucket are restored, you need to assign permissions to the IAM user and bucket to perform this action.

1. In the source account, create an IAM user.
2. Create an AWS IAM customer-managed policy that grants access to retrieve objects from the source bucket and copy them into the destination bucket. The policy should look similar to the following:

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": [
        "s3:ListBucket",
        "s3:GetObject"
      ],
      "Resource": [
        "arn:aws:s3:::source-DOC-EXAMPLE-BUCKET",
        "arn:aws:s3:::source-DOC-EXAMPLE-BUCKET/*"
      ]
    },
    {
      "Effect": "Allow",
      "Action": [
        "s3:ListBucket",
        "s3:PutObject",
        "s3:PutObjectAcl"
      ],
      "Resource": [
        "arn:aws:s3:::destination-DOC-EXAMPLE-BUCKET",
        "arn:aws:s3:::destination-DOC-EXAMPLE-BUCKET/*"
      ]
    }
  ]
}
```

3. Attach this customer-managed policy to the IAM user you created.

## Step 4: Modify Destination Bucket Policy

In the destination account, modify the destination bucket policy to grant the source account permissions to upload objects. Include a condition that requires object uploads to set the ACL to `bucket-owner-full-control`. The policy should look similar to the following:

```json
{
  "Version": "2012-10-17",
  "Id": "Policy1611277539797",
  "Statement": [
    {
      "Sid": "Stmt1611277535086",
      "Effect": "Allow",
      "Principal": {
        "AWS": "arn:aws:iam::222222222222:user/Jane"
      },
      "Action": "s3:PutObject",
      "Resource": "arn:aws:s3:::destination-DOC-EXAMPLE-BUCKET/*",
      "Condition": {
        "StringEquals": {
          "s3:x-amz-acl": "bucket-owner-full-control"
        }
      }
    },
    {
      "Sid": "Stmt1611277877767",
      "Effect": "Allow",
      "Principal": {
        "AWS": "arn:aws:iam::222222222222:user/Jane"
      },
      "Action": "s3:ListBucket",
      "Resource": "arn:aws:s3:::destination-DOC-EXAMPLE-BUCKET"
    }
  ]
}
```

**Reference:** [AWS Knowledge Center](https://repost.aws/knowledge-center/copy-s3-objects-account)

## Step 5: Verify File Copying

Now, check if you can copy files from the source account to the destination account's S3 bucket using the following command:

```bash
aws s3 cp s3://bucket-name/testfile.txt s3://bucket-name/test-folder/ --acl bucket-owner-full-control
```

### Optional: Check the Status of Restored Objects

You can check the status of the restored object using the bucket name and key name (Object Name) with the following command:

```bash
aws s3api head-object --bucket bucket-name --key key-name
```

## Step 6: Copy All Restored Files

If you are able to copy files from the source account's S3 bucket to the destination account's S3 bucket, you can loop through the `s3_objects.txt` file to copy all the restored files from one S3 account bucket to another using the following script:

```bash
#!/bin/bash

# Set the log file name
LOG_FILE="copy_log.txt"

while read line; do
    echo "Copying object: $line" >> $LOG_FILE
    aws s3 cp s3://bucket-name/$line s3://bucket-name/test-folder/ --acl bucket-owner-full-control >> $LOG_FILE 2>&1
    if [ $? -eq 0 ]; then
        echo "Copy successful for object: $line" >> $LOG_FILE
    else
        echo "Copy failed for object: $line" >> $LOG_FILE
    fi
done < s3_objects.txt

echo "Copy process completed. Log file: $LOG_FILE"
```

Now you should be able to successfully restore objects from Glacier Deep Archive and copy them into an S3 bucket of another AWS account. Ensure that you have the necessary permissions and policies in place to facilitate the transfer.
