---
title : "Giới thiệu"
date : 2024-01-01 
weight : 1
chapter : false
pre : " <b> 5.1. </b> "
---

# Bài toán Cloud & DevOps: Đưa hệ thống lên AWS Cloud, xây dựng hạ tầng container + serverless và giám sát

Nhằm đáp ứng yêu cầu về khả năng triển khai linh hoạt, mở rộng, bảo mật và tự động hóa một phần quy trình vận hành, đề tài thực hiện triển khai ứng dụng **RakiBookery** (hệ thống quản lý/bán sách trực tuyến) trên nền tảng Amazon Web Services (AWS). Kiến trúc được tổ chức thành các luồng chính: luồng xử lý yêu cầu người dùng, luồng xử lý serverless (Lambda + Bedrock), và luồng giám sát – thông báo.

### Luồng CI/CD (Pipeline Flow)
Quy trình tích hợp và triển khai liên tục được thiết kế nhằm: tự động hóa toàn bộ quá trình từ khi lập trình viên cập nhật mã nguồn đến khi phiên bản mới của ứng dụng được triển khai trên môi trường vận hành. Cụ thể, khi Developer thực hiện thao tác push code lên kho mã nguồn GitHub, GitHub Actions sẽ được kích hoạt để khởi chạy pipeline. Pipeline sử dụng cơ chế xác thực OIDC (OpenID Connect) để liên kết với AWS Security Token Service (STS) và cấp phát thông tin xác thực tạm thời (temporary credentials), thay thế cho việc sử dụng Access Key/Secret Key tĩnh nhằm mục đích tăng cường mức độ an toàn bảo mật. Sau khi xác thực thành công, pipeline tiến hành build Docker image của ứng dụng và đẩy image lên Amazon Elastic Container Registry (ECR). Tiếp theo, dịch vụ Amazon ECS Fargate sẽ tự động pull image mới nhất từ ECR và thực hiện triển khai phiên bản mới của ứng dụng mà không cần can thiệp thủ công.

- **Luồng xử lý:** Developer → GitHub → GitHub Actions → OIDC Authentication → AWS STS → Amazon ECR → Amazon ECS Fargate.

### Luồng User (Request Flow)

Luồng xử lý yêu cầu được xây dựng theo mô hình phân phối nội dung kết hợp cân bằng tải nhằm đảm bảo hiệu năng truy cập, khả năng chịu lỗi và bảo mật. Khi người dùng gửi yêu cầu truy cập, yêu cầu trước tiên được tiếp nhận bởi Amazon CloudFront (CDN). Đối với các tài nguyên tĩnh của giao diện, CloudFront truy xuất trực tiếp từ Amazon S3 Static Bucket. Đối với các yêu cầu động, CloudFront chuyển tiếp yêu cầu đến Application Load Balancer (ALB). ALB phân phối lưu lượng đến các task ECS Fargate đang chạy. Các container PHP kết nối đến Amazon RDS MySQL (đặt trong Private Subnet) để thực hiện thao tác đọc/ghi dữ liệu. Đồng thời, media (ảnh sách, file upload) được lưu trữ và truy xuất từ Amazon S3 Media Bucket.

- **Luồng xử lý:** User → CloudFront → (S3 Static hoặc ALB) → ECS Fargate → RDS MySQL (Private Subnet) + S3 Media

### Luồng Serverless & AI (Lambda + Bedrock)

Luồng serverless được thiết kế để xử lý các tác vụ phụ trợ một cách tự động, không cần duy trì máy chủ liên tục. Khi người dùng upload ảnh lên S3 Media Bucket, sự kiện S3 sẽ tự động kích hoạt AWS Lambda để thực hiện xử lý ảnh (resize, tối ưu…). Lambda cũng có thể được gọi để gửi thông báo đơn hàng hoặc sự kiện quan trọng thông qua Amazon SNS. Ngoài ra, hệ thống tích hợp Amazon Bedrock (thông qua Lambda hoặc API Gateway) để thực hiện chức năng gợi ý sách đơn giản dựa trên Knowledge Base. API Gateway đóng vai trò expose các endpoint của Lambda và Bedrock một cách an toàn, có hỗ trợ CORS.

- **Luồng xử lý:** S3 Media Upload → Lambda (xử lý ảnh / gửi thông báo) → SNS → Email
hoặc API Gateway → Lambda → Amazon Bedrock (gợi ý sách).

### Luồng Giám sát & Thông báo (Observability)

Để đảm bảo hệ thống vận hành ổn định và phát hiện sớm sự cố, đề tài triển khai cơ chế giám sát tập trung dựa trên các dịch vụ quản trị của AWS. Toàn bộ nhật ký hệ thống (logs từ ECS Fargate, Lambda, Application) và metrics (CPU, Memory, 5xx, RDS…) được thu thập và tập trung tại Amazon CloudWatch. CloudWatch Dashboard cung cấp cái nhìn tổng quan về tình trạng hệ thống. Các Alarm quan trọng được cấu hình; khi ngưỡng bị vượt quá, CloudWatch sẽ kích hoạt Amazon SNS để gửi email cảnh báo đến nhóm quản trị, hỗ trợ phản ứng kịp thời.

- **Luồng xử lý:** ECS / Lambda / RDS → CloudWatch (Logs + Metrics + Alarm) → Amazon SNS → Email Alert.

---

## Kiến trúc hệ thống

### Kiến trúc tổng thể

![AWS Diagram](/fcj-workshop-nhat/images/5-Workshop/5.1-Workshop-overview/aws_diagram.jpg)

Kiến trúc RakiBookery được chia thành các lớp chính:

#### 1. Presentation Layer
- Amazon CloudFront
- Amazon S3 Static Website
- (Tùy chọn) Route 53

Frontend tĩnh được phục vụ qua CloudFront để giảm độ trễ và tăng tốc độ truy cập.

#### 2. Application Layer
- Application Load Balancer (ALB)
- Amazon ECS Cluster (Fargate)
- PHP Application (Docker container)

Ứng dụng PHP chạy trên ECS Fargate, giúp loại bỏ việc quản lý máy chủ EC2. ALB thực hiện cân bằng tải và health check.

#### 3. Data Layer
- Amazon RDS MySQL (Private Subnet)
- Amazon S3 (Static + Media + Logs)
- Secrets Manager (lưu credential DB)

RDS được đặt trong private subnet, có encryption và backup. Secrets Manager lưu mật khẩu database an toàn.

#### 4. Serverless & AI Layer
- AWS Lambda (xử lý ảnh, thông báo, thống kê)
- Amazon SNS
- Amazon Bedrock (demo gợi ý sách)
- API Gateway

#### 5. Networking & Security Layer
- VPC (Public / Private Subnet)
- Internet Gateway + (NAT Gateway nếu cần)
- Security Groups
- IAM (Least Privilege) + Secrets Manager

#### 6. Monitoring Layer
- Amazon CloudWatch (Dashboard, Logs, Alarms)
- Amazon SNS

---

### Kiến trúc triển khai ECS

![ECS Deploy Diagram](/fcj-workshop-nhat/images/5-Workshop/5.1-Workshop-overview/aws_diagram2.jpg)


Hệ thống sử dụng **ECS Cluster (Fargate)** với:
- **Task Definition** cấu hình image từ ECR, environment variables và secrets từ Secrets Manager.
- Container PHP app kết nối RDS và S3.
- ALB Target Group + Health Check định tuyến traffic đến các task đang chạy.

---

## Thành phần AWS sử dụng

| Dịch vụ              | Mục đích |
|----------------------|----------|
| **IAM**              | Quản lý danh tính và phân quyền theo nguyên tắc Least Privilege. Tạo User/Group/Role/Policy cho team và dịch vụ. |
| **VPC**              | Xây dựng mạng riêng ảo với Public/Private Subnet, Route Table, Security Group. |
| **Internet Gateway** | Kết nối VPC với Internet cho Public Subnet (ALB, CloudFront origin…). |
| **ECS Fargate**      | Chạy container PHP app mà không cần quản lý EC2. |
| **ECR**              | Lưu trữ Docker image của ứng dụng PHP. |
| **ALB**              | Cân bằng tải HTTP/HTTPS, Health Check, định tuyến đến ECS tasks. |
| **RDS MySQL**        | Cơ sở dữ liệu quan hệ được quản lý (bookstoredb), đặt trong Private Subnet, có encryption & backup. |
| **S3**               | Lưu trữ Static Website, Media (ảnh sách), Logs. Có encryption + versioning (Media). |
| **CloudFront**       | CDN phân phối nội dung tĩnh và dynamic, giảm độ trễ. |
| **Secrets Manager**  | Lưu trữ an toàn DB password và các credential nhạy cảm. |
| **Lambda**           | Xử lý ảnh upload S3, gửi thông báo, thống kê đơn giản. |
| **SNS**              | Gửi thông báo email khi có sự kiện đơn hàng hoặc alarm. |
| **CloudWatch**       | Thu thập Logs, Metrics, tạo Dashboard và Alarm. |
| **API Gateway**      | Expose Lambda và Bedrock endpoint. |
| **Amazon Bedrock**   | Demo chức năng gợi ý sách (Knowledge Base + Guardrails). |

---

**Ghi chú triển khai thực tế:**
- Region: **ap-southeast-1 (Singapore)**
- Team: Văn An, Minh Nhật, Kiệt, Vũ Bảo, Gia Hào
- Timeline: 4 Sprints (Foundation → Compute → Serverless/Monitoring → Hoàn thiện & Docs)
- Các task chính đã/ đang thực hiện: VPC, RDS MySQL, S3 (3 buckets), IAM + Secrets Manager, ECR + Dockerfile PHP, ECS Fargate, ALB, CloudFront, Lambda, SNS, CloudWatch, Bedrock demo, API Gateway.