---
title : "Prerequisites"
date : 2024-01-01
weight : 2
chapter : false
pre : " <b> 5.2. </b> "
---

#### IAM permissions
The policy below defines the IAM permissions required to deploy the AWS services in the RakiBookery workshop.


```json
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Sid": "RakiBookeryFullWorkshopPolicy",
            "Effect": "Allow",
            "Action": [
                "ec2:*Vpc*",
                "ec2:*Subnet*",
                "ec2:*SecurityGroup*",
                "ec2:*InternetGateway*",
                "ec2:*Route*",
                "ec2:*NetworkInterface*",
                "ec2:*Address*",
                "ec2:Describe*",
                "ec2:CreateTags",
                "rds:*",
                "s3:*",
                "iam:CreateUser",
                "iam:CreateGroup",
                "iam:AddUserToGroup",
                "iam:CreateRole",
                "iam:DeleteRole",
                "iam:GetRole",
                "iam:ListRoles",
                "iam:CreatePolicy",
                "iam:DeletePolicy",
                "iam:GetPolicy",
                "iam:AttachRolePolicy",
                "iam:DetachRolePolicy",
                "iam:AttachGroupPolicy",
                "iam:PutRolePolicy",
                "iam:PassRole",
                "secretsmanager:*",
                "ecr:*",
                "ecs:*",
                "elasticloadbalancing:*",
                "cloudfront:*",
                "route53:*",
                "sns:*",
                "lambda:*",
                "logs:*",
                "cloudwatch:*",
                "bedrock:*",
                "aoss:*",
                "apigateway:*"
            ],
            "Resource": "*"
        }
    ]
}
```

#### Provision resources using CloudFormation

In this lab, we will use the **Asia Pacific (Singapore) region (ap-southeast-1)**.

To prepare the workshop environment, deploy the following CloudFormation template (click link): [PrivateLinkWorkshop](https://us-east-1.console.aws.amazon.com/cloudformation/home?region=us-east-1#/stacks/quickcreate?templateURL=https://s3.us-east-1.amazonaws.com/reinvent-endpoints-builders-session/Nested.yaml&stackName=PLCloudSetup). Keep all default settings.

![create stack](/fcj-workshop-nhat/images/5-Workshop/5.2-Prerequisite/cloudfomation_stack1.png)

+ Set the CloudFormation stack name: RakiBookery-5-2-S3-Gateway
+ Choose **Create stack**

![create stack](/fcj-workshop-nhat/images/5-Workshop/5.2-Prerequisite/cloudfomation_stack2.png)

The CloudFormation deployment takes approximately 15 minutes to complete.

![complete](/fcj-workshop-nhat/images/5-Workshop/5.2-Prerequisite/cloudfomation_success.png)

+ VPCs have been created

![vpcs](/fcj-workshop-nhat/images/5-Workshop/5.2-Prerequisite/vpc.png)

+ 3 EC2 instances have been created

![EC2](/fcj-workshop-nhat/images/5-Workshop/5.2-Prerequisite/ec2.png)
