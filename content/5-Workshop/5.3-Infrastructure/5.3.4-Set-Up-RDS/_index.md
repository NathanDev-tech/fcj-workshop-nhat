---
title : "Initialize and Configure RDS"
date : 2024-01-01 
weight : 4
chapter : false
pre : " <b> 5.3.4 </b> "
---

### Objective

Work provisions `bookstoredb` on Amazon RDS for MySQL and tests ECS Fargate connectivity within the VPC.

### Prerequisites

Prepare the VPC and two private subnets in distinct AZs from [section 5.3.2]({{< relref "/5-Workshop/5.3-Infrastructure/5.3.2-Set-Up-VPC/" >}}), the `bookstoredb.sql` file, a MySQL client, and authorized administrative access to the VPC. Store application credentials as described in [section 5.3.1]({{< relref "/5-Workshop/5.3-Infrastructure/5.3.1-Set-Up-IAM/" >}}).

### Deployment configuration

1. In **Aurora and RDS → Subnet groups**, create `rakibookery-db-subnetgroup` in `RakiBookery-vpc-01` using private subnets in `ap-southeast-1a` and `ap-southeast-1b`.
2. Create `RakiBookery-rds-sg` in that VPC. Allow TCP `3306` from the approved import host or application task security group.
3. Under **Create database**, select **Full configuration**, **MySQL 8.4.11**, **Dev/Test**, and **Single-AZ**. The report records `rakibookery-db-instance`, class `db.t4g.micro`, 20 GiB of gp3 storage, encryption, and automated backups. Select the project subnet and security groups and set **Public access = No**.

![Project MySQL instance in Available status](../../../images/5-Workshop/5.3-Infrastructure/rds-instance-available.png)


4. The team used an EC2 intermediary and MySQL Workbench over an SSH tunnel to run `bookstoredb.sql`. Verify the schema and the `book`, `category`, `user`, and `promo` tables after import. Keep passwords and SSH keys out of the report.

### Results and verification

The `rakibookery-rds-connectivity-test` task used a MySQL client to query the `book` table. The documented test used a public subnet for the temporary task. RDS retained **Public access = No**. CloudWatch recorded `COUNT(*) = 20`, confirming that the task connected and read the data. PHP workflows and CRUD testing belong to **APP-001**.

![CloudWatch logs showing COUNT(*) of 20 from the MySQL test task](../../../images/5-Workshop/5.3-Infrastructure/rds-fargate-query.png)


### Conclusion

The deployment record shows a private RDS instance, an imported schema, and a successful query from a Fargate test task.