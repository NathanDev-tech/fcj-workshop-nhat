---
title : "Thiết kế và xây dựng hạ tầng trên AWS"
date : 2024-01-01 
weight : 3
chapter : false
pre : " <b> 5.3. </b> "
---
## Mục tiêu

Trong dự án này, nhóm RakiBookery xây dựng nền tảng AWS cho ứng dụng PHP. Phạm vi gồm tổ chức mạng bằng VPC, lưu dữ liệu nghiệp vụ trên Amazon RDS for MySQL, chuẩn bị bucket S3 cho tệp tĩnh, media và log, rồi đóng gói ứng dụng thành Docker image để đưa lên ECR. IAM và Secrets Manager quản lý quyền truy cập và thông tin xác thực phục vụ triển khai.

## Kiến trúc tổng quan

Sơ đồ thể hiện hạ tầng nền tảng của RakiBookery và cách các tài nguyên này kết nối với ECS Fargate, Application Load Balancer và CloudFront ở giai đoạn tích hợp. Các kết nối trong sơ đồ mô tả thiết kế của hệ thống.

{{< mermaid >}}
graph TD
    Client["Người dùng"] --> CF["Amazon CloudFront"]
    CF --> S3["Amazon S3<br/>Static Web"]
    Client --> ALB
    subgraph Amazon VPC
        ALB["Application Load Balancer<br/>Public subnet"] --> APP["PHP - ECS Fargate<br/>Private subnet"]
        APP -->|MySQL TCP 3306| DB["Amazon RDS for MySQL<br/>Private subnet"]
    end
    ECR["Amazon ECR<br/>Docker image"] --> APP
    SM["AWS Secrets Manager<br/>Thông tin xác thực MySQL"] --> APP
{{< /mermaid >}}

- **Luồng ứng dụng.** Application Load Balancer tiếp nhận yêu cầu ở lớp mạng public rồi chuyển tiếp đến container PHP chạy trên ECS Fargate. Ứng dụng kết nối với RDS for MySQL qua mạng riêng trong VPC.
- **Luồng triển khai.** Khi khởi động task, ECS lấy Docker image từ ECR và đọc thông tin xác thực MySQL từ Secrets Manager theo cấu hình của task definition.
- **Luồng lưu trữ.** S3 gồm ba bucket cho Static Web, Media và Logs. Bucket Static Web phục vụ phân phối nội dung qua CloudFront. Hai bucket còn lại lưu tệp media và nhật ký theo cấu hình của ứng dụng và dịch vụ ghi log.

## Lựa chọn thiết kế và đầu ra

- **Mạng.** Nhóm đặt ALB hướng Internet trong các public subnet. Kiến trúc đặt task ứng dụng và RDS trong private subnet. Bước triển khai cần VPC ID, ID từng subnet và security group của ALB, task và cơ sở dữ liệu.
- **Quyền và dữ liệu.** Nhóm cấp role theo nhiệm vụ của từng dịch vụ, lưu thông tin xác thực cơ sở dữ liệu trong Secrets Manager và giữ các bucket S3 ở chế độ riêng tư. Task definition cần execution role và ARN của secret. CloudFront và Lambda sử dụng bucket tương ứng ở mục 5.4 và 5.5.
- **Đóng gói ứng dụng.** Container image trên ECR giúp nhóm triển khai cùng một bản build PHP lên Fargate. Ghi lại URI và digest của image để xác định chính xác bản build.

![Resource map with the four RakiBookery subnets](../../../images/5-Workshop/5.3-Infrastructure/5.3.2-Set-Up-VPC/01-vpc-resource-map.png)


## Kết luận

Hạ tầng xác định cách ứng dụng PHP được đóng gói, kết nối cơ sở dữ liệu và quản lý quyền trên AWS. Hai đầu vào trực tiếp cho giai đoạn tích hợp Fargate là Docker image trên ECR và thông tin kết nối RDS trong Secrets Manager. Kết quả khi triển khai được đánh giá qua khả năng khởi động container, kết nối MySQL từ Fargate và phân phối nội dung tĩnh qua CloudFront.

## Các thành phần hạ tầng

{{% children sort="Weight" /%}}

