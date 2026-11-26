---
title : "Initialize and Configure RDS"
date : 2024-01-01 
weight : 4
chapter : false
pre : " <b> 5.3.4 </b> "
---

### Objective

Our goal is to move `bookstoredb` to Amazon RDS for MySQL and check the connection from ECS Fargate in the VPC.

### What we need

Prepare two private subnets in different Availability Zones in the VPC that will host RDS. Check the VPC ID as shown in [section 5.3.2]({{< relref "/5-Workshop/5.3-Infrastructure/5.3.2-Set-Up-VPC/" >}}). Prepare the `bookstoredb.sql` file, a MySQL client, and an approved way for an administrator to connect to the VPC. Store the application credentials as described in [section 5.3.1]({{< relref "/5-Workshop/5.3-Infrastructure/5.3.1-Set-Up-IAM/" >}}).

The database stores the catalog and business data outside the PHP container. The DB subnet group tells RDS where it can place the instance. Our Dev/Test setup uses Single-AZ. Subnets in two Availability Zones meet the subnet-group requirement, while Single-AZ describes how the instance itself runs. [AWS RDS in a VPC](https://docs.aws.amazon.com/AmazonRDS/latest/UserGuide/USER_VPC.WorkingWithRDSInstanceinaVPC.html).

### Deployment setup

1. In **Aurora and RDS → Subnet groups**, create `rakibookery-db-subnetgroup`. Our RDS setup shows the name `RakiBookery-vpc-01`. Check the VPC ID before selecting the two private subnets in `ap-southeast-1a` and `ap-southeast-1b`.
2. Create `RakiBookery-rds-sg` in the same VPC. Allow TCP `3306` only from the approved security group for the import host or the application task.
3. In **Create database**, choose **Full configuration**, **MySQL 8.4.11**, **Dev/Test**, and **Single-AZ**. Our report shows the instance `rakibookery-db-instance` with class `db.t4g.micro`, 20 GiB of gp3 storage, encryption, and automatic backups. Choose the right subnet group and security group. Set **Public access = No**.

![RakiBookery MySQL instance with Available status](../../../images/5-Workshop/5.3-Infrastructure/5.3.4-Set-Up-RDS/01-rds-instance-available.png)

![RDS instance class db.t4g.micro selected in the report](../../../images/5-Workshop/5.3-Infrastructure/5.3.4-Set-Up-RDS/02-rds-instance-class.png)

![RDS gp3 storage with 20 GiB configured](../../../images/5-Workshop/5.3-Infrastructure/5.3.4-Set-Up-RDS/03-rds-storage.png)

![RDS Connectivity and security showing its subnet group and Publicly accessible No](../../../images/5-Workshop/5.3-Infrastructure/5.3.4-Set-Up-RDS/04-rds-private-connectivity.png)

4. Our team used an EC2 instance and opened an SSH tunnel from MySQL Workbench to run `bookstoredb.sql`. After the import, check the schema name and the tables `book`, `category`, `user`, and `promo`. Keep passwords and SSH keys out of the report.

![MySQL Workbench reports a successful connection through the SSH tunnel](../../../images/5-Workshop/5.3-Infrastructure/5.3.4-Set-Up-RDS/05-workbench-connection-test.png)

![MySQL Workbench Action Output after executing the database import script](../../../images/5-Workshop/5.3-Infrastructure/5.3.4-Set-Up-RDS/06-workbench-import-output.png)

After the instance becomes **Available**, copy its endpoint and port from **Connectivity & security**. From the approved import host, open the MySQL client, select `bookstoredb`, import the SQL, and check that the expected tables exist. Use a dedicated MySQL user for the PHP application with just the permissions it needs. Test this MySQL user separately from the user used to import data.

> **Screenshot to add.** Capture the DB subnet group details with the two subnet IDs and Availability Zones, plus the saved RDS inbound rules for approved clients. After import, show the schema, table list, and a test query result. Hide credentials and private data.

### Results and checks

We used a MySQL client in the `rakibookery-rds-connectivity-test` task to query the `book` table. The test task ran in a public subnet. RDS stayed set to **Public access = No**. CloudWatch recorded `COUNT(*) = 20`, which shows that the task could connect and read the data. Section 5.4.5 shows how to check reads and writes from the PHP website.

![CloudWatch Logs showing COUNT(*) = 20 from the MySQL test task](../../../images/5-Workshop/5.3-Infrastructure/5.3.4-Set-Up-RDS/07-rds-fargate-query.png)

For the PHP setup, prepare the RDS endpoint, the `bookstoredb` schema name, the approved source security group, and the secret reference. A database query from a test task checks connectivity from that workload. Section 5.4.5 then checks the same connection through the PHP application.

The ECS **Logs** screenshot shows the same task ID as the CloudWatch log stream above, with `COUNT(*) = 20`. ECS displays the test at 21:37 in UTC+07, while the CloudWatch screen displays it at 14:37 in UTC. We use the task ID and time zone to match the two screens.

![ECS Logs shows the task ID and the MySQL count from the same connectivity test](../../../images/5-Workshop/5.3-Infrastructure/5.3.4-Set-Up-RDS/08-ecs-task-query-log.png)


The Workbench screenshots show the connection test and SQL execution. For the PHP application test, use the application MySQL user and the PHP task's security group. Match the task ID and timestamp with the log. The Fargate query screenshot here comes from the separate database connectivity test.

### Conclusion

Our report shows that we set up RDS in a private network and imported the schema. The Fargate test task was able to run a query and get a result.