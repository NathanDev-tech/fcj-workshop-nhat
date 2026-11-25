---
title : "Tích hợp Bedrock"
date : 2024-01-01
weight : 3
chapter : false
pre : " <b> 5.5.3 </b> "
---

## Mục tiêu

Nhóm em triển khai trợ lý RakiBookery để khách có thể hỏi cách sử dụng website, tra cứu thông tin nhà sách và chọn sách theo nhu cầu. Hai loại câu hỏi dùng nguồn dữ liệu khác nhau. Giá và tồn kho lấy từ MySQL để phản ánh danh mục hiện tại. Thông tin liên hệ, chính sách và hướng dẫn sử dụng lấy từ tài liệu đã đưa vào Knowledge Base.

## Kiến trúc tổng quan

```text
Câu hỏi → Lambda Python 3.12 → Guardrail đầu vào → Xác định loại câu hỏi
                                   ├─ Hỏi nhà sách → Agent Runtime Retrieve → Knowledge Base
                                   └─ Gợi ý sách  → RDS MySQL → Danh mục còn hàng
                        Dữ liệu phù hợp → Nova Micro Converse → Kiểm tra đầu ra → Phản hồi
```

Trong mã nguồn, **Bedrock Agent Runtime** là API mà Lambda gọi để thực hiện `Retrieve` trên Knowledge Base. Lambda điều phối hai luồng dữ liệu và gọi `Converse` qua **Bedrock Runtime** để tạo câu trả lời. Các bước dưới đây mô tả cách nhóm triển khai theo cấu trúc này. [Amazon Bedrock Retrieve](https://docs.aws.amazon.com/bedrock/latest/APIReference/API_agent-runtime_Retrieve.html) · [Amazon Bedrock Converse](https://docs.aws.amazon.com/bedrock/latest/APIReference/API_runtime_Converse.html).

## Các bước triển khai

### Bước 1. Chuẩn bị Knowledge Base và nguồn S3

Nhóm em chuẩn bị tài liệu tra cứu và metadata đi kèm trong S3. Với Managed Knowledge Base, Lambda gọi `Retrieve` bằng `managedSearchConfiguration` và lọc theo trường `document_type`. Các câu hỏi về nhà sách và hướng dẫn sử dụng tra tài liệu loại `store_info`. Tài liệu minh họa được đánh dấu `environment=demo` để trợ lý nói rõ đây là thông tin mẫu. Khi không tìm được đoạn phù hợp, hàm trả thông báo thiếu dữ liệu.

![Knowledge Base đã được bật trong cấu hình Lambda](../../../../images/5-Workshop/5.5-Serverless/5.5.3-Bedrock-Demo/01-knowledge-base-config.png)

Ghi nhận cấu hình của nhóm cho thấy Knowledge Base ở trạng thái `Available`. Lượt đồng bộ được ghi nhận đã hoàn tất với **2 tài liệu và 2 tệp metadata, 0 lỗi**. Đây là mốc kiểm tra nguồn dữ liệu. Khi kiểm thử Lambda, nhóm cần đối chiếu thêm log `KB retrieved` và câu trả lời với tài liệu gốc.

### Bước 2. Cấu hình Lambda và Nova Micro

![Profile Nova Micro và Region trong cấu hình Lambda](../../../../images/5-Workshop/5.5-Serverless/5.5.3-Bedrock-Demo/02-nova-micro-config.png)

Nhóm chuyển sang cấu hình **Amazon Nova Micro** với profile `apac.amazon.nova-micro-v1:0`, gọi từ Region Singapore (`ap-southeast-1`). Lambda ký yêu cầu bằng thông tin xác thực của execution role. Các quyền tương ứng là `bedrock:Retrieve` cho Knowledge Base, `bedrock:InvokeModel` cho profile và model được phép, và `bedrock:ApplyGuardrail` cho Guardrail đang dùng. Nhóm dùng Secrets Manager cho thông tin đăng nhập MySQL.

### Bước 3. Kết nối danh mục RDS để gợi ý sách

![Cấu hình MySQL khi chẩn đoán Lambda, với thông tin kết nối được che](../../../../images/5-Workshop/5.5-Serverless/5.5.3-Bedrock-Demo/03-rds-connection-config.png)

Lambda kết nối MySQL trong VPC và đọc schema `bookstoredb`. Khi đối chiếu với SQL của dự án, nhóm điều chỉnh truy vấn để đọc `book.Name` và ghép bảng `category` qua `CategoryID`. Lambda chỉ đưa sách đang mở bán và còn hàng vào danh mục gửi cho model. Model trả tối đa ba ID, sau đó Lambda kiểm tra từng ID và lấy tên, giá, tồn kho từ kết quả SQL. Tài liệu sách trong Knowledge Base có thể bổ sung mô tả khi bật `BOOK_KB_ENABLED`.

### Bước 4. Áp dụng Guardrails cho đầu vào và đầu ra

![Guardrail phiên bản 1 trong cấu hình Lambda](../../../../images/5-Workshop/5.5-Serverless/5.5.3-Bedrock-Demo/04-guardrail-config.png)

Hồ sơ cấu hình ghi nhận Guardrail đã có phiên bản `1`. Lambda gọi `ApplyGuardrail` trước khi truy vấn dữ liệu và gọi lại với phần nội dung sẽ hiển thị cho người dùng, kể cả danh sách sách dự phòng. Khi Guardrail can thiệp, handler trả `GUARDRAIL_BLOCKED`. Nhóm chuẩn bị một ca câu hỏi bình thường và một ca dùng từ khóa thử có trong quy tắc đã publish để kiểm tra cả hai tình huống. [Amazon Bedrock ApplyGuardrail](https://docs.aws.amazon.com/bedrock/latest/APIReference/API_runtime_ApplyGuardrail.html).

## Kiểm thử và xử lý lỗi

Nhóm em chạy riêng ca hỏi thông tin nhà sách, ca gợi ý sách và ca kiểm tra quy tắc chặn. Mỗi ca có thể đối chiếu qua `request_id` trong CloudWatch. Với ca gợi ý, kiểm tra mã sách và giá với RDS. Với ca nhà sách, kiểm tra số đoạn truy xuất và đối chiếu câu trả lời với tài liệu trong S3.

![Hình trích từ log Lambda trên AWS về Guardrail và lỗi MySQL 1045](../../../../images/5-Workshop/5.5-Serverless/5.5.3-Bedrock-Demo/05-guardrail-mysql-diagnostic.png)

Trong lần chẩn đoán Lambda trên AWS, log ghi nhận Guardrail đã kiểm tra đầu vào với `action=NONE`. Cùng lần gọi, MySQL trả lỗi `1045` ở bước đăng nhập. Dưới đây là các trường liên quan được trích từ log.

```text
Guardrail source=INPUT action=NONE
MYSQL_CONNECTION_DIAGNOSTIC {"mysql_error": 1045, "stage": "connect", "attempt": 1}
MYSQL_CONNECTION_DIAGNOSTIC {"mysql_error": 1045, "stage": "connect", "attempt": 2}
```

Lỗi `1045` cho thấy Lambda đã nhận được phản hồi từ MySQL và bước xác thực bị từ chối. Nhóm cần đối chiếu thông tin đăng nhập trong secret với DB instance đang dùng trước khi chạy lại ca gợi ý. Bộ kiểm thử cục bộ của mã nguồn đạt **16/16 ca**, bao gồm lọc metadata, kiểm tra danh sách ID sách, xử lý quota và Guardrail đầu ra.

![Kết quả chạy bộ kiểm thử cục bộ đạt 16/16 ca](../../../../images/5-Workshop/5.5-Serverless/5.5.3-Bedrock-Demo/06-local-test-result.png)

## Kết luận

Nhóm đã chuẩn bị nguồn Knowledge Base, thực hiện đồng bộ tài liệu và ghi nhận một lần Guardrail kiểm tra đầu vào trên AWS. Luồng gợi ý sách cần kiểm thử lại sau khi xử lý lỗi đăng nhập MySQL. Kết quả `Retrieve` và `Converse` trên AWS cần được xác nhận qua log và phản hồi của từng ca. Phần kết nối trợ lý với website qua API Gateway được trình bày ở mục 5.5.4.
