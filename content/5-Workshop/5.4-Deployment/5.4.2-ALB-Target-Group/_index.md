---
title : "Create ALB + Target Group + Health Check"
date : 2026-11-12
weight : 2
chapter : false
pre : " <b> 5.4.2 </b> "
---


# ALB + Target Group + Health Check Configuration Guide

## 1. Create Target Group

### 1.1. Open Target Groups

1. Log in to the AWS EC2 Console.

2. Select the **Singapore (ap-southeast-1)** Region.

3. In the left navigation pane, scroll down to **Load Balancing**.

4. Select **Target Groups**.

5. Click **Create target group**.

### 1.2. Basic configuration

Fill in the information according to the following table:

| Field | Value | 
| ----- | ----- | 
| **Choose a target type** | IP addresses | 
| **Target group name** | RakiBookery-tg | 
| **Protocol** | HTTP | 
| **Port** | 80 | 
| **IP address type** | IPv4 | 
| **VPC** | VPC currently used for RakiBookery | 
| **Protocol version** | HTTP1 | 

> **Important:** Select **IP addresses**, do not select *Instances*, because ECS Fargate uses the `awsvpc` network mode.

![Basic Configuration Illustration](your_image_url_1)

### 1.3. Create Target Group

1. Click **Next**.

2. In the **Register targets** section, you do not need to enter any IP addresses yet.

3. Click **Create target group**.

*Note: The ECS Service will automatically register Fargate task IPs to the Target Group when we link them. After creation, it is completely normal for the Target Group to have **0 registered targets**.*

![Target Group Creation Illustration](your_image_url_2)

## 2. Create Application Load Balancer (ALB)

**Check and Initialize:**

1. Go to **EC2** → **Load Balancers**.

2. In the left menu, select **Load Balancing** → **Load Balancers** (Check the existing ALB list).

3. Click **Create load balancer** → Select **Application Load Balancer**.

### 2.1. Basic configuration

| Field | Value | 
| ----- | ----- | 
| **Load balancer name** | RakiBookery-alb | 
| **Scheme** | Internet-facing | 
| **IP address type** | IPv4 | 

### 2.2. Network mapping

| Field | Value | 
| ----- | ----- | 
| **VPC** | VPC of RakiBookery-tg | 
| **Availability Zones** | At least 2 different AZs | 
| **Subnets** | 2 corresponding public subnets | 

> **Note:** The Public subnet must have a `0.0.0.0/0` route pointing to the Internet Gateway.

![Network Mapping Illustration](your_image_url_3)

### 2.3. Security groups

You need to create a separate security group for the ALB.

**Step 1: Create Security Group for ALB**
Open a new AWS tab:

1. Go to **EC2** → **Security Groups**.

2. Click **Create security group**.

3. Enter the following information:

| Field | Value | 
| ----- | ----- | 
| **Security group name** | RakiBookery-alb-sg | 
| **Description** | Security group for RakiBookery ALB | 
| **VPC** | Same VPC as the ALB | 

**Step 2: Configure Rules**

* **Inbound rules:** Click Add rule and enter:

  * **Type:** HTTP

  * **Port:** 80

  * **Source:** Anywhere-IPv4 (0.0.0.0/0)

* **Outbound rules:** Keep the default *All traffic* for this demo step.

Click **Create security group**.
Then return to the ALB creation tab, select the newly created Security Group `RakiBookery-alb-sg` (which allows inbound HTTP port 80 from `0.0.0.0/0`).

### 2.4. Listeners and routing

| Field | Value | 
| ----- | ----- | 
| **Protocol** | HTTP | 
| **Port** | 80 | 
| **Default action** | Forward to target groups | 
| **Target group** | RakiBookery-tg | 

### 2.5. Create ALB

Click **Create load balancer** and wait for the status to change to **Active**.

* **Generated DNS name:** `RakiBookery-alb-1141293019.ap-southeast-1.elb.amazonaws.com`

![ALB Creation Illustration](your_image_url_4)

## 3. Attach ALB to ECS Service

1. Navigate to **Amazon ECS** → **Clusters** → Select `RakiBookery-cluster`.

2. Select the **Services** tab.

3. Click on the Service `RakiBookery-task-service-01`.

4. Click **Update** in the top right corner.

5. Locate the **Load balancing** section.

6. Configure with the following parameters:

| Section | Value | 
| ----- | ----- | 
| **Load balancer type** | Application Load Balancer | 
| **Use existing load balancer** | RakiBookery-alb | 
| **Listener** | HTTP:80 | 
| **Use existing target group** | RakiBookery-tg | 
| **Container** | Main | 
| **Container port** | 80 | 

![ECS Service Link Illustration](your_image_url_5)

> **Summary:** AWS-006 has completed the configuration of the ALB, Target Group, Health Check, Listener, and Security Groups. The ECS Service has been successfully linked with the ALB. We are currently waiting for the team to spin up Fargate to test the Health Check and access the actual website.