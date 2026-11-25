---
title : "Tạo và cấu hình IAM"
date : 2024-01-01 
weight : 1
chapter : false
pre : " <b> 5.3.1 </b> "
---

### Mục tiêu

Tổ chức quyền truy cập AWS cho nhóm RakiBookery và quyền của dịch vụ khi chạy ứng dụng PHP. AWS Secrets Manager lưu thông tin xác thực MySQL để ECS cung cấp cho container lúc khởi động.

### Cần chuẩn bị

- **Tài khoản và quyền triển khai.** Xác nhận đúng tài khoản RakiBookery, danh tính do admin cấp và danh sách thành viên. IAM có phạm vi toàn tài khoản. Với ECS, ECR, RDS và Secrets Manager, chọn Region Singapore (`ap-southeast-1`).
- **Phạm vi công việc.** Đối chiếu các action ở [mục 5.2]({{< relref "/5-Workshop/5.2-Prerequiste/" >}}) với nhiệm vụ của nhóm. Xác định ai quản lý IAM, ai triển khai task và ai quản lý thông tin xác thực.
- **Đầu vào để chạy thử.** Chuẩn bị URI và tag của backend image ở [mục 5.3.5]({{< relref "/5-Workshop/5.3-Infrastructure/5.3.5-Set-Up-ECR/" >}}), tên CloudWatch log group nếu dùng `awslogs`, và endpoint, tên database, tài khoản ứng dụng từ [mục 5.3.4]({{< relref "/5-Workshop/5.3-Infrastructure/5.3.4-Set-Up-RDS/" >}}). Xác định tên biến môi trường mà mã PHP đọc.

Có thể chuẩn bị IAM Group và execution role trước. Sau khi có RDS và ECR, bổ sung ARN của secret và kiểm tra với image thật.

### Cấu hình triển khai

#### 1. Rà soát IAM Group và tài khoản thành viên

1. Trong **IAM → User groups**, mở nhóm `RakiBookery`. Nếu dựng môi trường mới, admin tạo group rồi thêm từng tài khoản đã được cấp vào tab **Users**.
2. Trong tab **Permissions**, đối chiếu policy đang gắn với công việc của nhóm. Ảnh minh chứng cho thấy cấu hình đã dùng trong tài khoản dự án. Nếu người dùng vẫn kế thừa `AdministratorAccess`, việc gắn thêm một policy hẹp hơn không làm giảm quyền hiện có.
3. Với thành viên đăng nhập AWS Console bằng IAM user, kiểm tra MFA trong **Security credentials**. Ghi nhận nhóm và phạm vi quyền vào báo cáo.

![Tab Permissions của nhóm RakiBookery](../../../../images/5-Workshop/5.3-Infrastructure/iam-group-permissions.png)

*Hình 5.3.1a. Cấu hình ghi nhận của nhóm RakiBookery gồm 5 IAM user và hai policy được gắn vào group.*

#### 2. Chuẩn bị ECS task execution role

1. Mở **IAM → Roles**, tìm `ecsTaskExecutionRole`. Nếu role đã có, mở role để kiểm tra **Trust relationships** và **Permissions**.

   ![IAM Roles hiển thị ecsTaskExecutionRole và trusted entity ecs-tasks](../../../../images/5-Workshop/5.3-Infrastructure/iam-ecs-execution-role-list.png)

   *Hình 5.3.1b. Danh sách xác nhận role ecsTaskExecutionRole có thực thể tin cậy là ECS tasks.*

2. Nếu cần tạo mới, chọn **Create role → AWS service → Elastic Container Service**. Trong **Use case**, chọn **Elastic Container Service Task**. Kiểm tra trust policy có `sts:AssumeRole` cho `ecs-tasks.amazonaws.com`.
3. Trong **Add permissions**, gắn `AmazonECSTaskExecutionRolePolicy` ở danh sách **Permissions policies** để ECS kéo image từ ECR và ghi log qua `awslogs`. Permissions boundary chỉ đặt mức quyền tối đa. Nếu tài khoản yêu cầu boundary, dùng boundary do admin phê duyệt và kiểm tra nó cho phép các thao tác của role. Tạo trước log group nếu container dùng `awslogs`.

   ![Tab Permissions của ecsTaskExecutionRole hiển thị AmazonECSTaskExecutionRolePolicy](../../../../images/5-Workshop/5.3-Infrastructure/iam-ecs-execution-role-policy.png)

   *Hình 5.3.1c. Policy AmazonECSTaskExecutionRolePolicy đã được gắn vào execution role*

4. Sao chép ARN và chọn role này ở trường **Task execution role** của task definition. Nếu PHP gọi AWS API trực tiếp, chẳng hạn tải ảnh lên S3, cấp quyền cho một role riêng ở trường **Task role** và giới hạn theo bucket hoặc prefix cần dùng.
5. Cho phép danh tính triển khai dùng `iam:PassRole` trên ARN các role đã được phê duyệt, với `iam:PassedToService` là `ecs-tasks.amazonaws.com`. [AWS IAM PassRole](https://docs.aws.amazon.com/IAM/latest/UserGuide/id_roles_use_passrole.html).

#### 3. Chuẩn bị secret và khai báo trong task definition

Sau khi có tài khoản MySQL dành cho ứng dụng, mở **Secrets Manager → Store a new secret** trong Region dự án. Có thể lưu tên người dùng và mật khẩu dưới các khóa JSON `username` và `password`. Sao chép ARN đầy đủ từ trang chi tiết của secret. Nếu đã có secret dành cho ứng dụng, dùng lại secret đó và đối chiếu tên khóa trước khi cấu hình task.

Policy ở [mục 5.2]({{< relref "/5-Workshop/5.2-Prerequiste/" >}}) mô tả phạm vi thao tác triển khai của workshop. Policy mẫu bên dưới giới hạn quyền đọc secret của execution role theo ARN của secret ứng dụng. Nếu task nhận thông tin MySQL qua Secrets Manager, vào **Permissions → Add permissions → Create inline policy → JSON** trên execution role để bổ sung quyền theo mẫu. Ảnh 5.3.1c ghi nhận policy cơ bản tại thời điểm chụp. Kiểm tra quyền đọc secret sau khi bổ sung.

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": "secretsmanager:GetSecretValue",
      "Resource": "arn:aws:secretsmanager:ap-southeast-1:987989461846:secret:rakibookery/prod/rds/credentials-bUVjFm"
    }
  ]
}
```


### Kiểm tra cấu hình

- Trong IAM, đối chiếu nhóm thành viên, trust policy, role ARN và quyền đọc đúng secret của execution role. Ảnh danh sách role cho biết role tồn tại. Tab **Permissions** mới cho thấy các policy đang được gắn.
- Chạy thử một task ở môi trường thử nghiệm. Xác nhận task kéo được image, kết nối tới log group khi bật `awslogs` và đọc được secret. Nếu task dừng khi khởi tạo, xem **Stopped reason** và log để phân biệt thiếu quyền ECS với lỗi truy cập ECR, Secrets Manager hoặc CloudWatch Logs từ subnet.
- Khi task chạy, kiểm tra ứng dụng kết nối RDS và thực hiện truy vấn thử. Đối chiếu security group ở [mục 5.3.2]({{< relref "/5-Workshop/5.3-Infrastructure/5.3.2-Set-Up-VPC/" >}}) nếu MySQL bị timeout. Lưu kết quả kiểm tra dưới hạng mục **APP-001** và che giá trị secret trong ảnh minh chứng.

### Kết luận

Nhóm đã có IAM Group riêng và danh sách role dành cho dịch vụ. Policy `AdministratorAccess` có phạm vi rộng, nên tiêu chí đặc quyền tối thiểu cần được đánh giá theo nhiệm vụ thực tế của từng thành viên. Việc cấp secret và kết nối MySQL được kiểm tra cùng task Fargate.
