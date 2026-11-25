---
title: "Liên kết ECS Service với ALB"
date: 2024-01-01
weight: 3
chapter: false
pre: " <b> 5.4.3. </b> "
---

## ECS Service được liên kết với ALB để làm gì?

Ở [mục 5.4.2]({{< relref "/5-Workshop/5.4-Deployment/5.4.2-ALB-Target-Group" >}}), nhóm em đã có listener và Target Group. Để ALB biết task nào đang chạy website, nhóm cấu hình ECS Service sử dụng Target Group đó và chỉ rõ tên container PHP cùng cổng `80` từ Task Definition.

```text
Trình duyệt → ALB (listener HTTP:80) → Target Group (IP:80) → Task Fargate (PHP/Apache)
                                        ↑
                           ECS Service đăng ký IP của task
```

Ví dụ, khi triển khai revision mới, ECS có thể khởi chạy task với một địa chỉ IP khác. ECS tự đăng ký IP của task mới và hủy đăng ký task cũ theo quá trình triển khai. Người dùng vẫn truy cập cùng địa chỉ ALB. Với ECS Fargate, nhóm kiểm tra cấu hình ở cấp **ECS Service** thay vì nhập thủ công IP của từng task vào Target Group. [Tài liệu AWS về ECS và ALB](https://docs.aws.amazon.com/AmazonECS/latest/developerguide/alb.html).

## Đối chiếu cấu hình của RakiBookery

**1.** Vào **ECS → Clusters → Services**, chọn `RakiBookery-task-service-01` trong `RakiBookery-cluster` và xem phần **Load balancing**. Đối chiếu Target Group với Target Group được listener của `RakiBookery-alb` chuyển tiếp tới.

![Gắn container Main:80 với listener HTTP:80 và RakiBookery-tg](../../../../images/5-Workshop/5.4-Deployment/5.4.3-Link-ECS-with-ALB/01-ecs-load-balancing.png)

**2.** Kiểm tra cặp **container name / container port** là `Main:80` và khớp Task Definition của revision đang triển khai. Đối chiếu subnet và security group của task với cấu hình ở mục 5.4.2.

![Trang ECS Service ghi nhận cấu hình mạng và liên kết Load balancing](../../../../images/5-Workshop/5.4-Deployment/5.4.3-Link-ECS-with-ALB/02-ecs-service-configuration.png)

**3.** Khi Fargate được bật với số task mong muốn lớn hơn 0, theo dõi **Deployments**, **Events** và **Tasks**. Lấy địa chỉ IP của task đang chạy và đối chiếu với danh sách tại **Target Groups → Targets**.

## Cách nhóm xác nhận luồng truy cập

- **Task `RUNNING`.** ECS đã khởi chạy task. Nếu task dừng, đọc **Stopped reason** và log để kiểm tra image, secret hoặc lỗi khởi động.
- **Target `Healthy`.** ALB đã nhận phản hồi đúng theo cấu hình Health Check. Nếu target `Unhealthy`, dùng lý do hiển thị để khoanh vùng lỗi cổng, đường mạng hoặc phản hồi HTTP.
- **Website mở qua ALB.** Dùng DNS của ALB để mở một trang thực tế của ứng dụng. Sau đó mới kiểm tra luồng qua CloudFront ở mục 5.4.4 và thao tác đọc, ghi MySQL ở mục 5.4.5.

## Kết quả ghi nhận

Ghi chú triển khai của nhóm xác nhận phần **liên kết ECS Service với ALB đã cấu hình**. Tại thời điểm ghi chú, bước chạy Fargate để kiểm tra Health Check và mở website vẫn đang chờ. Khi kiểm thử lại, nhóm em sẽ lưu cùng một lượt ảnh cấu hình **Load balancing**, IP task trong tab **Targets** và kết quả mở website qua ALB. [Ghi chú cấu hình ALB của nhóm](https://app.notion.com/p/ALB-Target-Group-Health-Check-3f84cb9c2604807caf58cceadee8794c).
