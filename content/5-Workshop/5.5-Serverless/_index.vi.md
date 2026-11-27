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

Website PHP chạy trên Amazon ECS với Fargate và lưu dữ liệu nhà sách trong Amazon RDS for MySQL. Khi một ảnh được tải lên bằng thao tác PUT và khóa đối tượng bắt đầu bằng `images/`, sự kiện S3 kích hoạt hàm Lambda xử lý ảnh. Hàm ghi ảnh thu nhỏ vào `thumbnails/`. Tiền tố đầu ra nằm ngoài phạm vi của trigger nên ảnh thu nhỏ không kích hoạt lại hàm. Với luồng thông báo, một SNS topic phân phối bản tin đến email đã xác nhận đăng ký và gọi hàm Lambda nhận bản tin để ghi log.

Với trợ lý sách, nhóm thiết kế API Gateway làm đầu vào HTTP cho Lambda. Tùy nội dung câu hỏi, Lambda tra tài liệu nhà sách trong Knowledge Base hoặc đọc danh mục còn hàng từ RDS để đưa dữ liệu phù hợp cho Nova Micro. Guardrails kiểm tra nội dung đầu vào và đầu ra. Ví dụ, câu hỏi về chính sách của nhà sách cần tài liệu đã đồng bộ, còn câu hỏi về giá và tồn kho cần đối chiếu với dữ liệu MySQL.

## Lý do tách các luồng xử lý

SNS phân phối một bản tin đến các bên đã đăng ký. Lambda xử lý những sự kiện như tệp mới tải lên mà nhóm không cần duy trì thêm máy chủ. Trợ lý sách dùng HTTP vì website cần nhận phản hồi cho từng câu hỏi. Cách kiểm tra của mỗi luồng khác nhau. Với sự kiện, nhóm tìm object đầu ra, log hoặc email. Với trợ lý, nhóm đối chiếu phản hồi HTTP với lần gọi Lambda và nguồn dữ liệu được sử dụng.

## Trình tự thực hiện và cách kiểm tra

**Thông báo qua SNS.** Nhóm em tạo topic rồi thử gửi trực tiếp đến email đã xác nhận subscription. Ca này giúp xác nhận đường gửi email trước khi nối thêm nguồn thông báo từ ứng dụng. Khi tích hợp publisher, nhóm dùng nội dung thử có mã nhận diện để đối chiếu giữa log của lần gửi và email nhận được.

**Xử lý sự kiện bằng Lambda.** Sau khi tạo hàm, nhóm kiểm tra bằng sự kiện mẫu trong Console rồi thử kích hoạt từ dịch vụ thật. Với ảnh sách, nhóm đối chiếu cùng tên tệp ở S3 và log của lần chạy. Với hàm thống kê, kết quả cần so sánh với truy vấn trực tiếp trên RDS để biết số liệu có đúng hay không.

**Tích hợp trợ lý sách.** Nhóm chuẩn bị tài liệu và metadata trên S3, đồng bộ Knowledge Base rồi mới kiểm tra truy xuất. Sau đó, nhóm dùng hai prompt riêng cho tra cứu thông tin và gợi ý sách. Ca kiểm tra Guardrail được chạy riêng để có thể đọc kết quả của bộ lọc trước khi chẩn đoán lỗi ở bước gọi dữ liệu hoặc model.

**Kiểm thử qua API Gateway.** Quy trình kiểm thử bắt đầu bằng việc gửi yêu cầu JSON trực tiếp đến route của HTTP API. Sau đó, nhóm sẽ gọi cùng route từ giao diện website và đối chiếu URL, mã trạng thái HTTP, phản hồi hiển thị và `request_id` nếu Lambda trả về. Các thông tin này giúp xác định lỗi ở trình duyệt, API Gateway hay hàm xử lý.

## Kết quả và bài học

Nhóm em đã nhận được email thông báo thử gửi trực tiếp từ SNS Console. Khi tải ảnh thử lên S3, nhóm tìm thấy ảnh thu nhỏ trong `thumbnails/` và log stream của hàm xử lý ảnh. Hàm nhận SNS cũng xử lý được sự kiện mẫu khi chạy **Test** trong Lambda Console. Để xác nhận toàn bộ luồng thông báo, nhóm cần kiểm tra một lần gọi Lambda do SNS kích hoạt và một lần ứng dụng gửi bản tin qua hàm gửi thông báo (publisher).

Ở phần trợ lý sách, nhóm đã đồng bộ nguồn tài liệu vào Knowledge Base và chạy thành công 16 ca kiểm thử cục bộ. Trong lần thử trên AWS, log ghi nhận Guardrail đã kiểm tra câu hỏi đầu vào. Yêu cầu gợi ý sách dừng ở bước đăng nhập MySQL với lỗi `1045`. Phần API Gateway đã có cách cấu hình và kịch bản kiểm tra, còn kết quả gọi từ website cần đối chiếu với route thực tế và phản hồi trên trình duyệt.

Qua phần này, em học được cách lần theo một yêu cầu qua nhiều dịch vụ. Ví dụ, khi ảnh thu nhỏ xuất hiện, em có thể đối chiếu ảnh đầu vào, log Lambda và object đầu ra. Với trợ lý sách, em dựa vào mã lỗi và `request_id` để xác định lỗi nằm ở bước nào trước khi sửa cấu hình.

## Minh chứng cho lần trình bày

Với mỗi luồng, lưu dữ liệu thử, kết quả và một ID hoặc thời điểm để đối chiếu. Dùng mã đơn hàng cho SNS, object key cho xử lý ảnh và `request_id` của Lambda cho trợ lý. Với ca lỗi, ghi bước cuối cùng đã hoạt động và thay đổi cần kiểm tra ở lần chạy sau.


## Kết luận

Nhóm đã có minh chứng cho luồng gửi email qua SNS và tạo ảnh thu nhỏ bằng Lambda. Việc kiểm thử trợ lý từ website vẫn cần tiếp tục sau khi xử lý lỗi xác thực MySQL và đối chiếu cấu hình API Gateway. Phần [5.6 Vận hành]({{< relref "/5-Workshop/5.6-Operations/" >}}) trình bày cách nhóm theo dõi các luồng này và kiểm thử khi đưa chúng vào cùng một hệ thống.

## Các phần triển khai

{{% children sort="Weight" /%}}