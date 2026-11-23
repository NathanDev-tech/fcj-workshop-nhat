---
title : "Tạo và cấu hình VPC"
date : 2024-01-01 
weight : 2
chapter : false
pre : " <b> 5.3.2 </b> "
---


### Mục tiêu

Tổ chức mạng cho RakiBookery theo luồng **người dùng → Application Load Balancer (ALB) → ECS Fargate → RDS MySQL**. ALB nhận lưu lượng ở public subnet. Container PHP và RDS kết nối qua private subnet trong cùng VPC.

### Cần chuẩn bị

- Đăng nhập đúng tài khoản AWS của dự án, chọn **Asia Pacific (Singapore)** (`ap-southeast-1`) và dùng danh tính triển khai đã duyệt ở [mục 5.3.1]({{< relref "/5-Workshop/5.3-Infrastructure/5.3.1-Set-Up-IAM/" >}}).
- Xác nhận **VPC ID**, dải IPv4 và hai Availability Zone. Ghi nhận VPC `RakiBookery-vpc-01` có CIDR `10.11.0.0/16`. Ảnh RDS ở mục 5.3.4 ghi nhận một VPC cùng tên nhưng khác ID, nên khi triển khai tiếp cần lấy VPC ID từ chính môi trường đang dùng.
- Xác định cổng mà container PHP lắng nghe, cổng health check của ALB và các dịch vụ AWS mà task cần gọi khi khởi động.

### Cấu hình triển khai

#### 1. Tạo hoặc kiểm tra VPC và subnet

1. Mở **VPC → Your VPCs**. Nếu VPC dự án đã có, mở **Details** để đối chiếu CIDR và kiểm tra **DNS resolution** cùng **DNS hostnames**. Với môi trường mới, chọn **Create VPC → VPC only** và nhập CIDR đã được duyệt.
2. Mở **Subnets** và kiểm tra từng subnet thuộc đúng VPC. Nếu cần tạo mới, chọn **Create subnet** và dùng cấu hình mẫu đã ghi nhận trong ảnh Console:

   - `ap-southeast-1a`: public `10.11.1.0/24`, private `10.11.3.0/24`.
   - `ap-southeast-1b`: public `10.11.2.0/24`, private `10.11.4.0/24`.

![Resource map với bốn subnet của RakiBookery](../../../../images/5-Workshop/5.3-Infrastructure/vpc-resource-map.png)

*Hình 5.3.2a. Resource map ghi nhận hai AZ, mỗi AZ có một public subnet và một private subnet.*

#### 2. Kết nối Internet Gateway và phân tuyến

1. Trong **VPC → Internet gateways**, kiểm tra `RakiBookery-igw` đã gắn vào đúng VPC. Nếu dựng mới, tạo gateway và chọn **Attach to a VPC**.
2. Mở **Route tables → Routes** của bảng public. Tạo hoặc kiểm tra tuyến mặc định `0.0.0.0/0` trỏ tới Internet Gateway. Tuyến `10.11.0.0/16 → local` phục vụ lưu lượng nội bộ.

![Bảng định tuyến public của RakiBookery](../../../../images/5-Workshop/5.3-Infrastructure/vpc-public-route-table.png)

*Hình 5.3.2b. Route table public có tuyến mặc định qua Internet Gateway.*

3. Mở **Subnet associations** và đối chiếu hai public subnet với bảng public. Ảnh Console ghi nhận hai private subnet dùng main route table thông qua liên kết ngầm định. Khi triển khai, kiểm tra route table hiệu lực của từng private subnet và bảo đảm chúng không có tuyến mặc định trực tiếp tới Internet Gateway.

![Public subnet gắn rõ ràng với route table và private subnet dùng main route table](../../../../images/5-Workshop/5.3-Infrastructure/vpc-subnet-associations.png)

*Hình 5.3.2c. Liên kết route table và CIDR của bốn subnet tại thời điểm chụp.*

4. Chuẩn bị đường ra cho Fargate trước khi chạy task. Với NAT Gateway, dùng NAT công khai trong public subnet và thêm tuyến từ subnet của task tới NAT. Với VPC endpoint, cấu hình interface endpoint `ecr.api` và `ecr.dkr` có Private DNS cùng S3 gateway endpoint cho route table của task. Nếu task dùng Secrets Manager hoặc `awslogs`, kiểm tra đường tới các dịch vụ này. Security group của interface endpoint nhận HTTPS `443` từ security group của task. Các phương án này được đối chiếu riêng khi triển khai. [AWS về VPC với private subnet](https://docs.aws.amazon.com/vpc/latest/userguide/vpc-example-private-subnets-nat.html) · [AWS về ECR VPC endpoint](https://docs.aws.amazon.com/AmazonECR/latest/userguide/vpc-endpoints.html).

#### 3. Giới hạn kết nối bằng security group

Trong **VPC → Security groups**, chọn đúng VPC và cấu hình ba lớp quyền:

- **ALB.** Cho phép inbound ở cổng listener được dùng, ví dụ HTTPS `443` khi đã cấu hình HTTPS.
- **ECS Fargate.** Cho phép inbound từ security group của ALB tới cổng container và cổng health check nếu hai cổng khác nhau. Khi khởi chạy task trong private subnet, tắt **Assign public IP**.
- **RDS MySQL.** Cho phép inbound TCP `3306` từ security group của task. Gắn security group này với đúng DB instance ở [mục 5.3.4]({{< relref "/5-Workshop/5.3-Infrastructure/5.3.4-Set-Up-RDS/" >}}). [AWS về security group cho ALB](https://docs.aws.amazon.com/elasticloadbalancing/latest/application/load-balancer-update-security-groups.html).

### Kiểm tra cấu hình

- Đối chiếu VPC ID, CIDR, AZ và route table hiệu lực của từng subnet. Ghi lại hai subnet dùng cho DB subnet group và ID security group RDS.
- Khi triển khai Fargate, xác nhận task kéo được image và đọc secret. Sau đó kiểm tra target của ALB báo **Healthy** và chạy truy vấn MySQL từ môi trường ứng dụng.
- Ba ảnh minh chứng xác nhận cấu hình VPC, subnet và route table tại thời điểm chụp. Kết quả health check và kết nối MySQL được ghi riêng khi kiểm thử ứng dụng.

### Kết luận

Subnet và route table xác định đường đi của lưu lượng. Security group giới hạn các kết nối được phép trên đường ALB → Fargate → RDS. Kết quả của mục này là cấu hình mạng để bàn giao cho các bước triển khai ứng dụng và cơ sở dữ liệu.
