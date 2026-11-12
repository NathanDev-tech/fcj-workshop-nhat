---
title : "CloudFront + CDN Setup"
date : 2024-01-01
weight : 2
chapter : false
pre : " <b> 5.6.2 </b> "
---

### 5.5.2.1. Tạo CloudFront Distribution

Mở CloudFront Console → Distributions và chọn Create distribution, sau đó điền các thông số sau:

**Distribution name:** Nhập tên `raki-frontend-cdn`.

**Description – optional:** Có thể để trống hoặc nhập `CloudFront CDN for RakiBookery Frontend and API`.

**Distribution type:** Giữ tùy chọn **Single website or app**.

**Domain (Route 53 managed domain – optional):** Để trống vì dự án dùng URL mặc định `*.cloudfront.net` do AWS cấp.

![image011.png](/images/5-Workshop/5.5-Raki-Operations/image011.png)

### 5.5.2.3. Cấu hình S3 Origin và OAC

**Origin type:** Chọn **Amazon S3**.

**S3 origin:** Chọn bucket `raki-frontend-dev.s3.ap-southeast-1.amazonaws.com`.

**Origin path – optional:** Để trống, không nhập `/path` vì frontend được lưu tại thư mục gốc của bucket.

**Allow private S3 bucket access to CloudFront:** Giữ chọn **Allow private S3 bucket access to CloudFront – Recommended**. Đây là tính năng **Origin Access Control (OAC)**, cho phép CloudFront đọc bucket private và ngăn người dùng truy cập trực tiếp vào S3.

**Origin settings:** Giữ tùy chọn **Use recommended origin settings**.

**Cache settings:** Giữ tùy chọn **Use recommended cache settings tailored to serving S3 content**.

![image013.png](/images/5-Workshop/5.5-Raki-Operations/image013.png)

### 5.5.2.4. Điều chỉnh ALB Origin

Sau khi khởi tạo, vào **Distributions**, chọn distribution vừa tạo, mở tab **Origins** và chỉnh sửa origin Elastic Load Balancing đã liên kết. Đặt *Protocol* là **HTTP only** để phù hợp với ALB/API hiện tại, tránh lỗi giao tiếp hoặc phản hồi `400 Bad Request` do không khớp giao thức.

![image015.png](/images/5-Workshop/5.5-Raki-Operations/image015.png)

Đợi distribution có trạng thái **Enabled** và cập nhật hoàn tất, sau đó mở URL triển khai thực tế.

![image017.png](/images/5-Workshop/5.5-Raki-Operations/image017.png)