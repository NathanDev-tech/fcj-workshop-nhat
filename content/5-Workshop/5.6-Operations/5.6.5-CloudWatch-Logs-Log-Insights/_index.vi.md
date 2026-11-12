---
title : "CloudWatch Logs & Log Insights"
date : 2024-01-01
weight : 5
chapter : false
pre : " <b> 5.6.5 </b> "
---

Sau khi hoàn thành phần này, hệ thống sẽ đáp ứng các yêu cầu sau:

- Thiết lập thời gian lưu trữ Log là **30 ngày**
- Tạo Log Group cho **Application Logs**
- Khởi tạo Log Group phục vụ lưu trữ và quản lý nhật ký hệ thống
- Thực hiện truy vấn Log bằng **CloudWatch Logs Insights**
- Kích hoạt **VPC Flow Logs** để giám sát lưu lượng mạng trong VPC
- Kiểm tra khả năng thu thập và phân tích Log

### 5.5.5.1. Các bước thực hiện

#### Bước 1. Thiết lập thời gian lưu trữ Log (Retention Policy)

CloudWatch mặc định lưu trữ Log vô thời hạn. Để tối ưu chi phí lưu trữ, cấu hình thời gian lưu Log là **30 ngày**.

1. Đăng nhập **AWS Management Console**.
2. Truy cập dịch vụ **CloudWatch**.
3. Chọn **Log groups**.

![Hình 78.](/images/5-Workshop/5.5-Raki-Operations/image078.png)

4. Chọn Log Group cần cấu hình.
5. Chọn **Actions → Edit retention setting**.

![Hình 79.](/images/5-Workshop/5.5-Raki-Operations/image079.png)

6. Trong mục **Retention setting**, chọn **30 Days**.
7. Nhấn **Save** để lưu cấu hình.

![Hình 80.](/images/5-Workshop/5.5-Raki-Operations/image080.png)

> **Lưu ý:** Thực hiện tương tự cho tất cả Log Group của hệ thống để đảm bảo chính sách lưu trữ được áp dụng thống nhất.

---

#### Bước 2. Tạo Log Group

Log Group được sử dụng để lưu trữ và quản lý Log của các dịch vụ AWS như ECS, Lambda hoặc VPC Flow Logs.

1. Trong **CloudWatch**, chọn **Log groups**.
2. Nhấn **Create log group**.


3. Nhập tên Log Group, ví dụ:

```
/ecs/raki-task-be
```

hoặc

```
/ecs/raki-task-fe
```

4. Giữ nguyên các thiết lập mặc định.

![alt text](image.png)


5. Nhấn **Create** để hoàn tất.

6. Sau khi tạo thành công, Log Group sẽ xuất hiện trong danh sách.

![alt text](image-1.png)

---

#### Bước 3. Cấu hình ECS ghi Log vào CloudWatch

Để các container của ECS ghi Log trực tiếp lên CloudWatch, cần cấu hình **awslogs Log Driver** trong Task Definition.

1. Truy cập **Amazon ECS**.
2. Chọn **Task Definitions** và mở Task Definition của ứng dụng.
3. Chọn **Create new revision**.
4. Trong phần **Container Definitions**, cấu hình:

- **Log driver:** `awslogs`
- **Log group:** `/ecs/raki-backend`
- **AWS Region:** `ap-southeast-1`
- **Stream prefix:** `ecs`

![alt text](image-3.png)


5. Lưu Task Definition mới.
6. Cập nhật ECS Service sử dụng Revision vừa tạo.
7. Sau khi Deployment hoàn tất, Log sẽ tự động được ghi vào CloudWatch.

![Hình 106.](/images/5-Workshop/5.5-Raki-Operations/image106.png)

---

#### Bước 4. Truy vấn Log bằng CloudWatch Logs Insights

CloudWatch Logs Insights cho phép tìm kiếm, thống kê và phân tích Log theo thời gian thực.

1. Truy cập **CloudWatch → Logs Insights**.
2. Chọn Log Group cần phân tích.
3. Nhập truy vấn Log Insights hoặc sử dụng truy vấn đã lưu.
4. Nhấn **Run query** để thực hiện truy vấn.

Có thể sử dụng để:
- Tìm các bản ghi chứa Exception.
- Thống kê số lượng Error theo thời gian.
- Tìm HTTP Status Code 500.
- Phân tích Request có thời gian xử lý lớn.
- Tìm các IP truy cập nhiều nhất.
5. Nếu cần sử dụng lại, chọn **Save query**.
6. Đặt tên truy vấn và lưu lại.

![Hình 105.](/images/5-Workshop/5.5-Raki-Operations/image105.png)

![Hình 107.](/images/5-Workshop/5.5-Raki-Operations/image107.png)

![Hình 108.](/images/5-Workshop/5.5-Raki-Operations/image108.png)

---

#### Bước 5. Cấu hình VPC Flow Logs

VPC Flow Logs giúp ghi nhận toàn bộ lưu lượng mạng đi vào và đi ra khỏi VPC, hỗ trợ phân tích sự cố kết nối và kiểm tra lưu lượng truy cập.

1. Truy cập dịch vụ **Amazon VPC**.
2. Chọn **Your VPCs**.
3. Chọn VPC của hệ thống.

![Hình 98.](/images/5-Workshop/5.5-Raki-Operations/image098.png)

4. Chọn tab **Flow logs**.
5. Nhấn **Create flow log**.
6. Thiết lập các thông số:

- **Filter:** All
- **Destination:** Send to CloudWatch Logs
- **Destination log group:** Chọn Log Group đã tạo
- **IAM Role:** Chọn Role có quyền ghi Log vào CloudWatch


7. Nhấn **Create flow log** để hoàn tất.

![Hình 99.](/images/5-Workshop/5.5-Raki-Operations/image099.png)

![Hình 100.](/images/5-Workshop/5.5-Raki-Operations/image100.png)


![Hình 101.](/images/5-Workshop/5.5-Raki-Operations/image101.png)

![Hình 102.](/images/5-Workshop/5.5-Raki-Operations/image102.png)


![Hình 103.](/images/5-Workshop/5.5-Raki-Operations/image103.png)

---

#### Bước 6. Kiểm tra Log và truy vấn

Sau khi hoàn tất cấu hình, tiến hành kiểm tra khả năng thu thập và phân tích Log của hệ thống.

1. Truy cập **CloudWatch → Logs Insights**.
2. Chọn Log Group của ứng dụng.
3. Thực hiện các truy vấn đã tạo.
4. Kiểm tra kết quả:

- Log được ghi nhận đầy đủ.
- Có thể lọc theo khoảng thời gian.
- Truy vấn trả về đúng dữ liệu.
- Có thể tìm kiếm theo từ khóa hoặc mức độ Log.

5. Xác nhận Log mới tiếp tục được cập nhật khi ứng dụng hoạt động.
6. Hoàn tất cấu hình CloudWatch Logs và Log Insights.