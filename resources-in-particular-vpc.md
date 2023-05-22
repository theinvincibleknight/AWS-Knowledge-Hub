# Introduction

You might be running multiple Amazon Virtual Private Clouds (VPCs) in your AWS account to host different workloads and applications. Finding out which resource belongs to which VPC can be quite difficult when navigating the user interface in the AWS Console.

If you want to identify which resources are located under which VPC on your AWS account. You can do this by using the AWS Console and the AWS CLI.

## List all active AWS resources in a VPC in the AWS Console

To successfully find and list all AWS resources in a VPC you need to [enable AWS Config](https://docs.aws.amazon.com/config/latest/developerguide/gs-cli-subscribe.html) on your account. [**`AWS Config`**](https://docs.aws.amazon.com/config/latest/developerguide/WhatIsConfig.html) is a service that provides you with information on the configuration of the active AWS resources in your AWS account.

### 1. Open advanced queries in AWS Config

Then you need to use a feature called [Advanced queries](https://docs.aws.amazon.com/config/latest/developerguide/querying-AWS-resources.html) in AWS Config.

This feature provides a single query endpoint that allows you to use a query language to fetch the state of your AWS resources in one or more AWS accounts.

### 2. Run query

Press the **`New query`** button to start querying AWS Config. Then run the following query:

```sql
SELECT
  resourceId,
  resourceName,
  resourceType,
  tags,
  availabilityZone
WHERE
  relationships.resourceId = 'vpc-0ded1c524768bfc52'
```

Replace `'vpc-0ded1c524768bfc52'` with your VPC ID.

This will return all resources, including their id, name, type, tags, and availability zone that are related to your VPC ID.

You can choose to **`export`** the results as a `JSON` or `CSV` file for further processing.

## Find all active resources in a VPC in the AWS CLI

To make use of the AWS Command Line Interface (CLI) with your AWS account to run commands you should set up the required [CLI tool](https://docs.aws.amazon.com/cli/latest/userguide/getting-started-install.html) and [AWS profile](https://docs.aws.amazon.com/cli/latest/userguide/cli-configure-files.html#cli-configure-files-methods) first.

### 1. Query your resource configuration data in the terminal

Use the same query as we’ve used in the AWS Console but prefix it with the respective AWS Config service command:

```sql
aws configservice select-resource-config --expression "SELECT resourceId,  resourceName, resourceType, tags, availabilityZone WHERE relationships.resourceId='vpc-0ded1c524768bfc52'"
```

This will return all the AWS resources that are currently active in your Amazon VPC.

Output:
```
"Results": [
        "{\"resourceId\":\"acl-092a0e2c5b07b9247\",\"availabilityZone\":\"Multiple Availability Zones\",\"tags\":[],\"resourceType\":\"AWS::EC2::NetworkAcl\"}",
        "{\"resourceId\":\"arn:aws:cloudformation:eu-central-1:012345678912:stack/test-cloudNation-vpc/d8d995f0-543d-11ed-8f3d-026d4bea47f6\",\"resourceName\":\"test-cloudNation-vpc\",\"availabilityZone\":\"Regional>
        "{\"resourceId\":\"eni-01e24b83617a07dd2\",\"availabilityZone\":\"eu-central-1b\",\"tags\":[],\"resourceType\":\"AWS::EC2::NetworkInterface\"}",
        "{\"resourceId\":\"eni-0407590302acdec8a\",\"availabilityZone\":\"eu-central-1a\",\"tags\":[],\"resourceType\":\"AWS::EC2::NetworkInterface\"}",
        "{\"resourceId\":\"eni-053432ae4c7cd89c2\",\"availabilityZone\":\"eu-central-1b\",\"tags\":[],\"resourceType\":\"AWS::EC2::NetworkInterface\"}",
        "{\"resourceId\":\"eni-0605eff823362cc39\",\"availabilityZone\":\"eu-central-1a\",\"tags\":[],\"resourceType\":\"AWS::EC2::NetworkInterface\"}",
        "{\"resourceId\":\"eni-06a13f3838f940f8e\",\"availabilityZone\":\"eu-central-1a\",\"tags\":[],\"resourceType\":\"AWS::EC2::NetworkInterface\"}",
        "{\"resourceId\":\"eni-088be0e467c003785\",\"availabilityZone\":\"eu-central-1c\",\"tags\":[],\"resourceType\":\"AWS::EC2::NetworkInterface\"}",
        "{\"resourceId\":\"eni-0a55d9aef825f1d97\",\"availabilityZone\":\"eu-central-1c\",\"tags\":[],\"resourceType\":\"AWS::EC2::NetworkInterface\"}",
        "{\"resourceId\":\"eni-0d762e4ea5d10589b\",\"availabilityZone\":\"eu-central-1b\",\"tags\":[],\"resourceType\":\"AWS::EC2::NetworkInterface\"}",
        "{\"resourceId\":\"i-01b3ecb949db2b2f4\",\"availabilityZone\":\"eu-central-1b\",\"tags\":[{\"tag\":\"PreventCleanup\\u003dPlease\",\"value\":\"Please\",\"key\":\"PreventCleanup\"},{\"tag\":\"aws:cloudformati>
        "{\"resourceId\":\"i-025888f3bcbef23ef\",\"availabilityZone\":\"eu-central-1a\",\"tags\":[{\"tag\":\"PreventCleanup\\u003dPlease\",\"value\":\"Please\",\"key\":\"PreventCleanup\"},{\"tag\":\"aws:cloudformati>
        "{\"resourceId\":\"i-0e7795e95ec60e65c\",\"availabilityZone\":\"eu-central-1c\",\"tags\":[{\"tag\":\"PreventCleanup\\u003dPlease\",\"value\":\"Please\",\"key\":\"PreventCleanup\"},{\"tag\":\"aws:cloudformati>
        "{\"resourceId\":\"igw-0844a6a25180a154c\",\"availabilityZone\":\"Multiple Availability Zones\",\"tags\":[{\"tag\":\"PreventCleanup\\u003dPlease\",\"value\":\"Please\",\"key\":\"PreventCleanup\"},{\"tag\":\">
        "{\"resourceId\":\"rds:test-cloudnation-rds-serverlessclustera5ed910e-2i3epvn90ajv-2022-10-28-22-20\",\"resourceName\":\"rds:test-cloudnation-rds-serverlessclustera5ed910e-2i3epvn90ajv-2022-10-28-22-20\",\"a>
        "{\"resourceId\":\"rds:test-cloudnation-rds-serverlessclustera5ed910e-2i3epvn90ajv-2022-10-29-22-20\",\"resourceName\":\"rds:test-cloudnation-rds-serverlessclustera5ed910e-2i3epvn90ajv-2022-10-29-22-20\",\"a>
        "{\"resourceId\":\"rtb-001f2cbba7e9271d4\",\"availabilityZone\":\"Not Applicable\",\"tags\":[{\"tag\":\"PreventCleanup\\u003dPlease\",\"value\":\"Please\",\"key\":\"PreventCleanup\"},{\"tag\":\"aws:cloudform>
        "{\"resourceId\":\"rtb-00e069ff6195fe0bc\",\"availabilityZone\":\"Not Applicable\",\"tags\":[{\"tag\":\"PreventCleanup\\u003dPlease\",\"value\":\"Please\",\"key\":\"PreventCleanup\"},{\"tag\":\"aws:cloudform>
        "{\"resourceId\":\"rtb-02ed380487af3d753\",\"availabilityZone\":\"Not Applicable\",\"tags\":[{\"tag\":\"PreventCleanup\\u003dPlease\",\"value\":\"Please\",\"key\":\"PreventCleanup\"},{\"tag\":\"aws:cloudform>
        "{\"resourceId\":\"rtb-0499d4530876c98cb\",\"availabilityZone\":\"Not Applicable\",\"tags\":[{\"tag\":\"PreventCleanup\\u003dPlease\",\"value\":\"Please\",\"key\":\"PreventCleanup\"},{\"tag\":\"aws:cloudform>
        "{\"resourceId\":\"rtb-07ccadedb41fc2c8b\",\"availabilityZone\":\"Not Applicable\",\"tags\":[{\"tag\":\"PreventCleanup\\u003dPlease\",\"value\":\"Please\",\"key\":\"PreventCleanup\"},{\"tag\":\"aws:cloudform>
        "{\"resourceId\":\"rtb-0adb4dc7019440577\",\"availabilityZone\":\"Not Applicable\",\"tags\":[{\"tag\":\"PreventCleanup\\u003dPlease\",\"value\":\"Please\",\"key\":\"PreventCleanup\"},{\"tag\":\"aws:cloudform>
        "{\"resourceId\":\"rtb-0b1748558ede7296b\",\"availabilityZone\":\"Not Applicable\",\"tags\":[],\"resourceType\":\"AWS::EC2::RouteTable\"}",
        "{\"resourceId\":\"rtb-0c499abb3a009a3cb\",\"availabilityZone\":\"Not Applicable\",\"tags\":[{\"tag\":\"PreventCleanup\\u003dPlease\",\"value\":\"Please\",\"key\":\"PreventCleanup\"},{\"tag\":\"aws:cloudform>
        "{\"resourceId\":\"rtb-0c58f158336312ec9\",\"availabilityZone\":\"Not Applicable\",\"tags\":[{\"tag\":\"PreventCleanup\\u003dPlease\",\"value\":\"Please\",\"key\":\"PreventCleanup\"},{\"tag\":\"aws:cloudform>
        "{\"resourceId\":\"rtb-0d1896478397525fa\",\"availabilityZone\":\"Not Applicable\",\"tags\":[{\"tag\":\"PreventCleanup\\u003dPlease\",\"value\":\"Please\",\"key\":\"PreventCleanup\"},{\"tag\":\"aws:cloudform>
        "{\"resourceId\":\"sg-0316ae9b332eea428\",\"resourceName\":\"test-cloudNation-rds-serverlessClusterSecurityGroup15CA2923-6YX0ISL2DN66\",\"availabilityZone\":\"Not Applicable\",\"tags\":[{\"tag\":\"PreventCle>
        "{\"resourceId\":\"sg-0601ef39f9e2f00f0\",\"resourceName\":\"default\",\"availabilityZone\":\"Not Applicable\",\"tags\":[],\"resourceType\":\"AWS::EC2::SecurityGroup\"}",
        "{\"resourceId\":\"sg-0b7bbe3c4f3c766bd\",\"resourceName\":\"test-cloudNation-rds-serverlessClusterRotationSingleUserSecurityGroup0ED2D348-1QZEBFUNOV0W2\",\"availabilityZone\":\"Not Applicable\",\"tags\":[{\>
        "{\"resourceId\":\"sg-0c35f9949d1b6f560\",\"resourceName\":\"test-cloudNation-vpc-skeletonVPCNatSecurityGroup08D55A09-LCFW3O99AQ9G\",\"availabilityZone\":\"Not Applicable\",\"tags\":[{\"tag\":\"PreventCleanu>
        "{\"resourceId\":\"subnet-03a6f432748d890f3\",\"availabilityZone\":\"eu-central-1a\",\"tags\":[{\"tag\":\"aws-cdk:subnet-type\\u003dPrivate\",\"value\":\"Private\",\"key\":\"aws-cdk:subnet-type\"},{\"tag\":\>
        "{\"resourceId\":\"subnet-03b8f6b3857518c5c\",\"availabilityZone\":\"eu-central-1b\",\"tags\":[{\"tag\":\"PreventCleanup\\u003dPlease\",\"value\":\"Please\",\"key\":\"PreventCleanup\"},{\"tag\":\"aws-cdk:sub>
        "{\"resourceId\":\"subnet-03bacdfcf6aebd3a0\",\"availabilityZone\":\"eu-central-1c\",\"tags\":[{\"tag\":\"aws-cdk:subnet-type\\u003dIsolated\",\"value\":\"Isolated\",\"key\":\"aws-cdk:subnet-type\"},{\"tag\">
        "{\"resourceId\":\"subnet-04d6740bc3b0e6d41\",\"availabilityZone\":\"eu-central-1a\",\"tags\":[{\"tag\":\"PreventCleanup\\u003dPlease\",\"value\":\"Please\",\"key\":\"PreventCleanup\"},{\"tag\":\"aws-cdk:sub>
        "{\"resourceId\":\"subnet-0585664f00805edef\",\"availabilityZone\":\"eu-central-1b\",\"tags\":[{\"tag\":\"PreventCleanup\\u003dPlease\",\"value\":\"Please\",\"key\":\"PreventCleanup\"},{\"tag\":\"aws-cdk:sub>
        "{\"resourceId\":\"subnet-07a88b6e19ddf738d\",\"availabilityZone\":\"eu-central-1a\",\"tags\":[{\"tag\":\"aws-cdk:subnet-type\\u003dPublic\",\"value\":\"Public\",\"key\":\"aws-cdk:subnet-type\"},{\"tag\":\"P>
        "{\"resourceId\":\"subnet-0ae66b47932af7ab2\",\"availabilityZone\":\"eu-central-1c\",\"tags\":[{\"tag\":\"aws-cdk:subnet-type\\u003dPublic\",\"value\":\"Public\",\"key\":\"aws-cdk:subnet-type\"},{\"tag\":\"P>
        "{\"resourceId\":\"subnet-0bbcabe50c52f90f3\",\"availabilityZone\":\"eu-central-1b\",\"tags\":[{\"tag\":\"aws-cdk:subnet-type\\u003dPrivate\",\"value\":\"Private\",\"key\":\"aws-cdk:subnet-type\"},{\"tag\":\>
        "{\"resourceId\":\"subnet-0cb34da28adb8d560\",\"availabilityZone\":\"eu-central-1c\",\"tags\":[{\"tag\":\"PreventCleanup\\u003dPlease\",\"value\":\"Please\",\"key\":\"PreventCleanup\"},{\"tag\":\"aws-cdk:sub>
        "{\"resourceId\":\"test-cloudnation-rds-serverlessclustersubnetsbca74b96-ckxoi2t5hhjy\",\"resourceName\":\"test-cloudnation-rds-serverlessclustersubnetsbca74b96-ckxoi2t5hhjy\",\"availabilityZone\":\"Multiple>
        "{\"resourceId\":\"vpce-05961844e052082a6\",\"availabilityZone\":\"Not Applicable\",\"tags\":[],\"resourceType\":\"AWS::EC2::VPCEndpoint\"}",
        "{\"resourceId\":\"vpce-0beee10c8a718c923\",\"availabilityZone\":\"Not Applicable\",\"tags\":[],\"resourceType\":\"AWS::EC2::VPCEndpoint\"}"
    ],
    "QueryInfo": {
        "SelectFields": [
            {
                "Name": "resourceId"
            },
            {
                "Name": "resourceName"
            },
            {
                "Name": "resourceType"
            },
            {
                "Name": "tags"
            },
            {
                "Name": "availabilityZone"
            }
        ]
    }
}
```

## Conclusion

This is how you can list all resources from a certain VPC in an AWS account using the Advanced queries feature from AWS Config in the AWS Console and the AWS CLI.
