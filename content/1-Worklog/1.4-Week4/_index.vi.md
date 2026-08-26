---
title: "Worklog Tuần 4"
date: 2026-08-26
weight: 4
chapter: false
pre: " <b> 1.4. </b> "
---

### Mục tiêu tuần 8:

* Tìm hiểu kiến trúc Data Lake trên AWS và vai trò của các dịch vụ trong quá trình thu thập, lưu trữ, quản lý và phân tích dữ liệu.

### Các công việc đã làm trong tuần này:

| Thứ | Công việc | Ngày bắt đầu | Ngày hoàn thành | Nguồn tài liệu |
|:---:|---|:---:|:---:|---|
| 4 | - Tìm hiểu tổng quan về kiến trúc Data Lake trên AWS.<br>&emsp;+ Khái niệm Data Lake.<br>&emsp;+ Mục đích và lợi ích của Data Lake.<br>&emsp;+ Các loại dữ liệu có thể được lưu trữ trong Data Lake.<br>&emsp;+ Các thành phần chính trong kiến trúc Data Lake.<br>&emsp;+ Quy trình thu thập, lưu trữ, lập danh mục, xử lý và phân tích dữ liệu.<br>&emsp;+ Vai trò của Amazon S3 trong việc lưu trữ dữ liệu thô và dữ liệu đã xử lý.<br>&emsp;+ Vai trò của AWS Glue Crawler trong việc quét dữ liệu và nhận diện schema.<br>&emsp;+ Vai trò của AWS Glue Data Catalog trong việc quản lý metadata.<br>&emsp;+ Cách Amazon Athena sử dụng SQL để truy vấn dữ liệu trên Amazon S3.<br>&emsp;+ Tìm hiểu sơ đồ kiến trúc Data Lake theo các nhóm Ingest, Store, Catalog, Transform và Analyze.<br>&emsp;+ Tìm hiểu cách trình bày sơ đồ kiến trúc bằng các biểu tượng dịch vụ AWS.<br>&emsp;+ Tìm hiểu sơ bộ cách kiểm soát chi phí khi sử dụng S3, Glue và Athena. | 26/08/2026 | 26/08/2026 | [AWS Cloud Journey](https://cloudjourney.awsstudygroup.com/)<br>[Amazon S3 User Guide](https://docs.aws.amazon.com/AmazonS3/latest/userguide/Welcome.html)<br>[AWS Glue Crawler với Amazon Athena](https://docs.aws.amazon.com/athena/latest/ug/schema-crawlers.html) |
| 5 |  | 27/08/2026 | 27/08/2026 |  |
| 6 |  | 28/08/2026 | 28/08/2026 |  |

### Kết quả đạt được tuần 8:

&emsp;◉ Thứ 4:

<br>&emsp;&emsp;○ Hiểu Data Lake là một kho lưu trữ tập trung, có khả năng lưu trữ dữ liệu có cấu trúc, bán cấu trúc và phi cấu trúc.

<br>&emsp;&emsp;○ Hiểu sự khác nhau cơ bản giữa dữ liệu thô và dữ liệu đã qua xử lý trong kiến trúc Data Lake.

<br>&emsp;&emsp;○ Hiểu Amazon S3 đóng vai trò là lớp lưu trữ trung tâm cho dữ liệu thô, dữ liệu đã xử lý và kết quả truy vấn.

<br>&emsp;&emsp;○ Hiểu AWS Glue Crawler có nhiệm vụ quét các file dữ liệu trong Amazon S3, nhận diện cấu trúc và tạo thông tin bảng tự động.

<br>&emsp;&emsp;○ Hiểu AWS Glue Data Catalog lưu trữ metadata như tên bảng, tên cột, kiểu dữ liệu, định dạng file và vị trí dữ liệu.

<br>&emsp;&emsp;○ Hiểu Glue Data Catalog không trực tiếp lưu trữ dữ liệu nghiệp vụ mà chỉ quản lý thông tin mô tả dữ liệu.

<br>&emsp;&emsp;○ Hiểu Amazon Athena cho phép sử dụng câu lệnh SQL để truy vấn dữ liệu trực tiếp trên Amazon S3 mà không cần cài đặt máy chủ cơ sở dữ liệu.

<br>&emsp;&emsp;○ Hiểu luồng hoạt động cơ bản: dữ liệu được lưu trên Amazon S3, Glue Crawler nhận diện schema, Glue Data Catalog quản lý metadata và Athena thực hiện truy vấn.

<br>&emsp;&emsp;○ Biết cách đọc và giải thích sơ đồ Data Lake thông qua các nhóm chức năng Ingest, Store, Catalog, Transform và Analyze.

<br>&emsp;&emsp;○ Hiểu vai trò của các biểu tượng và đường kết nối khi trình bày một sơ đồ kiến trúc AWS.

<br>&emsp;&emsp;○ Hiểu chi phí Amazon Athena phụ thuộc vào lượng dữ liệu được quét và có thể tối ưu bằng cách giới hạn dữ liệu truy vấn.

<br>&emsp;&emsp;○ Hiểu AWS Glue Crawler chỉ nên được chạy khi cần cập nhật cấu trúc dữ liệu nhằm hạn chế chi phí phát sinh.

&emsp;◉ Thứ 5:


&emsp;◉ Thứ 6: