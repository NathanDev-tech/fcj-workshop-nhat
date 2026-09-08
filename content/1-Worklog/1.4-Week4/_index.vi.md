---
title: "Worklog Tuần 4"
date: 2026-08-26
weight: 4
chapter: false
pre: " <b> 1.4. </b> "
---

### Mục tiêu tuần 4:

* Tìm hiểu kiến trúc Data Lake trên AWS và vai trò của các dịch vụ trong quá trình thu thập, lưu trữ, quản lý và phân tích dữ liệu.

### Các công việc đã làm trong tuần này:

| Thứ | Công việc | Ngày bắt đầu | Ngày hoàn thành | Tài liệu tham khảo |
|:---:|---|:---:|:---:|---|
| 4 | - Tìm hiểu tổng quan về kiến trúc Data Lake trên AWS.<br>&emsp;+ Khái niệm Data Lake.<br>&emsp;+ Mục đích và lợi ích của Data Lake.<br>&emsp;+ Các loại dữ liệu có thể được lưu trữ trong Data Lake.<br>&emsp;+ Các thành phần chính trong kiến trúc Data Lake.<br>&emsp;+ Quy trình thu thập, lưu trữ, lập danh mục, xử lý và phân tích dữ liệu.<br>&emsp;+ Vai trò của Amazon S3 trong việc lưu trữ dữ liệu thô và dữ liệu đã xử lý.<br>&emsp;+ Vai trò của AWS Glue Crawler trong việc quét dữ liệu và nhận diện schema.<br>&emsp;+ Vai trò của AWS Glue Data Catalog trong việc quản lý metadata.<br>&emsp;+ Cách Amazon Athena sử dụng SQL để truy vấn dữ liệu trên Amazon S3.<br>&emsp;+ Tìm hiểu sơ đồ kiến trúc Data Lake theo các nhóm Ingest, Store, Catalog, Transform và Analyze.<br>&emsp;+ Tìm hiểu cách trình bày sơ đồ kiến trúc bằng các biểu tượng dịch vụ AWS.<br>&emsp;+ Tìm hiểu sơ bộ cách kiểm soát chi phí khi sử dụng S3, Glue và Athena. | 26/08/2026 | 26/08/2026 | [AWS Cloud Journey](https://cloudjourney.awsstudygroup.com/)<br>[Amazon S3 User Guide](https://docs.aws.amazon.com/AmazonS3/latest/userguide/Welcome.html)<br>[AWS Glue Crawler với Amazon Athena](https://docs.aws.amazon.com/athena/latest/ug/schema-crawlers.html) |
| Thứ 5 | - Hoàn thành triển khai và thực hành workshop AWS Data Lake on AWS.<br>&emsp;+ Ôn tập lại toàn bộ quy trình hoạt động của Data Lake từ bước thu thập dữ liệu đến bước phân tích dữ liệu.<br>&emsp;+ Thực hành tạo và quản lý Amazon S3 storage trong kiến trúc Data Lake.<br>&emsp;+ Tìm hiểu cách Amazon Kinesis Data Firehose thu thập dữ liệu streaming và đưa dữ liệu vào Amazon S3.<br>&emsp;+ Thực hành cấu hình luồng dữ liệu từ Kinesis Data Firehose đến Amazon S3.<br>&emsp;+ Tìm hiểu cách AWS Glue Crawler phân tích dữ liệu được lưu trữ trên Amazon S3 và tự động phát hiện schema.<br>&emsp;+ Tìm hiểu cách AWS Glue Data Catalog lưu trữ metadata bao gồm table, schema và vị trí dữ liệu.<br>&emsp;+ Sử dụng Amazon Athena để kiểm tra và truy vấn dữ liệu được lưu trữ trên Amazon S3 bằng SQL.<br>&emsp;+ Hiểu rõ mối quan hệ giữa Amazon S3, Kinesis Data Firehose, AWS Glue và Amazon Athena trong kiến trúc Data Lake.<br>&emsp;+ Kiểm tra lại toàn bộ tài nguyên AWS được tạo trong quá trình thực hành lab.<br>&emsp;+ Thực hiện cleanup và xóa toàn bộ tài nguyên sau khi hoàn thành workshop để tránh phát sinh chi phí.<br>&emsp;+ Tiếp tục với khóa AWS Certified Cloud Practitioner CLF-C02.<br>&emsp;+ Học về AWS Global Infrastructure bao gồm AWS Region, Availability Zone và Edge Location.<br>&emsp;+ Tìm hiểu cách AWS Global Infrastructure giúp tăng khả năng sẵn sàng, khả năng mở rộng và giảm độ trễ cho ứng dụng cloud.<br>&emsp;+ Xem video tổng hợp AWS Certified Cloud Practitioner CLF-C02 của ExamPro để củng cố kiến thức chuẩn bị cho kỳ thi. | 27/08/2026 | 27/08/2026 | [AWS Cloud Journey](https://cloudjourney.awsstudygroup.com/)<br>[Data Lake on AWS Workshop](https://000035.awsstudygroup.com/)<br>[AWS Certified Cloud Practitioner CLF-C02 - ExamPro Full Course](https://www.youtube.com/watch?v=7HKot-brXFE) |
| Thứ 6 | - Phân tích dữ liệu Data Lake được tạo từ workshop First Cloud Journey bằng Amazon Athena SQL.<br>&emsp;+ Kiểm tra resource inventory và xác nhận pipeline dữ liệu được tạo từ Thứ 5 đã sẵn sàng cho quá trình phân tích.<br>&emsp;+ Ôn lại mối liên hệ giữa Amazon S3, AWS Glue Data Catalog và Amazon Athena trong quy trình truy vấn dữ liệu.<br>&emsp;+ Khám phá schema của dữ liệu thông qua Glue Catalog bằng các câu lệnh Athena như SHOW TABLES và DESCRIBE.<br>&emsp;+ Thực hiện các câu truy vấn SQL để phân tích dữ liệu được lưu trữ trong Amazon S3 thông qua Athena.<br>&emsp;+ Thực hiện kiểm tra chất lượng dữ liệu bao gồm kiểm tra giá trị NULL, UUID trùng lặp và các giá trị activity không hợp lệ.<br>&emsp;+ Phân tích các chỉ số như tổng số event, số lượng thiết bị khác nhau, phân bố activity và các track có lượt phát cao nhất.<br>&emsp;+ Tìm hiểu cách chi phí Amazon Athena phụ thuộc vào lượng dữ liệu được quét và cách tối ưu câu truy vấn để giảm chi phí.<br>&emsp;+ Tìm hiểu CTAS (CREATE TABLE AS SELECT) để tạo dữ liệu processed dưới định dạng Parquet giúp tăng hiệu quả truy vấn.<br>&emsp;+ Hiểu sự khác biệt giữa raw data và processed data trong kiến trúc Data Lake.<br>&emsp;+ Thực hiện audit bằng AWS CloudTrail để tìm hiểu cách AWS ghi nhận hoạt động API và thay đổi tài nguyên.<br>&emsp;+ Kiểm tra thông tin Billing/Cost Management sau khi hoàn thành lab.<br>&emsp;+ Cleanup toàn bộ tài nguyên AWS được tạo trong workshop bao gồm Firehose, Glue, CloudFormation, IAM và dữ liệu trên S3.<br>&emsp;+ Hoàn thiện tổng kết tuần 8 và ghi nhận kết quả thực hành Data Lake trên AWS. | 28/08/2026 | 28/08/2026 | [Athena SQL, Audit, Cleanup và Tổng kết]<br>[AWS CloudTrail Documentation](https://docs.aws.amazon.com/awscloudtrail/latest/userguide/view-cloudtrail-events.html)<br>[Amazon Athena Documentation](https://docs.aws.amazon.com/athena/latest/ug/getting-started.html) |

### Kết quả đạt được tuần 4:

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

{{< figure src="images/Week8_Wednesday_S3_DataLake.png" title="Hình 1: Kiến trúc Data Lake trên AWS với Amazon S3, AWS Glue và Amazon Athena" width="850" >}}

&emsp;◉ Thứ 5:
<br>&emsp;&emsp;○ Hoàn thành workshop AWS Data Lake on AWS và hiểu được quy trình xây dựng một hệ thống Data Lake cơ bản trên AWS.
<br>&emsp;&emsp;○ Hiểu rằng Amazon S3 đóng vai trò là tầng lưu trữ trung tâm trong kiến trúc Data Lake.
<br>&emsp;&emsp;○ Hiểu cách Amazon Kinesis Data Firehose được sử dụng để thu thập dữ liệu streaming và đưa dữ liệu vào Amazon S3.
<br>&emsp;&emsp;○ Hiểu rằng AWS Glue Crawler có nhiệm vụ quét dữ liệu, phát hiện cấu trúc dữ liệu và tạo metadata thay vì lưu trữ dữ liệu thực tế.
<br>&emsp;&emsp;○ Hiểu vai trò của AWS Glue Data Catalog trong việc quản lý metadata phục vụ cho các dịch vụ phân tích dữ liệu.
<br>&emsp;&emsp;○ Hiểu cách Amazon Athena thực hiện truy vấn SQL trực tiếp trên dữ liệu lưu trữ trong Amazon S3 mà không cần quản lý database server.
<br>&emsp;&emsp;○ Hiểu cách các dịch vụ AWS kết hợp với nhau để xây dựng một kiến trúc Data Lake hoàn chỉnh.
<br>&emsp;&emsp;○ Hoàn thành việc kiểm tra và xóa toàn bộ tài nguyên AWS được tạo trong quá trình thực hành lab nhằm tránh phát sinh chi phí không cần thiết.
<br>&emsp;&emsp;○ Tiếp tục quá trình học và chuẩn bị cho chứng chỉ AWS Certified Cloud Practitioner CLF-C02.
<br>&emsp;&emsp;○ Hiểu các khái niệm cơ bản trong AWS Global Infrastructure gồm Region, Availability Zone và Edge Location.
<br>&emsp;&emsp;○ Hiểu cách AWS thiết kế hạ tầng toàn cầu nhằm đảm bảo tính sẵn sàng cao, khả năng chịu lỗi và khả năng truy cập với độ trễ thấp.
<br>&emsp;&emsp;○ Nâng cao khả năng hiểu và phân tích các khái niệm hạ tầng AWS phục vụ cho việc thiết kế các giải pháp cloud đáng tin cậy.

&emsp;◉ Thứ 6:
<br>&emsp;&emsp;○ Hoàn thành phân tích dữ liệu Data Lake bằng Amazon Athena SQL sau khi triển khai workshop AWS Data Lake.
<br>&emsp;&emsp;○ Hiểu cách Amazon Athena sử dụng metadata từ AWS Glue Data Catalog để truy vấn dữ liệu trên Amazon S3.
<br>&emsp;&emsp;○ Biết cách kiểm tra schema và cấu trúc bảng trước khi thực hiện truy vấn SQL.
<br>&emsp;&emsp;○ Thực hiện phân tích dữ liệu bằng SQL bao gồm thống kê event, phân tích activity và tìm kiếm dữ liệu nổi bật.
<br>&emsp;&emsp;○ Hiểu các bước kiểm tra chất lượng dữ liệu cơ bản như kiểm tra NULL, dữ liệu trùng lặp và giá trị không hợp lệ.
<br>&emsp;&emsp;○ Hiểu rằng chi phí Athena phụ thuộc vào lượng dữ liệu được scan và biết cách tối ưu câu truy vấn.
<br>&emsp;&emsp;○ Hiểu cách sử dụng CTAS để tạo lớp dữ liệu processed với định dạng Parquet.
<br>&emsp;&emsp;○ Phân biệt được raw data và processed data trong kiến trúc Data Lake.
<br>&emsp;&emsp;○ Hiểu cách AWS CloudTrail hỗ trợ audit và theo dõi các hoạt động trên AWS.
<br>&emsp;&emsp;○ Kiểm tra chi phí sau lab và hoàn thành cleanup toàn bộ tài nguyên AWS.
<br>&emsp;&emsp;○ Hoàn thiện kiến thức về toàn bộ luồng Data Lake: Ingestion → Storage → Catalog → Query → Insight.