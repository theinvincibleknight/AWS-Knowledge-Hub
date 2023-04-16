# Add tags to multiple Log Groups based on specific string in a name

## Prerequisites

- To access AWS services with the `AWS CLI`, you need an `AWS account` and `IAM` credentials. When running AWS CLI commands, the AWS CLI needs to have access to those AWS credentials.
- Install the latest release of the `AWS CLI` version on your computer.
- Configure the `AWS CLI`.

To add tags for multiple CloudWatch log groups based on their names, you can use the AWS Command Line Interface (CLI) and run a script that uses the `aws logs tag-log-group` command.

Here's an example script you can use:

```bash
#!/bin/bash

# Set the tag key and value
TAG_KEY="my-tag-key"
TAG_VALUE="my-tag-value"

# List all log group names
LOG_GROUPS=$(aws logs describe-log-groups --query "logGroups[*].logGroupName" --output text)

# Loop through the log groups and add the tags
for LOG_GROUP in $LOG_GROUPS
do
  if [[ $LOG_GROUP == *"my-log-group-name"* ]]; then
    aws logs tag-log-group --log-group-name $LOG_GROUP --tags $TAG_KEY=$TAG_VALUE
  fi
done
```

In this script, you first set the tag key and value that you want to add to the log groups. Then, you use the `aws logs describe-log-groups` command to list all the log group names. The `--query` parameter and `--output` parameter are used to filter the output to only return the log group names.

Next, you loop through each log group name and use an `if` statement to check if the log group name contains the string "my-log-group-name". You can replace this string with the actual name or pattern of the log group names you want to add tags to.

If the log group name matches the pattern, you use the `aws logs tag-log-group` command to add the tag to the log group.

Save this script as a file with a `.sh` extension, make it executable with the `chmod +x` command, and then run it from the command line with `./script-name.sh`.
