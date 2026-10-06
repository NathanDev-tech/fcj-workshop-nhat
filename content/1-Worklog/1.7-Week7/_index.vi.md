---

title: "Week 7 Worklog"

date: 2026-08-17

weight: 7

chapter: false

pre: " <b> 1.7. </b> "

---

### Mục tiêu Tuần 7:

- Hiểu AWS Lambda và mô hình Serverless.
- Ôn tập các nội dung AWS Cloud Practitioner CLF-C02 bằng ExamPro.
- Hiểu Amazon SQS và các khái niệm về message queue.
- Bắt đầu làm quen với Grafana và hiểu các chức năng cơ bản về giám sát và trực quan hóa dữ liệu.
- Tìm hiểu cách sử dụng IAM User, IAM Group và IAM Policy để quản lý quyền truy cập vào các AWS resources.
- Tìm hiểu AWS Secrets Manager và cách sử dụng dịch vụ để lưu trữ an toàn các thông tin xác thực và cấu hình nhạy cảm của dự án.
- Thực hành các dịch vụ AWS thông qua các workshop và liên hệ kiến thức với môi trường AWS thực tế.
- Áp dụng kiến thức AWS vào dự án đầu tiên mang tên RakiBookery.

### Công việc đã hoàn thành trong tuần:

| Ngày | Công việc | Ngày bắt đầu | Ngày hoàn thành | Tài liệu tham khảo |
|:---:|---|:---:|:---:|---|
| 2 | Tìm hiểu AWS Lambda và mô hình Serverless; cấu hình IAM Role, tạo Lambda Function và thực hành tự động hóa Start/Stop EC2 nhằm hỗ trợ tối ưu chi phí | 14/09/2026 | 14/09/2026 | [Optimize EC2 cost with Lambda](https://000022.awsstudygroup.com/) |
| 3 | Ôn tập AWS Certified Cloud Practitioner CLF-C02 bằng ExamPro; tìm hiểu Amazon SQS, Queue, Message, Producer và Consumer, đồng thời thực hành gửi và nhận message | 15/09/2026 | 15/09/2026 | [ExamPro CLF-C02](https://www.exampro.co/clf-c02) |
| 4 | Tìm hiểu Getting Started with Grafana Basic; thực hành triển khai và truy cập Grafana trên EC2, tạo IAM User và Group, gắn IAM Policy phù hợp và cấu hình quyền truy cập AWS Secrets Manager cho dự án RakiBookery | 16/09/2026 | 16/09/2026 | [Getting started with Grafana basic](https://000029.awsstudygroup.com/) |
| 5 |  | 17/09/2026 | 17/09/2026 |  |
| 6 |  | 18/09/2026 | 18/09/2026 |  |

### Kết quả đạt được trong Tuần 7:

&emsp;◉ Thứ Hai:
<br>&emsp;&emsp;○ Tìm hiểu AWS Lambda và mô hình Serverless.
<br>&emsp;&emsp;○ Tìm hiểu cách AWS Lambda thực thi code mà không yêu cầu tự quản lý trực tiếp máy chủ.
<br>&emsp;&emsp;○ Ôn lại mô hình thực thi theo event của Lambda Function.
<br>&emsp;&emsp;○ Tạo và cấu hình IAM Role cho Lambda, đồng thời tìm hiểu cách cấp quyền cần thiết cho function.
<br>&emsp;&emsp;○ Tạo Lambda Function và cấu hình để thực hiện các tác vụ vận hành trên EC2 instance.
<br>&emsp;&emsp;○ Thực hành sử dụng Lambda để tự động hóa việc Start và Stop EC2.
<br>&emsp;&emsp;○ Kiểm tra kết quả thực thi của Lambda Function và xem cách function tương tác với EC2.
<br>&emsp;&emsp;○ Tìm hiểu cách Lambda có thể giảm nhu cầu duy trì hạ tầng hoạt động liên tục và hỗ trợ tối ưu chi phí EC2.

&emsp;◉ Thứ Ba:
<br>&emsp;&emsp;○ Ôn tập các nội dung AWS Certified Cloud Practitioner CLF-C02 bằng ExamPro.
<br>&emsp;&emsp;○ Ôn tập Application Integration, Messaging, Serverless, Security, Shared Responsibility Model và các trường hợp sử dụng của AWS services.
<br>&emsp;&emsp;○ Tìm hiểu Amazon SQS và vai trò của SQS như một dịch vụ message queue.
<br>&emsp;&emsp;○ Tìm hiểu các khái niệm Queue, Message, Producer và Consumer.
<br>&emsp;&emsp;○ Tìm hiểu cách Producer gửi message vào Queue và Consumer lấy message để xử lý.
<br>&emsp;&emsp;○ Tìm hiểu cách Amazon SQS giúp tách các thành phần của ứng dụng và hỗ trợ xử lý message bất đồng bộ.
<br>&emsp;&emsp;○ Thực hành tạo và sử dụng SQS Queue.
<br>&emsp;&emsp;○ Thực hành gửi message vào Queue và nhận message để xử lý.
<br>&emsp;&emsp;○ Ôn lại vai trò của Standard Queue trong xử lý message có khả năng mở rộng và hiểu rằng thứ tự message không được đảm bảo tuyệt đối.

&emsp;◉ Thứ Tư:
<br>&emsp;&emsp;○ Tìm hiểu workshop Getting Started with Grafana Basic và làm quen với Grafana cùng giao diện cơ bản.
<br>&emsp;&emsp;○ Tìm hiểu mục đích cơ bản của Grafana trong việc monitoring và data visualization.
<br>&emsp;&emsp;○ Thực hành chuẩn bị môi trường EC2 để triển khai Grafana và tìm hiểu các bước cơ bản cần thiết để truy cập Grafana.
<br>&emsp;&emsp;○ Thực hành kết nối đến EC2 và kiểm tra các cấu hình cần thiết cho môi trường Grafana.
<br>&emsp;&emsp;○ Làm việc với IAM User và IAM Group để tổ chức quyền truy cập cho các thành viên trong dự án RakiBookery.
<br>&emsp;&emsp;○ Tạo IAM User cho các thành viên tham gia dự án và tổ chức các user vào IAM Group phù hợp.
<br>&emsp;&emsp;○ Gắn các IAM Policy phù hợp vào IAM Group để cấp các quyền cần thiết theo vai trò của từng thành viên.
<br>&emsp;&emsp;○ Tìm hiểu mối quan hệ giữa IAM User, IAM Group và IAM Policy trong việc quản lý quyền truy cập AWS resources.
<br>&emsp;&emsp;○ Cấu hình AWS Secrets Manager cho dự án RakiBookery để lưu trữ an toàn các thông tin xác thực và thông tin cấu hình nhạy cảm.
<br>&emsp;&emsp;○ Cấu hình quyền IAM phù hợp để các user được ủy quyền có thể truy cập các secret cần thiết mà không phải đưa trực tiếp thông tin nhạy cảm vào source code.
<br>&emsp;&emsp;○ Nhận thức rõ hơn về việc tách thông tin xác thực khỏi source code và sử dụng cơ chế quản lý secret tập trung để tăng tính bảo mật cho dự án.
<br>&emsp;&emsp;○ Thực hành truy cập môi trường Grafana và tìm hiểu vai trò của port 3000 khi kết nối đến giao diện web Grafana.
<br>&emsp;&emsp;○ Xây dựng nền tảng kiến thức ban đầu về monitoring, visualization, quản lý quyền truy cập bằng IAM và quản lý thông tin cấu hình an toàn trên AWS.