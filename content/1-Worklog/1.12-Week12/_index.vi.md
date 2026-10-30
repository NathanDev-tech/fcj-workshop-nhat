---
title: "Worklog Tuần 12"
date: 2026-10-19
weight: 12
chapter: false
pre: " <b> 1.12. </b> "
---

### Mục tiêu Tuần 12:

- Ghi nhận việc setup Amazon Bedrock thông qua AWS Lambda cho dự án RakiBookery.
- Tìm hiểu vai trò của AWS WAF trong việc bảo vệ ứng dụng web thông qua Web ACL và các nhóm rule.
- Nắm các khái niệm cơ bản của Amazon ElastiCache for Redis như cache, key/value, TTL, cache hit và cache miss.
- Tìm hiểu Amazon CloudFront, origin, cache behavior và cách phân phối nội dung tĩnh được lưu trên Amazon S3.
- Thiết kế sơ đồ kiến trúc khái niệm cho web app quản lý laptop, thể hiện riêng frontend, API/backend và DynamoDB.
- Phân biệt nội dung thiết kế kiến trúc với việc triển khai tài nguyên AWS thực tế; chú ý quyền truy cập và chi phí.

### Công việc đã hoàn thành trong tuần:

| Ngày | Công việc | Ngày bắt đầu | Ngày hoàn thành | Tài liệu tham khảo |
|:---:|---|:---:|:---:|---|
| 2 |  | 19/10/2026 | 19/10/2026 |  |
| 3 | Setup Amazon Bedrock thông qua AWS Lambda cho dự án RakiBookery | 20/10/2026 | 20/10/2026 | [Amazon Bedrock](https://docs.aws.amazon.com/bedrock/), [AWS Lambda](https://docs.aws.amazon.com/lambda/latest/dg/welcome.html) |
| 4 | Tìm hiểu AWS WAF, Web ACL, managed rule groups và custom rules ở mức kiến thức và thiết kế kiểm soát request | 21/10/2026 | 21/10/2026 | [AWS WAF Workshop](https://000026.awsstudygroup.com/vi/) |
| 5 | Tìm hiểu Amazon ElastiCache for Redis, caching, key/value, TTL, cache hit/miss và vai trò bổ trợ cho DynamoDB | 22/10/2026 | 22/10/2026 | [ElastiCache Workshop](https://000061.awsstudygroup.com/vi/), [Amazon ElastiCache](https://docs.aws.amazon.com/AmazonElastiCache/latest/dg/WhatIs.html) |
| 6 | Tìm hiểu CloudFront và thiết kế sơ đồ kiến trúc khái niệm cho web app quản lý laptop với S3, API/backend và DynamoDB | 23/10/2026 | 23/10/2026 | [CloudFront with S3 Workshop](https://000094.awsstudygroup.com/vi/), [Amazon CloudFront](https://docs.aws.amazon.com/AmazonCloudFront/latest/DeveloperGuide/Introduction.html) |

### Kết quả đạt được trong Tuần 12:

&emsp;◉ Thứ Ba:

<br>&emsp;&emsp;○ Hoàn thành việc setup Amazon Bedrock thông qua AWS Lambda cho dự án RakiBookery.
<br>&emsp;&emsp;○ Củng cố cách kết hợp dịch vụ AI của AWS với một hàm Lambda làm lớp xử lý trung gian cho ứng dụng.

&emsp;◉ Thứ Tư:

<br>&emsp;&emsp;○ Tìm hiểu AWS WAF và vai trò của Web Application Firewall trong việc kiểm soát request gửi đến ứng dụng web.
<br>&emsp;&emsp;○ Phân biệt Web ACL, managed rule groups và custom rules; xem xét cách cho phép hoặc chặn request theo điều kiện đã cấu hình.
<br>&emsp;&emsp;○ Ghi nhận AWS WAF là lớp bảo vệ cần được cấu hình phù hợp với điểm phân phối ứng dụng; không xem việc đọc workshop là bằng chứng đã triển khai WAF lên môi trường thật.

&emsp;◉ Thứ Năm:

<br>&emsp;&emsp;○ Tìm hiểu Amazon ElastiCache for Redis và mục đích sử dụng cache để lưu tạm dữ liệu thường xuyên được truy cập.
<br>&emsp;&emsp;○ Ôn các khái niệm key/value, TTL, cache hit và cache miss; hiểu rằng dữ liệu trong cache không thay thế nguồn dữ liệu chính.
<br>&emsp;&emsp;○ Liên hệ vai trò của ElastiCache với DynamoDB: DynamoDB tiếp tục lưu dữ liệu chính, còn Redis có thể hỗ trợ giảm các lần truy vấn lặp lại khi workload phù hợp.
<br>&emsp;&emsp;○ Lưu ý không tạo ElastiCache cluster chỉ để thử nghiệm khi chưa xác nhận quyền và ngân sách, do tài nguyên có thể phát sinh chi phí.

&emsp;◉ Thứ Sáu:
<br>&emsp;&emsp;○ Tìm hiểu Amazon CloudFront, CDN, distribution, origin, cache behavior, TTL và invalidation.
<br>&emsp;&emsp;○ Phân biệt Amazon S3 dùng để lưu trữ frontend tĩnh với CloudFront dùng để phân phối và cache nội dung đến người dùng.
<br>&emsp;&emsp;○ Thiết kế sơ đồ khái niệm cho web app quản lý laptop, mô tả luồng người dùng → CloudFront/WAF → frontend tĩnh trên S3 hoặc API/backend → DynnamoDB.
<br>&emsp;&emsp;○ Thể hiện rõ frontend, API/backend và cơ sở dữ liệu là các thành phần riêng biệt; backend là lớp xử lý nghiệp vụ và thực hiện đọc/ghi DynamoDB.
<br>&emsp;&emsp;○ Hoàn thiện hình sơ đồ và ghi chú vai trò của các dịch vụ AWS. Sơ đồ là thiết kế đề xuất, không phải bằng chứng cho thấy CloudFront, WAF, S3 hay backend đã được triển khai thực tế.

![AWS](/fcj-workshop-nhat/images/AWS_LAPTOP.png)