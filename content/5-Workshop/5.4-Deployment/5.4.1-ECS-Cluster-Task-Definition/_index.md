---
title : "Create ECS Cluster + Task Definition"
date : 2024-01-01
weight : 1
chapter : false
pre : " <b> 5.4.1 </b> "
---

## Objective

After we pushed the Docker image to ECR, our team needed to set up the PHP application on AWS. We use an **ECS cluster** to manage the RakiBookery tasks. The **task definition** stores the settings for the container, including its image, CPU, memory, port, roles, and environment variables. We chose Fargate so AWS could manage the servers that run the tasks.

## What we need before creating the task

Prepare the image URI in ECR, `ecsTaskExecutionRole`, and the ARN of the database secret from section 5.3. Check that the log group is ready if the task uses `awslogs`. [ECS deployment document](https://docs.google.com/document/d/1_qk9uf_5MlByeEo4Yaxt8YJSsoqWjTnO/edit). The cluster is where we run and watch tasks. The task definition describes the image, port, CPU, memory, and role. An ECS service later uses that definition to keep tasks running.

## Step 1. Create the ECS cluster

In the Singapore Region (`ap-southeast-1`), we opened **Amazon ECS → Clusters → Create cluster**. We named the cluster `RakiBookery-cluster` and chose **Fargate only** under infrastructure. ECS showed a success message and added the cluster to the list. The new cluster had no running tasks when we took the screenshot.

![Open Create cluster in Amazon ECS](../../../images/5-Workshop/5.4-Deployment/5.4.1-ECS-Cluster-Task-Definition/01-cluster-create.png)

![Name the cluster RakiBookery-cluster and choose Fargate only](../../../images/5-Workshop/5.4-Deployment/5.4.1-ECS-Cluster-Task-Definition/02-cluster-fargate.png)

![ECS confirms that it created the RakiBookery cluster](../../../images/5-Workshop/5.4-Deployment/5.4.1-ECS-Cluster-Task-Definition/03-cluster-created.png)

## Step 2. Set up the task definition

1. Open **Task definitions → Create new task definition**. Name the family `RakiBookery-task` and choose **AWS Fargate**.
2. Choose **Linux/X86_64**, keep the `awsvpc` network mode, and set **0.25 vCPU and 1 GB of memory**, as shown in our setup screenshot. The x86_64 architecture matches the `linux/amd64` image from section 5.3.5.
3. Name the main container `Main`, select **Essential**, and enter the image URI from the `rakibookery` ECR repository. The screenshot shows that we used the `latest` tag for this setup. Set **Container port** to `80`, **Protocol** to `TCP`, and **App protocol** to `HTTP`. These settings match Apache and the ALB setup in the next sections.

![Open Create new task definition](../../../images/5-Workshop/5.4-Deployment/5.4.1-ECS-Cluster-Task-Definition/04-task-create.png)

![Choose the RakiBookery-task family and AWS Fargate](../../../images/5-Workshop/5.4-Deployment/5.4.1-ECS-Cluster-Task-Definition/05-task-family.png)

![Linux X86_64 with 0.25 vCPU and 1 GB of memory](../../../images/5-Workshop/5.4-Deployment/5.4.1-ECS-Cluster-Task-Definition/06-task-resources.png)

![The Main container uses an ECR image and port 80](../../../images/5-Workshop/5.4-Deployment/5.4.1-ECS-Cluster-Task-Definition/07-task-container.png)

After we saved the task definition, the Console showed `RakiBookery-task:1` as `ACTIVE`. This means ECS has registered the revision and can use it to start a task. We still need a running task and its logs to check the PHP application. The details also showed `ecsTaskExecutionRole` as the task execution role.

![RakiBookery-task:1 is ACTIVE after creation](../../../images/5-Workshop/5.4-Deployment/5.4.1-ECS-Cluster-Task-Definition/08-task-created.png)


## Step 3. Give ECS permission to read the secret

In **IAM → Roles → ecsTaskExecutionRole → Permissions**, we checked the `AmazonECSTaskExecutionRolePolicy` policy. It lets ECS pull the image and write logs to CloudWatch Logs. We added the inline policy `ECSSecretsAccessPolicy` so ECS could read the database login details from Secrets Manager when the container starts.

The demo policy gave access to a wide range of resources. For the PHP setup, give `secretsmanager:GetSecretValue` permission on the ARN of the RDS secret. The screenshot shows the default `aws/secretsmanager` key. If we use a KMS key managed by the team, also give `kms:Decrypt` permission on that key's ARN. Add these permissions to the **task execution role** because ECS reads the secret when the task starts. [AWS ECS task execution role](https://docs.aws.amazon.com/AmazonECS/latest/developerguide/task_execution_IAM_role.html).

![The ecsTaskExecutionRole permissions page and the inline policy option](../../../images/5-Workshop/5.4-Deployment/5.4.1-ECS-Cluster-Task-Definition/09-execution-role.png)

![IAM confirms that it added ECSSecretsAccessPolicy](../../../images/5-Workshop/5.4-Deployment/5.4.1-ECS-Cluster-Task-Definition/10-execution-policy-created.png)

## Step 4. Create a revision and set environment variables

Open **Secrets Manager → Secrets**. Open the RDS secret to find its **Secret ARN**. Then open the `RakiBookery-task` family, choose **Create new revision**, and go to **Environment variables** for the `Main` container.

![The Secret ARN field in Secrets Manager, with identifiers hidden](../../../images/5-Workshop/5.4-Deployment/5.4.1-ECS-Cluster-Task-Definition/11-database-secret.png)

![Open Create new revision in the RakiBookery-task family](../../../images/5-Workshop/5.4-Deployment/5.4.1-ECS-Cluster-Task-Definition/12-task-revision.png)

![Container environment variables in the new revision](../../../images/5-Workshop/5.4-Deployment/5.4.1-ECS-Cluster-Task-Definition/13-environment-editor.png)

![ValueFrom for the two login fields](../../../images/5-Workshop/5.4-Deployment/5.4.1-ECS-Cluster-Task-Definition/14-secret-valuefrom.png)

The screenshot shows `username` and `password` with the `ValueFrom` type. For PHP, use the names that the application reads. In this project they are `DB_USER` and `DB_PASS`. Each one must point to its matching JSON key in the secret.

- `DB_USER` gets its value from `<SECRET_ARN>:username::`.
- `DB_PASS` gets its value from `<SECRET_ARN>:password::`.
- `DB_HOST` uses the RDS endpoint, and `DB_NAME` uses `bookstoredb`.

When we enter the full secret ARN without a JSON key at the end, ECS puts the whole secret in the variable. To pick one JSON key, use Fargate platform version **1.4.0 or later** on Linux. Save the new revision and record its number for the test run. If the secret value changes later, start a new task so it receives the new value. [AWS guide to using secrets in ECS environment variables](https://docs.aws.amazon.com/AmazonECS/latest/developerguide/secrets-envvar-secrets-manager.html).

## Step 5. Run a test task and check the logs

1. In `RakiBookery-cluster`, open **Tasks → Run new task**. Choose Fargate and the revision with the secret settings. Check the VPC, subnet, and security group before starting the task. For the test run, use a public subnet and a public IP. For a private subnet, make sure the task can reach ECR, S3, Secrets Manager, and CloudWatch Logs through NAT or the right endpoints.
2. Wait for the task to reach `RUNNING`, then open **Logs** to check if the container started. If the task reaches `STOPPED`, read **Stopped reason** and the container error. Check whether the task could pull the image, read the secret, and start the application.
3. To check if PHP can connect to MySQL, open a page that reads data from the database. Check the PHP task log for that request. See [section 5.4.5]({{< relref "/5-Workshop/5.4-Deployment/5.4.5-Connect-PHP-App-To-RDS/" >}}).

Before testing with the ALB, check that the task uses the expected revision. In the task details, record the image digest if the Console shows it. Check the Apache startup message in the log stream for container `Main`. After a test with a standalone task, stop that task when it is no longer needed.


## Conclusion

We created the ECS cluster and the first task definition for the PHP container. We also added permission for the task execution role to read the database secret. The image, port, environment variables, and secret settings must match the PHP application. This setup provides the base for running tasks and creating the ECS service in the next sections.