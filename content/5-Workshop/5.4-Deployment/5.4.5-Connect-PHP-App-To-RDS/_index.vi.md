---
title: "Kết nối ứng dụng PHP với RDS"
date: 2024-01-01
weight: 5
chapter: false
pre: " <b> 5.4.5. </b> "
---

## Mục tiêu

Log CloudWatch của task kiểm thử MySQL hiển thị kết quả `COUNT(*) = 20`. Kết quả này cho thấy task kiểm thử đã kết nối được đến RDS và đọc được dữ liệu. Mục tiêu ở đây là kiểm tra kết nối từ chính container PHP khi người dùng đọc hoặc thay đổi dữ liệu trên website.

![Log CloudWatch của task Fargate kiểm thử MySQL, truy vấn RDS trả về 20 bản ghi](../../../../images/5-Workshop/5.4-Deployment/5.4.5-Connect-PHP-App-To-RDS/01-rds-fargate-query.png)

## Cấu hình kết nối PHP

**1. Kiểm tra cấu hình mạng.** Xác nhận RDS for MySQL có **Publicly accessible = No**. Trong security group của RDS, kiểm tra quy tắc inbound cho phép TCP `3306` từ security group của task PHP. Dùng DNS endpoint của RDS cho `DB_HOST` và `bookstoredb` cho `DB_NAME`.

![Ảnh nguồn RDS thể hiện endpoint, cổng 3306, subnet group và Publicly accessible No](../../../../images/5-Workshop/5.4-Deployment/5.4.5-Connect-PHP-App-To-RDS/02-rds-private-network.png)

**2. Khai báo biến kết nối cơ sở dữ liệu.** Ánh xạ thông tin đăng nhập trong Secrets Manager vào `DB_USER` và `DB_PASS`. Kiểm tra tên các khóa trong secret và tên biến PHP đọc, sau đó triển khai revision mới cho ECS Service.

**3. Kiểm tra revision đã triển khai.** Chờ ECS Service triển khai ổn định. Xác nhận task đang phục vụ website dùng revision có cấu hình mới rồi mới thử từ trình duyệt.

Tab **Logs** của task kiểm thử MySQL hiển thị ID task trùng với log CloudWatch ở trên. Nhờ đó, có thể đối chiếu kết quả truy vấn với đúng task kiểm tra kết nối cơ sở dữ liệu. Các bước bên dưới kiểm tra thao tác dữ liệu từ website PHP.

![ECS Logs hiển thị ID task kiểm thử MySQL và kết quả COUNT bằng 20](../../../../images/5-Workshop/5.4-Deployment/5.4.5-Connect-PHP-App-To-RDS/03-ecs-mysql-test-log.png)

## Kiểm thử một thao tác đọc và một thao tác ghi

Dùng cùng revision của task definition trong ECS Service và tài khoản thử đã được cấp quyền cho cả hai ca. Với ca đọc, chọn một sách mẫu rồi so sánh tên và giá trên trang danh mục với dữ liệu MySQL. Với ca ghi, chọn thao tác được phép với dữ liệu thử, chẳng hạn chỉnh sửa một sách mẫu. Tải lại trang và xác nhận giá trị vừa lưu. Nếu truy cập MySQL được từ client đã duyệt, đối chiếu bản ghi theo ID của sách thử.


## Kiểm tra lỗi kết nối

Nếu kết nối hết thời gian chờ, nhóm em kiểm tra DNS và đường mạng. Nếu MySQL từ chối đăng nhập, nhóm đối chiếu tài khoản MySQL và các giá trị secret đã truyền vào container. Minh chứng ở cấp ứng dụng cần gắn với thao tác cụ thể trên trình duyệt và log tương ứng.

[AWS ECS và Secrets Manager](https://docs.aws.amazon.com/AmazonECS/latest/developerguide/secrets-envvar-secrets-manager.html).

## Kết luận

Truy vấn trả về `COUNT(*) = 20`, xác nhận task Fargate kiểm thử đã kết nối RDS và đọc được dữ liệu. Với ứng dụng PHP, kết nối phụ thuộc vào endpoint RDS, quy tắc security group và thông tin đăng nhập từ Secrets Manager. Cần đối chiếu thao tác đọc và thay đổi đã lưu sau khi tải lại trang để xác nhận kết nối cơ sở dữ liệu từ website.

