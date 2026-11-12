---
title : "Cấu hình CloudFront + Route 53"
date : 2024-01-01
weight : 4
chapter : false
pre : " <b> 5.4.4 </b> "
---

# Hướng dẫn cấu hình CloudFront + Route 53

## Bước 1 — Kiểm tra S3 Static Web (AWS-003)

1. Vào **Amazon S3**.
2. Xác định bucket chứa file tĩnh của website RakiBookery và kiểm tra xem đã upload các file CSS, JS, hình ảnh hay chưa.
3. Giữ **Block Public Access** ở trạng thái BẬT. Không cần bật tính năng *S3 Static Website Hosting* nếu bạn dự định dùng CloudFront với S3 REST origin và OAC.

![Kiểm tra S3 Static Web](url_hinh_anh_cua_ban_1)

## Bước 2 — Tạo CloudFront Distribution

1. Mở **AWS CloudFront Console**.
2. Nhấn **Create distribution**.
3. Nếu AWS hỏi loại distribution, hãy chọn loại dành cho website/web application.
4. Đến phần **Origin**, chọn S3 bucket `rakibookery-static-web-246111`.
5. Nếu có phần **Origin access**, chọn **Origin access control (OAC)** hoặc tùy chọn được đề xuất để truy cập S3 private.

**Các lựa chọn cấu hình quan trọng:**

| Mục | Giá trị |
| --- | --- |
| **Origin** | S3 Static Web bucket |
| **Origin access** | Origin Access Control (OAC) |
| **Viewer protocol policy** | Redirect HTTP to HTTPS |
| **Price class** | Chọn theo ngân sách |
| **WAF** | Không bắt buộc cho bài tập |

> *Lưu ý:* AWS có thể yêu cầu tạo OAC mới; hãy chọn **Create new OAC**.

![Cấu hình CloudFront Distribution](url_hinh_anh_cua_ban_2)

## Bước 3 — Cấp quyền CloudFront đọc S3

Sau khi tạo Distribution, hãy mở thông báo hướng dẫn cập nhật S3 Bucket Policy.
1. Nếu AWS cung cấp nút **Copy policy**, hãy sao chép policy đó.
2. Vào **S3** → Chọn **Bucket** → **Permissions** → **Bucket policy** → **Edit**.
3. Dán policy để cho phép CloudFront Distribution đọc object bằng hành động `s3:GetObject`.

> *Lưu ý:* Policy nên giới hạn bằng ARN của Distribution cụ thể, tuyệt đối không cấp quyền đọc công khai (public read) cho mọi người.

## Bước 4 — Thêm ALB làm Origin thứ hai

### 4.1. Thêm ALB Origin
1. Vào AWS CloudFront Console.
2. Chọn Distribution `RakiBookery-cloudfront` vừa tạo.
3. Chọn tab **Origins**.
4. Nhấn **Create origin**.

![Thêm ALB Origin](url_hinh_anh_cua_ban_3)

### 4.2. Điền cấu hình ALB Origin
Ở màn hình Create origin, chọn các giá trị sau:

| Trường | Giá trị |
| --- | --- |
| **Origin type** (nếu có) | Elastic Load Balancer |
| **Origin domain** | `RakiBookery-alb-1141293019.ap-southeast-1.elb.amazonaws.com` |
| **Origin name / ID** | RakiBookery-ALB-Origin |
| **Origin path** | *Để trống* |
| **Protocol** | HTTP only |
| **HTTP port** | 80 |
| **HTTPS port** | *Giữ mặc định nếu không sử dụng* |
| **Origin Shield** | Disabled |
| **Connection attempts / timeout**| *Giữ mặc định* |

> **Vì sao chọn HTTP only?** ALB của bạn hiện chỉ có listener `HTTP:80`, chưa có `HTTPS:443`. Nếu chọn HTTPS only, CloudFront sẽ không kết nối được với ALB theo cấu hình hiện tại. Đây là cấu hình để thực hành. Khi triển khai hệ thống thật, nên dùng HTTPS giữa CloudFront và ALB.

![Cấu hình ALB Origin chi tiết](url_hinh_anh_cua_ban_4)

### 4.3. Tạo Origin
Nhấn **Create origin**. Sau khi tạo, tab Origins dự kiến sẽ có hai Origin như sau:

| Origin | Mục đích |
| --- | --- |
| **S3 `rakibookery-static-web-246111`** | Phục vụ file tĩnh (CSS, JS, ảnh) |
| **RakiBookery-ALB-Origin** | Ứng dụng/Website PHP trên Fargate |


## Bước 5 — Cấu hình Cache Behaviors

Vào CloudFront → Distribution → **Behaviors**. Thiết lập theo ví dụ dưới đây:

| Path pattern | Origin | Cache policy |
| --- | --- | --- |
| `/assets/*` | S3 Static Web | CachingOptimized |
| `Default (*)` | RakiBookery-alb | CachingDisabled |

> **Lưu ý cấu hình Behavior:**
> * Với behavior mặc định chuyển tới ALB, hãy chọn các HTTP methods phù hợp với ứng dụng PHP (ví dụ: `GET, HEAD, OPTIONS, PUT, POST, PATCH, DELETE` nếu cần).
> * Cấu hình chuyển tiếp cookies, query strings và headers theo yêu cầu của ứng dụng; có thể bắt đầu với managed origin request policy `AllViewer` khi phù hợp.
> * **Quan trọng:** Đường dẫn `/assets/*` chỉ đúng nếu URL thực tế của website dùng đường dẫn đó và object trong S3 có key tương ứng. Nếu ứng dụng dùng `/css/*`, `/js/*` hoặc `/images/*`, bạn cần tạo behavior theo các đường dẫn thực tế đó.
> * Không nên cache HTML động, trang đăng nhập hoặc giỏ hàng bằng chính sách cache tĩnh.

![Tạo Cache Behavior](url_hinh_anh_cua_ban_5)
![Cấu hình Cache Behavior S3](url_hinh_anh_cua_ban_6)

## Bước 6 — Kiểm tra CloudFront

1. Chờ Distribution chuyển sang trạng thái **Deployed**.
2. AWS sẽ cung cấp domain có dạng: `dxxxxxxxxxxxxx.cloudfront.net`
3. Thử truy cập một file tĩnh đã có trong S3, ví dụ: 
   `https://dxxxxxxxxxxxxx.cloudfront.net/assets/logo.png`
   *(Thay domain và đường dẫn bằng giá trị thực tế của bạn).*

Nếu file trả về mã `HTTP 200`, CloudFront đã phân phối nội dung tĩnh thành công. Nếu gặp lỗi `403 AccessDenied`, hãy kiểm tra lại cấu hình OAC, S3 Bucket Policy và object key.

![Kiểm tra truy cập CloudFront](url_hinh_anh_cua_ban_7)

## Bước 7 — Route 53 (Tùy chọn)

*Chỉ thực hiện nếu nhóm có domain riêng.*

1. Tạo hoặc sử dụng Hosted Zone trong **Route 53**.
2. Cấu hình chứng chỉ HTTPS bằng **ACM** (tại region `us-east-1` bắt buộc cho CloudFront).
3. Thêm domain vào cấu hình Distribution.
4. Tạo **DNS Alias record** loại A (có thể thêm AAAA nếu dùng IPv6) trỏ đến CloudFront.

Nếu nhóm chưa có domain, việc dùng domain mặc định `cloudfront.net` là đủ cho phần tùy chọn của bài thực hành AWS-007.