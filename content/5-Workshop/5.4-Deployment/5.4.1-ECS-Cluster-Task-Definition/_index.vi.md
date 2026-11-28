---
title : "Tạo ECS Cluster + Task Definition"
date : 2024-01-01
weight : 1
chapter : false
pre : " <b> 5.4.1 </b> "
---

## Mục tiêu

Nhóm dùng Amazon ECS Fargate để chạy ứng dụng PHP của RakiBookery tại Singapore (`ap-southeast-1`). Phần này ghi nhận cluster và service của ứng dụng, hướng dẫn kiểm tra task definition và điều chỉnh số task khi chạy thử.

## Kiến trúc tổng quan

**Image PHP trên Amazon ECR → Task definition → Task chạy trên AWS Fargate**

Task definition lưu cấu hình như image, cổng container, CPU và bộ nhớ. ECS Service dùng một revision của task definition để duy trì số task theo `desiredCount`. Cluster là nơi nhóm theo dõi service và task. AWS Fargate quản lý hạ tầng chạy task nên nhóm có thể triển khai container mà không cần quản lý máy EC2 cho cluster. [Tài liệu AWS về ECS Service](https://docs.aws.amazon.com/AmazonECS/latest/developerguide/ecs_services.html)

## 1. Xác nhận cluster và service

Trong **Amazon ECS → Clusters**, nhóm chọn `rakibookery-prod-cluster`. Danh sách cluster hiển thị phương thức chạy **Fargate**, một service và **No tasks running** tại thời điểm chụp.

![Danh sách ECS cluster ở Singapore hiển thị rakibookery-prod-cluster và một service](../../../../images/5-Workshop/5.4-Deployment/5.4.1-ECS-Cluster-Task-Definition/ecs-cluster-list.png)

Trong tab **Services**, nhóm dùng `rakibookery-prod-service` với kiểu lập lịch **Replica** để điều chỉnh số task của ứng dụng. Ảnh chi tiết của service nằm ở bước xác nhận task đã dừng.

## 2. Kiểm tra Task Definition

Trong danh sách **Services**, mở liên kết ở cột **Task definition** để xem revision service đang dùng. Đối chiếu các thiết lập trước khi bật task.

1. Trong **Container details**, kiểm tra URI image từ ECR, tên container và cổng ứng dụng. Đối chiếu URI của revision với image trong Amazon ECR. Cách kiểm tra tag và digest được trình bày ở [mục 5.3.5]({{< relref "/5-Workshop/5.3-Infrastructure/5.3.5-Set-Up-ECR/" >}}). Khi service gắn với ALB, tên và cổng container phải khớp cấu hình gắn target group. Xem [mục 5.4.3]({{< relref "/5-Workshop/5.4-Deployment/5.4.3-Link-ECS-with-ALB/" >}}).
2. Kiểm tra **Task size**, hệ điều hành và kiến trúc CPU. Kiến trúc của task phải khớp với image đã build. Fargate dùng network mode `awsvpc`.
3. Kiểm tra **Task execution role** để ECS tải image và ghi CloudWatch Logs theo cấu hình. Nếu task lấy thông tin kết nối cơ sở dữ liệu từ Secrets Manager, đối chiếu tên biến với ứng dụng và quyền đọc secret của execution role.

Khi đổi image hoặc cấu hình container, đăng ký revision rồi dùng **Update service** để chọn revision đó. [Tài liệu AWS về task definition](https://docs.aws.amazon.com/AmazonECS/latest/developerguide/task_definitions.html)

![Danh sách ECS cluster ở Singapore hiển thị rakibookery-prod-cluster và một service](../../../../images/5-Workshop/5.4-Deployment/5.4.1-ECS-Cluster-Task-Definition/ECS3.png)

## 3. Bật task để chạy thử

Mở Bash trong AWS CloudShell và chạy các khối lệnh dưới đây trong cùng phiên.

~~~bash
REGION=ap-southeast-1
CLUSTER=rakibookery-prod-cluster
SERVICE=rakibookery-prod-service
~~~

Đặt `desiredCount=1` để ECS khởi động một task. Lệnh `wait services-stable` chờ service ổn định sau khi cập nhật.

~~~bash
aws ecs update-service --region "$REGION" --cluster "$CLUSTER" \
  --service "$SERVICE" --desired-count 1 \
  --output json --no-cli-pager &&
aws ecs wait services-stable --region "$REGION" --cluster "$CLUSTER" \
  --services "$SERVICE" --no-cli-pager
~~~

**Trích kết quả từ `update-service`**

~~~text
{
  "service": {
    "serviceName": "rakibookery-prod-service",
    "status": "ACTIVE",
    "desiredCount": 1,
    "runningCount": 0,
    "pendingCount": 0,

    ...
    
    "enableExecuteCommand": true,
    "availabilityZoneRebalancing": "ENABLED",
    "resourceManagementType": "CUSTOMER"
  }
}
~~~

Phản hồi lúc bật ghi nhận `desiredCount=1` và `runningCount=0`. ECS đã nhận yêu cầu chạy một task. Bộ đếm sẽ thay đổi khi task khởi động xong.

## 4. Dừng task và xác nhận trạng thái

Sau khi chạy thử, đặt `desiredCount=0` và chờ service ổn định. Chạy lại `describe-services` ở bước 3 để xác nhận **0 desired / 0 running / 0 pending**.

~~~bash
aws ecs update-service --region "$REGION" --cluster "$CLUSTER" \
  --service "$SERVICE" --desired-count 0 \
  --output json --no-cli-pager &&
aws ecs wait services-stable --region "$REGION" --cluster "$CLUSTER" \
  --services "$SERVICE" --no-cli-pager
~~~

**Trích kết quả từ `update-service`**

~~~text
{
  "service": {
    "serviceName": "rakibookery-prod-service",
    "status": "ACTIVE",
    "desiredCount": 0,
    "runningCount": 1,
    "pendingCount": 0,

    ...

    "enableExecuteCommand": true,
    "availabilityZoneRebalancing": "ENABLED",
    "resourceManagementType": "CUSTOMER"
  }
}
~~~

Phản hồi lúc tắt ghi nhận `desiredCount=0` và `runningCount=1`. Chờ task dừng rồi kiểm tra lại số task của service.

Dùng `list-tasks` để kiểm tra riêng task thuộc service.

~~~bash
aws ecs list-tasks \
  --cluster rakibookery-prod-cluster \
  --service-name rakibookery-prod-service \
  --region ap-southeast-1 \
  --output json --no-cli-pager
~~~

Kết quả cần thấy sau khi đã giảm số task về `0`.

~~~json
{
  "taskArns": []
}
~~~

`list-tasks` mặc định lọc theo trạng thái mong muốn `RUNNING`. Đối chiếu danh sách rỗng với các bộ đếm trên service để xác nhận task đã dừng. [Tài liệu AWS CLI về list-tasks](https://docs.aws.amazon.com/cli/latest/reference/ecs/list-tasks.html)

Trong Console, tab **Services** hiển thị `rakibookery-prod-service` ở trạng thái **Active** với **0/0 Tasks running**. Phần **Cluster overview** ghi nhận **0 running / 0 pending**.

![Tab Services hiển thị rakibookery-prod-service Active và 0 trên 0 task đang chạy](../../../../images/5-Workshop/5.4-Deployment/5.4.1-ECS-Cluster-Task-Definition/ecs-cluster-services.png)

## Kết luận

Console ghi nhận `rakibookery-prod-cluster` dùng Fargate và có một service là `rakibookery-prod-service`. Tại thời điểm chụp, service ở trạng thái **Active** với **0/0 Tasks running**. Nhóm dùng cùng service để chạy một task khi cần thử ứng dụng và giảm về `0` khi hoàn tất.

