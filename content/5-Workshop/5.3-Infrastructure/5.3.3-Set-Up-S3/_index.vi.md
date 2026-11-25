---
title : "Tạo và cấu hình S3"
date : 2024-01-01 
weight : 3
chapter : false
pre : " <b> 5.3.3 </b> "
---

### Mục tiêu

Tổ chức lưu trữ trên Amazon S3 cho ba loại dữ liệu của RakiBookery tại Singapore (`ap-southeast-1`).

### Cần chuẩn bị

Dùng tài khoản triển khai đã được duyệt ở [mục 5.3.1]({{< relref "/5-Workshop/5.3-Infrastructure/5.3.1-Set-Up-IAM/" >}}). Khi dựng lại môi trường, chọn tên bucket khả dụng trong namespace tương ứng.

### Cấu hình triển khai

1. Trong **S3 → General purpose buckets**, tạo `rakibookery-prod-static-987989461846` cho tài nguyên tĩnh, `rakibookery-prod-logs-987989461846 ` cho nhật ký và `rakibookery-prod-media-987989461846` cho ảnh bìa sách, banner và tệp media.
2. Chọn **General purpose**, giữ **ACLs disabled (Bucket owner enforced)** và **Block all public access** cho cả ba bucket.

![Tạo bucket Media ở Singapore](../../../../images/5-Workshop/5.3-Infrastructure/s3-media-creation.png)


3. Bật **Bucket Versioning** cho Media để giữ các phiên bản object khi ghi đè.

![Block all public access và Versioning được chọn cho Media](../../../../images/5-Workshop/5.3-Infrastructure/s3-media-versioning.png)

### Kết quả và kiểm tra

Tài liệu ghi nhận quy trình tạo ba bucket và cấu hình Versioning cho Media. Khi nghiệm thu, đối chiếu **Permissions** và **Properties**. Kiểm tra sau tích hợp bằng cách mở tệp tĩnh qua CloudFront, xác nhận ảnh tải lên Media và log xuất hiện trong Logs khi dịch vụ gửi log đã được cấu hình.

### Kết luận

Ba bucket đảm nhiệm ba luồng dữ liệu riêng. Cấu hình chặn truy cập công khai được duy trì, và Versioning hỗ trợ lấy lại phiên bản trước của tệp media.
