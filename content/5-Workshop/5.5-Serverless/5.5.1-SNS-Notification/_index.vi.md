---
title: "Thông báo qua SNS"
date: 2024-01-01
weight: 1
chapter: false
pre: " <b> 5.5.1. </b> "
aliases:
  - "/5-workshop/5.5-operations/5.5.1-sns-notification/"
---

## Mục tiêu

Nhóm em dùng Amazon SNS để gửi thông báo nội bộ từ RakiBookery, chẳng hạn khi có đơn hàng mới hoặc khi ứng dụng cần báo lỗi. Nhóm kiểm tra đường gửi đến email trước, rồi mới gắn Lambda để xử lý thông báo theo sự kiện. Toàn bộ phần này dùng Region Singapore (`ap-southeast-1`).

## Triển khai

### Bước 1. Tạo topic `raki-noti-sns`

Trong **Amazon SNS → Topics → Create topic**, chọn loại **Standard** và đặt tên `raki-noti-sns`. Nhóm chọn Standard vì cần gửi trực tiếp tới email và hỗ trợ đăng ký Lambda. Sau khi tạo, lấy **Topic ARN** từ màn hình chi tiết để dùng lại khi tạo subscription hoặc cấu hình hàm gửi. [AWS về SNS email subscription](https://docs.aws.amazon.com/sns/latest/dg/sns-email-notifications.html).


![Chọn Standard và đặt tên topic raki-noti-sns](../../../../images/5-Workshop/5.5-Serverless/5.5.1-SNS-Notification/01-topic-create.png)


![Topic raki-noti-sns xuất hiện trong danh sách SNS](../../../../images/5-Workshop/5.5-Serverless/5.5.1-SNS-Notification/02-topic-list.png)


### Bước 2. Đăng ký email nhóm

Trong topic vừa tạo, chọn **Create subscription**, đặt **Protocol** là `Email` và nhập địa chỉ nhận ở **Endpoint**. Nhóm mở thư xác nhận từ AWS Notifications, chọn **Confirm subscription**, rồi quay lại Console để kiểm tra trạng thái `Confirmed`. Sau bước này, địa chỉ đã sẵn sàng nhận thông báo từ topic.


![Tạo subscription với giao thức Email](../../../../images/5-Workshop/5.5-Serverless/5.5.1-SNS-Notification/03-email-subscribe.png)


![Subscription email ở trạng thái Confirmed](../../../../images/5-Workshop/5.5-Serverless/5.5.1-SNS-Notification/04-email-confirmed.png)


### Bước 3. Kết nối Lambda với topic

Nhóm cấu hình một subscription **AWS Lambda** trỏ tới `raki-sns-handler`. Khi có thông báo mới, SNS gọi hàm này để đọc tiêu đề, nội dung và ghi log. Cấu hình thực tế của hàm nhận được trình bày ở [mục 5.5.2]({{< relref "/5-Workshop/5.5-Serverless/5.5.2-Lambda-Functions/" >}}).

Ở chiều gửi, tài liệu của nhóm có mẫu Lambda publisher đọc ARN qua biến `SNS_TOPIC_ARN`, gọi `sns.publish()` và ghi `MessageId`. Với publisher, quyền cần thiết là `sns:Publish` trên ARN của `raki-noti-sns`. Giữ luồng gửi riêng với handler nhận để một thông báo không bị publish ngược lại chính topic đã kích hoạt hàm. [AWS về Lambda nhận sự kiện SNS](https://docs.aws.amazon.com/lambda/latest/dg/with-sns.html).


![Chọn AWS Lambda và endpoint raki-sns-handler](../../../../images/5-Workshop/5.5-Serverless/5.5.1-SNS-Notification/05-lambda-subscribe.png)


![Subscription Lambda ở trạng thái Confirmed](../../../../images/5-Workshop/5.5-Serverless/5.5.1-SNS-Notification/06-lambda-confirmed.png)


## Kiểm thử và kết quả


![Nút Publish message trên trang topic raki-noti-sns](../../../../images/5-Workshop/5.5-Serverless/5.5.1-SNS-Notification/07-publish-console.png)


1. **Gửi nội dung riêng theo giao thức.** Trong topic, chọn **Publish message**, dùng chế độ payload theo giao thức và điền nội dung cho `email`. Hộp thư trong ảnh đã nhận được nội dung mẫu dành cho email.
2. **Gửi cùng nội dung đến các subscription.** Ở chế độ payload giống nhau, nhóm em gửi thông báo giả lập đơn hàng `#1001`. Ảnh hộp thư ghi nhận nội dung đã đến người nhận.
3. **Kiểm tra luồng Lambda publisher.** Với một đơn hàng thử, đối chiếu thời điểm gọi hàm, `MessageId` trả về hoặc ghi trong CloudWatch Logs và thư nhận được. Đây là kết quả mong đợi của lượt thử publisher. Hai ảnh email dưới đây ghi lại lượt gửi trực tiếp qua SNS Console.


![Email nhận được nội dung mẫu dành riêng cho giao thức Email](../../../../images/5-Workshop/5.5-Serverless/5.5.1-SNS-Notification/08-email-protocol-test.png)


![Email nhận được thông báo thử nghiệm cho đơn hàng #1001](../../../../images/5-Workshop/5.5-Serverless/5.5.1-SNS-Notification/09-email-order-test.png)


## Kết luận

Nhóm em đã xác nhận topic Standard chuyển được thông báo thử đến email và đã gắn subscription cho Lambda. Kết quả này xác nhận đường **SNS → email**. Để chốt đường **ứng dụng → publisher → SNS → email**, nhóm cần thêm log của publisher và thư tương ứng với cùng một lần gọi.
