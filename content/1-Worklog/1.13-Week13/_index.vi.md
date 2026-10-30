---
title: "Worklog Tuần 13"
date: 2026-10-26
weight: 13
chapter: false
pre: " <b> 1.13. </b> "
---

### Mục tiêu Tuần 13:

- Củng cố Python phục vụ tự động hóa tác vụ AWS, sử dụng môi trường ảo `.venv` và thư viện Boto3.
- Kiểm tra AWS CLI, danh tính AWS, Region và cấu hình credentials trước khi gọi dịch vụ bằng Python.
- Thực hành tạo và đọc dữ liệu từ Amazon DynamoDB bằng Python/Boto3.
- Xây dựng mini project DynamoDB Exporter để đọc dữ liệu laptop, xử lý phân trang và xuất CSV/JSON.
- Áp dụng logging, xử lý lỗi và nguyên tắc không đưa thông tin xác thực vào mã nguồn.

### Công việc đã hoàn thành trong tuần:

| Ngày | Công việc | Ngày bắt đầu | Ngày hoàn thành | Tài liệu tham khảo |
|:---:|---|:---:|:---:|---|
| 2 |  | 26/10/2026 | 26/10/2026 |  |
| 3 | Chuẩn bị môi trường Python `.venv`, cài Boto3, kiểm tra AWS CLI, AWS identity và Region; xử lý xung đột lệnh `aws` trên Windows bằng đường dẫn AWS CLI chính thức | 27/10/2026 | 27/10/2026 | [Boto3 Quickstart](https://docs.aws.amazon.com/boto3/latest/guide/quickstart.html), [Boto3 Credentials](https://docs.aws.amazon.com/boto3/latest/guide/credentials.html), [AWS CLI Installation](https://docs.aws.amazon.com/cli/latest/userguide/getting-started-install.html) |
| 4 | Ôn cấu trúc dữ liệu inventory và cách tổ chức đầu vào/đầu ra CSV, JSON để phục vụ các mini project Python | 28/10/2026 | 28/10/2026 | [Python CSV](https://docs.python.org/3/library/csv.html), [Python JSON](https://docs.python.org/3/library/json.html) |
| 5 | Phân tích yêu cầu DynamoDB Exporter; xác định cách đọc dữ liệu bằng Scan, tiếp tục qua `LastEvaluatedKey` và tạo báo cáo thống kê | 29/10/2026 | 29/10/2026 | [DynamoDB with Boto3](https://docs.aws.amazon.com/amazondynamodb/latest/developerguide/programming-with-python.html), [Boto3 Quickstart](https://docs.aws.amazon.com/boto3/latest/guide/quickstart.html) |
| 6 | Tạo bảng `LaptopInventory` ở Region Singapore, nạp 5 bản ghi laptop mẫu bằng Boto3, xác minh bằng Scan và xây dựng mã nguồn DynamoDB Exporter xuất CSV/JSON | 30/10/2026 | 30/10/2026 | [DynamoDB Scan](https://docs.aws.amazon.com/amazondynamodb/latest/developerguide/Scan.html), [Boto3 DynamoDB Table](https://docs.aws.amazon.com/boto3/latest/reference/services/dynamodb/table/scan.html) |

### Kết quả đạt được trong Tuần 13:

&emsp;◉ Thứ Ba:
<br>&emsp;&emsp;○ Tạo môi trường Python riêng `.venv` cho dự án `MiniProject2` và cài đặt thư viện Boto3.
<br>&emsp;&emsp;○ Kiểm tra cấu hình AWS CLI, credentials và Region `ap-southeast-1`; sử dụng `aws sts get-caller-identity` để xác minh danh tính AWS hiện tại.
<br>&emsp;&emsp;○ Xử lý lỗi khi lệnh `aws` trên Windows gọi nhầm `D:\aws.exe` và phát sinh lỗi thiếu module `_socket`; xác định và gọi trực tiếp AWS CLI v2 chính thức trong thư mục cài đặt theo tài khoản.
<br>&emsp;&emsp;○ Kiểm tra cấu hình bằng `aws configure list` và xác nhận AWS CLI v2 hoạt động với phiên bản được cài đặt.

&emsp;◉ Thứ Tư:
<br>&emsp;&emsp;○ Ôn cách tổ chức dữ liệu inventory dạng CSV/JSON và xác định những trường dữ liệu cần dùng cho bài toán laptop.
<br>&emsp;&emsp;○ Chuẩn bị cách tách dữ liệu đầu vào, dữ liệu đã xử lý và báo cáo đầu ra để chương trình Python dễ kiểm tra và mở rộng.

&emsp;◉ Thứ Năm:
<br>&emsp;&emsp;○ Phân tích yêu cầu của DynamoDB Exporter: đọc các item trong bảng, xử lý trường hợp kết quả Scan được chia thành nhiều trang và tổng hợp dữ liệu.
<br>&emsp;&emsp;○ Tìm hiểu vai trò của `LastEvaluatedKey` và `ExclusiveStartKey` để tiếp tục đọc dữ liệu cho đến khi không còn trang tiếp theo.
<br>&emsp;&emsp;○ Xác định hai đầu ra chính là `laptop_export.csv` và `inventory_summary.json`.

&emsp;◉ Thứ Sáu:
<br>&emsp;&emsp;○ Tạo mới bảng DynamoDB `LaptopInventory` trong Region `ap-southeast-1`, sử dụng billing mode `PAY_PER_REQUEST` và partition key `laptop_id` kiểu String.
<br>&emsp;&emsp;○ Viết `seed_data.py` bằng Boto3 để thêm 5 laptop mẫu gồm mã laptop, hãng, model, RAM, giá và số lượng. Chương trình sử dụng điều kiện `attribute_not_exists(laptop_id)` để tránh ghi đè item đã có.
<br>&emsp;&emsp;○ Chạy chương trình nạp dữ liệu và nhận kết quả thêm thành công 5 bản ghi; sau đó sử dụng AWS CLI gọi trực tiếp AWS CLI v2 chính thức để Scan bảng và xác nhận `Count = 5`, `ScannedCount = 5`.
<br>&emsp;&emsp;○ Xây dựng `dynamodb_exporter.py` bằng Python/Boto3: kết nối bảng, đọc dữ liệu bằng Scan, xử lý phân trang qua `LastEvaluatedKey`, ghi CSV bằng `csv.DictWriter` và tạo báo cáo JSON.
<br>&emsp;&emsp;○ Bổ sung logging, xử lý lỗi AWS, chuyển đổi kiểu `Decimal` khi ghi JSON và tổ chức thư mục `output` để lưu kết quả xuất.
<br>&emsp;&emsp;○ Chuẩn bị nội dung README và `.gitignore`, đồng thời rà soát nguyên tắc không đưa credentials, `.env` hoặc dữ liệu nhạy cảm lên GitHub.