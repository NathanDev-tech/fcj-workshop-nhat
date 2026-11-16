---
title : "Kết nối PHP app với RDS"
date : 2024-01-01
weight : 5
chapter : false
pre : " <b> 5.4.5 </b> "
---

# Kết nối Ứng dụng PHP trên Amazon ECS với Amazon RDS

Tài liệu này hướng dẫn từng bước cách cấu hình biến môi trường từ AWS Secrets Manager, import dữ liệu vào Amazon RDS thông qua một ECS Task độc lập (giữ nguyên tính bảo mật Private cho RDS), xử lý lỗi phân biệt chữ hoa/thường (case-sensitivity) của MySQL trên Linux và kiểm tra kết nối toàn diện qua Application Load Balancer (ALB) và Amazon CloudFront.

---

## Phần 1: Cấu hình Biến môi trường & Secrets Manager cho ECS

Ứng dụng PHP cần đọc thông tin kết nối cơ sở dữ liệu thông qua các biến môi trường chuẩn (`DB_HOST`, `DB_NAME`, `DB_USER`, `DB_PASS`). Chúng ta sẽ ánh xạ các khóa (Key) từ AWS Secrets Manager sang tên biến môi trường tương ứng trong ECS Task Definition.

### Bước 1: Tạo Revision mới và cập nhật tên biến môi trường trong ECS Task Definition

1. Truy cập **Amazon ECS** $\rightarrow$ **Task definitions** $\rightarrow$ chọn **RakiBookery-task**.
2. Nhấn **Create new revision**.
![Giao diện tạo Revision mới](images/createRevision.png)
3. Cuộn xuống phần **Container - 1** (Main), tìm mục **Environment variables**.
4. Giữ nguyên giá trị `ValueFrom` (ARN của Secrets Manager), chỉ cập nhật cột **Key** theo bảng ánh xạ sau:

| Key hiện tại | Đổi thành |
| :--- | :--- |
| `host` | `DB_HOST` |
| `dbname` | `DB_NAME` |
| `username` | `DB_USER` |
| `password` | `DB_PASS` |

> **Lưu ý:** Giữ nguyên các biến khác như `engine`, `port`, `dbInstanceIdentifier`, Image URI và Port mappings.
![Giao diện giá trị của Environment Variables](images/EnvironmentVariables.png)
![Giao diện giá trị của Environment Variables](images/EnvironmentVariables_2.png)
5. Cuộn xuống cuối trang và nhấn **Create** để tạo revision mới (ví dụ: `RakiBookery-task:3`).
6. Kiểm tra lại tại tab **Environment and secrets** trong chi tiết Container để xác nhận các biến đã được cập nhật chính xác.
   ![Giao diện kiểm tra giá trị Environment and secrets](images/CheckEnvironmentVariables.png)

### Bước 2: Cập nhật tên Database trong AWS Secrets Manager

Mặc định khi khởi tạo RDS, trường `dbname` trong Secret có thể mang giá trị `mysql`. Chúng ta cần trỏ chính xác về database của ứng dụng là `bookstoredb`.

1. Mở **AWS Secrets Manager** $\rightarrow$ chọn secret **fcj/sprint1/rds/mysql**.
2. Tại mục **Secret value**, nhấn **Edit**.
   ![Giao diện chỉnh sửa Secret value](images/updatedb_img5.png)
3. Chuyển sang tab **Plaintext**, tìm trường `"dbname"` và đổi giá trị từ `"mysql"` thành `"bookstoredb"`.
4. Nhấn **Save** để lưu thay đổi.
   ![Giao diện lưu thay đổi dbname](images/updatedb_3_img6.png)

### Bước 3: Cập nhật ECS Service để áp dụng cấu hình mới

1. Truy cập **Amazon ECS** $\rightarrow$ **Clusters** $\rightarrow$ **RakiBookery-cluster** $\rightarrow$ tab **Services**.
2. Chọn **RakiBookery-task-service-01** và nhấn **Update service**.
   ![Giao diện cập nhật ECS Service](images/updateECS_img7.png)
3. Trong mục **Deployment configuration**:
   - **Task definition revision:** Chọn revision mới nhất (ví dụ: `3 (LATEST)`).
   - **Desired tasks:** Đảm bảo giá trị là `1` (nếu đang là `0` thì đổi thành `1`).
   - Tích chọn **Force new deployment** nếu bạn vừa cập nhật giá trị bên trong Secrets Manager mà không tạo revision mới.
   ![Giao diện cấu hình Deployment](images/updateECS_2_img8.png)
4. Nhấn **Update** và chờ cho đến khi Task mới chuyển sang trạng thái **Running**.
   ![Giao diện Task Running sau khi update](images/updateECS_3_img9.png)

---

## Phần 2: Import Dữ liệu vào Private RDS thông qua ECS Fargate Task

Để đảm bảo an toàn bảo mật, RDS được đặt trong Private Subnet và không mở cổng `3306` ra Internet. Thay vì mở public access, chúng ta sẽ đóng gói file `bookstoredb.sql` vào một Docker image tạm thời và chạy một **One-off ECS Fargate Task** nằm cùng VPC với RDS để thực thi lệnh import.

### Bước 1: Chuẩn bị môi trường Docker trên máy Local

1. Đảm bảo máy tính đã cài đặt WSL 2 và **Docker Desktop** (chọn chế độ sử dụng WSL 2).
2. Mở **PowerShell** và kiểm tra Docker hoạt động:
   ```powershell
   docker --version
   docker info
   ```

### Bước 2: Tạo Dockerfile và Build Image chứa dữ liệu SQL

1. Di chuyển vào thư mục gốc của dự án và tạo thư mục con `db-import`:
   ```powershell
   cd D:\Documents\AWS_FCAJ\RakiBookery
   mkdir db-import
   ```
2. Tạo `Dockerfile` bên trong thư mục `db-import`:
   ```powershell
   @'
   FROM mysql:8.0
   WORKDIR /import
   COPY bookstoredb.sql /import/bookstoredb.sql
   '@ | Set-Content -Encoding ascii .\db-import\Dockerfile
   ```
3. Sao chép file `bookstoredb.sql` vào thư mục `db-import` và kiểm tra:
   ```powershell
   Copy-Item .\bookstoredb.sql .\db-import\bookstoredb.sql
   Get-ChildItem .\db-import
   ```
   ![Kiểm tra thư mục db-import](images/docker_img10.png)
4. Build Docker image với tag `rakibookery-db-import:latest`:
   ```powershell
   docker build -t rakibookery-db-import:latest .\db-import
   ```

### Bước 3: Đẩy (Push) Image lên Amazon ECR

1. Đăng nhập Docker vào Amazon ECR (thay đổi Account ID và Region nếu cần):
   ```powershell
   aws ecr get-login-password --region ap-southeast-1 | docker login --username AWS --password-stdin 147956724787.dkr.ecr.ap-southeast-1.amazonaws.com
   ```
   ![Đăng nhập Docker vào ECR thành công](images/loginDockerToECR_img11.png)
2. Gắn tag ECR riêng (`db-import`) để không ghi đè lên image của ứng dụng web:
   ```powershell
   docker tag rakibookery-db-import:latest 147956724787.dkr.ecr.ap-southeast-1.amazonaws.com/rakibookery:db-import
   docker images
   ```
   ![Giao diện Docker images sau khi tag](images/DockerImage_img12.png)
3. Push image lên ECR:
   ```powershell
   docker push 147956724787.dkr.ecr.ap-southeast-1.amazonaws.com/rakibookery:db-import
   ```
   ![Giao diện push image lên ECR](images/PushImage_img13.png)
4. Kiểm tra trên **AWS Console $\rightarrow$ Amazon ECR $\rightarrow$ rakibookery** để xác nhận tag `db-import` đã xuất hiện.
   ![Kiểm tra tag db-import trên ECR](images/CheckImage_img14.png)

### Bước 4: Tạo Task Definition riêng cho tác vụ Import DB

1. Truy cập **Amazon ECS $\rightarrow$ Task definitions $\rightarrow$ Create new task definition**.
   ![Giao diện tạo Task Definition mới](images/createTask_img15.png)
2. Cấu hình thông số cơ bản:
   - **Task definition family:** `RakiBookery-db-import-task`
   - **Launch type:** `AWS Fargate`
   - **OS/Architecture:** `Linux/X86_64`
   - **CPU / Memory:** `0.25 vCPU` / `0.5 GB`
   - **Task role:** Để trống (`None`)
   - **Task execution role:** `ecsTaskExecutionRole`
   ![Cấu hình thông số cơ bản Task Definition](images/createTask_img16.png)
3. Cấu hình **Container - 1**:
   - **Name:** `db-import`
   - **Image URI:** Chọn image `rakibookery:db-import` từ ECR.
   ![Cấu hình Container Name và Image URI](images/img17.png)
   ![Cấu hình Container tiếp theo](images/img18.png)
   ![Cấu hình Container tiếp theo](images/img19.png)
   - **Port mappings:** Nhấn **Remove** để xóa cổng `80` (tác vụ chạy ngầm không cần mở cổng).
   ![Xóa Port mapping](images/img20.png)
4. Thêm **Environment variables** trỏ tới Secrets Manager tương tự như ứng dụng chính (`DB_HOST`, `DB_NAME`, `DB_USER`, `DB_PASS`).
   ![Cấu hình Environment variables từ Secrets Manager](images/img21.png)
5. Cấu hình **Logging** (CloudWatch Logs) với log group `/ecs/RakiBookery-db-import-task`.
   ![Cấu hình CloudWatch Logging](images/img22.png)
6. Tại mục **Docker configuration**, nhập **Entry point** là `sh,-c` và **Command** để nạp dữ liệu từ `/import/bookstoredb.sql` vào RDS:
   ```bash
   mysql -h "$DB_HOST" -P 3306 -u "$DB_USER" -p"$DB_PASS" "$DB_NAME" < /import/bookstoredb.sql
   ```
   *(Sau khi import xong, bạn có thể tạo một revision mới với lệnh `SELECT` để kiểm tra số lượng bản ghi trong database).*
7. Nhấn **Create** để hoàn tất Task Definition.
   ![Hoàn tất tạo Task Definition](images/img23.png)

### Bước 5: Chạy One-off Task để thực thi Import và Kiểm tra Logs

1. Vào **Amazon ECS $\rightarrow$ Clusters $\rightarrow$ RakiBookery-cluster $\rightarrow$ tab Tasks $\rightarrow$ Run new task**.
   ![Giao diện Run new task](images/img24.png)
2. Chọn **Task definition family** là `RakiBookery-db-import-task` (bản `Latest`), **Launch type** là `Fargate`.
   ![Chọn Task definition family](images/img25.png)
   ![Chọn Launch type Fargate](images/img26.png)
3. Tại mục **Networking**:
   - **VPC:** `RakiBookery-vpc-01`
   - **Subnets:** Chọn 2 subnet giống với ECS Service của web.
   - **Security group:** Chọn `RakiBookery-fargate-sg` (Security Group đã được RDS Inbound Rule cho phép cổng `3306`).
   - **Public IP:** `Turned on` (giúp task tải image từ ECR và đọc Secret nếu VPC không có NAT Gateway/VPC Endpoint).
   ![Cấu hình Networking cho Task](images/img27.png)
4. Nhấn **Create**. Task sẽ chạy lệnh import và tự động chuyển sang trạng thái **Stopped** sau khi hoàn tất.
5. Mở chi tiết Task vừa chạy $\rightarrow$ tab **Logs** để kiểm tra kết quả. Nếu chạy lệnh kiểm tra dữ liệu, log sẽ hiển thị danh sách các bảng (`book`, `category`, `user`,...) và tổng số bản ghi đã được nạp thành công.
   ![Logs Task import - kết quả](images/img31.png)
   ![Logs Task import - chi tiết](images/img28.png)
   ![Logs Task import - bảng dữ liệu](images/img29.png)
   ![Logs Task import - xác nhận hoàn tất](images/img30.png)
   

---

## Phần 3: Xử lý Lỗi Phân biệt Hoa/Thường (Case-Sensitivity) & Cập nhật Mã nguồn PHP

Khi chạy MySQL trên môi trường Linux (Amazon RDS), tên bảng mặc định **phân biệt chữ hoa và chữ thường** (`lower_case_table_names=0`). Trong khi đó, các bảng trong file SQL được tạo với tên viết thường (`book`, `category`, `order`,...), nhưng một số câu truy vấn trong code PHP lại viết hoa chữ cái đầu (`Book`, `Order`,...).

### Bước 1: Chuẩn hóa tên bảng trong mã nguồn PHP

Kiểm tra thư mục `src/app/models/` và cập nhật toàn bộ tên bảng trong các câu truy vấn SQL sang chữ thường cho 4 file sau:
- `src/app/models/Book.php`
- `src/app/models/Order.php`
- `src/app/models/Address.php`
- `src/app/models/StatisticsModel.php`

### Bước 2: Build và Push Image ứng dụng PHP đã sửa lỗi

1. Tại thư mục gốc dự án (`D:\Documents\AWS_FCAJ\RakiBookery`), build lại Docker image với tag mới:
   ```powershell
   docker build -t rakibookery:rds-casefix-20261030 .
   docker images rakibookery
   ```
   ![Build Docker image với tag rds-casefix](images/img32.png)
2. Đăng nhập ECR, gắn tag và đẩy image mới lên repository:
   ```powershell
   aws ecr get-login-password --region ap-southeast-1 | docker login --username AWS --password-stdin 147956724787.dkr.ecr.ap-southeast-1.amazonaws.com
   docker tag rakibookery:rds-casefix-20261030 147956724787.dkr.ecr.ap-southeast-1.amazonaws.com/rakibookery:rds-casefix-20261030
   docker push 147956724787.dkr.ecr.ap-southeast-1.amazonaws.com/rakibookery:rds-casefix-20261030
   ```
   ![Đăng nhập ECR thành công](images/img33.png)
   ![Gắn tag image mới](images/img34.png)
   ![Push image lên ECR](images/img35.png)
3. Xác nhận tag `rds-casefix-20261030` đã có trên Amazon ECR.
   ![Xác nhận tag trên Amazon ECR](images/img36.png)

### Bước 3: Triển khai Image mới lên ECS Service

1. Vào **Amazon ECS $\rightarrow$ Task definitions $\rightarrow$ RakiBookery-task $\rightarrow$ Create new revision**.
   ![Tạo revision mới cho Task Definition](images/img37.png)
2. Cập nhật **Image URI** của container chính thành `147956724787.dkr.ecr.ap-southeast-1.amazonaws.com/rakibookery:rds-casefix-20261030`.
   ![Cập nhật Image URI](images/img38.png)
3. Giữ nguyên các biến môi trường Secrets Manager (`DB_HOST`, `DB_NAME`, `DB_USER`, `DB_PASS`) và IAM Roles, nhấn **Create** (tạo ra revision mới, ví dụ `RakiBookery-task:5`).
   ![Giữ nguyên biến môi trường](images/img39.png)
   ![Xác nhận cấu hình IAM Roles](images/img40.png)
   ![Nhấn Create tạo revision mới](images/img41.png)
   <!-- [img42] -->
4. Vào **Clusters $\rightarrow$ RakiBookery-cluster $\rightarrow$ Services $\rightarrow$ RakiBookery-task-service-01 $\rightarrow$ Update service**.
   ![Giao diện Update ECS Service](images/img43.png)
5. Chọn **Task definition revision** là bản `Latest` (`5`) và nhấn **Update**. Chờ quá trình deployment hoàn tất.
   ![Chọn Task definition revision Latest](images/img44.png)
   ![Deployment hoàn tất](images/img45.png)

---

## Phần 4: Kiểm tra Kết nối & Xác minh Hoạt động Website

### Bước 1: Kiểm tra Health Check của Target Group & Truy cập qua ALB

1. Mở **EC2 $\rightarrow$ Target Groups $\rightarrow$ RakiBookery-tg**, kiểm tra tab **Targets** để đảm bảo trạng thái là **Healthy**.
2. Mở **EC2 $\rightarrow$ Load Balancers $\rightarrow$ RakiBookery-alb**, sao chép **DNS name** và truy cập qua trình duyệt (`http://<ALB-DNS-name>`).

### Bước 2: Kiểm tra qua CloudFront & Xử lý tài nguyên tĩnh (CSS/JS) nếu thiếu

1. Mở **CloudFront $\rightarrow$ Distributions $\rightarrow$ RakiBookery-cloudfront**, sao chép **Distribution domain name** (dạng `xxxxx.cloudfront.net`) và mở trên trình duyệt.
   ![Truy cập website qua CloudFront](images/img48.png)
2. Nếu giao diện hiển thị thiếu CSS/JS (kiểm tra bằng `F12` $\rightarrow$ tab `Network` thấy lỗi `403` cho các file `.css`/`.js`):
   ![Lỗi 403 cho CSS/JS](images/img49.png)
   <!-- [img50]
   [img51]
   [img52] 
   !-- - Kiểm tra **Origin** của CloudFront để xác nhận bucket S3 lưu trữ static web (ví dụ: `rakibookery-static-web-246111`).
   [img53]
   [img54]
   - Truy cập **Amazon S3**, mở bucket trên và upload thư mục `client/` (chứa `css/` và `js/`) vào bên trong thư mục `assets/` để đạt cấu trúc `assets/client/css/...` và `assets/client/js/...`.
   [img55]
   [img56]
   [img57]
   [img58]
   [img59]
   [img60]
   [img61]
   [img62]
   [img63]
   - Quay lại trang CloudFront và nhấn `Ctrl + Shift + R` để tải lại trang bỏ qua cache.
   [img64]
   [img65]
   [img66] -->

### Bước 3: Xác minh Truy vấn Dữ liệu từ RDS

Trên giao diện website đã kết nối thành công với RDS, thực hiện kiểm tra các tính năng tương tác cơ sở dữ liệu:
1. **Hiển thị danh sách sản phẩm:** Trang chủ tải đầy đủ 20 đầu sách kèm hình ảnh, tác giả, thể loại và giá bán từ RDS.
   ![Danh sách sản phẩm từ RDS](images/img48.png)
2. **Tìm kiếm & Lọc theo tên/thể loại:** Nhập từ khóa (ví dụ: `midnight`, `harry`) và nhấn **Tìm kiếm**.
   ![Tìm kiếm sản phẩm theo tên](images/img68.png)
3. **Lọc theo khoảng giá:** Kéo thanh trượt giá hoặc nhập khoảng giá (`Từ` - `Đến`) và kiểm tra kết quả trả về.
   ![Lọc theo khoảng giá](images/img69.png)
4. **Sắp xếp dữ liệu:** Thử chức năng sắp xếp theo `Giá: Thấp đến cao` và `Giá: Cao đến thấp`.
   ![Sắp xếp theo giá tăng dần](images/img70.png)
   ![Sắp xếp theo giá giảm dần](images/img71.png)

> **Dọn dẹp sau thực hành:** Sau khi hoàn tất triển khai và kiểm tra, hãy tắt **Docker Desktop** trên máy cá nhân (chuột phải vào biểu tượng Docker ở khay hệ thống $\rightarrow$ **Quit Docker Desktop**) để giải phóng tài nguyên hệ thống.
> ![Tắt Docker Desktop](images/img74.png)
