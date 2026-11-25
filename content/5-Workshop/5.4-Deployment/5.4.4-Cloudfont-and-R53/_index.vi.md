---
title: "Thiết lập CloudFront và Route 53"
date: 2024-01-01
weight: 4
chapter: false
pre: " <b> 5.4.4. </b> "
---

## Mục tiêu

Nhóm em cấu hình CloudFront cho hai loại nội dung của RakiBookery. Ảnh, CSS và JavaScript có thể lấy từ S3 và lưu cache. Những yêu cầu như đăng nhập, giỏ hàng và đặt hàng cần được chuyển về website PHP trên Fargate qua ALB. Route 53 là phần gắn tên miền riêng khi nhóm có tên miền để quản lý.

## Cấu hình theo tài liệu của nhóm

**1. Origin S3 cho tệp tĩnh.** Distribution `RakiBookery-cloudfront` dùng bucket `rakibookery-static-web-246111` qua S3 REST origin. Nhóm giữ **Block Public Access** và dùng **Origin Access Control (OAC)**. Bucket policy cấp `s3:GetObject` và giới hạn theo ARN của distribution. [Tài liệu AWS về OAC](https://docs.aws.amazon.com/AmazonCloudFront/latest/DeveloperGuide/private-content-restricting-access-to-s3.html).

![Bucket rakibookery-static-web-246111 trong danh sách S3](../../../../images/5-Workshop/5.4-Deployment/5.4.4-Cloudfont-and-R53/01-cloudfront-s3-bucket.png)

![Cấu hình S3 origin và tùy chọn cấp CloudFront quyền truy cập bucket private](../../../../images/5-Workshop/5.4-Deployment/5.4.4-Cloudfont-and-R53/02-cloudfront-private-s3-origin.png)

![Origin S3 đã có Origin Access Control trong tab Origins](../../../../images/5-Workshop/5.4-Deployment/5.4.4-Cloudfont-and-R53/03-cloudfront-s3-oac.png)

**2. Origin ALB cho website PHP.** Origin `RakiBookery-ALB-Origin` trỏ về DNS của `RakiBookery-alb` ở Singapore. Tài liệu workshop chọn **HTTP only**, cổng `80`, vì listener hiện được ghi nhận là `HTTP:80`. Chính sách **Redirect HTTP to HTTPS** áp dụng ở chặng từ trình duyệt tới CloudFront. Hai chặng có cấu hình giao thức riêng.

![Origin ALB dùng HTTP only trên cổng 80](../../../../images/5-Workshop/5.4-Deployment/5.4.4-Cloudfont-and-R53/04-cloudfront-alb-origin-http.png)

![Distribution RakiBookery-cloudfront có hai origin S3 và ALB](../../../../images/5-Workshop/5.4-Deployment/5.4.4-Cloudfont-and-R53/05-cloudfront-two-origins.png)

**3. Behavior theo URL của ứng dụng.** Behavior mặc định `*` chuyển về ALB với `CachingDisabled`. Nhóm kiểm tra các phương thức HTTP, cookie, query string và header mà PHP cần để xử lý phiên. Với tệp tĩnh, tài liệu dùng `/assets/*` trỏ về S3 với `CachingOptimized`. Mẫu này cần khớp URL mà website gọi và key trong bucket, ví dụ `assets/logo.png`.

![Chính sách CachingDisabled cho luồng PHP về ALB](../../../../images/5-Workshop/5.4-Deployment/5.4.4-Cloudfont-and-R53/06-cloudfront-dynamic-no-cache.png)

![Tạo behavior /assets/* trỏ về S3](../../../../images/5-Workshop/5.4-Deployment/5.4.4-Cloudfont-and-R53/07-cloudfront-static-assets.png)

![Hai behavior sau khi lưu cấu hình, với chính sách cache tương ứng](../../../../images/5-Workshop/5.4-Deployment/5.4.4-Cloudfont-and-R53/08-cloudfront-two-behaviors.png)

**4. Tên miền riêng qua Route 53.** Nếu nhóm có tên miền riêng, dùng public hosted zone, thêm tên miền vào distribution, gắn chứng chỉ ACM tại `us-east-1` và tạo Alias A trỏ về CloudFront. Nếu distribution bật IPv6, tạo thêm Alias AAAA. Domain mặc định `cloudfront.net` dùng được để kiểm tra phần distribution. [Tài liệu AWS về Route 53 và CloudFront](https://docs.aws.amazon.com/Route53/latest/DeveloperGuide/routing-to-cloudfront-distribution.html) · [Yêu cầu chứng chỉ CloudFront](https://docs.aws.amazon.com/AmazonCloudFront/latest/DeveloperGuide/cnames-and-https-requirements.html).

## Kiểm tra và ghi nhận kết quả

Sau khi distribution triển khai xong, nhóm em kiểm tra một URL tệp tĩnh có object thật trong S3. Nếu trả `403`, đối chiếu OAC, bucket policy và object key. Tiếp theo, sau khi hoàn thành bài kiểm tra ở mục 5.4.3, nhóm thử trang PHP qua domain CloudFront và thực hiện một thao tác có phiên, chẳng hạn đăng nhập hoặc xem giỏ hàng.

Tài liệu triển khai CloudFront ghi cấu hình hai origin và chính sách cache, đồng thời mô tả cách thử tệp tĩnh. Phần báo cáo ghi riêng kết quả truy cập tệp tĩnh, website động và tên miền riêng. Mỗi kết quả cần ảnh hoặc phản hồi HTTP tương ứng. Các ảnh trên xác nhận cấu hình origin và behavior. Tài liệu nhóm chưa có ảnh bản ghi Route 53 hoặc ảnh trình duyệt xác nhận truy cập thành công. [Ghi chú cấu hình CloudFront của nhóm](https://app.notion.com/p/CloudFront-Route-53-3f94cb9c2604806e8cb2ea5279aa1114?source=copy_link).
