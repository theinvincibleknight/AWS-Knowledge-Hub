# Bash script that uses the AWS CLI to retrieve the name of the key pair used for multiple EC2 instances

## Prerequisites
- To access AWS services with the `AWS CLI`, you need an `AWS account` and `IAM` credentials. When running AWS CLI commands, the AWS CLI needs to have access to those AWS credentials.
- Install the latest release of the `AWS CLI` version on your computer.
- Configure the `AWS CLI`

Here is the bash script:

```bash
#!/bin/bash

# set the region
region=ap-south-1

# set the instance ids
instance_ids="i-0016a559a927f2452 i-0c6f05f8ab0694103"

# loop through each instance id and retrieve the key pair name
for id in $instance_ids; do
    key_name=$(aws ec2 describe-instances --region $region --instance-ids $id --query 'Reservations[*].Instances[*].KeyName' --output text)
    echo "Instance $id is using key pair: $key_name"
done
```

To use this script, replace the `region` variable with your desired region code and update the `instance_ids` variable with the IDs of the instances you want to check.

Save this script as a file with a `.sh` extension, make it executable with the `chmod +x` command.

```bash
chmod +x keypair.sh
```

Then, simply run the script in your terminal and it will output the name of the key pair used for each instance.

[Optional] You can also use `>` operator to save the output in txt file.
```bash
sh keypair.sh > output.txt
```
