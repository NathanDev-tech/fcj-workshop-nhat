---
title : "Prerequisites"
date : 2024-01-01
weight : 2
chapter : false
pre : " <b> 5.2. </b> "
---

#### IAM permissions

To deploy and clean up resources in this workshop, the AWS account must have the appropriate IAM permissions.

In the RakiBookery project, we use IAM Users for all team members, belonging to the RakiBookery IAM Group. The account has MFA enabled and is granted the AdministratorAccess policy through the IAM Group. Additionally, the IAMUserChangePassword policy is attached directly to the User. Therefore, the team does not need to attach the sample IAM policy provided in the original lab.

The AdministratorAccess policy currently in use is defined as follows:

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": "*",
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
