---
title: "Week 6 Worklog"
date: 2026-09-07
weight: 6
chapter: false
pre: " <b> 1.6. </b> "
---

### Week 6 Objectives:

- Strengthen practical knowledge of EC2, storage, databases, networking, monitoring, and IAM.
- Understand how AWS services work together in a highly available architecture.
- Practice building, reviewing, monitoring, and cleaning up AWS resources.

### Tasks completed this week:
| Day | Task | Start Date | Completion Date | Reference Material |
|:---:|---|:---:|:---:|---|
| 2 | - Practiced **EC2, EBS, S3, IAM Role, CloudShell, and AWS CLI**.<br>- Launched a Linux EC2 instance, configured Security Group/Key Pair, connected by SSH, and deployed a simple web server.<br>- Practiced **User Data, EBS Volume, Snapshot, AMI, S3 Versioning, Block Public Access**, and EC2 → S3 access through IAM Role.<br>- Reviewed the simple architecture **Internet → EC2**, with EBS and IAM Role/S3, then cleaned up test resources. | 07/09/2026 | 07/09/2026 | [ExamPro CLF-C02](https://www.youtube.com/watch?v=NhDYbskXRgc)<br>[EC2 Lab](https://000004.awsstudygroup.com/vi/)<br>[S3 Lab](https://000057.awsstudygroup.com/vi/)<br>[Amazon EC2 Docs](https://docs.aws.amazon.com/ec2/)<br>[Amazon S3 Docs](https://docs.aws.amazon.com/s3/) |
| 3 | - Studied **Amazon RDS** and **EC2 Auto Scaling**.<br>- Reviewed the move from a single EC2 instance to an **Application + Database + Scaling** architecture.<br>- Practiced through the RDS and Auto Scaling labs, then completed review and cleanup. | 08/09/2026 | 08/09/2026 | [ExamPro CLF-C02](https://www.youtube.com/watch?v=NhDYbskXRgc)<br>[RDS Lab](https://000005.awsstudygroup.com/vi/)<br>[Auto Scaling Lab](https://000006.awsstudygroup.com/vi/)<br>[Amazon RDS Docs](https://docs.aws.amazon.com/rds/) |
| 4 | - Studied **Elastic Load Balancing, CloudWatch, Route 53, and AWS CLI**.<br>- Learned ALB/NLB, Target Group, Listener, Health Check, CloudWatch Metrics/Logs/Alarms/Dashboard, and Route 53 DNS/routing concepts.<br>- Reviewed the architecture **Route 53 → ALB → Target Group → EC2 → RDS**, with CloudWatch monitoring.<br>- Practiced basic AWS CLI commands and cleaned up test resources. | 09/09/2026 | 09/09/2026 | [ExamPro CLF-C02](https://www.youtube.com/watch?v=NhDYbskXRgc)<br>[CloudWatch Lab](https://000008.awsstudygroup.com/vi/)<br>[Route 53 Lab](https://000010.awsstudygroup.com/vi/)<br>[AWS CLI Lab](https://000011.awsstudygroup.com/vi/)<br>[ELB Docs](https://docs.aws.amazon.com/elasticloadbalancing/)<br>[CloudWatch Docs](https://docs.aws.amazon.com/cloudwatch/)<br>[Route 53 Docs](https://docs.aws.amazon.com/route53/) |
| 5 | - Studied **Amazon DynamoDB** and **Amazon ElastiCache**.<br>- Reviewed the roles of **NoSQL** databases and **in-memory cache** in AWS architectures.<br>- Focused on seeing AWS as a system made of multiple services, followed by quiz and cleanup. | 10/09/2026 | 10/09/2026 | [DynamoDB Lab](https://000060.awsstudygroup.com/vi/)<br>[ElastiCache Lab](https://000061.awsstudygroup.com/vi/)<br>[DynamoDB Docs](https://docs.aws.amazon.com/dynamodb/) |
| 6 | - Studied **Windows on AWS** and **Managed Microsoft AD** concepts.<br>- Completed a **Highly Available Web Application Capstone** using Route 53, CloudFront, ALB, EC2, Auto Scaling, RDS, CloudWatch, IAM Role, and S3 concepts.<br>- Reviewed **High Availability, Scalability, Fault Tolerance, Monitoring, Security, Cost Optimization**, and AWS Well-Architected thinking.<br>- Performed final review and cleanup of unnecessary resources. | 11/09/2026 | 11/09/2026 | [ExamPro CLF-C02](https://www.youtube.com/watch?v=NhDYbskXRgc)<br>[Windows on AWS](https://000093.awsstudygroup.com/)<br>[Highly Available Web Application](https://www.youtube.com/watch?v=NhDYbskXRgc) |

### Week 6 Achievements:

&emsp;◉ Monday:
<br>&emsp;&emsp;○ Practiced EC2, EBS, S3, IAM Role, CloudShell, and AWS CLI.
<br>&emsp;&emsp;○ Deployed a simple web server and tested EC2 → S3 access without hard-coded Access Keys.
<br>&emsp;&emsp;○ Practiced User Data, EBS, Snapshot, AMI, and S3 Versioning.

&emsp;◉ Tuesday:
<br>&emsp;&emsp;○ Studied Amazon RDS and EC2 Auto Scaling.
<br>&emsp;&emsp;○ Understood the transition from one EC2 instance to an application with database and scaling.

&emsp;◉ Wednesday:
<br>&emsp;&emsp;○ Learned ALB/NLB, Target Group, Listener, Health Check, Route 53, and CloudWatch.
<br>&emsp;&emsp;○ Built the flow **Route 53 → ALB → Target Group → EC2 → RDS** and reviewed monitoring with CloudWatch.
<br>&emsp;&emsp;○ Practiced basic AWS CLI commands.

&emsp;◉ Thursday:
<br>&emsp;&emsp;○ Studied DynamoDB and ElastiCache.
<br>&emsp;&emsp;○ Understood the basic roles of NoSQL and in-memory caching.

&emsp;◉ Friday:
<br>&emsp;&emsp;○ Reviewed Windows on AWS and Managed Microsoft AD.
<br>&emsp;&emsp;○ Completed the Highly Available Web Application Capstone.
<br>&emsp;&emsp;○ Connected the week’s knowledge into one architecture covering High Availability, Scaling, Monitoring, Security, and Cost Optimization.
<br>&emsp;&emsp;○ Completed final review and cleanup of AWS lab resources.