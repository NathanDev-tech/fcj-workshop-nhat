---
title: "Bản đề xuất"
date: 2026-09-09
weight: 2
chapter: false
pre: "<b>2. </b>"
---

# RakiBookery
## Tự động hóa quy trình CI/CD trên nền tảng AWS
### 1. Tổng quan dự án

RakiBookery là nền tảng thương mại điện tử chuyên cung cấp dịch vụ quản lý và bán sách trực tuyến. Hệ thống tích hợp các chức năng cốt lõi bao gồm: danh mục sản phẩm, giỏ hàng, quản lý đơn hàng (Order), chương trình khuyến mãi (Promo), xử lý tài nguyên ảnh tải lên, gợi ý sản phẩm thông minh và gửi thông báo đơn hàng. Với đặc điểm xử lý giao dịch mua bán theo thời gian thực và lưu trữ tài nguyên đa phương tiện, hệ thống cần được triển khai trên một hạ tầng có khả năng mở rộng linh hoạt, bảo mật cao và dễ dàng vận hành.

Đề xuất này trình bày giải pháp triển khai hệ thống RakiBookery trên nền tảng Amazon Web Services (AWS) tại Region Singapore (`ap-southeast-1`) theo kiến trúc Cloud-Native, đáp ứng các yêu cầu về tính sẵn sàng cao, bảo mật nhiều lớp và tự động hóa vận hành. Mục tiêu của giải pháp là xây dựng một hạ tầng có khả năng tái sử dụng, chuẩn hóa quy trình theo định hướng DevOps trong môi trường Production.

- **Thời gian thực hiện dự án:** 4 Sprints / 4 tuần
- **Quy mô nhân sự:** 5 thành viên (Product Owner: Văn An | Scrum Master: Trương Gia Hào | AWS Admin: Minh Nhật | DevOps & Cloud Engineers: Kiệt, Vũ Bảo)

---

### 2. Phát biểu vấn đề

#### Hiện trạng
Trước khi triển khai đề xuất, dự án RakiBookery mới chỉ tồn tại ở dạng mã nguồn ứng dụng (PHP Web Application & MySQL Database) hoạt động cục bộ, gặp phải một số hạn chế:

- **Bảo mật chưa được thắt chặt:** Chưa áp dụng quy định bắt buộc xác thực đa yếu tố (Force MFA Policy) cho toàn bộ IAM users, chưa áp dụng nguyên tắc phân quyền tối thiểu (Least Privilege) và thông tin kết nối Database còn lưu trữ dưới dạng plain-text.
- **Hạ tầng đơn lẻ, thiếu phân tách mạng:** Ứng dụng và Database chạy chung hoặc chưa được phân tách rõ ràng vào các Private Subnets trong VPC.
- **Quy trình xử lý thủ công:** Việc tải và xử lý ảnh sản phẩm chưa được tự động hóa; hệ thống thiếu các cơ chế thông báo tự động và AI hỗ trợ gợi ý sản phẩm cho người dùng.

#### Mục tiêu
- Chuẩn hóa việc đóng gói ứng dụng PHP bằng **Docker Image** và quản lý tập trung trên **Amazon ECR**.
- Triển khai ứng dụng dạng Serverless Container với **Amazon ECS Fargate** kết hợp **Application Load Balancer (ALB)**.
- Xây dựng hạ tầng mạng riêng **Amazon VPC** phân tách Public/Private Subnets tại region Singapore (`ap-southeast-1`).
- Bắt buộc kích hoạt **Force MFA Policy**, quản lý IAM Least Privilege và lưu trữ thông tin nhạy cảm bí mật trên **AWS Secrets Manager**.
- Tự động hóa xử lý sự kiện tải ảnh bằng **AWS Lambda**, gửi cảnh báo/thông báo đơn hàng qua **Amazon SNS** và tích hợp AI gợi ý sách bằng **Amazon Bedrock**.
- Tăng cường bảo mật ứng dụng Web PHP bằng **AWS WAF** tại tầng CloudFront.

#### Giải pháp
- Thiết kế và khởi tạo hạ tầng VPC Multi-Subnet và RDS MySQL trong Private Subnet.
- Đóng gói ứng dụng PHP với Dockerfile và đẩy lên Amazon ECR.
- Định tuyến lưu lượng người dùng qua CloudFront CDN và Application Load Balancer tới ECS Fargate.
- Quản lý secrets tập trung bằng AWS Secrets Manager và bảo vệ thông tin xác thực.
- Thiết lập quy trình Serverless Event-driven với S3 Event Trigger, Lambda và SNS.
- Giám sát toàn bộ hệ thống bằng CloudWatch Dashboards, Alarms và Log Groups.

#### Lợi tức đầu tư
- **Tối ưu hóa chi phí (Cost Efficiency):** Sử dụng mô hình Serverless (ECS Fargate, Lambda) giúp chỉ trả chi phí theo tài nguyên thực tế tiêu thụ.
- **Tăng cường bảo mật (Security First):** Đảm bảo an toàn dữ liệu với Force MFA, Secrets Manager, Private Subnet cách ly Database và AWS WAF chống tấn công Web PHP.
- **Tự động hóa vận hành (Operational Excellence):** Giảm thiểu thao tác thủ công trong quản lý hạ tầng và xử lý dữ liệu truyền thông (media).

---

### 3. Sơ đồ kiến trúc

#### Kiến trúc tổng thể

Cần bổ sung

#### Kiến trúc kết nối dịch vụ

Cần bổ sung

#### Các thành phần kiến trúc

| Trụ cột Well-Architected | Dịch vụ AWS | Vai trò & Chức năng trong Hệ thống |
| --- | --- | --- |
| **Operational Excellence** | **Amazon ECR, Docker** | Viết Dockerfile đóng gói ứng dụng PHP, lưu trữ và quản lý phiên bản Container Images trên ECR. |
| **Security** | **AWS IAM, AWS Secrets Manager, AWS WAF** | Bắt buộc thiết lập Force MFA Policy, phân quyền IAM Least Privilege, bảo mật DB credentials bằng Secrets Manager, gắn AWS WAF ngăn chặn tấn công Web PHP. |
| **Reliability** | **Amazon VPC, Amazon RDS MySQL** | Thiết lập VPC tại Singapore (`ap-southeast-1`) với Public/Private Subnets, Internet Gateway; RDS MySQL chạy trong Private Subnet bật tính năng tự động Backup & Encryption. |
| **Performance Efficiency** | **Amazon ECS Fargate, ALB, CloudFront** | Vận hành ứng dụng dạng Serverless Container trên ECS Fargate trong Private Subnet, điều phối lưu lượng qua ALB và tăng tốc phân phối dữ liệu tĩnh/media qua CloudFront CDN. |
| **Cost Optimization** | **Amazon S3, AWS Fargate** | Phân loại lưu trữ dung lượng qua 3 S3 Buckets chuyên biệt (Static Web, Media bật Encryption & Versioning, Logs); Fargate tính phí theo tài nguyên thực dùng. |
| **Sustainability & Integration** | **AWS Lambda, Amazon SNS, Amazon Bedrock** | Lambda tự động xử lý ảnh upload lên S3 & gửi tin nhắn; Amazon SNS gửi thông báo đơn hàng/lỗi; Amazon Bedrock thử nghiệm gợi ý sách thông minh. |

---

### 4. Timeline & Milestones

| Sprint / Giai đoạn | Hạng mục công việc chính | 
| :--- | :--- | 
| **SPRINT 1: Foundation & Infrastructure** | - **Setup VPC (`AWS-001`):** Tạo VPC, Internet Gateway, Public/Private Subnets, Route Tables, Security Groups.<br>- **RDS MySQL Provisioning (`AWS-002`):** Khởi tạo RDS MySQL trong Private Subnet, bật Backup/Encryption, import `bookstoredb.sql`.<br>- **S3 Buckets (`AWS-003`):** Tạo 3 S3 Buckets (Static Web, Media, Logs), cấu hình Block Public Access & Encryption.<br>- **IAM & Secrets Manager (`IAM-001`):** Tạo IAM Users/Groups, cấu hình Force MFA, lưu DB credentials vào Secrets Manager.<br>- **ECR + Dockerfile (`AWS-004`):** Viết Dockerfile ứng dụng PHP, tạo ECR Repository, build & push image.<br>- **Order + Promo Fullstack (`BE-001`):** Cập nhật CSDL Order và hoàn thiện tính năng Khuyến mãi. |
| **SPRINT 2: Compute & Core Services** | - **Fargate & Task Definition (`AWS-005`):** Khởi tạo ECS Cluster (Fargate), tạo Task Definition kết nối env/secrets từ Secrets Manager.<br>- **ALB + Target Group (`AWS-006`):** Tạo Application Load Balancer, Target Group và Health Check.<br>- **CloudFront & CDN (`AWS-007`):** Phân phối static/media content từ S3 & ALB qua CloudFront Distribution.<br>- **Kết nối PHP app với RDS (`APP-001`):** Cấu hình connection string bí mật và kiểm thử các chức năng CRUD cơ bản. |
| **SPRINT 3: Serverless, Monitoring & Hoàn thiện** | - **SNS Notification (`AWS-008`):** Tạo SNS Topic gửi thông báo đơn hàng/lỗi qua Email.<br>- **Lambda Functions (`AWS-009`):** Viết Lambda xử lý ảnh upload lên S3 & gửi thông báo tự động.<br>- **CloudWatch Monitoring (`OPS-001`):** Tạo CloudWatch Dashboard, cấu hình Alarms (CPU, 5xx, RDS) và Log Groups.<br>- **Amazon Bedrock & AWS WAF (`BEDROCK-001`):**
| **SPRINT 4: Testing & Documents** | - **E2E Testing & Runbook (`QA-001` / `DOCS-001`):** Kiểm thử toàn bộ flow mua hàng, upload ảnh và đóng gói tài liệu vận hành. |

---

### 5. Ngân sách dự kiến

Hệ thống tối ưu hóa chi phí dựa trên mô hình **AWS Pay-As-You-Go** và **Serverless**, tận dụng tối đa các hạn mức dịch vụ tại Region Singapore (`ap-southeast-1`).

| Dịch vụ AWS | Cấu hình & Vai trò theo kiến trúc |
| --- | --- |
| **Amazon ECS Fargate** | Chạy backend PHP container theo nhu cầu tài nguyên thực tế của Task Definition, tối ưu chi phí idle. |
| **Amazon RDS MySQL** | Khởi tạo RDS instance trong Private Subnet với dung lượng lưu trữ phù hợp cho `bookstoredb.sql`. |
| **Amazon S3 & CloudFront** | Lưu trữ tệp tĩnh, media và log với 3 buckets chuyên biệt; CloudFront CDN giảm tải băng thông truy cập gốc. |
| **AWS Secrets Manager & ECR** | Lưu trữ an toàn DB credentials (tính phí theo secret/tháng) và lưu trữ Docker Images trên ECR repository. |
| **AWS Lambda & Amazon SNS** | Tính phí theo số lượng yêu cầu (requests) và thời gian thực thi (execution time) khi trigger xử lý ảnh/thông báo. |
| **Amazon CloudWatch** | Cấu hình thu thập log cho Fargate/Lambda và cảnh báo theo ngưỡng chi phí/hiệu năng. |

---

### 6. Đánh giá rủi ro

#### Ma trận rủi ro & Kế hoạch ứng phó

| Rủi ro nhận diện | Khả năng | Ảnh hưởng | Biện pháp giảm thiểu & Kế hoạch ứng phó |
| --- | --- | --- | --- |
| **Lộ thông tin nhạy cảm / DB Credentials** | Thấp | Rất cao | Lưu trữ toàn bộ credentials trên Secrets Manager; bắt buộc áp dụng **Force MFA Policy** cho toàn bộ IAM Accounts. |
| **Sự cố kết nối Database từ Fargate** | Trung bình | Cao | Đặt RDS MySQL trong Private Subnet; cấu hình Security Group chỉ cho phép truy cập nội bộ từ ECS Fargate Tasks. |
| **Tấn công lỗ hổng bảo mật Web PHP** | Trung bình | Cao | Tích hợp **AWS WAF** tại tầng CloudFront để ngăn chặn các kiểu tấn công phổ biến vào ứng dụng PHP. |
| **Lỗi phát sinh khi xử lý upload ảnh** | Trung bình | Trung bình | Tách rời logic xử lý ảnh sang các hàm **AWS Lambda** chạy độc lập theo S3 Event Trigger. |

---

### 7. Kết quả mong đợi

Sau khi hoàn thành triển khai dự án RakiBookery, hệ thống dự kiến đạt được các kết quả sau:

- **Hạ tầng chuẩn hóa Cloud-Native:** Toàn bộ hệ thống Web E-commerce RakiBookery vận hành ổn định, bảo mật trên AWS ECS Fargate và RDS MySQL tại region Singapore.
- **Tự động hóa & Tích hợp AI:** Tự động hóa quy trình xử lý tài nguyên media bằng Serverless Lambda, phát cảnh báo đơn hàng qua SNS và thử nghiệm tính năng AI gợi ý sản phẩm dựa trên Amazon Bedrock.
- **Bảo mật và Vận hành chuẩn mực:** Tuân thủ quy định Force MFA, kiểm soát IAM Least Privilege, mã hóa tài nguyên S3/Secrets và đóng gói bộ tài liệu hướng dẫn vận hành (Runbook) đầy đủ cho nhóm.

---

### 8. Tài liệu tham khảo

[1]: [AWS Well-Architected Framework](https://aws.amazon.com/vi/architecture/well-architected/)

[2]: [Amazon ECS Documentation](https://docs.aws.amazon.com/ecs/)

[3]: [AWS Secrets Manager Developer Guide](https://docs.aws.amazon.com/secretsmanager/)