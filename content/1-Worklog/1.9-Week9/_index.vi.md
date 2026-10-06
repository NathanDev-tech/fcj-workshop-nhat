---
title: "Worklog Tuần 9"
date: 2026-09-28
weight: 9
chapter: false
pre: " <b> 1.9. </b> "
---

### Mục tiêu tuần 9:
- Hiểu cách tối ưu kích thước EC2 dựa trên dữ liệu sử dụng thực tế.
- Tìm hiểu CloudWatch Agent và cách thu thập các metric phục vụ việc phân tích EC2.
- Hiểu AWS Compute Optimizer và vai trò của các đề xuất tối ưu tài nguyên.
- Hiểu AWS Service Quotas và cách kiểm tra giới hạn của các AWS service.
- Tìm hiểu quy trình yêu cầu tăng quota khi nhu cầu sử dụng vượt quá giới hạn hiện tại.
- Hiểu cách sử dụng IAM Policy để kiểm soát việc sử dụng tài nguyên AWS.
- Thực hành giới hạn quyền theo Region, EC2 family, instance size và EBS volume.
- Liên hệ kiến thức Monitoring, Resource Optimization, IAM và Cost Management trong môi trường AWS.

### Các công việc đã làm trong tuần này:

### Công việc đã hoàn thành trong tuần:

| Ngày | Công việc | Ngày bắt đầu | Ngày hoàn thành | Tài liệu tham khảo |
|:---:|---|:---:|:---:|---|
| 2 |  | 28/09/2026 | 28/09/2026 |  |
| 3 |  | 29/09/2026 | 29/09/2026 |  |
| 4 | Tìm hiểu EC2 Right-Sizing; tạo IAM Role, gắn Role cho EC2, cài đặt và cấu hình CloudWatch Agent, thu thập metric và tìm hiểu đề xuất tối ưu từ AWS Compute Optimizer | 30/09/2026 | 30/09/2026 | [EC2 Right-Sizing](https://000032.awsstudygroup.com/vi/) |
| 5 | Tìm hiểu AWS Service Quotas; kiểm tra quota của AWS service, xem giới hạn hiện tại và thực hành tìm hiểu quy trình yêu cầu tăng quota khi cần thêm tài nguyên | 01/10/2026 | 01/10/2026 | [AWS Service Quotas](https://000063.awsstudygroup.com/vi/) |
| 6 | Tìm hiểu Cost & Usage Management với IAM; tạo IAM Group, User và Policy, thực hành giới hạn quyền theo Region, EC2 Family, Instance Size và EBS Volume, sau đó kiểm tra kết quả của Policy | 02/10/2026 | 02/10/2026 | [Cost and Usage Management](https://000064.awsstudygroup.com/vi/) |

### Kết quả đạt được tuần 9:

&emsp;◉ Thứ Tư:
<br>&emsp;&emsp;○ Tìm hiểu khái niệm EC2 Right-Sizing và vai trò của việc lựa chọn kích thước EC2 phù hợp với nhu cầu sử dụng thực tế.
<br>&emsp;&emsp;○ Làm quen với Amazon CloudWatch và tìm hiểu cách sử dụng CloudWatch để thu thập các metric phục vụ việc theo dõi tài nguyên EC2.
<br>&emsp;&emsp;○ Tạo IAM Role dành cho CloudWatch Agent và tìm hiểu cách cấp quyền cần thiết để EC2 có thể gửi dữ liệu monitoring.
<br>&emsp;&emsp;○ Gắn IAM Role vào EC2 instance và kiểm tra việc cấp quyền cho instance.
<br>&emsp;&emsp;○ Cài đặt và cấu hình CloudWatch Agent trên EC2 để thu thập thông tin sử dụng tài nguyên, trong đó có dữ liệu liên quan đến memory utilization.
<br>&emsp;&emsp;○ Tìm hiểu mối liên hệ giữa EC2, IAM Role, CloudWatch Agent và dữ liệu monitoring trong quá trình tối ưu tài nguyên.
<br>&emsp;&emsp;○ Tìm hiểu EC2 Resource Optimization và cách sử dụng dữ liệu thu thập được để đánh giá cấu hình EC2.
<br>&emsp;&emsp;○ Tìm hiểu AWS Compute Optimizer và cách sử dụng các recommendation để hỗ trợ lựa chọn cấu hình EC2 phù hợp hơn.
<br>&emsp;&emsp;○ Nhận ra rằng việc tối ưu EC2 cần dựa trên dữ liệu sử dụng thực tế thay vì chỉ lựa chọn instance theo cấu hình mặc định.

&emsp;◉ Thứ Năm:
<br>&emsp;&emsp;○ Tìm hiểu AWS Service Quotas và khái niệm quota/giới hạn đối với các AWS service.
<br>&emsp;&emsp;○ Tìm hiểu giao diện Service Quotas và cách kiểm tra quota của từng AWS service.
<br>&emsp;&emsp;○ Tìm hiểu cách xác định giá trị quota hiện tại và theo dõi giới hạn sử dụng của tài khoản.
<br>&emsp;&emsp;○ Tìm hiểu các bước chuẩn bị trước khi thực hiện yêu cầu tăng quota.
<br>&emsp;&emsp;○ Thực hành tìm hiểu quy trình Request quota increase khi nhu cầu sử dụng tài nguyên vượt quá giới hạn hiện tại.
<br>&emsp;&emsp;○ Tìm hiểu rằng việc tăng quota cần được AWS xem xét và không phải mọi yêu cầu đều được chấp thuận hoàn toàn.
<br>&emsp;&emsp;○ Hiểu vai trò của Service Quotas trong việc chuẩn bị và mở rộng hệ thống AWS khi số lượng tài nguyên ngày càng tăng.
<br>&emsp;&emsp;○ Liên hệ kiến thức quota với việc lập kế hoạch triển khai và khả năng mở rộng hệ thống trong môi trường AWS.

&emsp;◉ Thứ Sáu:
<br>&emsp;&emsp;○ Tìm hiểu workshop Cost and Usage Management và vai trò của IAM trong việc kiểm soát việc sử dụng tài nguyên AWS.
<br>&emsp;&emsp;○ Chuẩn bị IAM Group, IAM User và IAM Policy theo cấu trúc của workshop.
<br>&emsp;&emsp;○ Tìm hiểu cách sử dụng IAM Policy để giới hạn quyền sử dụng tài nguyên theo từng điều kiện cụ thể.
<br>&emsp;&emsp;○ Thực hành giới hạn quyền sử dụng tài nguyên theo Region.
<br>&emsp;&emsp;○ Thực hành giới hạn EC2 theo EC2 family nhằm kiểm soát loại instance mà user được phép sử dụng.
<br>&emsp;&emsp;○ Thực hành giới hạn EC2 theo instance size để kiểm soát kích thước tài nguyên có thể được triển khai.
<br>&emsp;&emsp;○ Thực hành giới hạn việc tạo và sử dụng EBS volume theo chính sách đã cấu hình.
<br>&emsp;&emsp;○ Kiểm tra lại hiệu quả của IAM Policy bằng cách thực hiện các thao tác tạo tài nguyên và quan sát trường hợp được phép hoặc bị từ chối.
<br>&emsp;&emsp;○ Hiểu rõ hơn cách IAM có thể được sử dụng không chỉ để kiểm soát quyền truy cập mà còn hỗ trợ quản trị tài nguyên và kiểm soát chi phí.
<br>&emsp;&emsp;○ Kiểm tra và cleanup các AWS resources sau khi hoàn thành workshop.