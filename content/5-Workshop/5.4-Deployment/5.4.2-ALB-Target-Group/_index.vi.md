---
title : "Tạo ALB, Target Group và Health Check"
date : 2026-11-12
weight : 2
chapter : false
pre : " <b> 5.4.2 </b> "
---

# Hướng dẫn cấu hình ALB + Target Group + Health Check

## 1. Tạo Target Group

### 1.1. Mở Target Groups
1. Đăng nhập AWS EC2 Console.
2. Chọn Region **Singapore (ap-southeast-1)**.
3. Trong menu bên trái, kéo xuống phần **Load Balancing**.
4. Chọn **Target Groups**.
5. Nhấn **Create target group**.

### 1.2. Basic configuration
Điền thông tin theo bảng sau:

| Trường | Giá trị |
| :--- | :--- |
| **Choose a target type** | IP addresses |
| **Target group name** | RakiBookery-tg |
| **Protocol** | HTTP |
| **Port** | 80 |
| **IP address type** | IPv4 |
| **VPC** | VPC đang sử dụng cho RakiBookery |
| **Protocol version** | HTTP1 |

> **Quan trọng:** Chọn **IP addresses**, không chọn *Instances*, vì ECS Fargate sử dụng network mode `awsvpc`.

![Cấu hình Basic Target Group](url_hinh_anh_cua_ban_1)

### 1.3. Tạo Target Group
1. Nhấn **Next**.
2. Đến phần **Register targets**, chưa cần nhập IP nào.
3. Nhấn **Create target group**.

*Lưu ý: ECS Service sẽ tự động đăng ký IP của Fargate task vào Target Group khi chúng ta liên kết chúng. Sau khi tạo, Target Group có **0 registered targets** là điều hoàn toàn bình thường.*

![Register targets & Tạo Target Group](url_hinh_anh_cua_ban_2)

---

## 2. Tạo Application Load Balancer (ALB)

**Kiểm tra và Khởi tạo:**
1. Vào **EC2** → **Load Balancers**.
2. Trong menu bên trái, chọn **Load Balancing** → **Load Balancers** (Kiểm tra danh sách ALB hiện có).
3. Nhấn **Create load balancer** → Chọn **Application Load Balancer**.

### 2.1. Basic configuration
| Trường | Giá trị |
| :--- | :--- |
| **Load balancer name** | RakiBookery-alb |
| **Scheme** | Internet-facing |
| **IP address type** | IPv4 |

### 2.2. Network mapping
| Trường | Giá trị |
| :--- | :--- |
| **VPC** | VPC của RakiBookery-tg |
| **Availability Zones** | Ít nhất 2 AZ khác nhau |
| **Subnets** | 2 public subnets tương ứng |

> **Lưu ý:** Public subnet cần có route `0.0.0.0/0` đến Internet Gateway.

![Network Mapping](url_hinh_anh_cua_ban_3)

### 2.3. Security groups
Cần tạo security group riêng cho ALB.

**Bước 1: Tạo Security Group cho ALB**
Mở tab AWS mới:
1. Vào **EC2** → **Security Groups**.
2. Nhấn **Create security group**.
3. Nhập thông tin:

| Trường | Giá trị |
| :--- | :--- |
| **Security group name** | RakiBookery-alb-sg |
| **Description** | Security group for RakiBookery ALB |
| **VPC** | Cùng VPC với ALB |

**Bước 2: Cấu hình Rules**
* **Inbound rules:** Nhấn Add rule và nhập:
  * **Type:** HTTP
  * **Port:** 80
  * **Source:** Anywhere-IPv4 (0.0.0.0/0)
* **Outbound rules:** Giữ mặc định *All traffic* cho bước demo.

Nhấn **Create security group**.
Sau đó quay lại tab tạo ALB, chọn Security Group `RakiBookery-alb-sg` vừa tạo (cho phép inbound HTTP port 80 từ `0.0.0.0/0`).

### 2.4. Listeners and routing
| Trường | Giá trị |
| :--- | :--- |
| **Protocol** | HTTP |
| **Port** | 80 |
| **Default action** | Forward to target groups |
| **Target group** | RakiBookery-tg |

### 2.5. Tạo ALB
Nhấn **Create load balancer** và đợi trạng thái chuyển sang **Active**.

* **DNS name thu được:** `RakiBookery-alb-1141293019.ap-southeast-1.elb.amazonaws.com`

![Hoàn tất tạo ALB](url_hinh_anh_cua_ban_4)

---

## 3. Gắn ALB vào ECS Service

1. Vào **Amazon ECS** → **Clusters** → Chọn `RakiBookery-cluster`.
2. Chọn tab **Services**.
3. Nhấn vào Service `RakiBookery-task-service-01`.
4. Nhấn **Update** ở góc trên bên phải.
5. Tìm đến phần **Load balancing**.
6. Cấu hình theo thông số sau:

| Mục | Giá trị |
| :--- | :--- |
| **Load balancer type** | Application Load Balancer |
| **Use existing load balancer** | RakiBookery-alb |
| **Listener** | HTTP:80 |
| **Use existing target group** | RakiBookery-tg |
| **Container** | Main |
| **Container port** | 80 |

![Gắn ALB vào ECS](url_hinh_anh_cua_ban_5)

---
> **Tổng kết:** AWS-006 đã hoàn thành cấu hình ALB, Target Group, Health Check, Listener và Security Groups. ECS Service đã được liên kết với ALB. Hiện đang chờ nhóm bật Fargate để kiểm thử Health Check và truy cập website thực tế.