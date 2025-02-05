# Automate Exporting of CloudWatch Log Groups to S3 Bucket

Storing logs in CloudWatch can be expensive. To save costs, we can reduce the retention period of log groups. However, in some cases, we need to retain logs for audit or compliance purposes. In such cases, we should store the logs in an S3 bucket. While we can export CloudWatch Log Groups to an S3 bucket, this process is typically manual.

To automate the daily export of logs to an S3 bucket, we can use an AWS Lambda function in conjunction with Amazon EventBridge to invoke the Lambda function on a schedule.

## Steps to Automate Log Export

1. **Create an S3 Bucket**  
   Create an S3 bucket in the same region as your CloudWatch log groups. Once the bucket is created, click on the bucket, navigate to the "Permissions" tab, and edit the bucket policy. Add the following policy:

   ```json
   {
       "Version": "2012-10-17",
       "Statement": [
           {
               "Effect": "Allow",
               "Principal": {
                   "Service": "logs.ap-south-1.amazonaws.com"
               },
               "Action": "s3:GetBucketAcl",
               "Resource": "arn:aws:s3:::cloudwatch-loggroups-archive-uat",
               "Condition": {
                   "StringEquals": {
                       "aws:SourceAccount": "123456789098"
                   },
                   "ArnLike": {
                       "aws:SourceArn": "arn:aws:logs:ap-south-1:123456789098:log-group:*"
                   }
               }
           },
           {
               "Effect": "Allow",
               "Principal": {
                   "Service": "logs.ap-south-1.amazonaws.com"
               },
               "Action": "s3:PutObject",
               "Resource": "arn:aws:s3:::cloudwatch-loggroups-archive-uat/*",
               "Condition": {
                   "StringEquals": {
                       "aws:SourceAccount": "123456789098",
                       "s3:x-amz-acl": "bucket-owner-full-control"
                   },
                   "ArnLike": {
                       "aws:SourceArn": "arn:aws:logs:ap-south-1:123456789098:log-group:*"
                   }
               }
           }
       ]
   }
   ```

   **Note:** Update the `aws:SourceAccount` and `Resource` values with the correct account number and bucket ARN.

2. **Create a Lambda Function**  
   Create a Lambda function and paste the following Python code:

```python
import boto3
import os
from pprint import pprint
import time

logs = boto3.client('logs')
ssm = boto3.client('ssm')

def lambda_handler(event, context):
    extra_args = {}
    log_groups = []
    log_groups_to_export = []
    
    if 'S3_BUCKET' not in os.environ:
        print("Error: S3_BUCKET not defined")
        return
    
    print("--> S3_BUCKET=%s" % os.environ["S3_BUCKET"])
    
    while True:
        response = logs.describe_log_groups(**extra_args)
        log_groups = log_groups + response['logGroups']
        
        if not 'nextToken' in response:
            break
        extra_args['nextToken'] = response['nextToken']
    
    for log_group in log_groups:
        response = logs.list_tags_log_group(logGroupName=log_group['logGroupName'])
        log_group_tags = response['tags']
        if 'ExportToS3' in log_group_tags and log_group_tags['ExportToS3'] == 'true':
            log_groups_to_export.append(log_group['logGroupName'])
    
    for log_group_name in log_groups_to_export:
        ssm_parameter_name = ("/log-exporter-last-export/%s" % log_group_name).replace("//", "/")
        try:
            ssm_response = ssm.get_parameter(Name=ssm_parameter_name)
            ssm_value = ssm_response['Parameter']['Value']
        except ssm.exceptions.ParameterNotFound:
            ssm_value = "0"
        
        export_to_time = int(round(time.time() * 1000))
        
        print("--> Exporting %s to %s" % (log_group_name, os.environ['S3_BUCKET']))
        
        if export_to_time - int(ssm_value) < (24 * 60 * 60 * 1000):
            # Haven't been 24hrs from the last export of this log group
            print("    Skipped until 24hrs from last export is completed")
            continue
        
        try:
            response = logs.create_export_task(
                logGroupName=log_group_name,
                fromTime=int(ssm_value),
                to=export_to_time,
                destination=os.environ['S3_BUCKET'],
                destinationPrefix=log_group_name.strip("/")
            )
            print("    Task created: %s" % response['taskId'])
            time.sleep(60)
            
        except logs.exceptions.LimitExceededException:
            print("    Need to wait until all tasks are finished (LimitExceededException). Continuing later...")
            return
        
        except Exception as e:
            print("    Error exporting %s: %s" % (log_group_name, getattr(e, 'message', repr(e))))
            continue
        
        ssm_response = ssm.put_parameter(
            Name=ssm_parameter_name,
            Type="String",
            Value=str(export_to_time),
            Overwrite=True)
```

### Key Points:
- The environment variable `S3_BUCKET` must be set. This variable specifies the bucket to which the logs will be exported.
- The function creates a CloudWatch Logs Export Task.
- It only exports logs from log groups that have a tag `ExportToS3=true`.
- The log group name is used as the prefix folder when exporting.
- A checkpoint is saved in AWS Systems Manager (SSM) to ensure that the next export starts from the last exported timestamp.
- The function only exports logs if 24 hours have passed since the last export.

3. **Configure the Lambda Function**  
   After creating the Lambda function, click on it, go to the "Configuration" tab, and then click on "Environment Variables." Add a variable with the key `S3_BUCKET` and the value set to your S3 bucket name.

   In the "Configuration" tab, click on "General Configuration" and increase the timeout based on the size or quantity of your log groups. If you anticipate that execution may take longer, you can increase the timeout to a maximum of 15 minutes.

4. **Set Permissions**  
   In the "Configuration" tab, click on "Permissions," then click on the IAM Role associated with the Lambda function. This role should have the `LambdaBasicExecutionRole` policy. Click on "Add permissions" and then "Create inline policy." Add the following inline policy:

   ```json
   {
       "Version": "2012-10-17",
       "Statement": [
           {
               "Effect": "Allow",
               "Action": [
                   "logs:DescribeLogGroups",
                   "logs:ListTagsLogGroup",
                   "logs:CreateExportTask"
               ],
               "Resource": "*"
           },
           {
               "Effect": "Allow",
               "Action": [
                   "ssm:GetParameter",
                   "ssm:PutParameter"
               ],
               "Resource": "*"
           },
           {
               "Effect": "Allow",
               "Action": "s3:PutObject",
               "Resource": "arn:aws:s3:::cloudwatch-loggroups-archive-uat/*"
           }
       ]
   }
   ```

### Important Considerations:
- AWS allows only one export task to run per account at a time. If the Lambda function attempts to export multiple log groups simultaneously, you may encounter a `LimitExceededException` error.
- To handle this, a `time.sleep(60)` is included to allow the current export task to finish before starting the next one.
- If the export task takes longer than 60 seconds or if you have many log groups, consider running the Lambda function every 4 hours. This will ensure that only log groups that haven’t been exported in the last 24 hours are processed, preventing overlapping exports.

5. **Tag Log Groups**  
   Add the tag `ExportToS3=true` to all the log groups you want to export to S3. In the Lambda function, click on "Test" to verify that the function works as expected. You can check the CloudWatch console and click on "View all exports to Amazon S3" to see new tasks initiated by the Lambda function in the "Export tasks" section.

6. **Schedule the Lambda Function**  
   Once you confirm that the Lambda function is working correctly, go to Amazon EventBridge and create a schedule to invoke the Lambda function on a regular basis.

By using EventBridge and the Lambda function, you can automate the export of CloudWatch log groups to an S3 bucket without any manual intervention.

Additionally, you can create a lifecycle management policy for the S3 bucket to move objects to different storage classes, such as Glacier, to further reduce costs.

Reference: https://medium.com/dnx-labs/exporting-cloudwatch-logs-automatically-to-s3-with-a-lambda-function-80e1f7ea0187
