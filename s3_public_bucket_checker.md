# AWS S3 Public Bucket Checker

## Overview

By default, AWS does not provide an option to filter out publicly accessible S3 buckets using the AWS Console. This simple Bash script that can be used to list publicly accessible S3 buckets.

## Prerequisites

Before running the script, you need to install and configure the AWS CLI on your Ubuntu system.

## Usage

Once you have the AWS CLI set up, you can run the following script to check for publicly accessible S3 buckets:

```bash
#!/bin/bash

# Get the list of all S3 buckets
buckets=$(aws s3api list-buckets --query "Buckets[].Name" --output text)

echo "Buckets with public access:"

# Loop through each bucket and check its public access settings
for bucket in $buckets; do
    # Get the public access block configuration
    public_access=$(aws s3api get-public-access-block --bucket "$bucket" 2>/dev/null)

    # Check if the bucket has public access
    if [ $? -ne 0 ]; then
        # If the command failed, it means the bucket does not exist or is not accessible
        continue
    fi

    # Check if the public access block is set
    if echo "$public_access" | grep -q '"BlockPublicAcls": false' || echo "$public_access" | grep -q '"IgnorePublicAcls": false'; then
        echo "$bucket"
    fi
done
```

## Conclusion

This script provides a simple way to identify publicly accessible S3 buckets in your AWS account. Make sure to run it in a secure environment and handle the output responsibly.
