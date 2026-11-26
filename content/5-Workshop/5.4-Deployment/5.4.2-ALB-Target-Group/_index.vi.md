---
title : "Tạo ALB, Target Group và Health Check"
date : 2024-01-01
weight : 2
chapter : false
pre : " <b> 5.4.2 </b> "
---


## Mục tiêu

Sau khi chuẩn bị ECS Cluster và Task Definition, nhóm em cần một địa chỉ để trình duyệt truy cập website PHP của RakiBookery. Khi ECS triển khai phiên bản mới, service có thể thay thế các task Fargate. Nhóm dùng Application Load Balancer (ALB) để nhận yêu cầu web. Target Group quản lý các địa chỉ IP của task đã đăng ký.

## Chuẩn bị listener và mạng

Dùng đúng VPC của ECS Service. Chuẩn bị hai public subnet thuộc hai Availability Zone, security group của ALB và security group của task. Nếu dựng mới, vào **EC2 → Load Balancers → Create load balancer → Application Load Balancer**. Chọn **Internet-facing**, hai public subnet và security group của ALB. Sau đó áp dụng cấu hình listener và Target Group dưới đây. 
![Ảnh nguồn cho thấy RakiBookery-alb chọn Internet-facing và IPv4](../../../../images/5-Workshop/5.4-Deployment/5.4.2-ALB-Target-Group/01-alb-internet-facing.png)

## Cấu hình ALB và Target Group

**1. Tạo Target Group dạng IP.** Task Fargate dùng chế độ mạng `awsvpc` và có địa chỉ IP riêng. Vì vậy, nhóm tạo `RakiBookery-tg` với target type `IP` và dùng cổng HTTP `80` theo container Apache ở mục 5.4.1. Khi service khởi chạy task, ECS đăng ký IP của task vào Target Group này.

![Tạo Target Group RakiBookery-tg kiểu IP với HTTP cổng 80](../../../../images/5-Workshop/5.4-Deployment/5.4.2-ALB-Target-Group/02-target-group-ip.png)

![Target Group vừa tạo có 0 target đăng ký](../../../../images/5-Workshop/5.4-Deployment/5.4.2-ALB-Target-Group/03-target-group-created.png)

**2. Kiểm tra listener trên ALB.** ALB `RakiBookery-alb` ở Region Singapore (`ap-southeast-1`) dùng listener `HTTP:80` trong cấu hình workshop. Quy tắc listener chuyển tiếp yêu cầu đến Target Group của ứng dụng. Listener là nơi nhận yêu cầu từ phía truy cập, còn Target Group xác định nhóm task đích.

![ALB được đặt trong hai public subnet của VPC RakiBookery](../../../../images/5-Workshop/5.4-Deployment/5.4.2-ALB-Target-Group/04-alb-network-mapping.png)

![ALB Active với listener HTTP:80 chuyển tiếp về RakiBookery-tg](../../../../images/5-Workshop/5.4-Deployment/5.4.2-ALB-Target-Group/05-alb-http-listener.png)

**3. Kiểm tra security group ở hai đầu.** Security group của ALB cho phép lưu lượng đi vào cổng listener từ nguồn truy cập được cấu hình. Security group của task cho phép TCP `80` từ security group của ALB. Đối chiếu cả đường đi từ ALB tới cổng Health Check nếu dùng cổng khác.

![Quy tắc inbound HTTP cổng 80 của security group dành cho ALB](../../../../images/5-Workshop/5.4-Deployment/5.4.2-ALB-Target-Group/06-alb-security-group.png)

**4. Cấu hình Health Check.** ALB gửi yêu cầu HTTP đến đường dẫn kiểm tra trên từng target. Ảnh cấu hình cho thấy nhóm chọn đường dẫn `/`, traffic port `80` và chấp nhận mã `200`. Chu kỳ kiểm tra là `30` giây, timeout `5` giây, ngưỡng Healthy và Unhealthy đều là `2` lần liên tiếp. Khi chạy thử, nhóm cần xác nhận trang `/` trả mã `200` đúng như cấu hình. [AWS về Health Check của ALB](https://docs.aws.amazon.com/elasticloadbalancing/latest/application/target-group-health-checks.html).

![Health Check dùng HTTP với đường dẫn /](../../../../images/5-Workshop/5.4-Deployment/5.4.2-ALB-Target-Group/07-health-check-path.png)

![Health Check dùng traffic port và chấp nhận mã HTTP 200](../../../../images/5-Workshop/5.4-Deployment/5.4.2-ALB-Target-Group/08-health-check-settings.png)


## Kết luận

Nhóm em đã cấu hình ALB dạng Internet-facing với listener HTTP cổng `80`, Target Group dạng IP và Health Check HTTP. Các thiết lập này xác định đường chuyển tiếp yêu cầu đến container PHP và cách ALB kiểm tra từng target. Bước tiếp theo là liên kết ECS Service với Target Group này và kiểm tra truy cập website qua ALB.
 