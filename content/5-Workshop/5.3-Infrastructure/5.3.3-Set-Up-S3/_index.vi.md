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

Dùng danh tính triển khai đã được duyệt ở [mục 5.3.1]({{< relref "/5-Workshop/5.3-Infrastructure/5.3.1-Set-Up-IAM/" >}}). Khi dựng lại môi trường, chọn tên bucket còn khả dụng tại thời điểm tạo.

Nhóm dùng bucket riêng vì mỗi loại tệp có mục đích và thành phần truy cập khác nhau. CloudFront đọc tệp tĩnh qua OAC. Hàm xử lý ảnh đọc ảnh tải lên và ghi ảnh thu nhỏ trong bucket Media. Bucket Logs nhận tệp từ các nguồn ghi log đã được bật. Trước khi tích hợp, nhóm đối chiếu tên bucket thực tế trong Console của dự án.

### Cấu hình triển khai

1. Trong **S3 → General purpose buckets**, tạo `rakibookery-static-web-246111` cho tài nguyên tĩnh, `rakibookery-logs-246111` cho nhật ký và `rakibookery-media-246111` cho ảnh bìa sách, banner và tệp media.
2. Chọn **General purpose**, giữ **ACLs disabled (Bucket owner enforced)** và **Block all public access** cho cả ba bucket.

![Tạo bucket Media ở Singapore](../../../../images/5-Workshop/5.3-Infrastructure/5.3.3-Set-Up-S3/01-s3-media-creation.png)

![Tên bucket Static Web, Region Singapore và ACLs disabled trong báo cáo nguồn](../../../../images/5-Workshop/5.3-Infrastructure/5.3.3-Set-Up-S3/02-s3-static-creation.png)

![Tên bucket Logs và ACLs disabled trong báo cáo nguồn](../../../../images/5-Workshop/5.3-Infrastructure/5.3.3-Set-Up-S3/03-s3-logs-creation.png)

3. Bật **Bucket Versioning** cho Media để giữ các phiên bản object khi ghi đè.

![Block all public access và Versioning được chọn cho Media](../../../../images/5-Workshop/5.3-Infrastructure/5.3.3-Set-Up-S3/04-s3-media-versioning.png)

Danh sách bucket trong phần triển khai CloudFront ghi nhận đủ ba bucket RakiBookery tại Singapore. Các tên này khớp với báo cáo S3 và bucket Media dùng cho hàm xử lý ảnh.

![Danh sách đã lưu có ba bucket Static Web, Logs và Media tại Singapore](../../../../images/5-Workshop/5.3-Infrastructure/5.3.3-Set-Up-S3/05-s3-project-bucket-list.png)


### Kết quả và kiểm tra

Báo cáo S3 có các màn hình tạo ba bucket và thao tác chọn Versioning cho Media. Khi nghiệm thu, đối chiếu **Permissions** và **Properties**. Kiểm tra sau tích hợp bằng cách mở tệp tĩnh qua CloudFront, xác nhận ảnh tải lên Media và log xuất hiện trong Logs khi dịch vụ gửi log đã được cấu hình.

Tài liệu kiểm thử Lambda còn ghi nhận các tệp đã tải vào `images/` của bucket Media. Ảnh này xác nhận bucket đã lưu object tại thời điểm chạy thử.

![Các object ảnh đã được tải vào bucket Media của RakiBookery](../../../../images/5-Workshop/5.3-Infrastructure/5.3.3-Set-Up-S3/06-s3-media-uploaded-objects.png)

Dùng danh tính đã được duyệt để tải lần lượt hai tệp thử lên cùng object key trong bucket Media. Trong **Objects**, bật **Show versions** và kiểm tra S3 hiển thị hai version ID. Với tệp tĩnh, kiểm tra object có trong bucket rồi truy cập qua CloudFront sau mục 5.4.4. Với bucket Logs, kiểm tra tệp log thật sau khi cấu hình dịch vụ gửi log.


### Kết luận

Ba bucket đảm nhiệm ba luồng dữ liệu riêng. Cấu hình chặn truy cập công khai được duy trì, và Versioning hỗ trợ lấy lại phiên bản trước của tệp media.