---
title : "Create ALB + Target Group + Health Check"
date : 2024-01-01
weight : 2
chapter : false
pre : " <b> 5.4.2 </b> "
---

## Objective

After we set up the ECS cluster and task definition, our team needed an address the browser could use to open the RakiBookery PHP website. During an ECS deployment, the service can replace Fargate tasks. We use an Application Load Balancer (ALB) to receive web requests. Its target group holds the registered task IP addresses.

## Prepare the listener and network

Use the VPC that runs the ECS service. Prepare two public subnets in different Availability Zones. Also prepare the ALB security group and the task security group. For a new environment, choose **EC2 → Load Balancers → Create load balancer → Application Load Balancer**. Select **Internet-facing**, the two public subnets, and the ALB security group. Use the listener and target group settings below. [ALB deployment document](https://app.notion.com/p/ALB-Target-Group-Health-Check-3f84cb9c2604807caf58cceadee8794c).

![Source screenshot showing RakiBookery-alb with Internet-facing and IPv4 selected](../../../images/5-Workshop/5.4-Deployment/5.4.2-ALB-Target-Group/01-alb-internet-facing.png)

## Set up the ALB and target group

**1. Create an IP target group.** Fargate tasks use `awsvpc` and each task has its own IP address. We created `RakiBookery-tg` with the `IP` target type. We used HTTP port `80` to match the Apache container in section 5.4.1. When the service starts tasks, ECS registers their IP addresses with this target group.

![Create the RakiBookery-tg IP target group on HTTP port 80](../../../images/5-Workshop/5.4-Deployment/5.4.2-ALB-Target-Group/02-target-group-ip.png)

![The new target group has zero registered targets](../../../images/5-Workshop/5.4-Deployment/5.4.2-ALB-Target-Group/03-target-group-created.png)

**2. Check the ALB listener.** In the workshop setup, `RakiBookery-alb` in Singapore (`ap-southeast-1`) uses the `HTTP:80` listener. Its rule sends each request to the application's target group. The listener receives the request, and the target group lists the tasks that can receive it.

![The ALB uses two public subnets in the RakiBookery VPC](../../../images/5-Workshop/5.4-Deployment/5.4.2-ALB-Target-Group/04-alb-network-mapping.png)

![The active ALB uses HTTP:80 to send requests to RakiBookery-tg](../../../images/5-Workshop/5.4-Deployment/5.4.2-ALB-Target-Group/05-alb-http-listener.png)

**3. Check both security groups.** The ALB security group allows incoming traffic on the listener port from the sources in its rules. The task security group allows traffic on TCP port `80` from the ALB security group. If the health check uses a different port, check whether the ALB can reach that port too.

![The ALB security group's inbound rule for HTTP port 80](../../../images/5-Workshop/5.4-Deployment/5.4.2-ALB-Target-Group/06-alb-security-group.png)

**4. Set up the health check.** The ALB sends an HTTP request to a test path on each target. The screenshots show the path `/`, traffic port `80`, and success code `200`. The source settings show an interval of `30` seconds and a timeout of `5` seconds. The healthy and unhealthy thresholds are both `2` checks in a row. During testing, check that `/` returns `200`. [AWS ALB health checks](https://docs.aws.amazon.com/elasticloadbalancing/latest/application/target-group-health-checks.html).

![The HTTP health check uses the / path](../../../images/5-Workshop/5.4-Deployment/5.4.2-ALB-Target-Group/07-health-check-path.png)

![The health check uses the traffic port and accepts HTTP 200](../../../images/5-Workshop/5.4-Deployment/5.4.2-ALB-Target-Group/08-health-check-settings.png)

## Conclusion

We configured an internet-facing ALB with an HTTP listener on port `80`, an IP target group, and an HTTP health check. These settings define how requests reach the PHP container and how the ALB checks each target. The next step is to connect the ECS service to this target group and test website access through the ALB.

