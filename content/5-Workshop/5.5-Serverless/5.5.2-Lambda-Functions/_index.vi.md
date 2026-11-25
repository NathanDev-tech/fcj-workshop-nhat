---
title: "Triển khai hàm Lambda"
date: 2024-01-01
weight: 2
chapter: false
pre: " <b> 5.5.2. </b> "
aliases:
  - "/5-workshop/5.5-operations/5.5.2-lambda-functions/"
---

## Mục tiêu

Nhóm em dùng Lambda cho ba việc của RakiBookery: tạo ảnh thu nhỏ khi có ảnh sách mới, nhận thông báo từ SNS để ghi log và đọc số liệu đơn hàng từ MySQL. Danh sách Lambda của dự án ghi nhận `raki-image-processor`, `raki-sns-handler` và `raki-stats-service` dùng Python 3.12 ở Singapore.


![Danh sách các hàm RakiBookery và runtime Python 3.12](../../../../images/5-Workshop/5.5-Serverless/5.5.2-Lambda-Functions/01-lambda-inventory.png)


## Hàm xử lý ảnh `raki-image-processor`

**1. Tạo hàm và cấp quyền.** Tạo function Python 3.12 và đóng gói Pillow tương thích với runtime và kiến trúc Lambda. Execution role cần quyền đọc ảnh ở `images/`, ghi kết quả vào `thumbnails/` trong bucket Media và ghi log CloudWatch.

![Tạo raki-image-processor với Python 3.12](../../../../images/5-Workshop/5.5-Serverless/5.5.2-Lambda-Functions/02-image-function.png)


![Execution role và phần quyền ghi log của raki-image-processor](../../../../images/5-Workshop/5.5-Serverless/5.5.2-Lambda-Functions/03-image-permissions.png)

**2. Gắn trigger S3.** Trong **Configuration → Triggers → Add trigger**, chọn bucket `rakibookery-media-246111`, sự kiện `s3:ObjectCreated:Put` và prefix `images/`. Hàm dùng Pillow để thu nhỏ ảnh trong khung tối đa `300 × 300`, giữ tỷ lệ và ghi đầu ra vào `thumbnails/`. Hai prefix tách riêng để ảnh kết quả không gọi lại hàm. [AWS về S3 trigger](https://docs.aws.amazon.com/lambda/latest/dg/with-s3.html).

![Chọn bucket Media và sự kiện PUT cho trigger S3](../../../../images/5-Workshop/5.5-Serverless/5.5.2-Lambda-Functions/04-s3-put-trigger.png)


![Trigger S3 đã được gắn với raki-image-processor](../../../../images/5-Workshop/5.5-Serverless/5.5.2-Lambda-Functions/05-s3-trigger-attached.png)

**3. Thử với ảnh sách.** Nhóm em tải một ảnh JPG lên `images/` và kiểm tra object tương ứng trong `thumbnails/`. Các ảnh còn ghi nhận log stream tại `/aws/lambda/raki-image-processor`, nơi nhóm có thể đối chiếu lần gọi theo thời gian.

![Ảnh thử nghiệm được tải lên prefix images/](../../../../images/5-Workshop/5.5-Serverless/5.5.2-Lambda-Functions/06-s3-source-image.png)


![Các log stream của raki-image-processor trong CloudWatch](../../../../images/5-Workshop/5.5-Serverless/5.5.2-Lambda-Functions/07-image-log-streams.png)


![S3 hiển thị ảnh thu nhỏ trong prefix thumbnails/](../../../../images/5-Workshop/5.5-Serverless/5.5.2-Lambda-Functions/08-s3-generated-thumbnail.png)


## Hàm nhận thông báo `raki-sns-handler`

**1. Gắn topic.** Chọn **Add trigger → SNS** và chọn `raki-noti-sns` từ mục 5.5.1. Handler duyệt `Records`, đọc `Sns.Subject` và `Sns.Message` rồi ghi log. SNS gọi handler khi có thông báo, đồng thời phân phối email qua subscription đã cấu hình.

![Chọn raki-noti-sns khi thêm trigger SNS](../../../../images/5-Workshop/5.5-Serverless/5.5.2-Lambda-Functions/09-sns-trigger-setup.png)


![Trigger SNS xuất hiện trong cấu hình raki-sns-handler](../../../../images/5-Workshop/5.5-Serverless/5.5.2-Lambda-Functions/10-sns-trigger-attached.png)

**2. Kiểm thử với sự kiện mẫu.** Nhóm dùng chức năng **Test** trong Console với JSON có cấu trúc giống sự kiện SNS. Ảnh kết quả hiển thị lần chạy thành công, log chứa nội dung thông báo thử và phản hồi `statusCode: 200`. Ca này xác nhận handler đọc được payload mẫu. Để xác minh SNS gọi hàm thật, đối chiếu log của handler sau một lần publish từ topic. [AWS về cấu trúc sự kiện SNS](https://docs.aws.amazon.com/lambda/latest/dg/with-sns.html).

![Lambda Test thành công với sự kiện SNS mẫu và phản hồi statusCode 200](../../../../images/5-Workshop/5.5-Serverless/5.5.2-Lambda-Functions/11-sns-handler-test.png)


## Hàm thống kê `raki-stats-service`

**1. Chuẩn bị kết nối.** Dùng Python 3.12 với thư viện `PyMySQL`. Đặt Lambda trong mạng có thể kết nối RDS ở mục 5.3.4 và mở TCP `3306` từ security group của Lambda trên security group RDS. Dùng tài khoản MySQL chỉ có quyền đọc dữ liệu cần thiết, lấy thông tin đăng nhập từ Secrets Manager.

![Execution role và phần quyền ghi log của raki-stats-service](../../../../images/5-Workshop/5.5-Serverless/5.5.2-Lambda-Functions/12-stats-execution-role.png)

**2. Kiểm tra truy vấn thống kê.** Truy vấn trong hồ sơ tính `total_orders` và `total_revenue` từ bảng `orders`. Khi chạy thử, dùng một mốc dữ liệu đã biết và đối chiếu hai giá trị trả về với kết quả MySQL. Ảnh danh sách Lambda xác nhận có hàm `raki-stats-service`. Hồ sơ chưa có ảnh kết quả truy vấn từ chính hàm này. [AWS về kết nối Lambda với RDS](https://docs.aws.amazon.com/lambda/latest/dg/services-rds.html).

## Kết luận

Minh chứng của nhóm ghi lại ảnh đầu vào và thumbnail trong S3, log stream của hàm xử lý ảnh và kết quả test sự kiện mẫu của hàm nhận SNS. Khi tổng hợp kết quả chạy trên AWS, nhóm em sẽ bổ sung log do SNS kích hoạt cùng kết quả truy vấn RDS từ hàm thống kê.
