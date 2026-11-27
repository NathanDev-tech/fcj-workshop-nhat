---
title : "Kiểm thử, vận hành và triển khai liên tục"
date : 2024-01-01
weight : 6
chapter : false
pre : " <b> 5.6 </b> "
---

### Mục tiêu

Trong phần này, dự án RakiBookery được vận hành và kiểm tra theo một quy trình end-to-end trên hạ tầng AWS, nhằm đảm bảo hệ thống hoạt động ổn định, có khả năng tự động mở rộng, được giám sát đầy đủ và đáp ứng các luồng nghiệp vụ chính của người dùng.

Các mục tiêu chính bao gồm:

* Khởi tạo và vận hành ECS Service với cơ chế Rolling Update.
* Cấu hình Auto Scaling cho Backend Service dựa trên mức sử dụng CPU.
* Cấu hình Amazon CloudFront để phân phối Frontend và định tuyến API đến Backend thông qua Application Load Balancer.
* Thiết lập CloudWatch Dashboard, Metrics và Alarms để giám sát tình trạng hệ thống.
* Thu thập và quản lý Application Logs, ALB Access Logs và VPC Flow Logs.
* Thiết lập AWS Budgets, Cost Anomaly Detection và các cảnh báo chi phí.
* Thực hiện End-to-End Testing nhằm kiểm tra toàn bộ luồng hoạt động từ Frontend, CloudFront, ALB, ECS, RDS đến S3.
* Kiểm tra các tình huống lỗi và khả năng Responsive trên Mobile và Desktop.
* Dọn dẹp các tài nguyên AWS không còn sử dụng nhằm hạn chế phát sinh chi phí.

### Tổng quan

Quy trình vận hành hệ thống RakiBookery gồm các giai đoạn chính:

1. Khởi tạo ECS Service với Fargate và Rolling Update.
2. Cấu hình Auto Scaling cho Backend với ngưỡng CPU 70%.
3. Cấu hình CloudFront để phân phối Frontend và route API qua ALB.
4. Thiết lập CloudWatch Dashboard, Logs và Alarms để giám sát hệ thống.
5. Cấu hình VPC Flow Logs, Cost Monitoring và AWS Budget Alerts.
6. Thực hiện End-to-End Testing cho các luồng chính như đăng ký, đăng nhập, POI, audio, tour và thanh toán.
7. Kiểm tra các tình huống lỗi và khả năng Responsive trên Mobile và Desktop.
8. Dọn dẹp toàn bộ tài nguyên AWS đã tạo trong workshop.

### Kết luận triển khai

Sau khi hoàn thành quá trình cấu hình, giám sát và kiểm thử, hệ thống RakiBookery đã được vận hành theo mô hình production-like trên AWS.

Các kết quả chính đạt được:

* ECS Service hoạt động ổn định trên Fargate với cơ chế Rolling Update.
* Backend có khả năng tự động scale từ `2` đến `6` tasks dựa trên CPU utilization.
* CloudFront phân phối Frontend từ S3 và định tuyến API đến Backend thông qua ALB.
* Application Load Balancer phân phối traffic đến ECS Backend Service.
* CloudWatch Dashboard cung cấp khả năng theo dõi Metrics, Logs và trạng thái hệ thống.
* CloudWatch Alarm hỗ trợ phát hiện sớm các lỗi HTTP 5xx và các bất thường trong quá trình vận hành.
* VPC Flow Logs và Application Logs hỗ trợ troubleshooting và phân tích nguyên nhân sự cố.
* AWS Budgets và Cost Anomaly Detection hỗ trợ kiểm soát chi phí sử dụng AWS.
* End-to-End Testing xác nhận các chức năng chính như đăng ký, đăng nhập, xem POI, phát audio, tour booking và thanh toán sandbox hoạt động đúng.
* Hệ thống được kiểm tra thêm với các tình huống lỗi và khả năng Responsive trên nhiều thiết bị.
* Sau khi hoàn thành thực hành, các tài nguyên AWS không còn sử dụng có thể được dọn dẹp để tránh phát sinh chi phí.

Nếu không còn nhu cầu sử dụng môi trường, có thể thực hiện cleanup theo thứ tự phù hợp, bao gồm ECS Service, Task Definition không còn sử dụng, ALB, Target Group, CloudFront Distribution, CloudWatch Alarm/Dashboard, Log Group và ECR Repository.

### Các thành phần hạ tầng

{{% children sort="Weight" /%}}