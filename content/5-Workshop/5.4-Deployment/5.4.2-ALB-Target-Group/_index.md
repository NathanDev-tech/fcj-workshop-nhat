---
title : "Create ALB + Target Group + Health Check"
date : 2024-01-01
weight : 2
chapter : false
pre : " <b> 5.4.2 </b> "
---

## Objective

After preparing the ECS cluster and task definition, our team needed a browser entry point for the RakiBookery PHP site. Fargate tasks may be replaced during deployment, so the Application Load Balancer (ALB) receives requests and the target group tracks the task IPs that can serve them.

## ALB and target group configuration

**1. Use IP targets.** Fargate tasks use `awsvpc` and have their own IPs. Create `RakiBookery-tg` with `IP` as the target type and use HTTP port `80` to match the Apache container in section 5.4.1. ECS registers the task IPs when the service starts.

![Creating the IP target group RakiBookery-tg on HTTP port 80](../../../images/5-Workshop/5.4-Deployment/5.4.2-ALB-Target-Group/01-target-group-ip.png)

![The newly created target group has no registered targets](../../../images/5-Workshop/5.4-Deployment/5.4.2-ALB-Target-Group/02-target-group-created.png)

**2. Check the ALB listener.** The workshop configuration uses listener `HTTP:80` on `RakiBookery-alb` in Singapore (`ap-southeast-1`). The listener forwards requests to the application target group.

![ALB network mapping across two RakiBookery public subnets](../../../images/5-Workshop/5.4-Deployment/5.4.2-ALB-Target-Group/03-alb-network-mapping.png)

![Active ALB with HTTP:80 forwarding to RakiBookery-tg](../../../images/5-Workshop/5.4-Deployment/5.4.2-ALB-Target-Group/04-alb-http-listener.png)

**3. Check the security groups at both ends.** Allow traffic from the configured sources on the ALB listener. Allow TCP `80` from the ALB security group on the task security group. Review the path to the health check port if it differs from the application port.

![HTTP port 80 inbound rule on the ALB security group](../../../images/5-Workshop/5.4-Deployment/5.4.2-ALB-Target-Group/05-alb-security-group.png)

**4. Configure the health check.** The ALB sends an HTTP request to the configured path on each target. The screenshots show path `/`, traffic port `80`, and success code `200`. The interval is `30` seconds, the timeout is `5` seconds, and both the healthy and unhealthy thresholds are `2` consecutive checks. During testing, verify that `/` returns `200`.

![HTTP health check on the / path](../../../images/5-Workshop/5.4-Deployment/5.4.2-ALB-Target-Group/06-health-check-path.png)

![Health check uses the traffic port and accepts HTTP 200](../../../images/5-Workshop/5.4-Deployment/5.4.2-ALB-Target-Group/07-health-check-settings.png)

## Results and evidence

The team notes record the ALB, target group, health check, listener, and security groups as configured. At the time of the notes, the team was waiting to start Fargate to confirm healthy targets and reach the site. [Section 5.4.3]({{< relref "/5-Workshop/5.4-Deployment/5.4.3-Link-ECS-with-ALB" >}}) explains task registration and browser verification.

The screenshots above document the ALB and target group configuration at setup time. [Team ALB notes](https://app.notion.com/p/ALB-Target-Group-Health-Check-3f84cb9c2604807caf58cceadee8794c) · [AWS health check documentation](https://docs.aws.amazon.com/elasticloadbalancing/latest/application/target-group-health-checks.html).

