---
title: "Triển khai trên AWS"
date: 2024-01-01
weight: 4
chapter: false
pre: " <b> 5.4. </b> "
---

## Mục tiêu

Sau khi chuẩn bị VPC, Amazon RDS và Amazon ECR ở mục 5.3, nhóm em triển khai website bán sách RakiBookery lên AWS. Ở máy cá nhân, nhóm có thể mở website và kiểm tra dữ liệu ngay. Khi chạy bằng ECS Fargate, cùng một thao tác mở trang danh mục phụ thuộc vào nhiều thành phần: image phải tải được, container phải khởi động, bộ cân bằng tải phải chuyển được yêu cầu và PHP phải kết nối được MySQL.

## Kiến trúc tổng quan

- **Website PHP.** Trình duyệt gửi yêu cầu đến Application Load Balancer (ALB). ALB chuyển tiếp tới container PHP trên ECS Fargate qua Target Group. Khi CloudFront phân phối cả website, CloudFront chuyển các yêu cầu động tới ALB origin.
- **Tệp tĩnh.** CloudFront có thể lấy nội dung tĩnh từ bucket S3 qua Origin Access Control (OAC). Khi có tên miền riêng, bản ghi Alias trong Route 53 trỏ về distribution.
- **Dữ liệu.** Container PHP kết nối Amazon RDS for MySQL qua mạng nội bộ. ECR cung cấp image và Secrets Manager cung cấp thông tin kết nối khi task khởi động.

## Đầu vào từ phần hạ tầng

Trước khi cấu hình ECS, nhóm ghi lại URI và digest của image, task execution role, ARN của secret RDS, VPC ID và ID các security group. Nhóm đối chiếu các giá trị này khi cấu hình ALB và ECS Service. Đồng thời, xác nhận bucket S3 nào chứa những tệp sẽ được phân phối qua CloudFront.

## Trình tự triển khai

Nhóm em chia phần triển khai theo các mốc có thể kiểm tra riêng. Trước hết, kiểm tra ECS có tải được image và khởi động container. Tiếp theo, kiểm tra đường truy cập qua ALB. Sau đó mới đối chiếu nội dung tĩnh qua CloudFront và thử chức năng cần truy vấn MySQL. Cách chia này giúp nhóm biết nên kiểm tra log của container, trạng thái Target Group hay kết nối tới cơ sở dữ liệu khi gặp lỗi.

## Tiêu chí kiểm tra từng bước

1. **Khởi động task.** Task dùng revision mới kéo đúng image và khởi động container PHP. Đối chiếu trạng thái task và log từ chính task đó.
2. **Truy cập qua ALB.** IP của task đã được đăng ký và vượt qua Health Check. Dùng DNS của ALB để mở một trang thật của website.
3. **Truy cập qua CloudFront.** Mở một tệp tĩnh và một trang PHP. Thử thao tác cần phiên đăng nhập để xác nhận yêu cầu PHP vẫn đến ứng dụng cùng dữ liệu cần thiết.
4. **Dữ liệu ứng dụng.** Mở danh mục và lưu một thay đổi thử. Tải lại trang để xác nhận thay đổi vẫn tồn tại.

## Kết luận

Phần này trình bày cách kết nối các thành phần triển khai của RakiBookery. ECS Fargate chạy container PHP, ALB chuyển tiếp yêu cầu web, còn CloudFront phân phối tệp tĩnh và chuyển yêu cầu động về ứng dụng. PHP kết nối RDS qua mạng nội bộ. Nhóm em kiểm tra lần lượt quá trình khởi động task, trạng thái target, truy cập website và thao tác dữ liệu để đánh giá việc triển khai.

{{% children sort="Weight" /%}}

