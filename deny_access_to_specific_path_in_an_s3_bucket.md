## Overview

1. Deny access to a specific path in an S3 bucket for all users except approved IAM roles and users.
2. Prevent the bucket policy from being modified or deleted.
3. Provide a solution for regaining access if the bucket is accidentally locked out.

## Prerequisites

- An AWS account with permissions to manage S3 buckets and IAM roles.
- IAM roles/users that you want to grant access to the S3 bucket.

## Step 1: Define the Bucket Policy

Use the following bucket policy to restrict access and prevent modifications:

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "DenyAccessToSpecificPath",
      "Effect": "Deny",
      "Principal": "*",
      "Action": "s3:*",
      "Resource": "arn:aws:s3:::delete-me-new-bucket-1714/folder1/subfolder1/*",
      "Condition": {
        "StringNotLike": {
          "aws:PrincipalARN": [
            "arn:aws:iam::9876543210:user/delete-me-iam-user",
            "arn:aws:iam::9876543210:role/aws-reserved/sso.amazonaws.com/ap-south-1/AWSReservedSSO_AWS_AdminAcc_Admins_*"
          ]
        }
      }
    },
    {
      "Effect": "Deny",
      "Principal": "*",
      "Action": "s3:ListBucket",
      "Resource": "arn:aws:s3:::delete-me-new-bucket-1714",
      "Condition": {
        "StringNotLike": {
          "aws:PrincipalARN": [
            "arn:aws:iam::9876543210:user/delete-me-iam-user",
            "arn:aws:iam::9876543210:role/aws-reserved/sso.amazonaws.com/ap-south-1/AWSReservedSSO_AWS_AdminAcc_Admins_*"
          ]
        },
        "StringLike": {
          "s3:prefix": "folder1/*"
        }
      }
    },
    {
      "Sid": "PreventBucketPolicyModification",
      "Effect": "Deny",
      "Principal": "*",
      "Action": [
        "s3:PutBucketPolicy",
        "s3:DeleteBucketPolicy"
      ],
      "Resource": "arn:aws:s3:::delete-me-new-bucket-1714",
      "Condition": {
        "StringNotLike": {
          "aws:PrincipalARN": "arn:aws:iam::9876543210:role/aws-reserved/sso.amazonaws.com/ap-south-1/AWSReservedSSO_AWS_AdminAcc_Admins_*"
        }
      }
    }
  ]
}
```

> Note: In our case, since all users log in to the AWS portal using AWS SSO, we were unable to provide the direct ARN of the SSO user in the policy. Instead, we used the IAM role associated with that SSO user. You can edit the policy as per your usecase.

Step 2: Apply the Bucket Policy
- Log in to the AWS Management Console.
- Navigate to the S3 service.
- Select the bucket you want to modify.
- Go to the "Permissions" tab.
- Click on "Bucket Policy" and paste the policy defined above.
- Save the changes.

Step 3: Regaining Access After Lockout
If you accidentally deny access to everyone and lock yourself out of the bucket, follow these steps to regain access:

- Sign in to your AWS account using the root user email.
- Navigate to the S3 bucket that you are unable to edit the policy for or delete due to the current policy.
- Go to the "Permissions" tab and edit the "Bucket Policy."

Follow the guide provided by AWS: [How do I regain access to my Amazon S3 bucket after I accidentally denied everyone access?](https://repost.aws/knowledge-center/s3-accidentally-denied-access)
