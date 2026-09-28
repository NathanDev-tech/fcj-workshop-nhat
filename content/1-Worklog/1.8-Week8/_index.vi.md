---
title: "Worklog Tuần 8"
date: 2026-09-21
weight: 8
chapter: false
pre: " <b> 1.8. </b> "
---

### Mục tiêu Tuần 8:

- Hiểu AWS Systems Manager và cách quản lý tập trung các EC2 instance.
- Tìm hiểu Patch Manager và Run Command.
- Hiểu Infrastructure as Code với AWS CloudFormation.
- Tìm hiểu CloudFormation Template, Stack, Parameters, Resources và Outputs.
- Hiểu VPC Flow Logs và vai trò của nó trong việc giám sát mạng.
- Thực hành giám sát và quản lý tài nguyên AWS, đồng thời kiểm tra và cleanup tài nguyên sau mỗi bài lab.

### Công việc đã hoàn thành trong tuần:

| Ngày | Công việc | Ngày bắt đầu | Ngày hoàn thành | Tài liệu tham khảo |
|:---:|---|:---:|:---:|---|
| 2 |  | 21/09/2026 | 21/09/2026 |  |
| 3 |  | 22/09/2026 | 22/09/2026 |  |
| 4 | AWS Systems Manager - Patch Manager và Run Command | 23/09/2026 | 23/09/2026 | [AWS Systems Manager](https://000031.awsstudygroup.com/vi/) |
| 5 | AWS CloudFormation - Infrastructure as Code | 24/09/2026 | 24/09/2026 | [AWS CloudFormation](https://000037.awsstudygroup.com/vi/) |
| 6 | VPC Flow Logs - Network Monitoring | 25/09/2026 | 25/09/2026 | [VPC Flow Logs](https://000074.awsstudygroup.com/vi/) |

### Kết quả đạt được trong Tuần 8:


&emsp;◉ Thứ Tư:

<br>&emsp;&emsp;○ Tìm hiểu AWS Systems Manager và cách dịch vụ này hỗ trợ quản lý tập trung các EC2 instance.

<br>&emsp;&emsp;○ Thực hành cấu hình IAM Role cho EC2 và đưa instance vào danh sách managed nodes của Systems Manager.

<br>&emsp;&emsp;○ Thực hành Patch Manager với tùy chọn Scan and install để quản lý patch trên các Windows EC2 instance.

<br>&emsp;&emsp;○ Thực hành Run Command với AWS-RunPowerShellScript và thực hiện command trên nhiều managed instance.

<br>&emsp;&emsp;○ Hiểu cách Systems Manager hỗ trợ giảm nhu cầu quản lý từng EC2 instance riêng lẻ.

&emsp;◉ Thứ Năm:

<br>&emsp;&emsp;○ Tìm hiểu AWS CloudFormation và phương pháp Infrastructure as Code.

<br>&emsp;&emsp;○ Sử dụng AWS CloudShell làm môi trường thực hành các nội dung CloudFormation.

<br>&emsp;&emsp;○ Tìm hiểu các thành phần chính của CloudFormation Template gồm Parameters, Resources và Outputs.

<br>&emsp;&emsp;○ Tạo CloudFormation Template gồm Security Group, IAM Role, Instance Profile và EC2.

<br>&emsp;&emsp;○ Sử dụng cfn-lint để kiểm tra CloudFormation Template trước khi triển khai.

<br>&emsp;&emsp;○ Tạo CloudFormation Stack và kiểm tra kết quả triển khai.

&emsp;◉ Thứ Sáu:

<br>&emsp;&emsp;○ Tìm hiểu VPC Flow Logs và vai trò của dịch vụ trong việc giám sát mạng.

<br>&emsp;&emsp;○ Tạo và cấu hình VPC Flow Logs cho môi trường thực hành.

<br>&emsp;&emsp;○ Cấu hình CloudWatch Logs làm nơi nhận dữ liệu VPC Flow Logs.

<br>&emsp;&emsp;○ Tìm hiểu cách Flow Logs hỗ trợ theo dõi traffic và chẩn đoán các vấn đề liên quan đến Security Group.

<br>&emsp;&emsp;○ Ôn lại mối liên hệ giữa VPC, Network Interface, VPC Flow Logs và CloudWatch Logs.

<br>&emsp;&emsp;○ Kiểm tra và cleanup các AWS resources sau khi hoàn thành workshop.