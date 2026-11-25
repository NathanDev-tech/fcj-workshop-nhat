---
title : "Create ECS Cluster + Task Definition"
date : 2024-01-01
weight : 1
chapter : false
pre : " <b> 5.4.1 </b> "
---

## Objective

After pushing the Docker image to ECR, our team needed to define how the PHP application would run on AWS. An **ECS cluster** groups the tasks. A **task definition** specifies the image, resources, port, permissions, and environment variables. We chose Fargate so AWS would manage the compute infrastructure.

## Step 1. Create the ECS cluster

In Singapore (`ap-southeast-1`), open **Amazon ECS → Clusters → Create cluster**. Name it `RakiBookery-cluster` and select **Fargate only** under infrastructure. ECS confirmed creation and listed the cluster. At the time of the capture, the new cluster had no running tasks.

![Open Create cluster in Amazon ECS](../../../images/5-Workshop/5.4-Deployment/5.4.1-ECS-Cluster-Task-Definition/01-cluster-create.png)

![Name the cluster RakiBookery-cluster and select Fargate only](../../../images/5-Workshop/5.4-Deployment/5.4.1-ECS-Cluster-Task-Definition/02-cluster-fargate.png)

![ECS confirms that the RakiBookery cluster was created](../../../images/5-Workshop/5.4-Deployment/5.4.1-ECS-Cluster-Task-Definition/03-cluster-created.png)




## Step 2. Configure the task definition

1. Open **Task definitions → Create new task definition**. Set the family to `RakiBookery-task` and choose **AWS Fargate**.
2. Choose **Linux/X86_64**, keep the `awsvpc` network mode, and set the task size to **0.25 vCPU and 1 GB of memory**. The architecture matches the `linux/amd64` image from section 5.3.5.
3. Name the essential container `Main` and enter the image URI from the `rakibookery` ECR repository. The capture shows the `latest` tag used during setup. Set **Container port** to `80`, **Protocol** to `TCP`, and **App protocol** to `HTTP` to match Apache and the ALB setup in the following sections.

![Open Create new task definition](../../../images/5-Workshop/5.4-Deployment/5.4.1-ECS-Cluster-Task-Definition/04-task-create.png)

![Select the RakiBookery-task family and AWS Fargate](../../../images/5-Workshop/5.4-Deployment/5.4.1-ECS-Cluster-Task-Definition/05-task-family.png)

![Configure Linux X86_64 with 0.25 vCPU and 1 GB of memory](../../../images/5-Workshop/5.4-Deployment/5.4.1-ECS-Cluster-Task-Definition/06-task-resources.png)

![The Main container uses an ECR image and port 80](../../../images/5-Workshop/5.4-Deployment/5.4.1-ECS-Cluster-Task-Definition/07-task-container.png)


After saving, the console showed `RakiBookery-task:1` as `ACTIVE` and listed `ecsTaskExecutionRole` as the execution role.

![RakiBookery-task:1 is ACTIVE after creation](../../../images/5-Workshop/5.4-Deployment/5.4.1-ECS-Cluster-Task-Definition/08-task-created.png)


## Step 3. Grant ECS access to the secret

In **IAM → Roles → ecsTaskExecutionRole → Permissions**, check `AmazonECSTaskExecutionRolePolicy` for pulling the image and writing CloudWatch logs. Our team created the inline policy `ECSSecretsAccessPolicy` so ECS could retrieve the database credentials when the container starts.

The demo policy had broad resource scope. For the PHP deployment, grant `secretsmanager:GetSecretValue` on the specific RDS secret ARN. The source capture shows the default `aws/secretsmanager` key. If the secret uses a customer-managed KMS key, grant `kms:Decrypt` on that key's ARN. Add these permissions to the **task execution role** because ECS reads the secret during startup. [AWS ECS task execution role documentation](https://docs.aws.amazon.com/AmazonECS/latest/developerguide/task_execution_IAM_role.html).

![The ecsTaskExecutionRole permissions page and the inline policy option](../../../images/5-Workshop/5.4-Deployment/5.4.1-ECS-Cluster-Task-Definition/09-execution-role.png)

![IAM confirms that ECSSecretsAccessPolicy was created](../../../images/5-Workshop/5.4-Deployment/5.4.1-ECS-Cluster-Task-Definition/10-execution-policy-created.png)


## Step 4. Create a revision and map environment variables

Open the RDS secret in **Secrets Manager → Secrets** to obtain the **Secret ARN**. In the `RakiBookery-task` family, choose **Create new revision** and open **Environment variables** for container `Main`.

![Secret ARN field in Secrets Manager, with identifiers redacted](../../../images/5-Workshop/5.4-Deployment/5.4.1-ECS-Cluster-Task-Definition/11-database-secret.png)

![Open Create new revision in the RakiBookery-task family](../../../images/5-Workshop/5.4-Deployment/5.4.1-ECS-Cluster-Task-Definition/12-task-revision.png)

![Container environment variables in the new revision form](../../../images/5-Workshop/5.4-Deployment/5.4.1-ECS-Cluster-Task-Definition/13-environment-editor.png)

![Source capture showing ValueFrom for the two credential fields](../../../images/5-Workshop/5.4-Deployment/5.4.1-ECS-Cluster-Task-Definition/14-secret-valuefrom.png)

The setup capture shows `username` and `password` fields with the `ValueFrom` type. For the PHP website, match the variable names to the application and select the individual JSON keys.

- `DB_USER` reads `<SECRET_ARN>:username::`.
- `DB_PASS` reads `<SECRET_ARN>:password::`.
- `DB_HOST` is the RDS endpoint and `DB_NAME` is `bookstoredb`.

A full ARN without a JSON key suffix injects the entire secret into the variable. Selecting a JSON key requires Linux Fargate platform version **1.4.0 or later**. Save the revision and record its number so you can select it for the test run. [AWS documentation on injecting secrets into ECS environment variables](https://docs.aws.amazon.com/AmazonECS/latest/developerguide/secrets-envvar-secrets-manager.html).

## Step 5. Run a test task and inspect the logs

1. In `RakiBookery-cluster`, open **Tasks → Run new task** and select Fargate and the revision with the secret settings. Check the VPC, subnet, and security group. Our notes describe a trial using a public subnet and public IP. For private subnets, verify connectivity to ECR, S3, Secrets Manager, and CloudWatch Logs through NAT or the appropriate endpoints.
2. Watch for the task to reach `RUNNING`, then open **Logs** to check container startup. If it reaches `STOPPED`, inspect the **Stopped reason** and container error to narrow down image pulls, secret retrieval, or application startup.
3. To confirm PHP can connect to MySQL, perform a read through the website and inspect the corresponding PHP task log. See [section 5.4.5]({{< relref "/5-Workshop/5.4-Deployment/5.4.5-Connect-PHP-App-To-RDS/" >}}).

## Conclusion

The team report confirms that the cluster, first task definition revision, and inline policy were created. The final capture shows the `ValueFrom` form during revision setup. The source has no capture of a running PHP task or its Apache logs. The checks above describe how to verify that run before linking the ECS service to the ALB.