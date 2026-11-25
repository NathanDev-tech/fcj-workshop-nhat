---
title : "Tạo ECR Repository + Dockerfile"
date : 2024-01-01 
weight : 5
chapter : false
pre : " <b> 5.3.5 </b> "
---

### Mục tiêu

Đóng gói ứng dụng PHP của RakiBookery thành Docker image và đưa image lên private repository Amazon ECR tại Singapore (`ap-southeast-1`).

### Cần chuẩn bị

Cần mã nguồn backend, Docker chạy Linux container, AWS CLI và danh tính có quyền push lên repository. Nhóm dùng kiến trúc `linux/amd64` để build image.

### Cấu hình triển khai

1. Dùng image nền `php:8.2-apache`, cài `pdo_mysql` và `curl`, bật `mod_rewrite` và cấu hình Apache phục vụ thư mục `public` tại cổng `80`.
2. Thêm `.dockerignore` để loại `.env`, dữ liệu SQL và tệp không cần thiết khỏi build context. Build image `rakibookery:local` rồi kiểm tra với MySQL 8.0 trên Docker network riêng.
3. Tạo private repository `rakibookery` với tag mutability **Mutable**, scan on push bật và mã hóa `AES256` theo hồ sơ triển khai. Đăng nhập registry bằng token ECR tạm thời qua stdin. Gắn tag `latest` cho image, push, rồi pull lại để kiểm tra.

### Kết quả và kiểm tra

- Kiểm thử local trả HTTP `200` cho trang chủ, `/shop` và tài nguyên tĩnh. Route không tồn tại trả `404`. Báo cáo ghi nhận ứng dụng kết nối được với MySQL test.
- `aws ecr describe-images` xác nhận repository có các tag `latest` và `manual-test`. Báo cáo ghi nhận OCI index digest bắt đầu bằng `sha256:2364b729` và lần pull từ ECR thành công.

![Trích đoạn báo cáo về kết quả describe-images và xác minh digest trên ECR](../../../../images/5-Workshop/5.3-Infrastructure/ecr-image-report-p11.png)


### Kết luận

Image đã được build, chạy thử tại máy phát triển, push và pull qua ECR theo hồ sơ triển khai. URI kèm tag hoặc digest là đầu vào để khai báo image trong ECS task definition.