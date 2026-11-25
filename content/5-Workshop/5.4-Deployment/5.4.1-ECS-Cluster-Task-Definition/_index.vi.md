---
title : "Tạo ECS Cluster + Task Definition"
date : 2024-01-01
weight : 1
chapter : false
pre : " <b> 5.4.1 </b> "
---

## Mục tiêu

Sau khi đưa Docker image lên ECR, nhóm em cần khai báo cách chạy ứng dụng PHP trên AWS. **ECS Cluster** là nơi nhóm quản lý các task của RakiBookery. **Task Definition** lưu cấu hình chạy của container, gồm image, tài nguyên, cổng, quyền và biến môi trường. Nhóm chọn Fargate để AWS quản lý hạ tầng chạy task.

## Bước 1. Tạo ECS Cluster

Trong Region Singapore (`ap-southeast-1`), nhóm vào **Amazon ECS → Clusters → Create cluster**. Đặt tên `RakiBookery-cluster` và chọn **Fargate only** ở phần hạ tầng. Sau khi tạo, ECS hiển thị thông báo thành công và cluster xuất hiện trong danh sách. Tại thời điểm chụp ảnh, cluster vừa tạo chưa có task đang chạy.

![Mở chức năng Create cluster trong Amazon ECS](../../../../images/5-Workshop/5.4-Deployment/5.4.1-ECS-Cluster-Task-Definition/01-cluster-create.png)

![Đặt tên RakiBookery-cluster và chọn Fargate only](../../../../images/5-Workshop/5.4-Deployment/5.4.1-ECS-Cluster-Task-Definition/02-cluster-fargate.png)

![ECS xác nhận đã tạo cluster RakiBookery](../../../../images/5-Workshop/5.4-Deployment/5.4.1-ECS-Cluster-Task-Definition/03-cluster-created.png)


## Bước 2. Khai báo Task Definition

1. Vào **Task definitions → Create new task definition**. Đặt tên family `RakiBookery-task` và chọn **AWS Fargate**.
2. Chọn **Linux/X86_64**, giữ network mode `awsvpc` và đặt tài nguyên ở mức **0,25 vCPU, 1 GB bộ nhớ** theo cấu hình nhóm đã thử. Kiến trúc x86_64 phù hợp với image `linux/amd64` ở mục 5.3.5.
3. Khai báo container chính là `Main`, chọn **Essential**, rồi nhập URI image của kho `rakibookery` trên ECR. Ảnh ghi lại lần dùng tag `latest`. Đặt **Container port** là `80`, **Protocol** là `TCP` và **App protocol** là `HTTP`, khớp với Apache và cấu hình ALB ở các mục tiếp theo.

![Mở chức năng Create new task definition](../../../../images/5-Workshop/5.4-Deployment/5.4.1-ECS-Cluster-Task-Definition/04-task-create.png)

![Chọn family RakiBookery-task và AWS Fargate](../../../../images/5-Workshop/5.4-Deployment/5.4.1-ECS-Cluster-Task-Definition/05-task-family.png)

![Cấu hình Linux X86_64 với 0,25 vCPU và 1 GB bộ nhớ](../../../../images/5-Workshop/5.4-Deployment/5.4.1-ECS-Cluster-Task-Definition/06-task-resources.png)

![Container Main dùng image từ ECR và cổng 80](../../../../images/5-Workshop/5.4-Deployment/5.4.1-ECS-Cluster-Task-Definition/07-task-container.png)


Sau khi lưu, Console ghi nhận `RakiBookery-task:1` ở trạng thái `ACTIVE`. Ảnh chi tiết cũng cho thấy task dùng execution role `ecsTaskExecutionRole`.

![RakiBookery-task:1 ở trạng thái ACTIVE sau khi tạo](../../../../images/5-Workshop/5.4-Deployment/5.4.1-ECS-Cluster-Task-Definition/08-task-created.png)


## Bước 3. Cấp quyền đọc secret cho ECS

Trong **IAM → Roles → ecsTaskExecutionRole → Permissions**, nhóm kiểm tra policy `AmazonECSTaskExecutionRolePolicy` dành cho việc tải image và ghi CloudWatch Logs. Nhóm tạo thêm inline policy `ECSSecretsAccessPolicy` để ECS lấy thông tin đăng nhập từ Secrets Manager lúc khởi động container.

Cấu hình demo đã cấp quyền trên phạm vi rộng. Để triển khai cấu hình PHP, cấp `secretsmanager:GetSecretValue` trên ARN của đúng secret RDS. Ảnh nguồn cho thấy secret dùng khóa mặc định `aws/secretsmanager`. Nếu secret dùng khóa KMS do nhóm quản lý, bổ sung `kms:Decrypt` trên ARN của khóa đó. Gắn quyền này vào **task execution role** vì ECS đọc secret trong lúc khởi động. [AWS về quyền của ECS task execution role](https://docs.aws.amazon.com/AmazonECS/latest/developerguide/task_execution_IAM_role.html).

![Trang quyền của ecsTaskExecutionRole và tùy chọn thêm inline policy](../../../../images/5-Workshop/5.4-Deployment/5.4.1-ECS-Cluster-Task-Definition/09-execution-role.png)


![IAM xác nhận đã thêm ECSSecretsAccessPolicy](../../../../images/5-Workshop/5.4-Deployment/5.4.1-ECS-Cluster-Task-Definition/10-execution-policy-created.png)


## Bước 4. Tạo revision và ánh xạ biến môi trường

Vào **Secrets Manager → Secrets**, mở secret RDS để lấy **Secret ARN**. Sau đó vào family `RakiBookery-task`, chọn **Create new revision** và mở phần **Environment variables** của container `Main`.

![Vị trí trường Secret ARN trong Secrets Manager, với mã định danh được che](../../../../images/5-Workshop/5.4-Deployment/5.4.1-ECS-Cluster-Task-Definition/11-database-secret.png)

![Mở Create new revision trong family RakiBookery-task](../../../../images/5-Workshop/5.4-Deployment/5.4.1-ECS-Cluster-Task-Definition/12-task-revision.png)

![Phần Environment variables của container trong revision mới](../../../../images/5-Workshop/5.4-Deployment/5.4.1-ECS-Cluster-Task-Definition/13-environment-editor.png)

![Ảnh thao tác chọn ValueFrom cho hai trường thông tin đăng nhập](../../../../images/5-Workshop/5.4-Deployment/5.4.1-ECS-Cluster-Task-Definition/14-secret-valuefrom.png)



Ảnh thao tác ghi lại hai trường `username` và `password` với kiểu `ValueFrom`. Khi dùng revision để chạy website PHP, nhóm đối chiếu tên biến với mã ứng dụng và chọn từng khóa JSON trong secret như sau.

- `DB_USER` lấy giá trị từ `<SECRET_ARN>:username::`.
- `DB_PASS` lấy giá trị từ `<SECRET_ARN>:password::`.
- `DB_HOST` dùng endpoint RDS và `DB_NAME` dùng `bookstoredb`.

Với ARN đầy đủ của secret mà thiếu hậu tố khóa JSON, ECS đưa toàn bộ nội dung secret vào biến. Khi chọn từng khóa JSON, dùng Fargate platform **1.4.0 trở lên** trên Linux. Lưu revision mới và ghi lại số revision để chọn đúng khi chạy task. [AWS về đưa secret vào biến môi trường ECS](https://docs.aws.amazon.com/AmazonECS/latest/developerguide/secrets-envvar-secrets-manager.html).

## Bước 5. Chạy thử task và kiểm tra log

1. Trong `RakiBookery-cluster`, vào **Tasks → Run new task**, chọn Fargate và revision có cấu hình secret. Đối chiếu VPC, subnet và security group trước khi chạy. Tài liệu nhóm mô tả lượt thử dùng public subnet với public IP. Nếu chạy trong private subnet, kiểm tra đường tới ECR, S3, Secrets Manager và CloudWatch Logs qua NAT hoặc endpoint phù hợp.
2. Theo dõi trạng thái task đến `RUNNING` và mở tab **Logs** để kiểm tra container khởi động. Nếu task chuyển sang `STOPPED`, xem **Stopped reason** và lỗi của container để xác định có vướng ở kéo image, lấy secret hoặc khởi động ứng dụng.
3. Để xác nhận PHP kết nối MySQL, chạy một thao tác đọc dữ liệu từ website và đối chiếu với log của task PHP. Phần đối chiếu ứng dụng được trình bày ở [mục 5.4.5]({{< relref "/5-Workshop/5.4-Deployment/5.4.5-Connect-PHP-App-To-RDS/" >}}).

## Kết luận

Hồ sơ của nhóm xác nhận ECS Cluster, Task Definition revision đầu tiên và inline policy đã được tạo. Ảnh cuối ghi lại màn hình cấu hình `ValueFrom` khi tạo revision. Nguồn chưa có ảnh task PHP ở trạng thái `RUNNING` hoặc log Apache của lần chạy đó. Phần kiểm tra ở trên ghi các dấu hiệu cần đối chiếu trước khi gắn ECS Service với ALB.
