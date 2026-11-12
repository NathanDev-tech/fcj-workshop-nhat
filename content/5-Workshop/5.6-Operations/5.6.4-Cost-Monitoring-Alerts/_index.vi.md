---
title : "Cost Monitoring & Alerts"
date : 2024-01-01
weight : 4
chapter : false
pre : " <b> 5.6.4 </b> "
---

### 5.5.4.1. Thiết lập AWS Budget và Ngân sách hàng tháng

Mở **AWS Billing and Cost Management Console** → chọn **Budgets** và chọn **Create budget**, sau đó chọn loại **Cost budget** và thiết lập các thông số sau:

**Budget name:** Nhập tên `raki-monthly-budget`.

**Period:** Chọn **Monthly** để theo dõi ngân sách theo chu kỳ từng tháng.

**Budget renewal type:** Chọn **Recurring budget** (ngân sách tự động làm mới vào ngày đầu tiên của mỗi tháng).

**Start month:** Chọn tháng bắt đầu áp dụng (ví dụ: `Jul 2026`).

**Budgeting method:** Chọn **Fixed** để theo dõi dựa trên một số tiền cố định hàng tháng.

**Enter your budgeted amount ($):** Nhập `40.00` (đặt hạn mức ngân sách $40.00/tháng).

![image045.jpg](/images/5-Workshop/5.5-Raki-Operations/image045.jpg)

### 5.5.4.2. Cấu hình Phạm vi Ngân sách (Budget Scope) và Thẻ Tag

**Budget scope:** Chọn **Filter specific AWS cost dimensions** để lọc chi phí cho riêng dự án thay vì theo dõi toàn bộ tài khoản AWS.

**Filters:** Chọn lọc theo **Tag: Project included (1)** với giá trị `RakiBookery`.

**Advanced options:** Giữ **Aggregate costs by** là **Unblended costs**.

**Tags (optional):** Khai báo các thẻ quản lý tài nguyên cho Budget:

- **Key:** `Project` | **Value - optional:** `RakiBookery`
- **Key:** `ManagedBy` | **Value - optional:** `CloudFormation`

![image046.jpg](/images/5-Workshop/5.5-Raki-Operations/image046.jpg)

### 5.5.4.3. Thiết lập Cảnh báo (Alerts) và Kiểm tra Trang Chi tiết Ngân sách

Mở **Budgets** → chọn `raki-monthly-budget` để xem thông tin chi tiết và kiểm tra trạng thái **Budget health** cũng như cấu hình các ngưỡng cảnh báo (**Alerts**):

**Budget health:** Trạng thái hiển thị **Healthy** khi chi phí hiện tại (**Current vs. budgeted**) chưa vượt quá hạn mức $40.00.

**Cấu hình 3 ngưỡng cảnh báo (Alerts):**

- **Actual cost > 50%:** Kích hoạt cảnh báo khi chi phí thực tế vượt quá **50%** ($20.00) so với ngân sách $40.00.
- **Actual cost > 70%:** Kích hoạt cảnh báo khi chi phí thực tế vượt quá **70%** ($28.00) so với ngân sách $40.00.
- **Forecasted cost > 90%:** Kích hoạt cảnh báo dự báo khi chi phí ước tính đến cuối tháng vượt quá **90%** ($36.00) so với ngân sách $40.00.

![image047.jpg](/images/5-Workshop/5.5-Raki-Operations/image047.jpg)