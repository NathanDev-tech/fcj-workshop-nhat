---
title : "Initialize and Configure IAM"
date : 2024-01-01 
weight : 1
chapter : false
pre : " <b> 5.3.1 </b> "
---

### Objective

Our goal is to set up AWS access for our team and the services that run the RakiBookery PHP application. AWS Secrets Manager stores the MySQL credentials. ECS passes these details to the container when it starts.

### What we need

- **Account and deployment access.** Check that you are in the RakiBookery AWS account. Confirm the identity approved by the administrator and the list of team members. IAM applies to the whole account. For ECS, ECR, RDS, and Secrets Manager, use the Singapore Region (`ap-southeast-1`).
- **Team responsibilities.** Compare the actions in [section 5.2]({{< relref "/5-Workshop/5.2-Prerequiste/" >}}) with the team's work. Decide who manages IAM, who deploys tasks, and who manages the login details.
- **What we need for the test.** Prepare the backend image URI and tag from [section 5.3.5]({{< relref "/5-Workshop/5.3-Infrastructure/5.3.5-Set-Up-ECR/" >}}). If the task uses `awslogs`, prepare the CloudWatch log group name. Get the RDS endpoint, database name, and MySQL user for the application from [section 5.3.4]({{< relref "/5-Workshop/5.3-Infrastructure/5.3.4-Set-Up-RDS/" >}}). Check which environment variables the PHP code reads.

We can prepare the IAM group and the execution role first. After RDS and ECR are ready, we can add the secret ARN.

### Which identity performs each action?

The deployment identity registers the task definition and passes approved roles to ECS. The **task execution role** lets ECS pull the image, get secret values at startup, and send logs. The **task role** is used only if the PHP code calls an AWS API. MySQL access is checked separately using the database user and the security group rules. [AWS ECS execution role](https://docs.aws.amazon.com/AmazonECS/latest/developerguide/task_execution_IAM_role.html).

### Deployment setup

#### 1. Check the IAM group and its users

1. In **IAM → User groups**, open the `RakiBookery` group. For a new environment, the administrator creates the group and adds each approved IAM user in **Users**.
2. In **Permissions**, check if the policies match the team's work. The screenshot shows the setup in the project account. A policy with fewer permissions does not remove the permissions a user already gets from `AdministratorAccess`.
3. For team members who sign in to the AWS Console with an IAM user, check MFA in **Security credentials**. Record the group and the permissions in the report.

![Permissions tab for the RakiBookery group](../../../images/5-Workshop/5.3-Infrastructure/5.3.1-Set-Up-IAM/01-iam-group-permissions.png)

#### 2. Prepare the ECS task execution role

**1.** Open **IAM → Roles** and find `ecsTaskExecutionRole`. If the role already exists, open it and check **Trust relationships** and **Permissions**.

![IAM Roles showing ecsTaskExecutionRole and ecs-tasks as the trusted entity](../../../images/5-Workshop/5.3-Infrastructure/5.3.1-Set-Up-IAM/02-iam-ecs-execution-role-list.png)

**2.** To create the role, choose **Create role → AWS service → Elastic Container Service**. Under **Use case**, choose **Elastic Container Service Task**. Check that the trust policy allows `sts:AssumeRole` for `ecs-tasks.amazonaws.com`.

**3.** In **Add permissions**, attach `AmazonECSTaskExecutionRolePolicy` under **Permissions policies**. This lets ECS pull the image from ECR and send logs through `awslogs`. A permissions boundary limits the permissions a role can use. If the account requires a boundary, use one approved by the administrator and check that it allows the actions the role needs. Create the log group first if the container uses `awslogs`.

![Permissions tab for ecsTaskExecutionRole showing AmazonECSTaskExecutionRolePolicy](../../../images/5-Workshop/5.3-Infrastructure/5.3.1-Set-Up-IAM/03-iam-ecs-execution-role-policy.png)

**4.** Copy the ARN and select the role in **Task execution role** in the task definition. If PHP calls an AWS API directly, for example to upload an image to S3, use a separate role in **Task role**. Limit its permissions to the bucket or prefix that PHP needs.

**5.** Give the deployment identity `iam:PassRole` only for approved role ARNs. Set `iam:PassedToService` to `ecs-tasks.amazonaws.com`. [AWS IAM PassRole](https://docs.aws.amazon.com/IAM/latest/UserGuide/id_roles_use_passrole.html).

![Tab Permissions](../../../../images/5-Workshop/5.3-Infrastructure/5.3.1-Set-Up-IAM/04-iam-role-ecs.png)

#### 3. Prepare the secret and add it to the task definition

After you have a MySQL account for the application, open **Secrets Manager → Store a new secret** in the project Region. You can use the JSON keys `username` and `password`. Copy the full ARN from the secret details page. If the application already has a secret, check the key names before you use it.

The policy in [section 5.2]({{< relref "/5-Workshop/5.2-Prerequiste/" >}}) lists the actions for deploying the workshop. The example policy below allows the execution role to read one secret, using the secret ARN. If the task gets the MySQL credentials from Secrets Manager, open **Permissions → Add permissions → Create inline policy → JSON** on the execution role and add the policy. The **Permissions** screenshot above shows the base policy. Check that the role can read the secret after you add the inline policy.

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": "secretsmanager:GetSecretValue",
      "Resource": "arn:aws:secretsmanager:ap-southeast-1:987989461846:secret:rakibookery/prod/rds/credentials-bUVjFm"
    }
  ]
}
```

### Conclusion

Our team has its own IAM group and roles for the services. The `AdministratorAccess` policy gives broad permissions. We need to review the actions each member needs to follow least privilege. To verify the setup, run a Fargate task and check that the container gets the values from Secrets Manager and connects to MySQL.
