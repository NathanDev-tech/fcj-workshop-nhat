---
title: "Worklog Tuần 10 "
date: 2026-10-05
weight: 10
chapter: false
pre: " <b> 1.10. </b> "
---

### Mục tiêu Tuần 10:

- Hiểu AWS CDK và mối quan hệ giữa CDK với CloudFormation.
- Thực hành định nghĩa hạ tầng AWS bằng code thông qua CDK.
- Tìm hiểu cách CDK Advanced tổ chức architecture gồm nhiều AWS services.
- Làm quen với Amazon Bedrock, Foundation Models, prompt và inference.
- Củng cố kiến thức IAM User, IAM Group, IAM Policy và IAM Role.
- Tìm hiểu Assume Role, trust policy và cách sử dụng Condition để kiểm soát truy cập.
- Liên hệ Infrastructure as Code, Generative AI và IAM Security trong môi trường AWS.

### Công việc đã hoàn thành trong tuần:

| Ngày | Công việc | Ngày bắt đầu | Ngày hoàn thành | Tài liệu tham khảo |
|:---:|---|:---:|:---:|---|
| 2 |  | 05/10/2026 | 05/10/2026 |  |
| 3 |  | 06/10/2026 | 06/10/2026 |  |
| 4 | Tìm hiểu AWS CDK Essentials; chuẩn bị workspace, tạo và cập nhật CDK template, tìm hiểu CDK Stack và CloudFormation; thực hành Amazon Bedrock Text Playground với Foundation Model và prompt cơ bản | 07/10/2026 | 07/10/2026 | [AWS CDK Essentials](https://000038.awsstudygroup.com/vi/)<br>[Amazon Bedrock](https://docs.aws.amazon.com/bedrock/)<br>[Bedrock Console Guide](https://docs.aws.amazon.com/bedrock/latest/userguide/getting-started-console.html) |
| 5 | Tìm hiểu AWS CDK Advanced; nghiên cứu cách mô tả architecture gồm API Gateway, ALB, ECS, Lambda và S3 bằng CDK; tìm hiểu Nested Stack và quy trình triển khai/cleanup | 08/10/2026 | 08/10/2026 | [AWS CDK Advanced](https://000076.awsstudygroup.com/vi/) |
| 6 | Tìm hiểu IAM Role & Condition; thực hành IAM Group, User, Role và Assume Role; tìm hiểu trust policy, least privilege và điều kiện giới hạn truy cập theo IP/thời gian | 09/10/2026 | 09/10/2026 | [IAM Role & Condition](https://000044.awsstudygroup.com/vi/) |

### Kết quả đạt được trong Tuần 10:

&emsp;◉ Thứ Tư:
<br>&emsp;&emsp;○ Tìm hiểu AWS Cloud Development Kit (CDK) và mối quan hệ giữa CDK với CloudFormation.
<br>&emsp;&emsp;○ Chuẩn bị workspace cho CDK và tìm hiểu cấu trúc cơ bản của một CDK application.
<br>&emsp;&emsp;○ Tìm hiểu cách định nghĩa hạ tầng AWS bằng code, tạo CDK template và cập nhật template khi cấu hình thay đổi.
<br>&emsp;&emsp;○ Ôn lại mối liên hệ giữa CDK application, Stack, CloudFormation template và AWS resources.
<br>&emsp;&emsp;○ Làm quen với Amazon Bedrock và các khái niệm Foundation Model, prompt và inference.
<br>&emsp;&emsp;○ Truy cập Bedrock Console, chọn một model khả dụng và thử prompt trong Text Playground để quan sát phản hồi được tạo ra.
<br>&emsp;&emsp;○ Liên hệ khả năng sử dụng Foundation Models với việc xây dựng ứng dụng Generative AI trên AWS.

&emsp;◉ Thứ Năm:
<br>&emsp;&emsp;○ Tiếp tục workshop AWS CDK Advanced sau khi tìm hiểu CDK Essentials.
<br>&emsp;&emsp;○ Tìm hiểu cách định nghĩa một architecture nhiều dịch vụ AWS bằng CDK thay vì cấu hình thủ công từng resource.
<br>&emsp;&emsp;○ Ôn lại vai trò của API Gateway trong việc tiếp nhận request và ALB trong việc phân phối traffic đến application targets.
<br>&emsp;&emsp;○ Tìm hiểu vai trò của ECS trong việc chạy containerized applications và cách các thành phần này được kết hợp trong architecture.
<br>&emsp;&emsp;○ Xem xét các thành phần bổ sung như Lambda và S3 trong kiến trúc được mô tả bởi workshop.
<br>&emsp;&emsp;○ Tìm hiểu Nested Stack và cách chia hạ tầng thành các stack nhỏ hơn để dễ tổ chức và quản lý.
<br>&emsp;&emsp;○ Ôn lại quy trình triển khai bằng CDK và vai trò của CloudFormation trong quá trình tạo AWS resources.
<br>&emsp;&emsp;○ Kiểm tra và dọn dẹp các tài nguyên được tạo trong quá trình thực hành.

&emsp;◉ Thứ Sáu:

<br>&emsp;&emsp;○ Củng cố kiến thức IAM User, IAM Group, IAM Policy và IAM Role.
<br>&emsp;&emsp;○ Tìm hiểu quy trình Assume Role, vai trò của AWS STS và việc sử dụng temporary credentials.
<br>&emsp;&emsp;○ Tìm hiểu sự khác nhau giữa identity policy và trust policy khi kiểm soát việc sử dụng IAM Role.
<br>&emsp;&emsp;○ Thực hành tổ chức IAM User vào Group, kiểm tra quyền được cấp và tìm hiểu cách chuyển sang Role bằng Switch Role.
<br>&emsp;&emsp;○ Tìm hiểu cách sử dụng Condition trong trust policy để giới hạn việc assume role theo địa chỉ IP và thời gian.
<br>&emsp;&emsp;○ Liên hệ các điều kiện trong policy với nguyên tắc least privilege và yêu cầu bảo mật khi quản trị tài khoản AWS.
<br>&emsp;&emsp;○ Kiểm tra các quyền đã cấu hình và dọn dẹp IAM resources sau khi hoàn thành workshop.
