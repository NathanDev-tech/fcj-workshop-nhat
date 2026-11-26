---
title: "Liên kết ECS Service với ALB"
date: 2024-01-01
weight: 3
chapter: false
pre: " <b> 5.4.3. </b> "
---

## Mục tiêu

Ở [mục 5.4.2]({{< relref "/5-Workshop/5.4-Deployment/5.4.2-ALB-Target-Group" >}}), nhóm em đã có listener và Target Group. Để ALB biết task nào đang chạy website, nhóm cấu hình ECS Service sử dụng Target Group đó và chỉ rõ tên container PHP cùng cổng `80` từ Task Definition.

```text
Trình duyệt → ALB (listener HTTP:80) → Target Group (IP:80) → Task Fargate (PHP/Apache)
                                        ↑
                           ECS Service đăng ký IP của task
```

Ví dụ, khi triển khai revision mới, ECS có thể khởi chạy task với một địa chỉ IP khác. ECS đăng ký IP của task mới và hủy đăng ký IP của task cũ theo quá trình triển khai. DNS của ALB giữ nguyên trong quá trình này. ECS tự động quản lý việc đăng ký và hủy đăng ký IP của task cho service. Nhóm kiểm tra liên kết Target Group trong cấu hình **ECS Service**. [Tài liệu AWS về ECS và ALB](https://docs.aws.amazon.com/AmazonECS/latest/developerguide/alb.html).

## Tạo hoặc kiểm tra ECS Service

Nếu service đã có, mở cấu hình và đối chiếu với các bước bên dưới. Nếu dựng mới, tạo service trong `RakiBookery-cluster` bằng Fargate, chọn revision của task definition từ mục 5.4.1 và đặt số task mong muốn lớn hơn `0` cho lượt thử. Chọn VPC, các subnet của workload và security group của task. Trong **Load balancing**, chọn ALB và Target Group dạng IP từ mục 5.4.2, rồi chọn container `Main` và cổng `80`.

## Đối chiếu cấu hình của RakiBookery

**1.** Vào **ECS → Clusters → Services**, chọn `RakiBookery-task-service-01` trong `RakiBookery-cluster` và xem phần **Load balancing**. Đối chiếu Target Group với Target Group được listener của `RakiBookery-alb` chuyển tiếp tới.

![Gắn container Main:80 với listener HTTP:80 và RakiBookery-tg](../../../../images/5-Workshop/5.4-Deployment/5.4.3-Link-ECS-with-ALB/01-ecs-load-balancing.png)

**2.** Kiểm tra cặp **container name / container port** là `Main:80` và khớp Task Definition của revision đang triển khai. Đối chiếu subnet và security group của task với cấu hình ở mục 5.4.2.

![Trang ECS Service ghi nhận cấu hình mạng và liên kết Load balancing](../../../../images/5-Workshop/5.4-Deployment/5.4.3-Link-ECS-with-ALB/02-ecs-service-configuration.png)

**3.** Khi service khởi chạy task Fargate với số task mong muốn lớn hơn `0`, theo dõi **Deployments**, **Events** và **Tasks**. Lấy địa chỉ IP của task đang chạy và đối chiếu với danh sách tại **Target Groups → Targets**.

## Cách nhóm xác nhận luồng truy cập

- **Task `RUNNING`.** ECS đã khởi chạy task. Cần xem thêm log khởi động PHP. Nếu task dừng, đọc **Stopped reason** và log để kiểm tra image, secret hoặc lỗi khởi động.
- **Target `Healthy`.** Target đã vượt qua Health Check của ALB theo đường dẫn và mã phản hồi đã cấu hình. Nếu target `Unhealthy`, dùng lý do hiển thị để khoanh vùng lỗi cổng, đường mạng hoặc phản hồi HTTP.
- **Website mở qua ALB.** Dùng DNS của ALB để mở một trang thực tế của ứng dụng. Sau đó mới kiểm tra luồng qua CloudFront ở mục 5.4.4 và thao tác đọc, ghi MySQL ở mục 5.4.5.

Khi target chuyển sang `Healthy`, truy cập một URL thật qua ALB. Đối chiếu IP riêng của task với IP trong **Targets**. Khi thử sau một lần cập nhật, ghi lại revision và thời điểm gửi yêu cầu từ trình duyệt. Nếu log ứng dụng xác định được task xử lý yêu cầu, đối chiếu task đó với IP đã đăng ký.

## Kết luận

ECS Service đã được liên kết với ALB thông qua Target Group dạng IP và container `Main:80`. Liên kết này giúp ECS quản lý việc đăng ký target khi các task thay đổi trong quá trình triển khai. Để xác nhận website truy cập được, cần đối chiếu task đang chạy, trạng thái target và trang mở qua DNS của ALB.

