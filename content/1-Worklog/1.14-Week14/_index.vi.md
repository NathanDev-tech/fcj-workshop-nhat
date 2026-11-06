---
title: "Worklog Tuần 14"
date: 2026-11-02
weight: 14
chapter: false
pre: " <b> 1.14. </b> "
---

### Mục tiêu tuần 14:

- Củng cố cách sử dụng AWS Lambda với Python, đặc biệt là `lambda_handler`, `event`, `context` và logging.
- Thực hành đọc và kiểm tra file CSV, phát hiện dữ liệu không hợp lệ và xử lý lỗi định dạng đầu vào.
- Hoàn thiện kết nối AWS Lambda với Amazon RDS MySQL và gọi mô hình AI thông qua Amazon Bedrock cho dự án **RakiBookery**.
- Thiết lập Amazon Bedrock Knowledge Base với dữ liệu chứa thông tin của nhà sách RakiBookery.
- Rèn luyện cách kiểm tra lỗi theo từng lớp: dữ liệu đầu vào, kết nối cơ sở dữ liệu, quyền truy cập và dịch vụ AI.

### Các công việc đã làm trong tuần này:

| Thứ | Công việc | Ngày bắt đầu | Ngày hoàn thành | Nguồn tài liệu |
|:---:|---|:---:|:---:|---|
| 2 |  | 02/11/2026 | 02/11/2026 |  |
| 3 | Tìm hiểu AWS Lambda với Python, cấu trúc handler, `event`/`context` và cách thử bằng JSON mẫu (phần thực hành chưa có xác nhận hoàn thành riêng) | 03/11/2026 | 03/11/2026 | [Lambda Python Handler](https://docs.aws.amazon.com/lambda/latest/dg/python-handler.html), [Lambda Python Logging](https://docs.aws.amazon.com/lambda/latest/dg/python-logging.html) |
| 4 | Xây dựng và kiểm tra chương trình đọc CSV cho dữ liệu laptop; xử lý lỗi file có dòng trắng trước header | 04/11/2026 | 04/11/2026 | [Python CSV](https://docs.python.org/3/library/csv.html), [Python Decimal](https://docs.python.org/3/library/decimal.html) |
| 5 | Hoàn tất kết nối AWS Lambda với Amazon RDS MySQL và gọi AI thông qua Amazon Bedrock cho RakiBookery | 05/11/2026 | 05/11/2026 | [Lambda with RDS](https://docs.aws.amazon.com/lambda/latest/dg/services-rds.html), [Bedrock Inference API](https://docs.aws.amazon.com/bedrock/latest/userguide/inference-api.html) |
| 6 | Hoàn thành thiết lập Amazon Bedrock Knowledge Base với dữ liệu chứa thông tin nhà sách RakiBookery | 06/11/2026 | 06/11/2026 | [Knowledge Base Data Sync](https://docs.aws.amazon.com/bedrock/latest/userguide/kb-data-source-sync-ingest.html), [Knowledge Base Retrieval](https://docs.aws.amazon.com/bedrock/latest/userguide/kb-how-retrieval.html) |

### Kết quả đạt được tuần 14:

&emsp;◉ Thứ Ba:
<br>&emsp;&emsp;○ Tiếp cận cấu trúc hàm AWS Lambda với Python, gồm `lambda_handler(event, context)`, dữ liệu đầu vào từ event và cách xem log khi xử lý yêu cầu.
<br>&emsp;&emsp;○ Chuẩn bị nội dung thực hành sử dụng event JSON mẫu và chương trình Python chạy local. **Chưa có xác nhận riêng rằng bài chạy thử Thứ Ba đã hoàn thành**, nên nội dung này được ghi nhận ở mức chuẩn bị/học tập.

&emsp;◉ Thứ Tư:
<br>&emsp;&emsp;○ Thực hành chương trình đọc dữ liệu laptop từ `samples/laptops_week14.csv` với các trường `laptop_id`, `brand`, `model`, `ram_gb`, `price` và `quantity`.
<br>&emsp;&emsp;○ Xây dựng bước kiểm tra cấu trúc CSV, các giá trị số không hợp lệ và mã laptop trùng trong cùng file; dữ liệu mẫu có 6 dòng được thiết kế để kiểm tra 4 dòng hợp lệ và 2 dòng lỗi.
<br>&emsp;&emsp;○ Gặp lỗi `ValueError: CSV không có header` khi chạy `day04_validate_csv.py`. Kiểm tra kích thước file và nội dung byte đầu cho thấy file có dữ liệu nhưng bắt đầu bằng `\r\n`, tức có một dòng trắng trước header.
<br>&emsp;&emsp;○ Xác định nguyên nhân `csv.DictReader` nhận nhầm dòng đầu tiên làm header, sau đó xử lý dòng trắng đầu file và hoàn thành bài thực hành Thứ Tư.

&emsp;◉ Thứ Năm:
<br>&emsp;&emsp;○ Hoàn tất kết nối từ AWS Lambda đến cơ sở dữ liệu Amazon RDS MySQL của dự án **RakiBookery**.
<br>&emsp;&emsp;○ Tiếp tục xử lý bài toán kết nối Lambda–RDS theo cấu hình mạng và quyền truy cập phù hợp; phân biệt lỗi kết nối cơ sở dữ liệu với lỗi thực thi hàm.
<br>&emsp;&emsp;○ Thực hiện thành công lời gọi mô hình AI thông qua Amazon Bedrock trong luồng xử lý của RakiBookery, giúp kết hợp dữ liệu nhà sách và khả năng xử lý của AI.

&emsp;◉ Thứ Sáu:
<br>&emsp;&emsp;○ Hoàn thành việc thiết lập **Amazon Bedrock Knowledge Base** với dữ liệu chứa thông tin của nhà sách RakiBookery.
<br>&emsp;&emsp;○ Tìm hiểu vai trò của Knowledge Base trong việc quản lý nguồn dữ liệu riêng và hỗ trợ truy xuất thông tin cho các chức năng AI của nhà sách.
<br>&emsp;&emsp;○ Phân biệt việc **thiết lập Knowledge Base** với bước kiểm thử sâu chất lượng truy xuất, tích hợp ứng dụng và quy trình RAG đầu cuối; các bước sau chưa được ghi nhận là đã hoàn thành.
