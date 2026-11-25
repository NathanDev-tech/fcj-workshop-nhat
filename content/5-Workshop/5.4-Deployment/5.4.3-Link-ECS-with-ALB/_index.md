---
title: "Link ECS Service with ALB"
date: 2024-01-01
weight: 3
chapter: false
pre: " <b> 5.4.3. </b> "
---

## Why link the ECS service to the ALB?

In [section 5.4.2]({{< relref "/5-Workshop/5.4-Deployment/5.4.2-ALB-Target-Group" >}}), our team prepared the listener and target group. We configure the ECS service with that target group and the PHP container name and port `80` from the task definition so the ALB can reach the website tasks.

```text
Browser → ALB (HTTP:80 listener) → Target group (IP:80) → Fargate task (PHP/Apache)
                                    ↑
                          ECS service registers task IPs
```

A new revision may start a task with a different IP address. ECS registers new targets and deregisters old targets during deployment, while users continue to use the same ALB address. The team checks this mapping on the **ECS service**. [AWS documentation on ECS and ALB](https://docs.aws.amazon.com/AmazonECS/latest/developerguide/alb.html).

## Reviewing the RakiBookery configuration

**1.** Open **ECS → Clusters → Services**, select `RakiBookery-task-service-01` in `RakiBookery-cluster`, and inspect **Load balancing**. Match the target group to the one in the `RakiBookery-alb` listener rule.

![Connecting Main:80 to the HTTP:80 listener and RakiBookery-tg](../../../images/5-Workshop/5.4-Deployment/5.4.3-Link-ECS-with-ALB/01-ecs-load-balancing.png)

**2.** Match `Main:80` to the configured task definition revision. Check the subnet and task security group against section 5.4.2.

![ECS service page showing networking and the load balancing association](../../../images/5-Workshop/5.4-Deployment/5.4.3-Link-ECS-with-ALB/02-ecs-service-configuration.png)

**3.** When Fargate is started with a desired count above zero, watch **Deployments**, **Events**, and **Tasks**. Match the running task IP to **Target Groups → Targets**.

## Checking the full request path

- **Task `RUNNING`.** ECS has started the task. If it stops, use its stopped reason and logs to investigate image, secret, or startup failures.
- **Target `Healthy`.** The ALB has received an accepted health check response. Use the reason for an unhealthy status to narrow down port, network, or HTTP response issues.
- **Website through the ALB.** Open an actual application page through the ALB DNS name. Then test CloudFront in section 5.4.4 and MySQL reads and writes in section 5.4.5.

## Recorded result

The deployment notes confirm that the ECS service was linked to the ALB. At the time of the notes, the Fargate health check and website access test were pending. In a verification run, capture the **Load balancing** configuration, task IP in **Targets**, and browser result together. [Team ALB notes](https://app.notion.com/p/ALB-Target-Group-Health-Check-3f84cb9c2604807caf58cceadee8794c).
