---
title : "Tạo và cấu hình S3"
date : 2024-01-01 
weight : 3
chapter : false
pre : " <b> 5.3.3 </b> "
---

### Mục tiêu

Nhóm cấu hình Amazon S3 tại Singapore (`ap-southeast-1`) để lưu tệp tĩnh, ảnh và nhật ký của RakiBookery. Nhóm dùng ba bucket riêng để đặt quyền truy cập phù hợp với từng loại dữ liệu.

## Kiến trúc tổng quan

- **Media** (`rakibookery-prod-media-987989461846`) lưu ảnh của nhà sách. Bucket bật Versioning để giữ các phiên bản trước của object khi ghi đè.
- **Static** (`rakibookery-prod-static-987989461846`) lưu tài nguyên tĩnh của website. CloudFront có thể đọc tệp qua Origin Access Control (OAC) và bucket policy. Phần cấu hình origin được trình bày ở [mục 5.4.4]({{< relref "/5-Workshop/5.4-Deployment/5.4.4-Cloudfont-and-R53/" >}}).
- **Logs** (`rakibookery-prod-logs-987989461846`) dành cho tệp nhật ký. Tệp xuất hiện khi dịch vụ gửi log đã được cấu hình đích và cấp quyền ghi.

## 1. Tạo các bucket

Dùng danh tính triển khai có quyền phù hợp ở [mục 5.3.1]({{< relref "/5-Workshop/5.3-Infrastructure/5.3.1-Set-Up-IAM/" >}}). Trong **Amazon S3 → General purpose buckets → Create bucket**, chọn Region **Asia Pacific (Singapore)**, loại **General purpose** và **Global namespace** như trong ảnh. Nhập tên từng bucket. Khi dựng lại, dùng tên còn khả dụng trong namespace đã chọn. [Tài liệu AWS về cách đặt tên bucket](https://docs.aws.amazon.com/AmazonS3/latest/userguide/bucketnamingrules.html)

Ở **Object Ownership**, giữ **ACLs disabled (Bucket owner enforced)** để quản lý quyền bằng policy. Giữ **Block all public access** để chặn truy cập công khai trực tiếp. CloudFront có thể đọc bucket Static theo bucket policy ở bước tích hợp. [Tài liệu AWS về Object Ownership](https://docs.aws.amazon.com/AmazonS3/latest/userguide/about-object-ownership.html)

### Bucket Media

Nhập `rakibookery-prod-media-987989461846`. Màn hình tạo bucket ghi nhận Region Singapore và **ACLs disabled**. Trước khi lưu cấu hình của Media, chọn thêm **Bucket Versioning → Enable** như ở bước 2.

![Tạo bucket Media ở Singapore với ACLs disabled](../../../../images/5-Workshop/5.3-Infrastructure/5.3.3-Set-Up-S3/01-s3-media-creation.png)

### Bucket Static

Nhập `rakibookery-prod-static-987989461846` và giữ **ACLs disabled**. Sau khi tạo, dùng tên bucket này khi kiểm tra S3 origin của CloudFront.

![Tên bucket Static và Object Ownership trong màn hình Create bucket](../../../../images/5-Workshop/5.3-Infrastructure/5.3.3-Set-Up-S3/02-s3-static-creation.png)

### Bucket Logs

Nhập `rakibookery-prod-logs-987989461846` và giữ **ACLs disabled** theo cấu hình trong ảnh. Bucket này là đích lưu trữ. Cấu hình gửi log được thực hiện ở dịch vụ tạo log.

![Tên bucket Logs và Object Ownership trong màn hình Create bucket](../../../../images/5-Workshop/5.3-Infrastructure/5.3.3-Set-Up-S3/03-s3-logs-creation.png)

## 2. Bật và xác nhận Versioning cho Media

Trong màn hình **Create bucket** của Media, ảnh ghi nhận **Block all public access** đang được chọn và **Bucket Versioning** đặt thành **Enable**.

![Chọn Block all public access và Enable Versioning khi tạo bucket Media](../../../../images/5-Workshop/5.3-Infrastructure/5.3.3-Set-Up-S3/04-s3-media-versioning.png)

## 3. Kiểm tra kết quả

Danh sách **General purpose buckets** hiển thị đủ ba bucket Media, Static và Logs ở `ap-southeast-1`.

![Danh sách ba bucket Media, Static và Logs ở Singapore](../../../../images/5-Workshop/5.3-Infrastructure/5.3.3-Set-Up-S3/05-s3-project-bucket-list.png)

Sau khi tạo, nhóm mở **Media → Properties → Bucket Versioning**. Console hiển thị **Enabled**, xác nhận cấu hình đã được lưu. Khi tải nội dung mới lên cùng object key, S3 tạo một phiên bản mới và giữ phiên bản trước. [Tài liệu AWS về S3 Versioning](https://docs.aws.amazon.com/AmazonS3/latest/userguide/Versioning.html)

![Trang Properties của bucket Media ghi Bucket Versioning Enabled](../../../../images/5-Workshop/5.3-Infrastructure/5.3.3-Set-Up-S3/06-s3-media-versioning-enabled.png)

- **Media.** Để kiểm tra Versioning, tải hai tệp có nội dung khác nhau lên cùng key thử, chẳng hạn `versioning-check/sample.txt`. Bật **Show versions** trong tab **Objects** và kiểm tra hai version ID. Kiểm tra trigger xử lý ảnh ở [mục 5.5.2]({{< relref "/5-Workshop/5.5-Serverless/5.5.2-Lambda-Functions/" >}}).
- **Static.** Sau khi gắn origin, mở một tệp tĩnh qua URL CloudFront để kiểm tra quyền đọc.
- **Logs.** Sau khi cấu hình dịch vụ gửi log, mở bucket Logs và kiểm tra object ở prefix đã chọn.

## Kết luận

Nhóm đã tạo ba bucket S3 của RakiBookery tại Singapore. Màn hình **Properties** xác nhận bucket Media đã bật Versioning. Các bucket được dùng cho từng nhóm dữ liệu và là đầu vào khi cấu hình CloudFront, xử lý ảnh và chuyển nhật ký.
