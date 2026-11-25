---
title : "Tạo và cấu hình RDS MySQL"
date : 2024-01-01 
weight : 4
chapter : false
pre : " <b> 5.3.4 </b> "
---

### Mục tiêu
Đưa dữ liệu `bookstoredb` lên Amazon RDS for MySQL và kiểm tra kết nối từ ECS Fargate trong VPC.

### Cần chuẩn bị

Dùng VPC và hai private subnet thuộc hai AZ từ [mục 5.3.2]({{< relref "/5-Workshop/5.3-Infrastructure/5.3.2-Set-Up-VPC/" >}}), tệp `bookstoredb.sql`, MySQL client và một đường kết nối quản trị được phép vào VPC. Lưu thông tin xác thực ứng dụng theo [mục 5.3.1]({{< relref "/5-Workshop/5.3-Infrastructure/5.3.1-Set-Up-IAM/" >}}).

### Cấu hình triển khai

1. Trong **Aurora and RDS → Subnet groups**, tạo `rakibookery-db-subnetgroup`, chọn `RakiBookery-vpc-01` và hai private subnet thuộc `ap-southeast-1a` và `ap-southeast-1b`.
2. Tạo `RakiBookery-rds-sg` cùng VPC. Chỉ mở TCP `3306` từ security group của máy nhập dữ liệu hoặc task ứng dụng đã được duyệt.
3. Trong **Create database**, chọn **Full configuration**, **MySQL 8.4.11**, **Dev/Test** và **Single-AZ**. Hồ sơ ghi nhận instance `rakibookery-db-instance`, lớp `db.t4g.micro`, ổ gp3 20 GiB, mã hóa và sao lưu tự động. Chọn đúng subnet group và security group, đặt **Public access = No**.

![Instance MySQL của dự án ở trạng thái Available](../../../../images/5-Workshop/5.3-Infrastructure/rds-instance-available.png)


4. Nhóm dùng EC2 trung gian và MySQL Workbench qua SSH tunnel để chạy `bookstoredb.sql`. Sau khi nhập, kiểm tra tên schema và các bảng `book`, `category`, `user`, `promo`. Giữ mật khẩu và khóa SSH ngoài tài liệu.

### Kết quả và kiểm tra

Task kiểm thử `rakibookery-rds-connectivity-test` dùng MySQL client để truy vấn bảng `book`. Lần thử trong tài liệu dùng public subnet cho task kiểm thử. RDS vẫn giữ **Public access = No**. CloudWatch ghi nhận `COUNT(*) = 20`, xác nhận task đã kết nối và đọc dữ liệu. Kiểm thử các luồng PHP và CRUD được ghi nhận riêng ở hạng mục **APP-001**.

![CloudWatch Logs ghi COUNT(*) bằng 20 từ task kiểm thử MySQL](../../../../images/5-Workshop/5.3-Infrastructure/rds-fargate-query.png)


### Kết luận

Hồ sơ ghi nhận RDS đã được khởi tạo trong mạng riêng, nhập schema và trả kết quả truy vấn từ task Fargate kiểm thử.