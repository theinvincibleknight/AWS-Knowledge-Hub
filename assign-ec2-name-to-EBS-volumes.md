# Assign the EC2 instance name to its corresponding EBS volume

## Introduction

We have the option to assign name tags to EBS volumes according to our requirements. However, if we don't assign names, the name tag will remain empty. This can lead to confusion when managing the volumes, as it won't be immediately clear which volume is connected to which EC2 instance. We would have to check the details to find out.

To address this, we can utilize the following Python script to assign the EC2 instance name to its corresponding EBS volume. This will provide better visibility and streamline the management process.

```python
import boto3

def assign_instance_name_to_volume(instance_id):
    ec2_client = boto3.client('ec2')

    # Get the instance name
    response = ec2_client.describe_instances(InstanceIds=[instance_id])
    instance_name = ""
    for tag in response['Reservations'][0]['Instances'][0]['Tags']:
        if tag['Key'] == 'Name':
            instance_name = tag['Value']
            break

    # Get the volume attached to the instance
    response = ec2_client.describe_volumes(Filters=[{'Name': 'attachment.instance-id', 'Values': [instance_id]}])
    volume_id = response['Volumes'][0]['VolumeId']

    # Assign the instance name as the volume's name
    ec2_client.create_tags(Resources=[volume_id], Tags=[{'Key': 'Name', 'Value': instance_name}])

    print(f"Assigned the name '{instance_name}' to volume {volume_id}")


# List of EC2 instance IDs
instance_ids = ['i-07621e98bfbc0ab60']

# Iterate over the instance IDs and assign names to volumes
for instance_id in instance_ids:
    assign_instance_name_to_volume(instance_id)
```
Replace `i-07621e98bfbc0ab60` with your required instance ID to assign the desired names to the volumes. You can use `,` and enter mutiple instance IDs, to assign the EC2 instance name to its corresponding EBS volume.
