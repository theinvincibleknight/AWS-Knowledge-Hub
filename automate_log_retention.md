## Automating Log Retention Management in Amazon CloudWatch Logs

By default, Amazon CloudWatch Logs stores your log data indefinitely. The persistence of logs from applications and operating systems allows you to search, filter, and store logs for future analysis. However, as your AWS workload logging increases over time, so do your log storage costs.

To manage this, we can automate the process using AWS EventBridge and a Lambda function. This Lambda function will be triggered on a scheduled basis, checking for log groups with a retention period set to "Never Expire" and updating it to 30 days.

Here is a sample implementation in Python:

```python
import boto3

def set_log_group_retention(region):
    client = boto3.client('logs', region_name=region)
    
    next_token = None
    while True:
        if next_token:
            response = client.describe_log_groups(nextToken=next_token)
        else:
            response = client.describe_log_groups()
        
        log_groups = response['logGroups']
        
        for log_group in log_groups:
            log_group_name = log_group['logGroupName']
            retention_in_days = log_group.get('retentionInDays', None)
            
            if retention_in_days is None:
                print(f"Updating retention period for log group {log_group_name} to 30 days.")
                client.put_retention_policy(logGroupName=log_group_name, retentionInDays=30)
            else:
                print(f"Retention period for log group {log_group_name} is already set to {retention_in_days} days. Skipping.")
        
        next_token = response.get('nextToken', None)
        if not next_token:
            break

def lambda_handler(event, context):
    set_log_group_retention('ap-south-1')
    set_log_group_retention('us-east-1')
    return {
        'statusCode': 200,
        'body': 'Log group retention periods updated successfully.'
    }
```

### Important Notes:
- If you have a large number of log groups, you may need to increase the `Timeout` setting in the **General Configuration** of the Lambda function.
- Ensure that the Lambda function has sufficient permissions to update the retention period of CloudWatch logs. If you are not familiar with IAM custom policies, you can assign the `CloudWatchFullAccess` policy.
- It is recommended to deploy and test this Lambda function before scheduling it with EventBridge.

Once tested, you can create an EventBridge schedule to invoke this Lambda function. This will check all log groups and update the retention period to 30 days if it finds that the retention period is set to "None" or "Never Expire."
