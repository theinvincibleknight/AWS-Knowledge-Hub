# Fetch List of Not Invoked Lambda Functions

In AWS, it's common to have multiple Lambda functions that may never be invoked after their creation. Identifying these unused functions can help in optimizing your AWS resources and potentially reducing costs. This guide will walk you through the process of creating a Lambda function that fetches a list of Lambda functions that have not been invoked in the last 30 days and sends this list to a specified email address via Amazon SNS (Simple Notification Service).

## Prerequisites

Before you begin, ensure you have the following:

1. **AWS Account**: You need access to an AWS account with permissions to create Lambda functions, SNS topics, and CloudWatch metrics.
2. **IAM Role**: Create an IAM role for the Lambda function with the following permissions:
   - `LambdaBasicExecutionRole`
   - `AmazonSNSFullAccess`
   - `AWSLambda_ReadOnlyAccess`
   - `CloudWatchReadOnlyAccess`
   
   You can fine-tune these permissions to include only the necessary actions.

3. **SNS Topic**: Create an SNS topic with at least one subscription (e.g., an email subscription) and copy the ARN (Amazon Resource Name) of the SNS topic.

## Step 1: Create the Lambda Function

1. **Navigate to the AWS Lambda Console**: Go to the AWS Management Console and open the Lambda service.
2. **Create a New Lambda Function**: Click on "Create function" and choose "Author from scratch."
3. **Configure the Function**:
   - **Function Name**: Give your function a meaningful name (e.g., `CheckUnusedLambdaFunctions`).
   - **Runtime**: Select Python 3.x as the runtime.
   - **Permissions**: Choose the IAM role you created earlier.

## Step 2: Implement the Lambda Function Code

Copy and paste the following code into the Lambda function editor. This code will fetch the list of Lambda functions that have not been invoked in the last 30 days and send the list to the specified SNS topic.

```python
import boto3
from datetime import datetime, timedelta

cloudwatch = boto3.client('cloudwatch', region_name='ap-south-1')
sns = boto3.client('sns', region_name='ap-south-1')
lambda_client = boto3.client('lambda', region_name='ap-south-1')

SNS_TOPIC_ARN = 'arn:aws:sns:ap-south-1:9876543210:CheckLambdaInvokeAlerts'

def lambda_handler(event, context):
    functions = lambda_client.list_functions()['Functions']
    non_invoked_functions = []

    for function in functions:
        function_name = function['FunctionName']
        response = cloudwatch.get_metric_statistics(
            Namespace='AWS/Lambda',
            MetricName='Invocations',
            Dimensions=[{'Name': 'FunctionName', 'Value': function_name}],
            StartTime=datetime.utcnow() - timedelta(days=30),
            EndTime=datetime.utcnow(),
            Period=86400,  # 1 day in seconds
            Statistics=['Sum']
        )
        if not response['Datapoints']:
            non_invoked_functions.append(function_name)

    if non_invoked_functions:
        subject = "List of Lambda functions not invoked in last 30 days in Old Prod Account"
        message = "The following Lambda functions have not been invoked in the last 30 days:\n" + "\n".join(non_invoked_functions)
        sns.publish(
            TopicArn=SNS_TOPIC_ARN,
            Subject=subject[:100],  # Ensure the subject is within the 100 character limit
            Message=message
        )
```

### Code Breakdown

```python
import boto3
from datetime import datetime, timedelta
```
- **Imports**: The code begins by importing the necessary libraries. `boto3` is the AWS SDK for Python, which allows you to interact with AWS services. `datetime` and `timedelta` are used to handle date and time operations.

```python
cloudwatch = boto3.client('cloudwatch', region_name='ap-south-1')
sns = boto3.client('sns', region_name='ap-south-1')
lambda_client = boto3.client('lambda', region_name='ap-south-1')
```
- **Creating Clients**: Here, three clients are created using `boto3`:
  - `cloudwatch`: This client is used to interact with Amazon CloudWatch, which provides metrics and logs for AWS services.
  - `sns`: This client is for Amazon Simple Notification Service (SNS), which is used to send notifications.
  - `lambda_client`: This client is for AWS Lambda, allowing you to interact with Lambda functions.

```python
SNS_TOPIC_ARN = 'arn:aws:sns:ap-south-1:9876543210:CheckLambdaInvokeAlerts'
```
- **SNS Topic ARN**: This variable holds the Amazon Resource Name (ARN) of the SNS topic where notifications will be sent. You need to replace this with your actual SNS topic ARN.

```python
def lambda_handler(event, context):
```
- **Lambda Handler Function**: This is the main entry point for the Lambda function. AWS Lambda invokes this function when the function is triggered. It takes two parameters:
  - `event`: Contains data passed to the function upon invocation.
  - `context`: Provides runtime information about the Lambda function.

```python
    functions = lambda_client.list_functions()['Functions']
    non_invoked_functions = []
```
- **Listing Lambda Functions**: The `list_functions` method of the `lambda_client` retrieves a list of all Lambda functions in the account. The result is stored in the `functions` variable.
- **Non-invoked Functions List**: An empty list `non_invoked_functions` is initialized to store the names of Lambda functions that have not been invoked.

```python
    for function in functions:
        function_name = function['FunctionName']
```
- **Iterating Through Functions**: A loop iterates over each function in the `functions` list. For each function, its name is extracted and stored in the `function_name` variable.

```python
        response = cloudwatch.get_metric_statistics(
            Namespace='AWS/Lambda',
            MetricName='Invocations',
            Dimensions=[{'Name': 'FunctionName', 'Value': function_name}],
            StartTime=datetime.utcnow() - timedelta(days=30),
            EndTime=datetime.utcnow(),
            Period=86400,  # 1 day in seconds
            Statistics=['Sum']
        )
```
- **Fetching Invocation Metrics**: The `get_metric_statistics` method of the `cloudwatch` client is called to retrieve invocation metrics for the current function. The parameters are:
  - `Namespace`: Specifies the AWS service (in this case, `AWS/Lambda`).
  - `MetricName`: The metric to retrieve, which is `Invocations`.
  - `Dimensions`: Filters the metrics by the function name.
  - `StartTime` and `EndTime`: Define the time range for the metrics (last 30 days).
  - `Period`: The granularity of the returned data (1 day in seconds).
  - `Statistics`: Specifies that we want the sum of invocations.

```python
        if not response['Datapoints']:
            non_invoked_functions.append(function_name)
```
- **Checking Invocation Data**: If the `Datapoints` in the response are empty, it means the function has not been invoked in the specified time frame. In this case, the function name is added to the `non_invoked_functions` list.

```python
    if non_invoked_functions:
        subject = "List of Lambda functions not invoked in last 30 days in Old Prod Account"
        message = "The following Lambda functions have not been invoked in the last 30 days:\n" + "\n".join(non_invoked_functions)
```
- **Preparing Notification**: If there are any non-invoked functions, a subject and message are prepared for the SNS notification. The message lists all the non-invoked functions.

```python
        sns.publish(
            TopicArn=SNS_TOPIC_ARN,
            Subject=subject[:100],  # Ensure the subject is within the 100 character limit
            Message=message
        )
```
- **Sending Notification**: The `publish` method of the `sns` client is called to send the notification.


## Step 3: Schedule the Lambda Function

To automate the process of checking for unused Lambda functions, you can set up an EventBridge rule to invoke the Lambda function on a monthly basis.

1. **Navigate to the EventBridge Console**: Go to the AWS Management Console and open the EventBridge service.
2. **Create a New Rule**: Click on "Rules" and then "Create rule."
3. **Configure the Rule**:
   - **Name**: Give your rule a name (e.g., `MonthlyUnusedLambdaCheck`).
   - **Schedule**: Set the schedule expression to run once a month (e.g., `cron(0 0 1 * ? *)` for the first day of every month).
   - **Target**: Select the Lambda function you created earlier as the target.
