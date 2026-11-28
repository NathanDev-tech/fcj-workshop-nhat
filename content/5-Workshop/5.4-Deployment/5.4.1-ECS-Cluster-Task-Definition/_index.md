---
title : "Create ECS Cluster + Task Definition"
date : 2024-01-01
weight : 1
chapter : false
pre : " <b> 5.4.1 </b> "
---

## Objective

We use Amazon ECS Fargate to run the RakiBookery PHP application in Singapore (`ap-southeast-1`). This section shows the application's cluster and service, how we check the task definition, and how we change the task count for a test.

## Architecture overview

**PHP image in Amazon ECR → Task definition → Task on AWS Fargate**

The task definition stores settings such as the image, container port, CPU, and memory. The ECS service uses a task definition revision to keep the number of tasks set in `desiredCount`. We use the cluster to view services and tasks. AWS Fargate manages the infrastructure for the tasks, so we can run containers without managing EC2 instances for the cluster. [AWS guide to ECS services](https://docs.aws.amazon.com/AmazonECS/latest/developerguide/ecs_services.html)

## 1. Check the cluster and service

In **Amazon ECS → Clusters**, we select `rakibookery-prod-cluster`. The list shows **Fargate**, one service, and **No tasks running** at the time of the screenshot.

![ECS cluster list in Singapore showing rakibookery-prod-cluster and one service](../../../images/5-Workshop/5.4-Deployment/5.4.1-ECS-Cluster-Task-Definition/ecs-cluster-list.png)

In the **Services** tab, we use `rakibookery-prod-service` with the **Replica** scheduling strategy to set the number of application tasks. The service screenshot is in the step that checks for stopped tasks.

## 2. Check the task definition

In the **Services** list, open the link in the **Task definition** column to see the revision used by the service. Check the settings before starting a task.

1. In **Container details**, check the ECR image URI, container name, and application port. Compare the revision's URI with the image in Amazon ECR. [Section 5.3.5]({{< relref "/5-Workshop/5.3-Infrastructure/5.3.5-Set-Up-ECR/" >}}) shows how to check the tag and digest. If the service uses an ALB, the container name and port must match the target group settings for the service. See [section 5.4.3]({{< relref "/5-Workshop/5.4-Deployment/5.4.3-Link-ECS-with-ALB/" >}}).
2. Check **Task size**, the operating system, and the CPU architecture. The task architecture must match the image build. Fargate uses the `awsvpc` network mode.
3. Check the **task execution role** so ECS can pull the image and send logs to CloudWatch as configured. If the task uses Secrets Manager for database login details, check the variable names in the application and the role's access to the secret.

When you change the image or container settings, register a revision. Use **Update service** to select that revision. [AWS guide to task definitions](https://docs.aws.amazon.com/AmazonECS/latest/developerguide/task_definitions.html)

![ECS service health page showing the app:80 container, its target group, and No targets](../../../images/5-Workshop/5.4-Deployment/5.4.1-ECS-Cluster-Task-Definition/ECS3.png)

## 3. Start a task for testing

Open Bash in AWS CloudShell and run the following blocks in the same session.

~~~bash
REGION=ap-southeast-1
CLUSTER=rakibookery-prod-cluster
SERVICE=rakibookery-prod-service
~~~

Set `desiredCount=1` to start one task. The `wait services-stable` command waits for the service to become stable after the update.

~~~bash
aws ecs update-service --region "$REGION" --cluster "$CLUSTER" \
  --service "$SERVICE" --desired-count 1 \
  --output json --no-cli-pager &&
aws ecs wait services-stable --region "$REGION" --cluster "$CLUSTER" \
  --services "$SERVICE" --no-cli-pager
~~~

**Excerpt from the `update-service` response**

~~~text
{
  "service": {
    "serviceName": "rakibookery-prod-service",
    "status": "ACTIVE",
    "desiredCount": 1,
    "runningCount": 0,
    "pendingCount": 0,

    ...

    "enableExecuteCommand": true,
    "availabilityZoneRebalancing": "ENABLED",
    "resourceManagementType": "CUSTOMER"
  }
}
~~~

The start response shows `desiredCount=1` and `runningCount=0`. ECS accepted the request to run one task. The count changes when the task starts.

## 4. Stop the task and check the result

After testing, set `desiredCount=0` and wait for the service to become stable. Use `describe-services` to confirm **0 desired / 0 running / 0 pending**.

~~~bash
aws ecs update-service --region "$REGION" --cluster "$CLUSTER" \
  --service "$SERVICE" --desired-count 0 \
  --output json --no-cli-pager &&
aws ecs wait services-stable --region "$REGION" --cluster "$CLUSTER" \
  --services "$SERVICE" --no-cli-pager
~~~

**Excerpt from the `update-service` response**

~~~text
{
  "service": {
    "serviceName": "rakibookery-prod-service",
    "status": "ACTIVE",
    "desiredCount": 0,
    "runningCount": 1,
    "pendingCount": 0,

    ...

    "enableExecuteCommand": true,
    "availabilityZoneRebalancing": "ENABLED",
    "resourceManagementType": "CUSTOMER"
  }
}
~~~

The stop response shows `desiredCount=0` and `runningCount=1`. Wait for the task to stop, then check the service counts again.

Use `list-tasks` to check the tasks for this service.

~~~bash
aws ecs list-tasks \
  --cluster rakibookery-prod-cluster \
  --service-name rakibookery-prod-service \
  --region ap-southeast-1 \
  --output json --no-cli-pager
~~~

This is the expected output after scaling to `0` tasks.

~~~json
{
  "taskArns": []
}
~~~

By default, `list-tasks` filters for a desired status of `RUNNING`. Check the empty list and the service counts to confirm the task has stopped. [AWS CLI guide to list-tasks](https://docs.aws.amazon.com/cli/latest/reference/ecs/list-tasks.html)

In the console, the **Services** tab shows `rakibookery-prod-service` as **Active** with **0/0 Tasks running**. **Cluster overview** shows **0 running / 0 pending**.

![Services tab showing rakibookery-prod-service as Active with zero out of zero tasks running](../../../images/5-Workshop/5.4-Deployment/5.4.1-ECS-Cluster-Task-Definition/ecs-cluster-services.png)

## Conclusion

The console shows that `rakibookery-prod-cluster` uses Fargate and has one service, `rakibookery-prod-service`. At the time of the screenshot, the service was **Active** with **0/0 Tasks running**. We use the same service to run one task when we test the application and scale to `0` when we finish.
