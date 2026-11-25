---
title: "Worklog Tuần 15"
date: 2026-11-09
weight: 15
chapter: false
pre: " <b> 1.15. </b> "
---

### Mục tiêu tuần 15:
- Tìm hiểu **AWS Step Functions** để tổ chức và điều phối các bước trong hệ thống serverless; phân biệt `Pass`, `Task`, `Choice`, `Succeed` và `Fail` state.
- Tìm hiểu `Retry`, `Catch`, cách truyền dữ liệu qua các state và đọc lịch sử thực thi nhằm phục vụ xử lý lỗi trong workflow.
- Nghiên cứu cơ chế xác thực ứng dụng bằng **Amazon Cognito**, tập trung vào User Pool, App Client, các bước đăng ký, xác nhận tài khoản và đăng nhập.
- Làm quen với **AWS AppSync** và **GraphQL**, hiểu quan hệ giữa schema, resolver và Amazon DynamoDB.
- Tham khảo ba workshop FCJ khác nhau, tách nội dung học tập khỏi các tài nguyên đang vận hành của dự án RakiBookery và lưu ý chi phí khi thực hành.

### Các công việc đã làm trong tuần này:

| Thứ | Công việc | Ngày bắt đầu | Ngày hoàn thành | Nguồn tài liệu |
|:---:|---|:---:|:---:|---|
| 2 |  | 09/11/2026 | 09/11/2026 |  |
| 3 | Tìm hiểu nền tảng AWS Step Functions, Amazon States Language và cách xây dựng workflow gồm các state cơ bản, phân nhánh xử lý. | 10/11/2026 | 10/11/2026 | [FCJ 000047 – AWS Step Functions](https://000047.awsstudygroup.com/vi/) |
| 4 | Nghiên cứu các phần Task State, State Input/Output và xử lý lỗi `Retry`/`Catch`; xem cách kiểm tra Execution History trong Step Functions. | 11/11/2026 | 11/11/2026 | [FCJ – Service Orchestration](https://000047.awsstudygroup.com/vi/3-serviceorchestration/), [FCJ – Error Handling](https://000047.awsstudygroup.com/vi/5-errorhandling/) |
| 5 | Tìm hiểu Amazon Cognito User Pool, App Client và quy trình đăng ký, xác nhận, đăng nhập; phân biệt User Pool và Identity Pool. | 12/11/2026 | 12/11/2026 | [FCJ 000081 – Amazon Cognito](https://000081.awsstudygroup.com/vi/), [Tạo User Pool](https://000081.awsstudygroup.com/vi/2-create-user-pool/) |
| 6 | Tìm hiểu AWS AppSync, GraphQL Schema, Query, Mutation và DynamoDB Resolver; phân tích cách đọc/ghi dữ liệu qua GraphQL API. | 13/11/2026 | 13/11/2026 | [FCJ 000086 – AWS AppSync](https://000086.awsstudygroup.com/), [DynamoDB Resolvers](https://000086.awsstudygroup.com/2-tutorial-dynamodb-resolvers/) |

### Kết quả đạt được tuần 15:

&emsp;◉ **Thứ Ba:**

- Tìm hiểu khái niệm *service orchestration* và vai trò của State Machine khi một yêu cầu cần được xử lý qua nhiều bước.
- Nghiên cứu cấu trúc cơ bản của Amazon States Language (ASL), các state `Pass`, `Choice`, `Task`, `Succeed`, `Fail` và điều kiện chuyển trạng thái.
- Liên hệ với dự án RakiBookery để hình dung một workflow kiểm tra dữ liệu đầu vào trước khi chuyển yêu cầu đến một Lambda thử nghiệm riêng; chưa thay đổi Lambda đang vận hành của dự án.

&emsp;◉ **Thứ Tư:**

- Đi sâu vào `Task` state để hiểu cách Step Functions điều phối Lambda và cách sử dụng `InputPath`, `Parameters`, `ResultPath` khi chuyển dữ liệu giữa các bước.
- Phân tích cơ chế `Retry` cho lỗi tạm thời và `Catch` để đưa workflow sang nhánh xử lý lỗi có kiểm soát.
- Xem quy trình kiểm tra Execution History, phân biệt lỗi của từng state với lỗi làm thất bại toàn bộ workflow; rút ra yêu cầu tách biệt logic xử lý và điều phối.

&emsp;◉ **Thứ Năm:**

- Tìm hiểu dịch vụ Amazon Cognito dành cho ứng dụng web/mobile, đặc biệt là chức năng xác thực và quản lý tài khoản thông qua User Pool.
- Nghiên cứu vai trò của App Client, luồng đăng ký – xác minh – đăng nhập và các token được phát hành sau xác thực.
- Phân biệt User Pool (xác thực người dùng ứng dụng) với Identity Pool (cấp AWS credentials tạm thời khi phù hợp); chú ý không đưa password, client secret và JWT lên repository hoặc ảnh báo cáo.

&emsp;◉ **Thứ Sáu:**

- Làm quen với GraphQL và sự khác biệt so với REST: client có thể xác định trường dữ liệu cần lấy thông qua `Query`.
- Tìm hiểu `Mutation`, GraphQL Schema và cơ chế resolver kết nối AppSync tới bảng DynamoDB.
- Phân tích ví dụ tạo, đọc và cập nhật dữ liệu từ workshop FCJ; xác định các thành phần cần có của một GraphQL API, bao gồm schema, data source, resolver, phương thức authorization và bước kiểm thử truy vấn.