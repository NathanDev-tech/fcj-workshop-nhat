---
title: "Thiết lập CloudFront và Route 53"
date: 2024-01-01
weight: 4
chapter: false
pre: " <b> 5.4.4. </b> "
---

## Mục tiêu

Nhóm em cấu hình CloudFront cho hai loại nội dung của RakiBookery. Ảnh, CSS và JavaScript có thể lấy từ S3 và lưu cache. Những yêu cầu như đăng nhập, giỏ hàng và đặt hàng cần được chuyển về website PHP trên Fargate qua ALB. Route 53 là phần gắn tên miền riêng khi nhóm có tên miền để quản lý.

## Chuẩn bị hai origin

Trước hết, mở website PHP qua ALB theo mục 5.4.3. Chuẩn bị một object tĩnh có key đã biết trong bucket S3 riêng tư. Dùng S3 REST origin để Origin Access Control (OAC) có thể ký các yêu cầu gửi tới bucket. Tên bucket trong ảnh CloudFront thuộc cấu hình đã ghi nhận. Khi thực hiện lại, đối chiếu tên bucket thực tế trước khi chọn origin.

## Tạo distribution và cấu hình origin

Trong CloudFront, chọn **Create distribution**. Nhóm đặt tên distribution là `RakiBookery-cloudfront` và chọn **Single website configuration**. Sau đó chọn bucket S3 riêng tư làm origin đầu tiên.

![Ảnh nguồn thể hiện tên distribution CloudFront và Single website configuration](../../../../images/5-Workshop/5.4-Deployment/5.4.4-Cloudfont-and-R53/01-cloudfront-distribution-name.png)

**1. Origin S3 cho tệp tĩnh.** Distribution `RakiBookery-cloudfront` dùng bucket `rakibookery-static-web-246111` qua S3 REST origin. Nhóm giữ **Block Public Access** và dùng **Origin Access Control (OAC)**. Bucket policy cấp `s3:GetObject` cho principal của dịch vụ CloudFront. Điều kiện `AWS:SourceArn` giới hạn quyền này theo ARN của distribution. [Tài liệu AWS về OAC](https://docs.aws.amazon.com/AmazonCloudFront/latest/DeveloperGuide/private-content-restricting-access-to-s3.html).

![Bucket rakibookery-static-web-246111 trong danh sách S3](../../../../images/5-Workshop/5.4-Deployment/5.4.4-Cloudfont-and-R53/02-cloudfront-s3-bucket.png)

![Cấu hình S3 origin và tùy chọn cấp CloudFront quyền truy cập bucket private](../../../../images/5-Workshop/5.4-Deployment/5.4.4-Cloudfont-and-R53/03-cloudfront-private-s3-origin.png)

![Origin S3 đã có Origin Access Control trong tab Origins](../../../../images/5-Workshop/5.4-Deployment/5.4.4-Cloudfont-and-R53/04-cloudfront-s3-oac.png)

**2. Origin ALB cho website PHP.** Origin `RakiBookery-ALB-Origin` trỏ về DNS của `RakiBookery-alb` ở Singapore. Cấu hình này dùng **HTTP only**, cổng `80`, vì listener hiện được ghi nhận là `HTTP:80`. Chính sách **Redirect HTTP to HTTPS** áp dụng ở chặng từ trình duyệt tới CloudFront. Nhóm cấu hình giao thức riêng cho kết nối từ CloudFront đến ALB.

![Origin ALB dùng HTTP only trên cổng 80](../../../../images/5-Workshop/5.4-Deployment/5.4.4-Cloudfont-and-R53/05-cloudfront-alb-origin-http.png)

![Distribution RakiBookery-cloudfront có hai origin S3 và ALB](../../../../images/5-Workshop/5.4-Deployment/5.4.4-Cloudfont-and-R53/06-cloudfront-two-origins.png)

**3. Cache behavior theo URL của ứng dụng.** Behavior mặc định `*` chuyển về ALB với `CachingDisabled`. Màn hình cấu hình nguồn cho phép đủ bảy phương thức HTTP và chọn **Redirect HTTP to HTTPS** cho yêu cầu từ trình duyệt. Nhóm đối chiếu các lựa chọn này với những route PHP đang kiểm thử. Với tệp tĩnh, nhóm dùng `/assets/*` trỏ về S3 với `CachingOptimized`. Mẫu này cần khớp URL mà website gọi và key trong bucket, ví dụ `assets/logo.png`.

![Ảnh nguồn thể hiện behavior mặc định về ALB, chuyển hướng HTTPS và các phương thức HTTP](../../../../images/5-Workshop/5.4-Deployment/5.4.4-Cloudfont-and-R53/07-cloudfront-dynamic-methods.png)

![Chính sách CachingDisabled cho luồng PHP về ALB](../../../../images/5-Workshop/5.4-Deployment/5.4.4-Cloudfont-and-R53/08-cloudfront-dynamic-no-cache.png)

![Tạo behavior /assets/* trỏ về S3](../../../../images/5-Workshop/5.4-Deployment/5.4.4-Cloudfont-and-R53/09-cloudfront-static-assets.png)

![Hai behavior sau khi lưu cấu hình, với chính sách cache tương ứng](../../../../images/5-Workshop/5.4-Deployment/5.4.4-Cloudfont-and-R53/10-cloudfront-two-behaviors.png)

**4. Tên miền riêng qua Route 53.** Nếu nhóm có tên miền riêng, dùng public hosted zone, thêm tên miền vào distribution, gắn chứng chỉ ACM tại `us-east-1` và tạo Alias A trỏ về CloudFront. Nếu distribution bật IPv6, tạo thêm Alias AAAA. Domain mặc định `cloudfront.net` dùng được để kiểm tra phần distribution. [Tài liệu AWS về Route 53 và CloudFront](https://docs.aws.amazon.com/Route53/latest/DeveloperGuide/routing-to-cloudfront-distribution.html) · [Yêu cầu chứng chỉ CloudFront](https://docs.aws.amazon.com/AmazonCloudFront/latest/DeveloperGuide/cnames-and-https-requirements.html).

Với behavior động, `CachingDisabled` tắt cache. Ảnh nguồn còn chọn origin request policy là `AllViewerExceptHostHeader`. Chính sách này chuyển cookie, query string và các header từ người truy cập, ngoại trừ header `Host` của người truy cập. CloudFront đặt header Host của yêu cầu tới origin theo tên miền của origin. Nhóm kiểm tra PHP có nhận đủ dữ liệu phiên cần thiết hay không. Cho phép các phương thức HTTP của ứng dụng, gồm `POST` cho biểu mẫu. Với `/assets/*`, kiểm tra behavior trỏ về S3 và đường dẫn khớp object key. [AWS về origin request policy](https://docs.aws.amazon.com/AmazonCloudFront/latest/DeveloperGuide/controlling-origin-requests.html) · [AWS về managed origin request policy](https://docs.aws.amazon.com/AmazonCloudFront/latest/DeveloperGuide/using-managed-origin-request-policies.html).

![Bucket policy](../../../../images/5-Workshop/5.4-Deployment/5.4.4-Cloudfont-and-R53/11-s3-bucket-policy.png)

## Kiểm tra và ghi nhận kết quả

Sau khi distribution triển khai xong, nhóm em kiểm tra một URL tệp tĩnh có object thật trong S3. Nếu trả `403`, đối chiếu OAC, bucket policy và object key. Tiếp theo, sau khi hoàn thành bài kiểm tra ở mục 5.4.3, nhóm thử trang PHP qua domain CloudFront và thực hiện một thao tác có phiên, chẳng hạn đăng nhập hoặc xem giỏ hàng. Sau khi đăng nhập, kiểm tra URL chuyển hướng và xác nhận trình duyệt vẫn dùng tên miền CloudFront.

Ghi riêng kết quả truy cập tệp tĩnh, website PHP và tên miền riêng. Đối chiếu mã HTTP, nội dung trả về và tên miền sau khi đăng nhập. Các bước này giúp xác định lỗi nằm ở quyền truy cập S3, cache behavior hay ứng dụng PHP.


## Kết luận

Nhóm em đã cấu hình CloudFront với origin S3 dành cho tệp tĩnh và origin ALB dành cho website PHP. Các cache behavior phân luồng yêu cầu tĩnh và động. OAC kiểm soát quyền truy cập bucket S3 riêng tư, còn `CachingDisabled` ngăn CloudFront lưu cache phản hồi PHP. Có thể bổ sung tên miền riêng qua Route 53 khi có tên miền và chứng chỉ ACM hợp lệ.