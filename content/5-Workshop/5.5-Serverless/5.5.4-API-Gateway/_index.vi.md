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

## Đầu vào kiểm thử và quyền truy cập

Chuẩn bị hàm trợ lý, một câu hỏi mẫu và origin của website RakiBookery. Xác định đối tượng được gọi route trước khi kết nối với giao diện. CORS kiểm soát việc trình duyệt đọc phản hồi từ origin khác. Với route yêu cầu đăng nhập, cấu hình authorizer phù hợp. Nếu route mở để thử, giữ trong phạm vi kiểm thử và đặt giới hạn tần suất phù hợp.

## Bước 1. Tạo HTTP API và gắn Lambda

Trong Region Singapore (`ap-southeast-1`), mở **API Gateway → Create API → HTTP API**. Chọn tích hợp **Lambda** và chọn đúng hàm trợ lý đã cấu hình ở [mục 5.5.3]({{< relref "/5-Workshop/5.5-Serverless/5.5.3-Bedrock-Demo/" >}}). Với tích hợp Lambda proxy, chọn định dạng payload **2.0**. Hàm xử lý lấy phương thức HTTP từ `requestContext.http.method` và phân tích JSON trong trường `body`. Kiểm tra chính sách gắn với tài nguyên (resource-based policy) của hàm có cấp cho API Gateway quyền `lambda:InvokeFunction` để gọi đúng hàm Lambda. [AWS về Lambda proxy integration](https://docs.aws.amazon.com/apigateway/latest/developerguide/http-api-develop-integrations-lambda.html).


## Bước 2. Khai báo route và stage
 
Dùng `POST /chat` làm route minh họa và gắn route với Lambda integration đã tạo. Trong **Stages**, kiểm tra stage đang nhận yêu cầu và trạng thái **Auto-deploy**. Nếu dùng stage `$default`, ghép `/chat` trực tiếp vào invoke URL. Nếu dùng stage có tên, lấy URL hiển thị trong Console rồi thêm đường dẫn route. Đối chiếu URL cuối với cấu hình gọi API của website. [AWS về stage của HTTP API](https://docs.aws.amazon.com/apigateway/latest/developerguide/http-api-stages.html).


## Bước 3. Cấu hình CORS cho website

Nếu website và API dùng hai origin khác nhau, đặt **Allow origins** là origin của website đang dùng, chẳng hạn địa chỉ HTTPS người dùng mở qua CloudFront. Cho phép phương thức `POST` và header `Content-Type` để trình duyệt gửi JSON. Nếu ứng dụng gửi token, cho phép thêm header `Authorization` và kiểm tra authorizer theo cơ chế đăng nhập của dự án. Khi bật CORS ở HTTP API, API Gateway xử lý preflight `OPTIONS` và trả header CORS theo cấu hình của API. [AWS về CORS cho HTTP API](https://docs.aws.amazon.com/apigateway/latest/developerguide/http-api-cors.html).


Ở lượt thử đầu, dùng invoke URL trong **Stages** và đường dẫn route thực tế. Nếu bật access log cho stage, cấu hình log có request ID của API Gateway, route và mã trạng thái. Trường `request_id` trong body phản hồi lấy từ Lambda và dùng để tìm log của hàm. [AWS về access log của HTTP API](https://docs.aws.amazon.com/apigateway/latest/developerguide/http-api-logging.html).

## Bước 4. Kiểm tra từ API đến trình duyệt

Gửi `POST` đến route với header `Content-Type: application/json` và nội dung sau.

```json
{
  "prompt": "Cho tôi thông tin liên hệ nhà sách."
}
```

- **Câu hỏi hợp lệ.** Kiểm tra mã HTTP và nội dung JSON. Khi tra được thông tin và sinh phản hồi thành công, handler dự kiến trả `200`, có trường `answer` và `request_id`. Với câu hỏi này, nhóm đối chiếu câu trả lời với tài liệu trong Knowledge Base.
- **Kiểm tra trên website.** Mở công cụ **Network** của trình duyệt và gửi câu hỏi từ giao diện. Đối chiếu URL, phương thức, mã HTTP và nội dung hiển thị. Dùng `request_id` trong phản hồi để đối chiếu với log Lambda theo [mục 5.6.5]({{< relref "/5-Workshop/5.6-Operations/5.6.5-CloudWatch-Logs-Log-Insights/" >}}).

Sau khi kiểm tra bằng công cụ gọi API, gửi cùng câu hỏi từ trình duyệt. Đối chiếu nội dung JSON với câu trả lời hiển thị trên giao diện để kiểm tra luồng từ website đến Lambda.

## Kết luận

Mã Lambda đã có xử lý body theo định dạng API Gateway. Cấu hình HTTP API trình bày cách kết nối website với hàm trợ lý bằng Lambda proxy integration, route, stage và CORS. Route `POST /chat` được dùng làm ví dụ để đối chiếu URL gọi API, nội dung JSON và câu trả lời trên giao diện.
