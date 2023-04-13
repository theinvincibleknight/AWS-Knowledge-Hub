# Grant Access to User-Specific Folders in an Amazon S3 Bucket

If the IAM user and S3 bucket belong to the same AWS account, then you can grant the user access to a specific bucket folder using an IAM policy. As long as the bucket policy doesn't explicitly deny the user access to the folder, you don't need to update the buck et policy if access is granted by the IAM policy.

If the IAM identity (user or role) and the S3 bucket belong to different AWS accounts, then you must grant access on both the IAM policy and the bucket policy.

The following example IAM policy allows a user to download objects from the folder **`DOC-EXAMPLE-BUCKET/media`** using the Amazon S3 console. The policy includes these statements:

- **`AllowUserToSeeBucketListInTheConsole`** allows the user to list the buckets that belong to their AWS account. The user needs this permission to be able to navigate to the bucket using the console.
- **`AllowRootAndHomeListingOfCompanyBucket`** allows the user to list the folders within **`DOC-EXAMPLE-BUCKET`**, which the user needs to be able to navigate to the folder using the console. The statement also allows the user to search on the prefix **media/** & **media/home** using the console. The **`"s3:delimiter":["/"]`**, which specifies the forward slash character (/) as the delimiter for folders within the path to an object. It's a best practice to specify the delimiter if the user makes requests using the AWS CLI or the Amazon S3 API.
- **`AllowListingOfUserFolder`** allows the user to list the contents within **`DOC-EXAMPLE-BUCKET/media/home`**.
- **`AllowAllS3ActionsInUserFolder`** allows the user to download **`(s3:GetObject)`** and upload **`(s3:PutObject)`** objects to the folder **`DOC-EXAMPLE-BUCKET/media/home/`**.

```json
{
 "Version":"2012-10-17",
 "Statement": [
   {
     "Sid": "AllowUserToSeeBucketListInTheConsole",
     "Action": ["s3:ListAllMyBuckets", "s3:GetBucketLocation"],
     "Effect": "Allow",
     "Resource": ["arn:aws:s3:::*"]
   },
  {
     "Sid": "AllowRootAndHomeListingOfCompanyBucket",
     "Action": ["s3:ListBucket"],
     "Effect": "Allow",
     "Resource": ["arn:aws:s3:::DOC-EXAMPLE-BUCKET"],
     "Condition":{"StringEquals":{"s3:prefix":["","media/", "media/home"],"s3:delimiter":["/"]}}
    },
  {
     "Sid": "AllowListingOfUserFolder",
     "Action": ["s3:ListBucket"],
     "Effect": "Allow",
     "Resource": ["arn:aws:s3:::DOC-EXAMPLE-BUCKET"],
     "Condition":{"StringLike":{"s3:prefix":["media/home/*"]}}
    },
   {
     "Sid": "AllowAllS3ActionsInUserFolder",
     "Effect": "Allow",
     "Action": ["s3:GetObject", "s3:PutObject"],
     "Resource": ["arn:aws:s3:::DOC-EXAMPLE-BUCKET/media/home/*"]
   }
 ]
}
```

This policy grants user, `Get` and `Put` access to only his folder **`(/media/home)`** and no one else’s. While you could simply grant each user access to his or her own bucket, keep in mind that an AWS account can have up to 100 buckets by default. By creating home folders and granting the appropriate permissions, you can instead have hundreds of users share a single bucket.

## Reference:

[1] [Grant Access to User-Specific Folders in an Amazon S3 Bucket](https://aws.amazon.com/blogs/security/writing-iam-policies-grant-access-to-user-specific-folders-in-an-amazon-s3-bucket/)

[2] [How can I grant a user access to a specific folder in my Amazon S3 bucket?](https://repost.aws/knowledge-center/s3-folder-user-access)

