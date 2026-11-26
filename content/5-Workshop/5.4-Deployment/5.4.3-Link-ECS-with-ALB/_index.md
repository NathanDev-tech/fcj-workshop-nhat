---
title: "Link ECS Service with ALB"
date: 2024-01-01
weight: 3
chapter: false
pre: " <b> 5.4.3. </b> "
---

## Objective

In [section 5.4.2]({{< relref "/5-Workshop/5.4-Deployment/5.4.2-ALB-Target-Group" >}}), our team set up a listener and a target group. To help the ALB find the tasks that run our website, we set the ECS service to use that target group. We used the PHP container name and port `80` from the task definition.

```text
Browser → ALB (HTTP:80 listener) → Target group (IP:80) → Fargate task (PHP/Apache)
                                    ↑
                          ECS service registers task IPs
```

For example, when we deploy a new revision, ECS can start a task with a new IP address. ECS registers the new task IP and removes the old task IP from the target group as the deployment progresses. The ALB DNS name stays the same during this process. ECS registers and removes task IPs automatically for the service. Our team checks the target group link in the **ECS service** settings. [AWS guide to using an ALB with ECS](https://docs.aws.amazon.com/AmazonECS/latest/developerguide/alb.html).

## Create or check the service

If the service already exists, open its settings and compare them with the steps below. For a new environment, create a service in `RakiBookery-cluster` with Fargate, the task definition revision from section 5.4.1, and a desired task count above `0` for the test. Select the VPC, the subnets where the tasks will run, and the task security group. In **Load balancing**, choose the ALB and the IP target group from section 5.4.2, then select container `Main` and port `80`.

## Check the RakiBookery settings

**1.** Open **ECS → Clusters → Services**. Select `RakiBookery-task-service-01` in `RakiBookery-cluster` and open **Load balancing**. Check that it uses the target group in the listener rule for `RakiBookery-alb`.

![Link Main:80 to the HTTP:80 listener and RakiBookery-tg](../../../images/5-Workshop/5.4-Deployment/5.4.3-Link-ECS-with-ALB/01-ecs-load-balancing.png)

**2.** Check that the **container name / container port** is `Main:80` and matches the task definition for the revision we are deploying. Check the task subnet and security group against the settings in section 5.4.2.

![The ECS service page shows the network settings and load balancing link](../../../images/5-Workshop/5.4-Deployment/5.4.3-Link-ECS-with-ALB/02-ecs-service-configuration.png)

**3.** When the service starts Fargate tasks with a desired task count above `0`, check **Deployments**, **Events**, and **Tasks**. Find the IP address of the running task and look for it under **Target Groups → Targets**.

## How we check the request path

- **Task `RUNNING`.** ECS has started the task. Check the PHP startup logs too. If the task stops, read **Stopped reason** and the logs. Check for a problem with the image, secret, or container start.
- **Target `Healthy`.** The target passed the ALB health check for the configured path and response code. If the target is `Unhealthy`, read the reason to see if the problem is with the port, network path, or HTTP response.
- **Website opens through the ALB.** Use the ALB DNS name to open an application page. Then test access through CloudFront in section 5.4.4 and MySQL reads and writes in section 5.4.5.

Check a real URL on the ALB after the target becomes `Healthy`. Then compare the task's private IP with the IP under **Targets**. After an update, record the revision and the time of the browser request. If the application logs identify the task that handled the request, match that task with its registered IP.

> **Screenshot to add.** Capture **Deployments** or **Tasks** with the running revision, then capture **Targets** with its IP and `Healthy` status. Save the RakiBookery page opened through the ALB DNS name from the same test.

## Conclusion

The ECS service is linked to the ALB through the IP target group and container `Main:80`. With this link, ECS manages task registration as tasks change during a deployment. To confirm website access, check the running task, its target health, and the page opened through the ALB DNS name.

