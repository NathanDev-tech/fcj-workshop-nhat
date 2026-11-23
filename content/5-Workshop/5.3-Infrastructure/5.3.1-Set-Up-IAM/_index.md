---
title : "Initialize and Configure IAM"
date : 2024-01-01 
weight : 1
chapter : false
pre : " <b> 5.3.1 </b> "
---

### Objective

Work organizes AWS access for the RakiBookery team and the services that run the PHP application. AWS Secrets Manager stores MySQL credentials for ECS to provide to the container at startup.

### Prerequisites

- **Account and deployment access.** Confirm the RakiBookery AWS account, the administrator-approved deployment identity, and the member list. IAM is account-wide. Use Singapore (`ap-southeast-1`) for ECS, ECR, RDS, and Secrets Manager.
- **Responsibilities.** Review the actions in [section 5.2]({{< relref "/5-Workshop/5.2-Prerequiste/" >}}) against the team's tasks. Identify who manages IAM, deploys tasks, and manages application credentials.
- **Test inputs.** Prepare the backend image URI and tag from [section 5.3.5]({{< relref "/5-Workshop/5.3-Infrastructure/5.3.5-Set-Up-ECR/" >}}), the CloudWatch log group if the task uses `awslogs`, and the RDS endpoint, database name, and application account from [section 5.3.4]({{< relref "/5-Workshop/5.3-Infrastructure/5.3.4-Set-Up-RDS/" >}}). Identify the environment variables read by the PHP application.

Prepare the IAM group and execution role first. When RDS and ECR are ready, add the secret ARN and test with the actual image.

### Deployment configuration

#### 1. Review the IAM group and team access

1. Open **IAM → User groups → RakiBookery**. For a new environment, an administrator creates the group and adds the approved member accounts in **Users**.
2. In **Permissions**, review the policies against team responsibilities. The screenshots show the project account's recorded configuration. Attaching a narrower policy does not reduce access for a user who still inherits `AdministratorAccess`.
3. Check MFA under **Security credentials** for members who use IAM users to sign in to the AWS Console. Record the group and its approved access scope in the report.

![Permissions tab for the RakiBookery IAM group](../../../images/5-Workshop/5.3-Infrastructure/iam-group-permissions.png)

*Figure 5.3.1a. The RakiBookery group shows five IAM users and two attached policies.*

#### 2. Prepare the ECS task execution role

1. Open **IAM → Roles** and find `ecsTaskExecutionRole`. If it already exists, open it to review **Trust relationships** and **Permissions**.

   ![IAM Roles showing ecsTaskExecutionRole and the ecs-tasks trusted entity](../../../images/5-Workshop/5.3-Infrastructure/iam-ecs-execution-role-list.png)

   *Figure 5.3.1b. The November 23, 2026 IAM role inventory shows ecsTaskExecutionRole with ECS tasks as its trusted entity.*

2. To create the role, choose **Create role → AWS service → Elastic Container Service**. Select **Elastic Container Service Task** as the use case. Verify that its trust policy allows `sts:AssumeRole` for `ecs-tasks.amazonaws.com`.
3. In **Add permissions**, attach `AmazonECSTaskExecutionRolePolicy` under **Permissions policies** so ECS can pull the ECR image and send logs through `awslogs`. A permissions boundary sets the maximum available permissions. If the account requires a boundary, use the administrator-approved boundary and verify that it permits the required operations. Create the log group first if the container uses `awslogs`.

   ![Permissions tab of ecsTaskExecutionRole showing AmazonECSTaskExecutionRolePolicy](../../../images/5-Workshop/5.3-Infrastructure/iam-ecs-execution-role-policy.png)

   *Figure 5.3.1c. AmazonECSTaskExecutionRolePolicy is attached to the execution role in this screenshot.*

4. Copy the ARN and select this role as the task definition’s **Task execution role**. If PHP calls an AWS API directly, such as to upload media to S3, assign a separate **Task role** and scope its permissions to the required bucket or prefix.
5. Scope the deployment identity’s `iam:PassRole` to the approved role ARNs with `iam:PassedToService` set to `ecs-tasks.amazonaws.com`. [AWS IAM PassRole](https://docs.aws.amazon.com/IAM/latest/UserGuide/id_roles_use_passrole.html).

#### 3. Prepare the secret and reference it in the task definition

After creating the MySQL application user, open **Secrets Manager → Store a new secret** in the project Region. The secret can use JSON keys such as `username` and `password`. Copy the full ARN from the secret details. If the application already has a secret, confirm the key names before reusing it.

The policy in [section 5.2]({{< relref "/5-Workshop/5.2-Prerequiste/" >}}) documents workshop deployment permissions. The example execution role policy scopes secret retrieval to the application secret ARN. If the task receives MySQL credentials from Secrets Manager, use **Permissions → Add permissions → Create inline policy → JSON** on the execution role to grant the access below. Figure 5.3.1c records the baseline policy. Verify secret access after adding the inline policy.

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


### Verify the configuration

- Check group membership, the trust policy, role ARN, and the execution role's access to the exact secret. The role list shows that the role exists. Its **Permissions** tab shows the attached policies.
- Run a test task. Confirm the image pull, log delivery when `awslogs` is enabled, and secret retrieval. If startup fails, use **Stopped reason** and logs to distinguish IAM failures from subnet connectivity to ECR, Secrets Manager, or CloudWatch Logs.
- When the task starts, run a test query from the application environment. If MySQL times out, check the security groups from [section 5.3.2]({{< relref "/5-Workshop/5.3-Infrastructure/5.3.2-Set-Up-VPC/" >}}). Record the result under **APP-001** and exclude secret values from screenshots.

### Conclusion

The project has a dedicated IAM group and service roles. The broad scope of `AdministratorAccess` means the **IAM-001** least-privilege criterion needs to be assessed against each member’s responsibilities. Secret injection and MySQL connectivity are verified with the Fargate task.