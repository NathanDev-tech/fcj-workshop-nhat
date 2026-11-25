---
title: "Triển khai trên AWS"
date: 2024-01-01
weight: 4
chapter: false
pre: " <b> 5.4. </b> "
---

## Mục tiêu

Sau khi chuẩn bị VPC, Amazon RDS và Amazon ECR ở mục 5.3, nhóm em chuyển sang bài toán đưa website bán sách RakiBookery lên AWS. Ở máy cá nhân, nhóm có thể mở website và kiểm tra dữ liệu ngay. Khi chạy bằng ECS Fargate, cùng một thao tác mở trang danh mục phụ thuộc vào nhiều thành phần: image phải tải được, container phải khởi động, bộ cân bằng tải phải chuyển được yêu cầu và PHP phải kết nối được MySQL.

## Kiến trúc tổng quan

- **Website PHP.** Trình duyệt gửi yêu cầu đến Application Load Balancer (ALB). ALB chuyển tiếp tới container PHP trên ECS Fargate qua Target Group. Nếu triển khai tên miền cho cả website qua CloudFront, ALB là một origin cho các yêu cầu động.
- **Tệp tĩnh.** CloudFront có thể lấy nội dung tĩnh từ bucket S3 qua Origin Access Control (OAC). Khi có tên miền riêng, bản ghi Alias trong Route 53 trỏ về distribution.
- **Dữ liệu.** Container PHP kết nối RDS MySQL qua mạng nội bộ. ECR cung cấp image và Secrets Manager cung cấp thông tin kết nối khi task khởi động.

## Trình tự triển khai

Nhóm em chia phần triển khai theo các mốc có thể kiểm tra riêng. Trước hết, kiểm tra ECS có tải được image và khởi động container. Tiếp theo, kiểm tra đường truy cập qua ALB. Sau đó mới đối chiếu nội dung tĩnh qua CloudFront và thử chức năng cần truy vấn MySQL. Cách chia này giúp nhóm biết nên kiểm tra log của container, trạng thái Target Group hay quyền kết nối cơ sở dữ liệu khi gặp lỗi.

{{% children sort="Weight" /%}}

## Kết luận

Hai đầu vào có minh chứng ở mục 5.3 là image trên ECR và truy vấn thử thành công từ một task Fargate tới RDS. Khi kiểm tra website, nhóm em cần thêm bằng chứng ở cấp ứng dụng, chẳng hạn ảnh trình duyệt hiển thị danh mục sách cùng thời điểm với log của task PHP đang phục vụ yêu cầu.
