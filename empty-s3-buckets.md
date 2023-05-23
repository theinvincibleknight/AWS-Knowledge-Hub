# List S3 Buckets which are Empty

There is no direct option in the AWS Console to check the number of empty S3 buckets. It is not feasible to manually check if there are a large number of S3 buckets. The Python script provided below will give you a list of empty S3 buckets.

```python
import boto3

# Create an S3 client
s3_client = boto3.client('s3')

# List all bucket names
response = s3_client.list_buckets()

# Iterate over each bucket
for bucket in response['Buckets']:
    bucket_name = bucket['Name']
    
    # Check if the bucket is empty
    response = s3_client.list_objects_v2(Bucket=bucket_name)
    if 'Contents' not in response:
        print(bucket_name)
```

## **Explanation:**

### Creating an S3 client:
```python
s3_client = boto3.client('s3')
```
This line creates an S3 client using the `boto3.client()`7 function. The client allows you to make requests to the S3 service.

### Listing all bucket names:
```python
response = s3_client.list_buckets()
```
This line calls the `list_buckets()` method on the S3 client. It retrieves a response that contains information about all the buckets in your AWS account.

### Iterating over each bucket and checking if it's empty:
```python
for bucket in response['Buckets']:
    bucket_name = bucket['Name']
    
    # Check if the bucket is empty
    response = s3_client.list_objects_v2(Bucket=bucket_name)
    if 'Contents' not in response:
        print(bucket_name)
```
This section of code iterates over each bucket in the response obtained from `list_buckets()`. For each bucket, it retrieves the bucket name using `bucket['Name']`.

Inside the loop, it calls the `list_objects_v2()` method on the S3 client to list the objects in the bucket. The `list_objects_v2()` method returns a response that contains information about the objects in the bucket.

If the `'Contents'` key is not present in the response, it means that the bucket is empty. In that case, the bucket name is printed.

By running this script, you will obtain a list of empty S3 bucket names.
