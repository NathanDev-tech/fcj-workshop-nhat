---
title: "Week 6 Worklog"
date: 2026-09-07
weight: 6
chapter: false
pre: " <b> 1.6. </b> "
---

### Week 6 Objectives:

### Tasks completed this week:
| Day | Task | Start Date | Completion Date | Reference Material |
|:---:|---|:---:|:---:|---|
| 2   | - Continued studying the **AWS Certified Cloud Practitioner (CLF-C02)** course from ExamPro, focusing on **Management and Development Tools** and the **Shared Responsibility Model**.<br>- Reviewed and studied concepts related to **Amazon EC2, AMI, Amazon EBS, Snapshots, and Amazon S3**.<br>- Practiced launching a Linux EC2 instance, configuring **Security Groups and Key Pairs**, and connecting to EC2 via **SSH**.<br>- Installed a Web Server on EC2 and deployed a simple web page.<br>- Learned and practiced using **User Data** to automatically install a Web Server during EC2 initialization.<br>- Created an **EBS Volume**, attached and mounted the volume to EC2 for use as Block Storage.<br>- Practiced creating **EBS Snapshots** and learned how **AMIs** can be used to store images and create EC2 instances.<br>- Learned the basics of **Amazon S3**, created a Bucket, performed upload/download operations for Objects, and explored **Versioning** and **Block Public Access**.<br>- Created an **IAM Role** for EC2 and tested EC2 access to S3 without using hard-coded Access Keys.<br>- Became familiar with **AWS CloudShell and AWS CLI** and executed basic commands to inspect account information and AWS resources.<br>- At the end of the session, reviewed the relationship between **EC2, EBS, S3, and IAM Roles** through a simple lab architecture and checked the resources that had been created. | 07/09/2026 | 07/09/2026 | [AWS Certified Cloud Practitioner CLF-C02 - ExamPro](https://www.youtube.com/watch?v=NhDYbskXRgc)<br>[AWS Cloud Journey](https://cloudjourney.awsstudygroup.com/vi/)<br>Amazon EC2 Lab<br>Amazon S3 Lab<br>[Amazon EC2 Documentation](https://docs.aws.amazon.com/ec2/)<br>[Amazon S3 Documentation](https://docs.aws.amazon.com/s3/) |
| 3 | - Continued studying the **AWS Certified Cloud Practitioner (CLF-C02)** course from ExamPro, focusing on **Compute and Storage** concepts.<br>- Studied **Amazon RDS** and learned the fundamentals of managed relational databases on AWS.<br>- Practiced creating a test RDS database, selecting a database engine, and reviewing configurations related to **DB Subnet Groups, Security Groups, and database networking**.<br>- Learned how an application running on EC2 can connect to RDS and reviewed the **Application → EC2 → RDS** architecture.<br>- Studied **RDS Backup and Restore** concepts and their role in database recovery.<br>- Learned and compared **Multi-AZ and Read Replica**, understanding that Multi-AZ focuses on **High Availability and Failover**, while Read Replica focuses on **Read Scaling**.<br>- Studied **EC2 Auto Scaling**, including **Launch Templates, Auto Scaling Groups, Desired Capacity, Minimum Capacity, and Maximum Capacity**.<br>- Practiced configuring an Auto Scaling Group and learned how Health Checks can detect unhealthy EC2 instances and replace them when necessary.<br>- Studied **Dynamic Scaling and Scheduled Scaling** to understand how EC2 capacity can automatically adjust according to workload requirements.<br>- Reviewed the **Application → EC2 Auto Scaling → RDS** architecture and reinforced the difference between **Scaling and Failover**.<br>- Completed a review quiz and cleaned up AWS resources created during the labs to avoid unnecessary costs. | 08/09/2026 | 08/09/2026 | [AWS Certified Cloud Practitioner CLF-C02 - ExamPro](https://www.youtube.com/watch?v=NhDYbskXRgc)<br>[AWS Cloud Journey](https://cloudjourney.awsstudygroup.com/vi/)<br>[RDS Lab](https://000005.awsstudygroup.com/vi/)<br>[Auto Scaling Lab](https://000006.awsstudygroup.com/vi/)<br>[AWS RDS Documentation](https://docs.aws.amazon.com/rds/) |
| 4 |  | 12/08/2026 | 12/08/2026 |  |
| 5 |  | 13/08/2026 | 13/08/2026 |  |
| 6 |  | 14/08/2026 | 14/08/2026 |  |

### Week 6 Achievements:

&emsp;◉ Monday:
<br>&emsp;&emsp;○ Completed the Management and Development Tools and Shared Responsibility Model sections of the ExamPro CLF-C02 course.
<br>&emsp;&emsp;○ Successfully launched and worked with a Linux EC2 instance, including Security Groups, Key Pairs, and SSH.
<br>&emsp;&emsp;○ Installed a Web Server on EC2 and practiced using User Data for automated configuration.
<br>&emsp;&emsp;○ Practiced creating, attaching, and mounting an EBS Volume, as well as learning about Snapshots and AMIs.
<br>&emsp;&emsp;○ Practiced basic S3 Bucket/Object operations and explored Versioning and Block Public Access.
<br>&emsp;&emsp;○ Created and tested an IAM Role for EC2 to access S3 without hard-coding Access Keys.
<br>&emsp;&emsp;○ Became familiar with AWS CloudShell/AWS CLI and executed basic commands for checking AWS resources.

&emsp;◉ Tuesday:
<br>&emsp;&emsp;○ Continued studying the Compute and Storage sections of the ExamPro CLF-C02 course.
<br>&emsp;&emsp;○ Studied Amazon RDS and learned how a managed relational database differs from running a database directly on an EC2 instance.
<br>&emsp;&emsp;○ Practiced creating and configuring a test RDS database, including the database engine, DB Subnet Group, Security Group, and database networking.
<br>&emsp;&emsp;○ Reviewed the Application → EC2 → RDS architecture and learned how an application can connect to a managed database service.
<br>&emsp;&emsp;○ Learned about RDS backup and restore concepts for database recovery.
<br>&emsp;&emsp;○ Clearly understood the difference between Multi-AZ and Read Replica: Multi-AZ is used for High Availability and Failover, while Read Replica is used for Read Scaling.
<br>&emsp;&emsp;○ Studied EC2 Auto Scaling and learned the purpose of Launch Templates and Auto Scaling Groups.
<br>&emsp;&emsp;○ Practiced configuring Desired Capacity, Minimum Capacity, and Maximum Capacity for an Auto Scaling Group.
<br>&emsp;&emsp;○ Learned how Health Checks can detect unhealthy EC2 instances and allow Auto Scaling to replace them.
<br>&emsp;&emsp;○ Studied Dynamic Scaling and Scheduled Scaling and how they can adjust EC2 capacity according to workload requirements.
<br>&emsp;&emsp;○ Reviewed the Application → EC2 Auto Scaling → RDS architecture and reinforced that Scaling and Failover solve different problems.
<br>&emsp;&emsp;○ Completed the review quiz and cleaned up test AWS resources to avoid unnecessary costs.