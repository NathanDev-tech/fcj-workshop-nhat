---
title: "Dịch vụ Serverless và AI"
date: 2024-01-01
weight: 5
chapter: false
pre: " <b> 5.5. </b> "
aliases:
  - "/5-workshop/5.5-operations/"
---

## Mục tiêu

Trong quá trình làm dự án, em cùng nhóm triển khai các chức năng hỗ trợ cho website bán sách RakiBookery. Khi quản trị viên tải ảnh sách, hệ thống cần tạo ảnh thu nhỏ để hiển thị trong danh mục. Nhóm cũng cần gửi thông báo thử đến email và xây dựng trợ lý để khách hỏi thông tin nhà sách hoặc tìm sách phù hợp. Phần này trình bày cách em và nhóm đưa những chức năng đó vào dự án và kiểm tra kết quả của từng luồng.

## Kiến trúc tổng quan

Website PHP tiếp tục chạy trên ECS Fargate và dùng RDS MySQL cho dữ liệu của nhà sách. Với luồng ảnh, S3 phát sự kiện khi có tệp mới trong `images/`. Lambda xử lý ảnh và lưu kết quả vào `thumbnails/`. Việc dùng hai thư mục riêng giúp ảnh thu nhỏ không kích hoạt lại hàm. Với thông báo, SNS phân phối nội dung đến các địa chỉ email đã xác nhận và hàm Lambda nhận thông báo để ghi log.

Với trợ lý sách, nhóm thiết kế API Gateway làm đầu vào HTTP cho Lambda. Tùy nội dung câu hỏi, Lambda tra tài liệu nhà sách trong Knowledge Base hoặc đọc danh mục còn hàng từ RDS để đưa dữ liệu phù hợp cho Nova Micro. Guardrails kiểm tra nội dung đầu vào và đầu ra. Ví dụ, câu hỏi về chính sách của nhà sách cần tài liệu đã đồng bộ, còn câu hỏi về giá và tồn kho cần đối chiếu với dữ liệu MySQL.

## Các phần triển khai

{{% children sort="Weight" /%}}

## Kết quả và bài học

Nhóm em đã nhận được email thông báo thử gửi trực tiếp từ SNS Console. Khi tải ảnh thử lên S3, nhóm tìm thấy ảnh thu nhỏ trong `thumbnails/` và log stream của hàm xử lý ảnh. Hàm nhận SNS cũng xử lý được sự kiện mẫu khi chạy **Test** trong Lambda Console. Để xác nhận toàn bộ luồng thông báo, nhóm còn cần đối chiếu một lần SNS gọi Lambda thật và một lần ứng dụng gửi thông báo qua publisher.

Ở phần trợ lý sách, nhóm đã đồng bộ nguồn tài liệu vào Knowledge Base và chạy thành công 16 ca kiểm thử cục bộ. Trong lần thử trên AWS, log ghi nhận Guardrail đã kiểm tra câu hỏi đầu vào. Yêu cầu gợi ý sách dừng ở bước đăng nhập MySQL với lỗi `1045`. Phần API Gateway đã có cách cấu hình và kịch bản kiểm tra, còn kết quả gọi từ website cần đối chiếu với route thực tế và phản hồi trên trình duyệt.

Qua phần này, em học được cách lần theo một yêu cầu qua nhiều dịch vụ. Ví dụ, khi ảnh thu nhỏ xuất hiện, em có thể đối chiếu ảnh đầu vào, log Lambda và object đầu ra. Với trợ lý sách, em dựa vào mã lỗi và `request_id` để xác định lỗi nằm ở bước nào trước khi sửa cấu hình.

## Kết luận

Nhóm đã có minh chứng cho luồng gửi email qua SNS và tạo ảnh thu nhỏ bằng Lambda. Việc kiểm thử trợ lý từ website vẫn cần tiếp tục sau khi xử lý lỗi xác thực MySQL và đối chiếu cấu hình API Gateway. Phần [5.6 Vận hành]({{< relref "/5-Workshop/5.6-Operations/" >}}) trình bày cách nhóm theo dõi các luồng này và kiểm thử khi đưa chúng vào cùng một hệ thống.
