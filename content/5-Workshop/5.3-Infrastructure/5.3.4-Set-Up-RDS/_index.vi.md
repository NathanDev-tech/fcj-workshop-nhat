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

Chuẩn bị hai private subnet thuộc hai Availability Zone trong VPC sẽ đặt RDS. Đối chiếu VPC ID theo [mục 5.3.2]({{< relref "/5-Workshop/5.3-Infrastructure/5.3.2-Set-Up-VPC/" >}}). Cần có tệp `bookstoredb.sql`, MySQL client và một đường kết nối quản trị được phép vào VPC. Lưu thông tin xác thực ứng dụng theo [mục 5.3.1]({{< relref "/5-Workshop/5.3-Infrastructure/5.3.1-Set-Up-IAM/" >}}).

RDS lưu danh mục và dữ liệu nghiệp vụ bên ngoài container PHP. DB subnet group xác định các subnet mà RDS có thể dùng để đặt instance. Môi trường Dev/Test của nhóm dùng Single-AZ. Việc chọn subnet ở hai Availability Zone đáp ứng yêu cầu của DB subnet group, còn Single-AZ mô tả cách instance thực tế được triển khai. [AWS về RDS trong VPC](https://docs.aws.amazon.com/AmazonRDS/latest/UserGuide/USER_VPC.WorkingWithRDSInstanceinaVPC.html).

### Cấu hình triển khai

1. Trong **Aurora and RDS → Subnet groups**, tạo `rakibookery-db-subnetgroup`. Cấu hình RDS của nhóm ghi nhận tên `RakiBookery-vpc-01`. Đối chiếu VPC ID trước khi chọn hai private subnet thuộc `ap-southeast-1a` và `ap-southeast-1b`.
2. Tạo `RakiBookery-rds-sg` cùng VPC. Chỉ mở TCP `3306` từ security group của máy nhập dữ liệu hoặc task ứng dụng đã được duyệt.
3. Trong **Create database**, chọn **Full configuration**, **MySQL 8.4.11**, **Dev/Test** và **Single-AZ**. Hồ sơ ghi nhận instance `rakibookery-db-instance`, lớp `db.t4g.micro`, ổ gp3 20 GiB, mã hóa và sao lưu tự động. Chọn đúng subnet group và security group, đặt **Public access = No**.

![Instance MySQL của dự án ở trạng thái Available](../../../../images/5-Workshop/5.3-Infrastructure/5.3.4-Set-Up-RDS/01-rds-instance-available.png)


![Chọn instance class db.t4g.micro trong báo cáo RDS](../../../../images/5-Workshop/5.3-Infrastructure/5.3.4-Set-Up-RDS/02-rds-instance-class.png)

![Cấu hình ổ gp3 với dung lượng 20 GiB](../../../../images/5-Workshop/5.3-Infrastructure/5.3.4-Set-Up-RDS/03-rds-storage.png)

![Connectivity and security hiển thị subnet group và Publicly accessible No](../../../../images/5-Workshop/5.3-Infrastructure/5.3.4-Set-Up-RDS/04-rds-private-connectivity.png)

4. Nhóm dùng EC2 trung gian và MySQL Workbench qua SSH tunnel để chạy `bookstoredb.sql`. Sau khi nhập, kiểm tra tên schema và các bảng `book`, `category`, `user`, `promo`. Giữ mật khẩu và khóa SSH ngoài tài liệu.

![MySQL Workbench thông báo kết nối thành công qua SSH tunnel](../../../../images/5-Workshop/5.3-Infrastructure/5.3.4-Set-Up-RDS/05-workbench-connection-test.png)

![Action Output của MySQL Workbench sau khi chạy script nhập cơ sở dữ liệu](../../../../images/5-Workshop/5.3-Infrastructure/5.3.4-Set-Up-RDS/06-workbench-import-output.png)

Khi instance ở trạng thái **Available**, lấy endpoint và cổng trong **Connectivity & security**. Từ máy nhập liệu đã được duyệt, mở MySQL client, chọn `bookstoredb`, nhập tệp SQL và kiểm tra các bảng cần có. Chuẩn bị tài khoản MySQL dành cho PHP với quyền cần thiết cho ứng dụng. Kiểm tra riêng tài khoản này với tài khoản dùng để nhập dữ liệu.

> **Ảnh cần bổ sung.** Chụp chi tiết DB subnet group có ID hai subnet và Availability Zone, cùng các inbound rule đã lưu của RDS cho client được duyệt. Sau khi nhập, chụp schema, danh sách bảng và kết quả truy vấn thử. Che thông tin xác thực và dữ liệu riêng tư.

### Kết quả và kiểm tra

Task kiểm thử `rakibookery-rds-connectivity-test` dùng MySQL client để truy vấn bảng `book`. Lần thử trong tài liệu dùng public subnet cho task kiểm thử. RDS vẫn giữ **Public access = No**. CloudWatch ghi nhận `COUNT(*) = 20`, xác nhận task đã kết nối và đọc dữ liệu. Phần kết nối PHP với RDS ở mục 5.4.5 trình bày cách kiểm tra các thao tác đọc và ghi từ website.

![CloudWatch Logs ghi COUNT(*) bằng 20 từ task kiểm thử MySQL](../../../../images/5-Workshop/5.3-Infrastructure/5.3.4-Set-Up-RDS/07-rds-fargate-query.png)


Đầu vào bàn giao cho ứng dụng gồm endpoint RDS, tên schema `bookstoredb`, security group nguồn đã được duyệt và tham chiếu secret. Truy vấn MySQL từ task thử xác nhận kết nối từ workload đó. Mục 5.4.5 tiếp tục kiểm tra kết nối qua chính ứng dụng PHP.

Ảnh tab **Logs** của ECS có ID task trùng với log stream CloudWatch ở trên và cùng kết quả `COUNT(*) = 20`. ECS hiển thị lần thử lúc 21:37 theo UTC+07, còn màn hình CloudWatch hiển thị lúc 14:37 theo UTC. Nhóm dùng ID task và múi giờ để đối chiếu hai màn hình.

![Tab ECS Logs hiển thị ID task và kết quả truy vấn cùng lần kiểm thử kết nối MySQL](../../../../images/5-Workshop/5.3-Infrastructure/5.3.4-Set-Up-RDS/08-ecs-task-query-log.png)


Ảnh Workbench ghi nhận lần kiểm tra kết nối và thực thi SQL. Khi thử ứng dụng PHP, dùng tài khoản MySQL của ứng dụng và security group của task PHP. Đối chiếu ID task và thời điểm chạy với log. Ảnh truy vấn Fargate ở đây thuộc lần kiểm thử kết nối cơ sở dữ liệu riêng.

### Kết luận

Báo cáo ghi nhận RDS đã được khởi tạo trong mạng riêng, nhập schema và trả kết quả truy vấn từ task Fargate kiểm thử.