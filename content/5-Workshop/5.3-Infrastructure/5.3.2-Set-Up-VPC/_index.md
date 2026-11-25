---
title : "Initialize and Configure VPC"
date : 2024-01-01 
weight : 2
chapter : false
pre : " <b> 5.3.2 </b> "
---


### Objective

Work establishes networking for **users → Application Load Balancer (ALB) → ECS Fargate → RDS for MySQL**. The ALB receives traffic in public subnets. The PHP container and RDS communicate in private subnets within the same VPC.

### Prerequisites

- Sign in to the project AWS account, select **Asia Pacific (Singapore)** (`ap-southeast-1`), and use the deployment identity reviewed in [section 5.3.1]({{< relref "/5-Workshop/5.3-Infrastructure/5.3.1-Set-Up-IAM/" >}}).
- Confirm the **VPC ID**, IPv4 CIDR, and two Availability Zones. The November 12, 2026 screenshots show `RakiBookery-vpc-01` with CIDR `10.11.0.0/16`. The RDS screenshot in section 5.3.4 shows a VPC with the same name and a different ID. Use the VPC ID for the environment you are configuring.
- Confirm the PHP container port, the ALB health check port, and the AWS services the task needs during startup.

### Deployment configuration

#### 1. Create or inspect the VPC and subnets

1. Open **VPC → Your VPCs**. If the project VPC exists, check the CIDR, **DNS resolution**, and **DNS hostnames** in **Details**. For a new environment, select **Create VPC → VPC only** and enter the approved CIDR.
2. Open **Subnets** and verify that each subnet belongs to the intended VPC. If provisioning new subnets, use **Create subnet** and refer to the layout recorded in the screenshots:

   - `ap-southeast-1a`: public `10.11.1.0/24`, private `10.11.3.0/24`.
   - `ap-southeast-1b`: public `10.11.2.0/24`, private `10.11.4.0/24`.

![RakiBookery VPC resource map with four subnets](../../../images/5-Workshop/5.3-Infrastructure/vpc-resource-map.png)

*Figure 5.3.2a. The resource map shows two AZs, each with one public and one private subnet.*

#### 2. Attach an Internet Gateway and configure routes

1. In **VPC → Internet gateways**, verify that `RakiBookery-igw` is attached to the intended VPC. For a new setup, create the gateway and select **Attach to a VPC**.
2. Open **Route tables → Routes** for the public route table. Add or verify the `0.0.0.0/0` default route to the Internet Gateway. The `10.11.0.0/16 → local` route supports internal traffic.

![RakiBookery public route table](../../../images/5-Workshop/5.3-Infrastructure/vpc-public-route-table.png)

*Figure 5.3.2b. The public route table sends its default route to the Internet Gateway.*

3. Under **Subnet associations**, verify both public subnets are associated with the public route table. The screenshots show that the private subnets use the main route table implicitly. At deployment, check the effective route table for each private subnet and make sure it has no direct default route to the Internet Gateway.

![Public subnet associations and private subnets using the main route table](../../../images/5-Workshop/5.3-Infrastructure/vpc-subnet-associations.png)

*Figure 5.3.2c. Route table associations and CIDRs for the four subnets at capture time.*

4. Prepare outbound connectivity before starting Fargate. With NAT, use a public NAT Gateway in a public subnet and route the task subnet to it. With VPC endpoints, configure `ecr.api` and `ecr.dkr` interface endpoints with private DNS and an S3 gateway endpoint on the task route table. If the task uses Secrets Manager or `awslogs`, verify connectivity to those services. Allow inbound HTTPS `443` from the task security group on the interface endpoint security group. Verify the chosen options during deployment. [AWS VPC private subnet guidance](https://docs.aws.amazon.com/vpc/latest/userguide/vpc-example-private-subnets-nat.html) · [AWS ECR VPC endpoints](https://docs.aws.amazon.com/AmazonECR/latest/userguide/vpc-endpoints.html).

#### 3. Restrict connections with security groups

Under **VPC → Security groups**, choose the intended VPC and configure three sets of inbound rules:

- **ALB.** Allow the active listener port, such as HTTPS `443` after configuring an HTTPS listener.
- **ECS Fargate.** Allow traffic from the ALB security group on the container port and the health check port if different. Disable **Assign public IP** when launching a task in the private subnets.
- **RDS for MySQL.** Allow TCP `3306` from the task security group. Assign the database group to the intended RDS instance in [section 5.3.4]({{< relref "/5-Workshop/5.3-Infrastructure/5.3.4-Set-Up-RDS/" >}}). [AWS ALB security group guidance](https://docs.aws.amazon.com/elasticloadbalancing/latest/application/load-balancer-update-security-groups.html).

### Verify the configuration

- Check the VPC ID, CIDR, AZ, and effective route table for each subnet. Record both subnets used by the DB subnet group and the RDS security group ID.
- During Fargate deployment, verify that the task pulls its image and reads the secret. Then check for a **Healthy** ALB target and run a MySQL query from the application environment.
- The three screenshots establish the VPC, subnet, and route table configuration at capture time. Record health check and MySQL connection results during application testing.

### Conclusion

Subnets and route tables define traffic paths. Security groups restrict connections along the ALB → Fargate → RDS path. These network settings are the inputs for application and database deployment.


