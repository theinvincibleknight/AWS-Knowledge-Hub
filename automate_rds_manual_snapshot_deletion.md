## Automating RDS Manual Snapshots Retention Management

To create a Python program that deletes all manual RDS snapshots older than 3 days using AWS Lambda, you can use the `boto3` library, which is the AWS SDK for Python. Below is a sample Lambda function that accomplishes this task.

### Prerequisites
1. **AWS Account**: You need an AWS account with permissions to manage RDS snapshots.
2. **IAM Role**: Create an IAM role for your Lambda function with the following permissions:
   - `rds:DescribeDBSnapshots`
   - `rds:DeleteDBSnapshot`
3. **Lambda Function**: Create a new Lambda function in the AWS Management Console.

### Lambda Function Code

```python
import boto3
import datetime

def lambda_handler(event, context):
    # Create an RDS client
    rds_client = boto3.client('rds')
    
    # Get the current time
    current_time = datetime.datetime.now(datetime.timezone.utc)
    
    # Calculate the cutoff time (3 days ago)
    cutoff_time = current_time - datetime.timedelta(days=3)
    
    # Describe DB snapshots
    response = rds_client.describe_db_snapshots(SnapshotType='manual')
    
    # Iterate through the snapshots
    for snapshot in response['DBSnapshots']:
        snapshot_time = snapshot['SnapshotCreateTime']
        
        # Check if the snapshot is older than 3 days
        if snapshot_time < cutoff_time:
            snapshot_id = snapshot['DBSnapshotIdentifier']
            print(f"Deleting snapshot: {snapshot_id} created on {snapshot_time}")
            
            # Delete the snapshot
            rds_client.delete_db_snapshot(DBSnapshotIdentifier=snapshot_id)
    
    return {
        'statusCode': 200,
        'body': 'Old manual RDS snapshots deleted successfully.'
    }
```

To manage this, we can automate the process using AWS EventBridge and a Lambda function. This Lambda function will be triggered on a scheduled basis, checks for the snapshots older than 3 days and deletes them.

### Important Notes:

- If you have a large number of manual snapshots, you may need to increase the `Timeout` setting in the General Configuration of the Lambda function.

- It is recommended to deploy and test this Lambda function before scheduling it with EventBridge.
