# Python script to describe all of your VPC resources

vpc-resources.py

```python
import logging
import boto3
from argparse import ArgumentParser, HelpFormatter
from botocore.exceptions import ClientError, ProfileNotFound

# logger config
logger = logging.getLogger()
logging.basicConfig(level=logging.INFO,
                    format='%(message)s')

# Argument parser config
formatter = lambda prog: HelpFormatter(prog, max_help_position=52)
parser = ArgumentParser(formatter_class=formatter)
# parser = ArgumentParser()
parser.add_argument("-v", "--vpc", required=True, help="The VPC to describe")
parser.add_argument("-r", "--region", default="us-east-1", help="AWS region that the VPC resides in")
parser.add_argument("-p", '--profile', default='default', help="AWS profile")
args = parser.parse_args()

# boto client config
try:
    session = boto3.Session(profile_name=args.profile)
except ProfileNotFound as e:
    logger.warning("{}, please provide a valid AWS profile name".format(e))
    exit(-1)

vpc_client = session.client("ec2", region_name=args.region)
elbV2_client = session.client('elbv2', region_name=args.region)
elb_client = session.client('elb', region_name=args.region)
lambda_client = session.client('lambda', region_name=args.region)
eks_client = session.client('eks', region_name=args.region)
asg_client = session.client('autoscaling', region_name=args.region)
rds_client = session.client('rds', region_name=args.region)
ec2 = session.resource('ec2', region_name=args.region)

vpc_id: str = args.vpc


def vpc_in_region():
    """
    Describes one or more of your VPCs.
    """
    vpc_exists = False
    try:
        vpcs = list(ec2.vpcs.filter(Filters=[]))
    except ClientError as e:
        logger.warning(e.response['Error']['Message'])
        exit()
    logger.info("VPCs in region {}:".format(args.region))
    for vpc in vpcs:
        logger.info(vpc.id)
        if vpc.id == vpc_id:
            vpc_exists = True

    logger.info("--------------------------------------------")
    return vpc_exists


def describe_asgs():
    logger.info("ASGs in VPC {}:".format(vpc_id))
    asgs = asg_client.describe_auto_scaling_groups()['AutoScalingGroups']
    for asg in asgs:
        asg_name = asg['AutoScalingGroupName']
        if asg_in_vpc(asg):
            logger.info(asg_name)

    logger.info("--------------------------------------------")
    return


def asg_in_vpc(asg):
    subnets_list = asg['VPCZoneIdentifier'].split(',')
    for subnet in subnets_list:
        try:
            sub_description = vpc_client.describe_subnets(SubnetIds=[subnet])['Subnets']
            if sub_description[0]['VpcId'] == vpc_id:
                logger.info("{} resides in {}".format(asg['AutoScalingGroupName'], vpc_id))
                return True
        except ClientError:
            pass

    return False


def describe_ekss():
    ekss = eks_client.list_clusters()['clusters']

    logger.info("EKSs in VPC {}:".format(vpc_id))
    for eks in ekss:
        eks_desc = eks_client.describe_cluster(name=eks)['cluster']
        if eks_desc['resourcesVpcConfig']['vpcId'] == vpc_id:
            logger.info(eks_desc['name'])

    logger.info("--------------------------------------------")
    return


def describe_ec2s():
    waiter = vpc_client.get_waiter('instance_terminated')
    reservations = vpc_client.describe_instances(Filters=[{"Name": "vpc-id",
                                                           "Values": [vpc_id]}])['Reservations']

    # Get a list of ec2s
    ec2s = [ec2['InstanceId'] for reservation in reservations for ec2 in reservation['Instances']]

    logger.info("EC2s in VPC {}:".format(vpc_id))
    for ec2 in ec2s:
        logger.info(ec2)

    logger.info("--------------------------------------------")
    return


def describe_lambdas():
    lmbds = lambda_client.list_functions()['Functions']

    lambdas_list = [lmbd['FunctionName'] for lmbd in lmbds
                    if 'VpcConfig' in lmbd and lmbd['VpcConfig']['VpcId'] == vpc_id]

    logger.info("Lambdas in VPC {}:".format(vpc_id))
    for lmbda in lambdas_list:
        logger.info(lmbda)

    logger.info("--------------------------------------------")
    return


def describe_rdss():
    rdss = rds_client.describe_db_instances()['DBInstances']

    rdsss_list = [rds['DBInstanceIdentifier'] for rds in rdss if rds['DBSubnetGroup']['VpcId'] == vpc_id]

    logger.info("RDSs in VPC {}:".format(vpc_id))
    for rds in rdsss_list:
        logger.info(rds)

    logger.info("--------------------------------------------")
    return


def describe_elbs():
    elbs = elb_client.describe_load_balancers()['LoadBalancerDescriptions']

    elbs = [elb['LoadBalancerName'] for elb in elbs if elb['VPCId'] == vpc_id]

    logger.info("Classic ELBs in VPC {}:".format(vpc_id))
    for elb in elbs:
        logger.info(elb)

    logger.info("--------------------------------------------")
    return


def describe_elbsV2():
    elbs = elbV2_client.describe_load_balancers()['LoadBalancers']

    elbs_list = [elb['LoadBalancerArn'] for elb in elbs if elb['VpcId'] == vpc_id]

    logger.info("ELBs V2 in VPC {}:".format(vpc_id))
    for elb in elbs_list:
        logger.info(elb)

    logger.info("--------------------------------------------")
    return


def describe_nats():
    nats = vpc_client.describe_nat_gateways(Filters=[{"Name": "vpc-id",
                                                      "Values": [vpc_id]}])['NatGateways']

    nats = [nat['NatGatewayId'] for nat in nats]
    logger.info("NAT GWs in VPC {}:".format(vpc_id))
    for nat in nats:
        logger.info(nat)

    logger.info("--------------------------------------------")
    return


def describe_enis():
    enis = vpc_client.describe_network_interfaces(Filters=[{"Name": "vpc-id", "Values": [vpc_id]}])['NetworkInterfaces']

    # Get a list of enis
    enis = [eni['NetworkInterfaceId'] for eni in enis]

    logger.info("ENIs in VPC {}:".format(vpc_id))
    for eni in enis:
        logger.info(eni)

    logger.info("--------------------------------------------")
    return


def describe_igws():
    """
  Describe the internet gateway
  """

    # Get list of dicts
    igws = vpc_client.describe_internet_gateways(
        Filters=[{"Name": "attachment.vpc-id",
                  "Values": [vpc_id]}])['InternetGateways']

    igws = [igw['InternetGatewayId'] for igw in igws]

    logger.info("IGWs in VPC {}:".format(vpc_id))
    for igw in igws:
        logger.info(igw)

    logger.info("--------------------------------------------")
    return


def describe_vpgws():
    """
  Describe the virtual private gateway
  """

    # Get list of dicts
    vpgws = vpc_client.describe_vpn_gateways(
        Filters=[{"Name": "attachment.vpc-id",
                  "Values": [vpc_id]}])['VpnGateways']

    vpgws = [vpgw['VpnGatewayId'] for vpgw in vpgws]

    logger.info("VPGWs in VPC {}:".format(vpc_id))
    for vpgw in vpgws:
        logger.info(vpgw)

    logger.info("--------------------------------------------")
    return


def describe_subnets():
    # Get list of dicts of metadata
    subnets = vpc_client.describe_subnets(Filters=[{"Name": "vpc-id",
                                                    "Values": [vpc_id]}])['Subnets']

    # Get a list of subnets
    subnets = [subnet['SubnetId'] for subnet in subnets]

    logger.info("Subnets in VPC {}:".format(vpc_id))
    for subnet in subnets:
        logger.info(subnet)

    logger.info("--------------------------------------------")
    return


def describe_acls():
    acls = vpc_client.describe_network_acls(Filters=[{"Name": "vpc-id",
                                                      "Values": [vpc_id]}])['NetworkAcls']

    # Get a list of subnets
    acls = [acl['NetworkAclId'] for acl in acls]
    logger.info("ACLs in VPC {}:".format(vpc_id))
    for acl in acls:
        logger.info(acl)

    logger.info("--------------------------------------------")
    return


def describe_sgs():
    sgs = vpc_client.describe_security_groups(Filters=[{"Name": "vpc-id",
                                                        "Values": [vpc_id]}])['SecurityGroups']

    # Get a list of subnets
    # sgs = [sg['GroupId'] for sg in sgs]
    logger.info("Security Groups in VPC {}:".format(vpc_id))

    for sg in sgs:
        logger.info(sg['GroupId'])


    logger.info("--------------------------------------------")
    return


def describe_rtbs():
    rtbs = vpc_client.describe_route_tables(Filters=[{"Name": "vpc-id",
                                                      "Values": [vpc_id]}])['RouteTables']
    # Get a list of Routing tables
    rtbs = [rtb['RouteTableId'] for rtb in rtbs]
    logger.info("Routing tables in VPC {}:".format(vpc_id))
    for rtb in rtbs:
        logger.info(rtb)

    logger.info("--------------------------------------------")
    return


def describe_vpc_epts():
    epts = vpc_client.describe_vpc_endpoints(Filters=[{"Name": "vpc-id",
                                                       "Values": [vpc_id]}])['VpcEndpoints']

    # Get a list of Routing tables
    epts = [ept['VpcEndpointId'] for ept in epts]
    logger.info("VPC EndPoints in VPC {}:".format(vpc_id))
    for ept in epts:
        logger.info(ept)


    logger.info("--------------------------------------------")
    return


if __name__ == '__main__':

    if vpc_in_region():
        describe_ekss()
        describe_asgs()
        describe_rdss()
        describe_ec2s()
        describe_lambdas()
        describe_elbs()
        describe_elbsV2()
        describe_nats()
        describe_vpc_epts()
        describe_igws()
        describe_vpgws()
        describe_enis()
        describe_sgs()
        describe_rtbs()
        describe_acls()
        describe_subnets()
    else:
        logger.info("The given VPC was not found in {}".format(args.region))
```

**Usage:** `vpc-resources.py [-h] -v VPC [-r REGION] [-p PROFILE]`

optional arguments:
  -h, --help                     show this help message and exit
  -v VPC, --vpc VPC              The VPC to annihilate
  -r REGION, --region REGION     AWS region that the VPC resides in
  -p PROFILE, --profile PROFILE  AWS profile
  
**Example:** `python vpc-resources.py -v vpc-0d151ef89787fac94 -r ap-south-1`
