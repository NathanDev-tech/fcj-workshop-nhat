---
title: "Kết nối ứng dụng PHP với RDS"
date: 2024-01-01
weight: 5
chapter: false
pre: " <b> 5.4.5. </b> "
---

## Từ truy vấn thử đến thao tác trên website

Báo cáo RDS của nhóm có ảnh CloudWatch ghi kết quả `COUNT(*) = 20` từ task kiểm thử MySQL. Kết quả này cho thấy task kiểm thử đã kết nối được đến RDS và đọc được dữ liệu. Bước tiếp theo là xác nhận chính container PHP dùng được kết nối đó khi người dùng thao tác trên website.

![Log truy vấn MySQL từ task Fargate kiểm thử](../../../../images/5-Workshop/5.4-Deployment/5.4.5-Connect-PHP-App-To-RDS/02-rds-fargate-query.png)

## Cấu hình kết nối PHP

1. Đối chiếu RDS MySQL có **Public access = No** và chỉ mở TCP `3306` từ security group của workload được cấp quyền. Dùng DNS endpoint của RDS cho `DB_HOST` và `bookstoredb` cho `DB_NAME`.
2. Ánh xạ thông tin đăng nhập trong Secrets Manager vào `DB_USER` và `DB_PASS`. Kiểm tra tên các khóa trong secret và tên biến PHP đọc, sau đó triển khai revision mới cho ECS Service.
3. Mở danh mục sách để kiểm tra thao tác đọc. Tiếp theo, dùng dữ liệu thử để thực hiện một thao tác ghi, tải lại trang rồi đối chiếu kết quả. Ghi thời điểm yêu cầu và xem log PHP nếu giao diện chỉ trả thông báo lỗi chung.

## Đánh giá kết quả

Khi gặp lỗi kết nối, nhóm em kiểm tra DNS và đường mạng nếu kết nối bị timeout, còn lỗi xác thực thì đối chiếu tài khoản và secret. Minh chứng ở cấp ứng dụng cần gắn với thao tác cụ thể trên trình duyệt và log tương ứng. Ảnh truy vấn thử ở trên là bằng chứng của bài kiểm tra mạng và MySQL.

[AWS ECS và Secrets Manager](https://docs.aws.amazon.com/AmazonECS/latest/developerguide/secrets-envvar-secrets-manager.html).
