---
title : "Initialize and Configure VPC"
date : 2024-01-01 
weight : 2
chapter : false
pre : " <b> 5.3.2 </b> "
---


### Objective

Our goal is to set up the RakiBookery network so user requests go to the **Application Load Balancer (ALB)**, then to **ECS Fargate**. The application uses **RDS for MySQL**. The ALB receives traffic in public subnets. In this design, the PHP tasks and RDS use private subnets in the same VPC.

### What we need

- Sign in to the project AWS account and select **Asia Pacific (Singapore)** (`ap-southeast-1`). Use the approved deployment identity from [section 5.3.1]({{< relref "/5-Workshop/5.3-Infrastructure/5.3.1-Set-Up-IAM/" >}}).
- Check the **VPC ID**, the IPv4 range, and the two Availability Zones. The VPC `RakiBookery-vpc-01` has the CIDR `10.11.0.0/16`. The RDS screenshot in section 5.3.4 shows a VPC with the same name and a different ID. Use the VPC ID from the environment where you are working.
- Check the PHP container port, the ALB health check port, and which AWS services the task needs when it starts.

### Why we use public and private subnets

The ALB receives web traffic in public subnets in two Availability Zones. The task accepts traffic from the ALB, and RDS accepts MySQL traffic from the approved application and import clients. Each subnet's route table controls where packets can go. Security groups control which connections the resources accept. We check both layers for the same VPC.

### Deployment setup

#### 1. Create or check the VPC and subnets

1. Open **VPC → Your VPCs**. If the VPC already exists, open **Details** and check the CIDR, **DNS resolution**, and **DNS hostnames**. For a new environment, choose **Create VPC → VPC only** and enter the approved CIDR.
2. Open **Subnets** and check that each subnet is in the right VPC. If you need new subnets, choose **Create subnet** and use the settings shown in the Console screenshot.

   - `ap-southeast-1a` uses public subnet `10.11.1.0/24` and private subnet `10.11.3.0/24`.
   - `ap-southeast-1b` uses public subnet `10.11.2.0/24` and private subnet `10.11.4.0/24`.

![Resource map with the four RakiBookery subnets](../../../images/5-Workshop/5.3-Infrastructure/5.3.2-Set-Up-VPC/01-vpc-resource-map.png)

#### 2. Attach the Internet Gateway and set up routes

1. In **VPC → Internet gateways**, check that `RakiBookery-igw` is attached to the right VPC. For a new setup, create the gateway and choose **Attach to a VPC**.
2. For the public route table, open **Route tables → Routes**. Add or check the default route `0.0.0.0/0` to the Internet Gateway. The route `10.11.0.0/16 → local` is for traffic inside the VPC.

![RakiBookery public route table](../../../images/5-Workshop/5.3-Infrastructure/5.3.2-Set-Up-VPC/02-vpc-public-route-table.png)

3. Open **Subnet associations** and check that both public subnets use the public route table. The Console screenshot shows that the private subnets use the main route table by default. During deployment, check which route table each private subnet uses. Make sure it has no default route that goes straight to the Internet Gateway.

![Public subnet associations and private subnets using the main route table](../../../images/5-Workshop/5.3-Infrastructure/5.3.2-Set-Up-VPC/03-vpc-subnet-associations.png)

4. Set up outbound access before you run a Fargate task. For a NAT setup, create a public NAT Gateway in a public subnet and add a route from the task subnet to that gateway. For a VPC endpoint setup, create `ecr.api` and `ecr.dkr` interface endpoints with Private DNS. Associate the S3 gateway endpoint with the route table used by the task subnet. If the task uses Secrets Manager or `awslogs`, check that it can reach those services too. On the interface endpoint security group, allow inbound HTTPS `443` from the task security group. Check the option you use during deployment. [AWS guide to private subnets](https://docs.aws.amazon.com/vpc/latest/userguide/vpc-example-private-subnets-nat.html) · [AWS guide to ECR VPC endpoints](https://docs.aws.amazon.com/AmazonECR/latest/userguide/vpc-endpoints.html).
#### 3. Limit connections with security groups

In **VPC → Security groups**, choose the right VPC and check the traffic rules for each component.

- **ALB.** Allow incoming traffic on the listener port in use. For example, use HTTPS `443` if you set up an HTTPS listener.
- **ECS Fargate.** Allow incoming traffic from the ALB security group on the container port. If the health check uses a different port, allow that port too. Turn off **Assign public IP** when starting a task in a private subnet.
- **RDS for MySQL.** Allow inbound TCP `3306` from the task security group. Attach this security group to the right RDS instance in [section 5.3.4]({{< relref "/5-Workshop/5.3-Infrastructure/5.3.4-Set-Up-RDS/" >}}). [AWS guide to ALB security groups](https://docs.aws.amazon.com/elasticloadbalancing/latest/application/load-balancer-update-security-groups.html).

### Configuration checks

- Check the VPC ID, the CIDR, the Availability Zone, and the route table each subnet uses. Record the two subnets in the DB subnet group and the RDS security group ID.
- During Fargate deployment, check that the task can pull the image and read the secret. Then check that the ALB target is **Healthy** and run a MySQL query from the application environment.
- The three screenshots show the VPC, subnet, and route table settings at the time we took them. We record the health check and MySQL connection results separately when we test the application.

Use the task ID to find the task's network interface, subnet, and security group. Check the path from the ALB to the task on TCP `80`, and from the task to RDS on TCP `3306`. Finally, check that the task can start with its ECR image and Secrets Manager values. These checks show whether the application can connect and reach the AWS services it needs at startup.

![IB Rules ALB RakiBookery](../../../../images/5-Workshop/5.3-Infrastructure/5.3.2-Set-Up-VPC/04-inbound-rules-alb.png)

![IB Rules ALB RakiBookery](../../../../images/5-Workshop/5.3-Infrastructure/5.3.2-Set-Up-VPC/05-inbound-rules-rds.png)

### Conclusion

Subnets and route tables show where traffic can go. Security groups control which connections are allowed along the ALB → Fargate → RDS path. We use these network settings in the next steps to deploy the application and database.



