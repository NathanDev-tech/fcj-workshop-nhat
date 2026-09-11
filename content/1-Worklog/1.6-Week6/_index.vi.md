---
title: "Worklog Tuần 6"
date: 2026-09-07
weight: 6
chapter: false
pre: " <b> 1.6. </b> "
---

### Mục tiêu tuần 6:

### Các công việc đã làm trong tuần này:
| Thứ | Công việc | Ngày bắt đầu | Ngày hoàn thành | Nguồn tài liệu |
|:---:|---|:---:|:---:|---|
| 2 | - Thực hành **EC2, EBS, S3, IAM Role, CloudShell và AWS CLI**.<br>- Khởi tạo Linux EC2, cấu hình Security Group/Key Pair, SSH vào EC2 và triển khai web server đơn giản.<br>- Thực hành **User Data, EBS Volume, Snapshot, AMI, S3 Versioning, Block Public Access** và EC2 → S3 thông qua IAM Role.<br>- Ôn lại kiến trúc **Internet → EC2**, EBS và IAM Role/S3, sau đó cleanup tài nguyên test. | 07/09/2026 | 07/09/2026 | [ExamPro CLF-C02](https://www.youtube.com/watch?v=NhDYbskXRgc)<br>[EC2 Lab](https://000004.awsstudygroup.com/vi/)<br>[S3 Lab](https://000057.awsstudygroup.com/vi/)<br>[Amazon EC2 Docs](https://docs.aws.amazon.com/ec2/)<br>[Amazon S3 Docs](https://docs.aws.amazon.com/s3/) |
| 3 | - Học về **Amazon RDS** và **EC2 Auto Scaling**.<br>- Ôn lại cách chuyển từ một EC2 đơn lẻ sang kiến trúc **Application + Database + Scaling**.<br>- Thực hành theo RDS Lab và Auto Scaling Lab, sau đó review và cleanup. | 08/09/2026 | 08/09/2026 | [ExamPro CLF-C02](https://www.youtube.com/watch?v=NhDYbskXRgc)<br>[RDS Lab](https://000005.awsstudygroup.com/vi/)<br>[Auto Scaling Lab](https://000006.awsstudygroup.com/vi/)<br>[Amazon RDS Docs](https://docs.aws.amazon.com/rds/) |
| 4 | - Học **Elastic Load Balancing, CloudWatch, Route 53 và AWS CLI**.<br>- Tìm hiểu ALB/NLB, Target Group, Listener, Health Check, CloudWatch Metrics/Logs/Alarm/Dashboard và Route 53 DNS/routing.<br>- Ôn kiến trúc **Route 53 → ALB → Target Group → EC2 → RDS**, có CloudWatch giám sát.<br>- Thực hành một số lệnh AWS CLI cơ bản và cleanup tài nguyên test. | 09/09/2026 | 09/09/2026 | [ExamPro CLF-C02](https://www.youtube.com/watch?v=NhDYbskXRgc)<br>[CloudWatch Lab](https://000008.awsstudygroup.com/vi/)<br>[Route 53 Lab](https://000010.awsstudygroup.com/vi/)<br>[AWS CLI Lab](https://000011.awsstudygroup.com/vi/)<br>[ELB Docs](https://docs.aws.amazon.com/elasticloadbalancing/)<br>[CloudWatch Docs](https://docs.aws.amazon.com/cloudwatch/)<br>[Route 53 Docs](https://docs.aws.amazon.com/route53/) |
| 5 | - Học **Amazon DynamoDB** và **Amazon ElastiCache**.<br>- Tìm hiểu vai trò cơ bản của **NoSQL** và **in-memory cache** trong kiến trúc AWS.<br>- Tập trung vào cách nhìn AWS như một hệ thống gồm nhiều service kết hợp, sau đó quiz và cleanup. | 10/09/2026 | 10/09/2026 | [DynamoDB Lab](https://000060.awsstudygroup.com/vi/)<br>[ElastiCache Lab](https://000061.awsstudygroup.com/vi/)<br>[DynamoDB Docs](https://docs.aws.amazon.com/dynamodb/) |
| 6 | - Học các khái niệm về **Windows on AWS** và **Managed Microsoft AD**.<br>- Thực hiện **Highly Available Web Application Capstone** với Route 53, CloudFront, ALB, EC2, Auto Scaling, RDS, CloudWatch, IAM Role và S3.<br>- Tổng hợp kiến thức về **High Availability, Scalability, Fault Tolerance, Monitoring, Security, Cost Optimization** và tư duy AWS Well-Architected.<br>- Final review và cleanup các resource không còn cần thiết. | 11/09/2026 | 11/09/2026 | [ExamPro CLF-C02](https://www.youtube.com/watch?v=NhDYbskXRgc)<br>[AWS Cloud Journey](https://cloudjourney.awsstudygroup.com/vi/)<br>[Windows on AWS](https://000093.awsstudygroup.com/)<br>[Highly Available Web Application](https://www.youtube.com/watch?v=NhDYbskXRgc) |

### Kết quả đạt được tuần 6:

&emsp;◉ Thứ 2:
<br>&emsp;&emsp;○ Thực hành EC2, EBS, S3, IAM Role, CloudShell và AWS CLI.
<br>&emsp;&emsp;○ Triển khai web server đơn giản và kiểm tra EC2 → S3 mà không hard-code Access Key.
<br>&emsp;&emsp;○ Thực hành User Data, EBS, Snapshot, AMI và S3 Versioning.

&emsp;◉ Thứ 3:
<br>&emsp;&emsp;○ Học Amazon RDS và EC2 Auto Scaling.
<br>&emsp;&emsp;○ Hiểu cách chuyển từ một EC2 đơn lẻ sang kiến trúc có Database và Scaling.

&emsp;◉ Thứ 4:
<br>&emsp;&emsp;○ Học ALB/NLB, Target Group, Listener, Health Check, Route 53 và CloudWatch.
<br>&emsp;&emsp;○ Xây dựng flow **Route 53 → ALB → Target Group → EC2 → RDS** và ôn cách CloudWatch giám sát hệ thống.
<br>&emsp;&emsp;○ Thực hành các lệnh AWS CLI cơ bản.

&emsp;◉ Thứ 5:
<br>&emsp;&emsp;○ Học DynamoDB và ElastiCache.
<br>&emsp;&emsp;○ Hiểu vai trò cơ bản của NoSQL và in-memory cache.

&emsp;◉ Thứ 6:
<br>&emsp;&emsp;○ Ôn Windows on AWS và Managed Microsoft AD.
<br>&emsp;&emsp;○ Hoàn thành Highly Available Web Application Capstone.
<br>&emsp;&emsp;○ Tổng hợp kiến thức trong tuần thành một kiến trúc có High Availability, Scaling, Monitoring, Security và Cost Optimization.
<br>&emsp;&emsp;○ Final review và cleanup các AWS resources sau khi hoàn thành lab.
