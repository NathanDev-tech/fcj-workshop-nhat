---
title: "Worklog Tuần 16"
date: 2026-11-16
weight: 16
chapter: false
pre: " <b> 1.16. </b> "
---

### Mục tiêu tuần 16:
- Tập trung biên soạn và hoàn thiện **báo cáo kỹ thuật / workshop của dự án RakiBookery**, bảo đảm nội dung phản ánh đúng hạ tầng đã sử dụng và cấu trúc triển khai trên AWS.
- Hoàn thiện cấu hình **Amazon Bedrock Agent** phục vụ định hướng chức năng AI của RakiBookery, kế thừa phần Knowledge Base về nhà sách đã chuẩn bị ở Tuần 14.
- Viết xong **mục 5.2 – Prerequisites** vào Thứ Ba (17/11/2026).
- Hoàn thành **mục 5.3 – Design and Build Infrastructure on AWS** vào Thứ Tư (18/11/2026), theo đúng năm tiểu mục của báo cáo.
- Chuẩn hóa nội dung tài liệu: trình tự thao tác, vai trò của từng dịch vụ, phân quyền cần thiết và mối liên hệ giữa các thành phần; phân biệt việc **viết tài liệu** với việc **tạo mới tài nguyên AWS**.

### Các công việc đã làm trong tuần này:

| Thứ | Công việc | Ngày bắt đầu | Ngày hoàn thành | Nguồn tài liệu |
|:---:|---|:---:|:---:|---|
| 2 | Rà soát nội dung báo cáo dự án RakiBookery, sắp xếp phần Workshop, đối chiếu kiến trúc và tiếp tục hoàn thiện thiết lập Amazon Bedrock Agent. | 16/11/2026 | 16/11/2026 | [Amazon Bedrock Agents – Create and Configure](https://docs.aws.amazon.com/bedrock/latest/userguide/agents-create.html) |
| 3 | Viết và hoàn thiện **5.2 – Prerequisites**, tập trung vào điều kiện chuẩn bị, môi trường AWS và phạm vi IAM permissions của các dịch vụ trước khi triển khai. | 17/11/2026 | 17/11/2026 | [AWS IAM Best Practices](https://docs.aws.amazon.com/IAM/latest/UserGuide/best-practices.html) |
| 4 | Hoàn tất **5.3 – Design and Build Infrastructure on AWS**, gồm IAM, VPC, S3, RDS, ECR Repository và Dockerfile; đồng thời hoàn thiện phần cấu hình Bedrock Agent cho RakiBookery. | 18/11/2026 | 18/11/2026 | [Amazon VPC](https://docs.aws.amazon.com/vpc/latest/userguide/what-is-amazon-vpc.html), [Amazon ECR](https://docs.aws.amazon.com/AmazonECR/latest/userguide/what-is-ecr.html), [Bedrock Agents](https://docs.aws.amazon.com/bedrock/latest/userguide/agents-create.html) |
| 5 |  | 19/11/2026 | 19/11/2026 |  |
| 6 |  | 20/11/2026 | 20/11/2026 |  |

### Kết quả đạt được tuần 16:

&emsp;◉ **Thứ Hai:**

- Tập trung rà soát tài liệu workshop của RakiBookery, kiểm tra cách dẫn dắt từ phần giới thiệu đến các bước chuẩn bị và xây dựng hạ tầng.
- Hệ thống hóa các dịch vụ được đề cập trong báo cáo: IAM, VPC, S3, RDS MySQL, Amazon ECR và Dockerfile, đồng thời đối chiếu với các phần AI đã triển khai trước đó bằng Lambda và Amazon Bedrock.
- Tiếp tục hoàn thiện thiết lập **Amazon Bedrock Agent**, rà soát các thành phần cấu hình phù hợp cho trợ lý AI của nhà sách; phần Knowledge Base về thông tin RakiBookery là nền tảng dữ liệu đã được chuẩn bị từ Tuần 14.
- Chỉnh lý cách trình bày để các phần Workshop có thứ tự, tên mục và nội dung nhất quán giữa các bước.

&emsp;◉ **Thứ Ba:**

- Viết và hoàn thành **5.2 – Prerequisites** cho báo cáo RakiBookery.
- Trình bày các yêu cầu cần kiểm tra trước khi thực hiện workshop: môi trường AWS, Region làm việc, quyền truy cập cần thiết, tình trạng tài nguyên hiện có và dữ liệu/cấu hình liên quan.
- Chú trọng **IAM permissions ở cấp dịch vụ/role/policy**, giải thích phạm vi quyền để các thành phần ứng dụng tương tác an toàn với nhau; không đồng nhất nội dung này với thao tác tạo IAM User hoặc Group.
- Kiểm tra nội dung 5.2 để không hướng dẫn tạo trùng những tài nguyên AWS đã có trong dự án; đồng thời bảo đảm đây là bước chuẩn bị cho mục 5.3.

&emsp;◉ **Thứ Tư:**

- **Hoàn thành phần 5.3 – Design and Build Infrastructure on AWS** trong tài liệu workshop, gồm đúng các tiểu mục:
  - **5.3.1 Initialize and Configure IAM:** vai trò của IAM và quyền các dịch vụ cần sử dụng.
  - **5.3.2 Initialize and Configure VPC:** phân chia mạng, subnet, routing và các yêu cầu kết nối.
  - **5.3.3 Initialize and Configure S3:** lưu trữ object và các tệp phục vụ ứng dụng.
- Hoàn thiện phần cấu hình **Amazon Bedrock Agent** theo phạm vi dự án, làm rõ mục đích sử dụng Agent trong hệ thống gợi ý/hỗ trợ nhà sách và mối liên hệ với dữ liệu đã chuẩn bị ở Knowledge Base.
- Rà soát thuật ngữ, tiêu đề và trình tự thao tác của phần báo cáo để người đọc có thể theo dõi từ khâu chuẩn bị đến xây dựng hạ tầng.