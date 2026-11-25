---
title : "CloudFront + CDN Setup"
date : 2024-01-01
weight : 2
chapter : false
pre : " <b> 5.6.2 </b> "
---

### 5.6.2.1. Tạo CloudFront Distribution

Mở CloudFront Console → Distributions và chọn Create distribution, sau đó điền các thông số sau:

**Distribution name:** Nhập tên `raki-prod-php-edge-cdn`.

**Description – optional:** Có thể để trống hoặc nhập `rakibookery-prod PHP on ALB, static on S3`.

**Distribution type:** Giữ tùy chọn **Single website or app**.

**Domain (Route 53 manage   d domain – optional):** Để trống vì dự án dùng URL mặc định `*.cloudfront.net` do AWS cấp.

![134447.png](/fcj-workshop-nhat/images/5-Workshop/5.6-Operations/5.6.2-CloudFront-CDN-Setup/134447.png)

### 5.6.2.2. Cấu hình S3 Origin và OAC

**Origin type:** Chọn **Amazon S3**.

**S3 origin:** Chọn bucket `rakibookery-prod-static-98798946184613 ap-southeast-1.amazonaws.com`.

**Origin path – optional:** Để trống, không nhập `/src/public` vì frontend được lưu tại thư mục public của bucket.

**Allow private S3 bucket access to CloudFront:** Giữ chọn **Allow private S3 bucket access to CloudFront – Recommended**. Đây là tính năng **Origin Access Control (OAC)**, cho phép CloudFront đọc bucket private và ngăn người dùng truy cập trực tiếp vào S3.

**Origin settings:** Giữ tùy chọn **Use recommended origin settings**.

**Cache settings:** Giữ tùy chọn **Use recommended cache settings tailored to serving S3 content**.

![134701.png](/fcj-workshop-nhat/images/5-Workshop/5.6-Operations/5.6.2-CloudFront-CDN-Setup/134701.png)

### 5.6.2.3. Điều chỉnh ALB Origin

Sau khi khởi tạo, vào **Distributions**, chọn distribution vừa tạo, mở tab **Origins** và chỉnh sửa origin Elastic Load Balancing đã liên kết. Đặt *Protocol* là **HTTP only** để phù hợp với ALB/API hiện tại, tránh lỗi giao tiếp hoặc phản hồi `400 Bad Request` do không khớp giao thức.

![134711.png](/fcj-workshop-nhat/images/5-Workshop/5.6-Operations/5.6.2-CloudFront-CDN-Setup/134711.png)

![135129.png](/fcj-workshop-nhat/images/5-Workshop/5.6-Operations/5.6.2-CloudFront-CDN-Setup/135129.png)

Đợi distribution có trạng thái **Enabled** và cập nhật hoàn tất, sau đó mở URL triển khai thực tế.

