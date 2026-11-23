---
title : "Các bước chuẩn bị"
date : 2024-01-01 
weight : 2
chapter : false
pre : " <b> 5.2. </b> "
---

#### IAM permissions
Chính sách dưới đây xác định các quyền IAM cần thiết để triển khai các dịch vụ AWS trong workshop RakiBookery.
```json
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Sid": "RakiBookeryFullWorkshopPolicy",
            "Effect": "Allow",
            "Action": [
                "ec2:CreateVpc",
                "ec2:ModifyVpcAttribute",
                "ec2:CreateSubnet",
                "ec2:CreateInternetGateway",
                "ec2:AttachInternetGateway",
                "ec2:CreateRouteTable",
                "ec2:AssociateRouteTable",
                "ec2:CreateRoute",
                "ec2:ReplaceRoute",
                "ec2:CreateSecurityGroup",
                "ec2:AuthorizeSecurityGroupIngress",
                "ec2:AuthorizeSecurityGroupEgress",
                "ec2:RevokeSecurityGroupIngress",
                "ec2:RevokeSecurityGroupEgress",
                "ec2:CreateVpcEndpoint",
                "ec2:ModifyVpcEndpoint",
                "ec2:CreateTags",
                "ec2:DescribeAvailabilityZones",
                "ec2:DescribeVpcs",
                "ec2:DescribeVpcAttribute",
                "ec2:DescribeSubnets",
                "ec2:DescribeInternetGateways",
                "ec2:DescribeRouteTables",
                "ec2:DescribeSecurityGroups",
                "ec2:DescribeSecurityGroupRules",
                "ec2:DescribeVpcEndpoints",
                "ec2:DescribeNetworkInterfaces",
                "rds:CreateDBSubnetGroup",
                "rds:CreateDBInstance",
                "rds:ModifyDBInstance",
                "rds:StartDBInstance",
                "rds:StopDBInstance",
                "rds:AddTagsToResource",
                "rds:ListTagsForResource",
                "rds:DescribeDBInstances",
                "rds:DescribeDBSubnetGroups",
                "rds:DescribeDBEngineVersions",
                "rds:DescribeOrderableDBInstanceOptions",
                "rds:DescribeDBSnapshots",
                "s3:CreateBucket",
                "s3:ListAllMyBuckets",
                "s3:GetBucketLocation",
                "s3:GetBucketPublicAccessBlock",
                "s3:PutBucketPublicAccessBlock",
                "s3:GetBucketOwnershipControls",
                "s3:PutBucketOwnershipControls",
                "s3:GetEncryptionConfiguration",
                "s3:PutEncryptionConfiguration",
                "s3:GetBucketVersioning",
                "s3:PutBucketVersioning",
                "s3:GetBucketPolicy",
                "s3:PutBucketPolicy",
                "s3:PutBucketTagging",
                "s3:ListBucket",
                "s3:ListBucketVersions",
                "s3:GetObject",
                "s3:GetObjectVersion",
                "s3:PutObject",
                "s3:AbortMultipartUpload",
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
                "secretsmanager:CreateSecret",
                "secretsmanager:DescribeSecret",
                "secretsmanager:ListSecrets",
                "secretsmanager:PutSecretValue",
                "secretsmanager:TagResource",
                "ecr:GetAuthorizationToken",
                "ecr:CreateRepository",
                "ecr:DescribeRepositories",
                "ecr:DescribeImages",
                "ecr:ListImages",
                "ecr:TagResource",
                "ecr:BatchCheckLayerAvailability",
                "ecr:BatchGetImage",
                "ecr:GetDownloadUrlForLayer",
                "ecr:InitiateLayerUpload",
                "ecr:UploadLayerPart",
                "ecr:CompleteLayerUpload",
                "ecr:PutImage",
                "ecs:CreateCluster",
                "ecs:CreateService",
                "ecs:RegisterTaskDefinition",
                "ecs:UpdateService",
                "ecs:RunTask",
                "ecs:StopTask",
                "ecs:DescribeClusters",
                "ecs:DescribeServices",
                "ecs:DescribeTasks",
                "ecs:DescribeTaskDefinition",
                "ecs:ListClusters",
                "ecs:ListServices",
                "ecs:ListTasks",
                "ecs:ListTaskDefinitions",
                "ecs:TagResource",
                "elasticloadbalancing:CreateLoadBalancer",
                "elasticloadbalancing:CreateTargetGroup",
                "elasticloadbalancing:CreateListener",
                "elasticloadbalancing:CreateRule",
                "elasticloadbalancing:ModifyListener",
                "elasticloadbalancing:ModifyRule",
                "elasticloadbalancing:DescribeLoadBalancers",
                "elasticloadbalancing:DescribeTargetGroups",
                "elasticloadbalancing:DescribeTargetHealth",
                "elasticloadbalancing:DescribeListeners",
                "elasticloadbalancing:DescribeRules",
                "elasticloadbalancing:AddTags",
                "cloudfront:CreateDistribution",
                "cloudfront:GetDistribution",
                "cloudfront:GetDistributionConfig",
                "cloudfront:UpdateDistribution",
                "cloudfront:ListDistributions",
                "cloudfront:CreateOriginAccessControl",
                "cloudfront:GetOriginAccessControl",
                "cloudfront:ListOriginAccessControls",
                "cloudfront:CreateInvalidation",
                "cloudfront:GetInvalidation",
                "cloudfront:TagResource",
                "route53:ListHostedZones",
                "route53:ListResourceRecordSets",
                "route53:ChangeResourceRecordSets",
                "route53:GetChange",
                "sns:CreateTopic",
                "sns:GetTopicAttributes",
                "sns:ListTopics",
                "sns:Subscribe",
                "sns:ListSubscriptionsByTopic",
                "sns:Publish",
                "lambda:CreateFunction",
                "lambda:GetFunction",
                "lambda:ListFunctions",
                "lambda:UpdateFunctionCode",
                "lambda:UpdateFunctionConfiguration",
                "lambda:InvokeFunction",
                "lambda:AddPermission",
                "lambda:GetPolicy",
                "lambda:TagResource",
                "logs:CreateLogGroup",
                "logs:DescribeLogGroups",
                "logs:DescribeLogStreams",
                "logs:GetLogEvents",
                "logs:FilterLogEvents",
                "logs:PutRetentionPolicy",
                "cloudwatch:GetMetricData",
                "cloudwatch:ListMetrics",
                "cloudwatch:PutMetricAlarm",
                "cloudwatch:DescribeAlarms",
                "cloudwatch:PutDashboard",
                "cloudwatch:GetDashboard",
                "bedrock:InvokeModel",
                "bedrock:Retrieve",
                "bedrock:ListFoundationModels",
                "bedrock:ListInferenceProfiles",
                "bedrock:GetInferenceProfile",
                "bedrock:CreateKnowledgeBase",
                "bedrock:GetKnowledgeBase",
                "bedrock:ListKnowledgeBases",
                "bedrock:CreateDataSource",
                "bedrock:GetDataSource",
                "bedrock:StartIngestionJob",
                "bedrock:GetIngestionJob",
                "bedrock:CreateGuardrail",
                "bedrock:CreateGuardrailVersion",
                "bedrock:GetGuardrail",
                "bedrock:ApplyGuardrail",
                "aoss:ListCollections",
                "aoss:BatchGetCollection",
                "apigateway:GET",
                "apigateway:POST",
                "apigateway:PUT",
                "apigateway:PATCH"
            ],
            "Resource": "*"
        }
    ]
}
```

#### Khởi tạo tài nguyên bằng CloudFormation

Trong lab này, chúng ta sử dụng Asia Pacific (Singapore) region (ap-southeast-1).

Để chuẩn bị cho môi trường làm workshop, chúng ta deploy CloudFormation template sau (click link): [PrivateLinkWorkshop ](https://us-east-1.console.aws.amazon.com/cloudformation/home?region=us-east-1#/stacks/quickcreate?templateURL=https://s3.us-east-1.amazonaws.com/reinvent-endpoints-builders-session/Nested.yaml&stackName=PLCloudSetup). Để nguyên các lựa chọn mặc định.

![create stack](/fcj-workshop-nhat/images/5-Workshop/5.2-Prerequisite/cloudfomation_stack1.png)

+ Đặt tên cloudformation: RakiBookery-5-2-S3-Gateway
+ Chọn Create stack

![create stack](/fcj-workshop-nhat/images/5-Workshop/5.2-Prerequisite/cloudfomation_stack2.png)

Quá trình triển khai CloudFormation cần khoảng 15 phút để hoàn thành.

![complete](/fcj-workshop-nhat/images/5-Workshop/5.2-Prerequisite/cfsuccess.png)

+ VPCs đã được tạo

![vpcs](/fcj-workshop-nhat/images/5-Workshop/5.2-Prerequisite/vpc.png)

+ 3 EC2s đã được tạo

![EC2](/fcj-workshop-nhat/images/5-Workshop/5.2-Prerequisite/ec2.png)