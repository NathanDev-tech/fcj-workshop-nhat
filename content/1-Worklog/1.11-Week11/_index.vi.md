---
title: "Worklog Tuần 11"
date: 2026-10-12
weight: 11
chapter: false
pre: " <b> 1.11. </b> "
---

### Mục tiêu Tuần 11:

- Sử dụng AWS CLI để kiểm tra danh tính AWS, Region và cấu hình profile.
- Ôn lại các khái niệm cơ bản của Amazon DynamoDB và thực hiện các thao tác dữ liệu CRUD.
- Làm quen với AWS Toolkit for Visual Studio Code và Amazon Q trong môi trường phát triển.
- Xây dựng mini project Python/Boto3 để quản lý dữ liệu laptop trên DynamoDB.
- Áp dụng cách quản lý credentials an toàn và kiểm tra, dọn dẹp tài nguyên sau khi thực hành.

### Công việc đã hoàn thành trong tuần:

| Ngày | Công việc | Ngày bắt đầu | Ngày hoàn thành | Tài liệu tham khảo |
|:---:|---|:---:|:---:|---|
| 2 | Ôn lại kiến thức Tuần 10, kiểm tra công cụ và chuẩn bị môi trường thực hành AWS/Python | 12/10/2026 | 12/10/2026 | [AWS Cloud Journey](https://cloudjourney.awsstudygroup.com/vi/) |
| 3 | Tìm hiểu AWS CLI, cách kiểm tra danh tính, Region và profile; làm quen với cách tương tác với AWS qua Terminal | 13/10/2026 | 13/10/2026 | [AWS CLI Workshop](https://000011.awsstudygroup.com/vi/) |
| 4 | Tìm hiểu Amazon DynamoDB, table, item, attribute và partition key; thực hiện các thao tác đọc/ghi/cập nhật/xóa dữ liệu | 14/10/2026 | 14/10/2026 | [Amazon DynamoDB Workshop](https://000060.awsstudygroup.com/vi/) |
| 5 | Làm quen với AWS Toolkit for VS Code, AWS Explorer và cách kết nối tài khoản theo phương thức xác thực được cấp | 15/10/2026 | 15/10/2026 | [AWS Toolkit for VS Code](https://000087.awsstudygroup.com/vi/) |
| 6 | Hoàn thành mini project AWS Laptop Inventory bằng Python, Boto3 và DynamoDB | 16/10/2026 | 16/10/2026 | [DynamoDB Workshop](https://000060.awsstudygroup.com/vi/), [Boto3 Credentials](https://docs.aws.amazon.com/boto3/latest/guide/credentials.html) |

### Kết quả đạt được trong Tuần 11:

&emsp;◉ Thứ Hai:
<br>&emsp;&emsp;○ Ôn lại các nội dung chính của Tuần 10 và chuẩn bị môi trường cho các bài thực hành tiếp theo.
<br>&emsp;&emsp;○ Rà soát công cụ sử dụng, cấu hình làm việc và lưu ý về chi phí trước khi tạo tài nguyên AWS.

&emsp;◉ Thứ Ba:
<br>&emsp;&emsp;○ Tìm hiểu AWS CLI và vai trò của công cụ dòng lệnh trong việc kiểm tra, thao tác với tài nguyên AWS.
<br>&emsp;&emsp;○ Thực hành các lệnh kiểm tra phiên bản công cụ, cấu hình profile/Region và danh tính AWS hiện tại.
<br>&emsp;&emsp;○ Củng cố nguyên tắc không đưa Access Key hoặc Secret Access Key vào mã nguồn, đồng thời sử dụng phương thức xác thực phù hợp với tài khoản được cấp.

&emsp;◉ Thứ Tư:
<br>&emsp;&emsp;○ Tìm hiểu Amazon DynamoDB, mô hình dữ liệu NoSQL và cách tổ chức dữ liệu theo table, item, attribute và partition key.
<br>&emsp;&emsp;○ Làm quen với các thao tác tạo, đọc, cập nhật và xóa dữ liệu; phân biệt mục đích sử dụng của Query và Scan.
<br>&emsp;&emsp;○ Tìm hiểu cách Python kết nối với DynamoDB thông qua AWS SDK for Python (Boto3).

&emsp;◉ Thứ Năm:
<br>&emsp;&emsp;○ Làm quen với AWS Toolkit for Visual Studio Code và AWS Explorer để xem các tài nguyên AWS được hỗ trợ trong môi trường phát triển.
<br>&emsp;&emsp;○ Tìm hiểu cách chọn Region và sử dụng profile/credentials phù hợp khi kết nối VS Code với AWS.
<br>&emsp;&emsp;○ Khám phá cách Amazon Q hỗ trợ giải thích mã và gợi ý khi phát triển, đồng thời cần kiểm tra lại kết quả trước khi áp dụng.

&emsp;◉ Thứ Sáu:
<br>&emsp;&emsp;○ Hoàn thành mini project AWS Laptop Inventory, sử dụng Python, Boto3 và Amazon DynamoDB để quản lý dữ liệu laptop.
<br>&emsp;&emsp;○ Tạo bảng `LaptopInventory` với partition key `laptop_id` kiểu String và kết nối bảng từ chương trình Python.
<br>&emsp;&emsp;○ Xây dựng các chức năng thêm laptop, xem danh sách, tìm theo mã, cập nhật giá/số lượng tồn và xóa laptop.
<br>&emsp;&emsp;○ Bổ sung kiểm tra đầu vào cơ bản, xác nhận trước khi xóa và xử lý một số lỗi AWS thông qua `ClientError`.
<br>&emsp;&emsp;○ Rà soát hướng dẫn dọn dẹp tài nguyên, giữ credentials ngoài source code và chuẩn bị cấu trúc dự án để có thể đưa lên GitHub an toàn.
