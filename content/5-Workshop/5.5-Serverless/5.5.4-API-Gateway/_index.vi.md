---
title: "Thiết lập API Gateway"
date: 2024-01-01
weight: 4
chapter: false
pre: " <b> 5.5.4. </b> "
aliases:
  - "/5-workshop/5.5-operations/5.5.4-api-gateway/"
---

## Mục tiêu

Sau khi kiểm thử cục bộ handler Lambda ở mục 5.5.3, nhóm em cần một địa chỉ HTTPS để website RakiBookery gửi câu hỏi đến trợ lý sách. API Gateway nhận yêu cầu từ website, chuyển phần thân JSON sang Lambda và trả kết quả về giao diện. Nhờ vậy, nhóm có thể kiểm tra đường gọi API riêng trước khi thử trên trình duyệt.

## Luồng xử lý

```text
Website RakiBookery → HTTP API → Lambda trợ lý sách → Bedrock và nguồn dữ liệu
Website RakiBookery ← Phản hồi HTTP từ API Gateway ← Kết quả Lambda
```

## Bước 1. Tạo HTTP API và gắn Lambda

Trong Region Singapore (`ap-southeast-1`), mở **API Gateway → Create API → HTTP API**. Chọn tích hợp **Lambda** và chọn đúng hàm trợ lý đã cấu hình ở [mục 5.5.3]({{< relref "/5-Workshop/5.5-Serverless/5.5.3-Bedrock-Demo/" >}}). Với Lambda proxy integration, nhóm dùng payload format **2.0**. Mã handler đọc được phương thức từ `requestContext.http.method` và giải mã phần thân trong trường `body`. Kiểm tra quyền invoke trong resource-based policy của hàm để API Gateway gọi được đúng Lambda. [AWS về Lambda proxy integration](https://docs.aws.amazon.com/apigateway/latest/developerguide/http-api-develop-integrations-lambda.html).

<!-- Ảnh 01 khi có: API Overview và Region. -->
<!-- Ảnh 02 khi có: Integrations hiển thị Lambda đích và payload format. -->

## Bước 2. Khai báo route và stage

Dùng `POST /chat` làm route minh họa và gắn route với Lambda integration đã tạo. Trong **Stages**, kiểm tra stage đang nhận yêu cầu và trạng thái **Auto-deploy**. Nếu dùng stage `$default`, ghép `/chat` trực tiếp vào invoke URL. Nếu dùng stage có tên, lấy URL hiển thị trong Console rồi thêm đường dẫn route. Đối chiếu URL cuối với cấu hình gọi API của website. [AWS về stage của HTTP API](https://docs.aws.amazon.com/apigateway/latest/developerguide/http-api-stages.html).

<!-- Ảnh 03 khi có: Routes hiển thị phương thức và integration. -->
<!-- Ảnh 04 khi có: Stages và invoke URL. -->

## Bước 3. Cấu hình CORS cho website

Nếu website và API dùng hai origin khác nhau, đặt **Allow origins** là origin của website đang dùng, chẳng hạn địa chỉ HTTPS người dùng mở qua CloudFront. Cho phép phương thức `POST` và header `Content-Type` để trình duyệt gửi JSON. Nếu ứng dụng gửi token, cho phép thêm header `Authorization` và kiểm tra authorizer theo cơ chế đăng nhập của dự án. Khi bật CORS ở HTTP API, API Gateway xử lý preflight `OPTIONS` và trả header CORS theo cấu hình của API. [AWS về CORS cho HTTP API](https://docs.aws.amazon.com/apigateway/latest/developerguide/http-api-cors.html).

<!-- Ảnh 05 khi có: CORS với origin, method và header. -->

## Bước 4. Kiểm tra từ API đến trình duyệt

Gửi `POST` đến route với header `Content-Type: application/json` và nội dung sau.

```json
{
  "prompt": "Cho tôi thông tin liên hệ nhà sách."
}
```

- **Câu hỏi hợp lệ.** Kiểm tra mã HTTP và nội dung JSON. Khi tra được thông tin và sinh phản hồi thành công, handler dự kiến trả `200`, có trường `answer` và `request_id`. Với câu hỏi này, nhóm đối chiếu câu trả lời với tài liệu trong Knowledge Base.
- **Thiếu câu hỏi.** Gửi `{}` đến cùng route. Handler dự kiến trả `400` với mã `BAD_REQUEST`. Ca này giúp kiểm tra API đã chuyển body đến Lambda và phản hồi lỗi được trả về cho client.
- **Kiểm tra trên website.** Mở công cụ **Network** của trình duyệt và gửi câu hỏi từ giao diện. Đối chiếu URL, phương thức, mã HTTP và nội dung hiển thị. Nếu gọi trực tiếp được nhưng trình duyệt báo lỗi, xem yêu cầu preflight và origin. Nếu Lambda trả lỗi, dùng `request_id` trong phản hồi để tra CloudWatch theo [mục 5.6.5]({{< relref "/5-Workshop/5.6-Operations/5.6.5-CloudWatch-Logs-Log-Insights/" >}}).

<!-- Ảnh 06 khi có: phản hồi HTTP cho ca hợp lệ và ca thiếu prompt. -->
<!-- Ảnh 07 khi có: yêu cầu trong tab Network hoặc phản hồi hiển thị trên website. -->

## Kết luận

Mã Lambda của nhóm đã có xử lý body theo định dạng API Gateway. Route `POST /chat` trong mục này minh họa cấu hình và cách nghiệm thu. Kết quả triển khai cần được đối chiếu với route, stage và phản hồi thực tế của môi trường RakiBookery.
