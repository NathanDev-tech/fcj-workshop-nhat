---
title : "Kiểm thử End-to-End"
date : 2024-01-01
weight : 6
chapter : false
pre : " <b> 5.5.6 </b> "
---

Sau khi hoàn thành phần này, hệ thống sẽ được xác nhận:

- Toàn bộ luồng người dùng từ Frontend qua ALB đến Backend (ECS) hoạt động đúng
- Các dịch vụ lưu trữ và CDN (S3, CloudFront, RDS MySQL) phục vụ đúng dữ liệu
- Không có lỗi, liên kết hỏng hoặc sự cố dữ liệu ở các tính năng cốt lõi
- Tích hợp serverless và demo Bedrock hoạt động ổn định

### 5.5.6.1. Các kịch bản kiểm thử

#### Kịch bản 1. Luồng đăng ký → đăng nhập

| Mục | Nội dung |
|-----|----------|
| **Mục tiêu** | Đảm bảo luồng tạo tài khoản và đăng nhập thành công |
| **Kết quả mong đợi** | Người dùng tạo tài khoản thành công, đăng nhập được và hệ thống cập nhật đúng trạng thái phiên |

**Quy trình thực hiện:**

1. Truy cập website qua CloudFront / ALB.
2. Chọn **Đăng ký** và điền đầy đủ thông tin form.
3. Sử dụng tài khoản vừa tạo để thực hiện **Đăng nhập**.
4. Kiểm tra thông tin người dùng hiển thị đúng sau khi đăng nhập.

**Trường hợp ngoại lệ:**

- TH1: Điền sai định dạng email / mật khẩu quá ngắn → form báo lỗi validation.
- TH2: Đăng nhập thất bại do mật khẩu không chính xác.
- TH3: Tài khoản đã tồn tại → hệ thống thông báo trùng email/username.

#### Kịch bản 2. Duyệt sách và xem chi tiết sản phẩm

| Mục | Nội dung |
|-----|----------|
| **Mục tiêu** | Kiểm thử khả năng tải danh sách sách và lấy thông tin chi tiết từ backend + RDS |
| **Kết quả mong đợi** | Danh sách sách hiển thị đầy đủ, hình ảnh tải từ S3 Media, thông tin mô tả và giá chính xác |

**Quy trình thực hiện:**

1. Tại trang chủ / trang danh mục, lướt xem danh sách sách.
2. Chọn một cuốn sách để xem trang chi tiết.
3. Kiểm tra hình ảnh, tên sách, mô tả, giá và thông tin liên quan hiển thị đúng.

**Trường hợp ngoại lệ:**

- TH1: Hình ảnh không tải được do lỗi kết nối S3 Media.
- TH2: Dữ liệu sách trống hoặc lỗi kết nối RDS.

#### Kịch bản 3. Thêm vào giỏ hàng → Đặt hàng + Áp dụng khuyến mãi (Promo)

| Mục | Nội dung |
|-----|----------|
| **Mục tiêu** | Kiểm thử toàn bộ luồng mua hàng và tính năng khuyến mãi |
| **Kết quả mong đợi** | Thêm sách vào giỏ thành công, áp dụng mã khuyến mãi đúng, tạo đơn hàng thành công và trạng thái được cập nhật |

**Quy trình thực hiện:**

1. Chọn sách → nhấn **Thêm vào giỏ hàng**.
2. Vào giỏ hàng, kiểm tra số lượng và tổng tiền.
3. Áp dụng mã khuyến mãi (nếu có).
4. Tiến hành đặt hàng (Checkout).
5. Kiểm tra đơn hàng được tạo và trạng thái hiển thị đúng (có thể nhận thông báo qua SNS).

**Trường hợp ngoại lệ:**

- TH1: Mã khuyến mãi không hợp lệ hoặc đã hết hạn.
- TH2: Lỗi khi tạo đơn hàng do mất kết nối RDS.

#### Kịch bản 4. Upload ảnh

| Mục | Nội dung |
|-----|----------|
| **Mục tiêu** | Đảm bảo chức năng upload ảnh hoạt động và được xử lý bởi Lambda |
| **Kết quả mong đợi** | Ảnh được upload thành công lên S3 Media, Lambda xử lý (resize/tối ưu) và hiển thị đúng trên giao diện |

**Quy trình thực hiện:**

1. Đăng nhập tài khoản.
2. Thực hiện upload ảnh (ảnh sách / ảnh đại diện / ảnh liên quan đến sản phẩm).
3. Kiểm tra ảnh xuất hiện trên giao diện sau khi upload.
4. Kiểm tra file đã được lưu trên S3 Media Bucket.

**Trường hợp ngoại lệ:**

- TH1: Upload file không đúng định dạng (không phải ảnh).
- TH2: File quá lớn → hệ thống từ chối.
- TH3: Lambda xử lý ảnh thất bại (ảnh gốc vẫn được lưu).

#### Kịch bản 5. Chức năng Admin

| Mục | Nội dung |
|-----|----------|
| **Mục tiêu** | Kiểm thử các chức năng quản trị cơ bản |
| **Kết quả mong đợi** | Admin có thể xem/quản lý sách, đơn hàng và người dùng một cách bình thường |

**Quy trình thực hiện:**

1. Đăng nhập bằng tài khoản Admin.
2. Kiểm tra danh sách sách, thêm/sửa/xóa sách (nếu có).
3. Xem danh sách đơn hàng và cập nhật trạng thái.
4. Kiểm tra thông tin người dùng.

**Trường hợp ngoại lệ:**

- TH1: Tài khoản thường không có quyền truy cập trang Admin.
- TH2: Lỗi khi cập nhật dữ liệu do mất kết nối database.

#### Kịch bản 6. Demo gợi ý sách bằng Amazon Bedrock

| Mục | Nội dung |
|-----|----------|
| **Mục tiêu** | Kiểm thử chức năng gợi ý sách thông qua Bedrock |
| **Kết quả mong đợi** | Hệ thống trả về danh sách gợi ý sách hợp lý dựa trên input của người dùng |

**Quy trình thực hiện:**

1. Truy cập tính năng gợi ý sách.
2. Nhập từ khóa / sở thích.
3. Kiểm tra kết quả gợi ý trả về từ Bedrock (qua Lambda / API Gateway).

**Trường hợp ngoại lệ:**

- TH1: Bedrock timeout hoặc lỗi quyền IAM.
- TH2: Đạt giới hạn Throttling API.
- TH3: Không có kết quả phù hợp.

#### Kịch bản 7. Kiểm thử xử lý lỗi

| Mục | Nội dung |
|-----|----------|
| **Mục tiêu** | Đảm bảo cơ chế xử lý ngoại lệ thân thiện với người dùng |
| **Kết quả mong đợi** | Giao diện hiển thị thông báo lỗi rõ ràng, không làm sập ứng dụng |

**Quy trình thực hiện:**

1. Nhập dữ liệu không hợp lệ vào các form (đăng ký, đặt hàng…).
2. Mô phỏng sự cố độ trễ mạng hoặc timeout.
3. Kiểm tra thông báo lỗi hiển thị đúng và hệ thống vẫn hoạt động bình thường.

#### Kịch bản 8. Kiểm thử giao diện đa nền tảng (Mobile Responsiveness)

| Mục | Nội dung |
|-----|----------|
| **Mục tiêu** | Đảm bảo hoạt động nhất quán trên Desktop và Mobile |
| **Kết quả mong đợi** | Giao diện co giãn linh hoạt, không bị tràn viền hay che khuất thành phần quan trọng |

**Quy trình thực hiện:**

1. Dùng Developer Tools trên trình duyệt (hoặc thiết bị thực tế) để giả lập các kích thước màn hình.
2. Kiểm tra UI/UX trên cả mobile và desktop (trang chủ, chi tiết sách, giỏ hàng, form đăng nhập…).

### 5.5.6.2. Tổng kết kiểm thử

Hầu hết các kịch bản kiểm thử đều hoạt động đúng như kỳ vọng. Các trường hợp ngoại lệ đã được ghi nhận và phần lớn đã được xử lý hoặc có phương án dự phòng.

| Kịch bản                          | Kết quả   |
|-----------------------------------|-----------|
| Đăng ký → Đăng nhập               | Passed    |
| Duyệt sách + Xem chi tiết         | Passed    |
| Thêm giỏ hàng → Đặt hàng + Promo  | Passed    |
| Upload ảnh                        | Passed    |
| Chức năng Admin                   | Passed    |
| Demo Bedrock gợi ý sách           | Passed    |
| Xử lý lỗi và timeout              | Passed    |
| Mobile responsiveness             | Passed    |
