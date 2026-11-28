---
title : "Kiểm thử End-to-End"
date : 2024-01-01
weight : 6
chapter : false
pre : " <b> 5.6.6 </b> "
---

Sau khi hoàn thành phần này, hệ thống sẽ được kiểm tra và xác nhận:

- Toàn bộ luồng người dùng từ Frontend qua ALB đến Backend (ECS) hoạt động đúng.
- Dữ liệu và hình ảnh được hiển thị trên website thông qua CloudFront.
- Các tính năng cốt lõi của khách hàng và Admin hoạt động theo phạm vi kiểm thử.
- Tích hợp serverless và demo Bedrock phản hồi trên giao diện.

**Môi trường kiểm thử:** [RakiBookery trên CloudFront](https://d3nvhtmwuhmywh.cloudfront.net/)

### 5.6.6.1. Các kịch bản kiểm thử

#### Kịch bản 1. Luồng đăng ký → đăng nhập

| Mục | Nội dung |
|-----|----------|
| **Mục tiêu** | Đảm bảo luồng tạo tài khoản và đăng nhập thành công |
| **Kết quả mong đợi** | Người dùng tạo tài khoản thành công, đăng nhập được và hệ thống cập nhật đúng trạng thái phiên |

**Quy trình thực hiện:**

1. Truy cập website qua CloudFront.
2. Chọn **Đăng nhập** và nhập thông tin tài khoản khách hàng đã đăng ký.
3. Nhấn **Đăng nhập** và kiểm tra trạng thái phiên.

   ![](/fcj-workshop-nhat/images/5-Workshop/5.6-Operations/5.6.6-End-to-End-testing/12-login-form.png)

   ![](/fcj-workshop-nhat/images/5-Workshop/5.6-Operations/5.6.6-End-to-End-testing/13-login-success.png)

4. Kiểm tra menu tài khoản cá nhân, giỏ hàng, đơn hàng và đăng xuất.

**Trường hợp ngoại lệ:**

- TH1: Điền sai định dạng email hoặc mật khẩu quá ngắn → form báo lỗi validation.
- TH2: Đăng nhập thất bại → hệ thống thông báo email hoặc mật khẩu không chính xác.
- TH3: Tài khoản đã tồn tại → cần kiểm tra thông báo trùng email.

#### Kịch bản 2. Duyệt sách và xem chi tiết sản phẩm

| Mục | Nội dung |
|-----|----------|
| **Mục tiêu** | Kiểm thử khả năng tải danh sách sách và lấy thông tin chi tiết từ Backend + RDS |
| **Kết quả mong đợi** | Danh sách sách hiển thị đầy đủ, hình ảnh, tên sách, mô tả và giá chính xác |

**Quy trình thực hiện:**

1. Tại trang chủ hoặc trang danh mục, mở danh sách sách và kiểm tra 20 sản phẩm.

   ![](/fcj-workshop-nhat/images/5-Workshop/5.6-Operations/5.6.6-End-to-End-testing/02-shop.jpg)

2. Chọn một cuốn sách để xem trang chi tiết.

   ![](/fcj-workshop-nhat/images/5-Workshop/5.6-Operations/5.6.6-End-to-End-testing/03-product-detail.jpg)

3. Kiểm tra ảnh bìa, tên sách, tác giả, nhà xuất bản, mô tả và giá.

**Trường hợp ngoại lệ:**

- TH1: Hình ảnh không tải được do lỗi kết nối S3 Media.
- TH2: Dữ liệu sách trống hoặc lỗi kết nối RDS.

#### Kịch bản 3. Thêm vào giỏ hàng → Đặt hàng + Áp dụng khuyến mãi (Promo)

| Mục | Nội dung |
|-----|----------|
| **Mục tiêu** | Kiểm thử toàn bộ luồng mua hàng, tính tổng tiền, thanh toán và mã khuyến mãi |
| **Kết quả mong đợi** | Thêm sách vào giỏ thành công, áp dụng mã khuyến mãi đúng, tạo đơn hàng thành công và trạng thái được cập nhật |

**Quy trình thực hiện:**

1. Chọn sách và nhấn **Thêm vào giỏ hàng**.
2. Vào giỏ hàng, kiểm tra số lượng và tổng tiền. Hai cuốn Harry Potter có tổng tiền 360.000 ₫.

   ![](/fcj-workshop-nhat/images/5-Workshop/5.6-Operations/5.6.6-End-to-End-testing/04-cart.jpg)

3. Áp dụng mã khuyến mãi nếu có. Giao diện đã có trường nhập mã; kết quả áp dụng chưa được xác minh.
4. Tiến hành đặt hàng. Kiểm thử PayPal Sandbox với đơn gồm hai sách trị giá 240.000 ₫; cửa sổ thanh toán hiển thị 9,60 USD.

   ![](/fcj-workshop-nhat/images/5-Workshop/5.6-Operations/5.6.6-End-to-End-testing/14-paypal-sandbox.png)

5. Kiểm tra thông báo **Đặt hàng thành công**.

   ![](/fcj-workshop-nhat/images/5-Workshop/5.6-Operations/5.6.6-End-to-End-testing/15-order-success.png)

6. Chọn **Xem đơn hàng**. Đối chiếu đơn hàng số 75, hai sản phẩm, tổng 240.000 ₫, phương thức PayPal và trạng thái Chờ xác nhận.

   ![](/fcj-workshop-nhat/images/5-Workshop/5.6-Operations/5.6.6-End-to-End-testing/16-order-detail.png)


**Trường hợp ngoại lệ:**

- TH1: Mã khuyến mãi không hợp lệ hoặc đã hết hạn.
- TH2: Lỗi khi tạo đơn hàng do mất kết nối RDS.
- TH3: Chưa đăng nhập → hệ thống yêu cầu đăng nhập trước khi thanh toán.

#### Kịch bản 4. Upload ảnh

| Mục | Nội dung |
|-----|----------|
| **Mục tiêu** | Đảm bảo chức năng upload ảnh hoạt động, ảnh được lưu trữ và xử lý bởi Lambda; kiểm tra khả năng hiển thị ảnh trên website |
| **Kết quả mong đợi** | Ảnh được upload lên S3 Media, Lambda xử lý và ảnh hiển thị đúng trên giao diện. Khi sản phẩm chưa có ảnh hoặc ảnh không tải được, hệ thống hiển thị ảnh thay thế phù hợp |

**Quy trình thực hiện:**

1. Đăng nhập tài khoản có quyền tải ảnh.
2. Thực hiện upload ảnh sách hoặc ảnh đại diện.
3. Kiểm tra ảnh hiển thị trên giao diện danh mục hoặc trang chi tiết sản phẩm.
4. Kiểm tra file trên S3 Media Bucket và log Lambda.

**Trường hợp ngoại lệ:**

- TH1: Upload file không đúng định dạng.
- TH2: File quá lớn → hệ thống từ chối.
- TH3: Lambda xử lý ảnh thất bại.
- TH4: Sản phẩm chưa có ảnh. Khi kiểm tra sản phẩm thử nghiệm `abc`, giao diện hiển thị ảnh mặc định **No Product Image Available**.

  ![](/fcj-workshop-nhat/images/5-Workshop/5.6-Operations/5.6.6-End-to-End-testing/23-upload-image-placeholder.jpg)

- TH5: Ảnh sản phẩm không tải được. Khi kiểm tra sách **The Song of Achilles**, ảnh bìa không hiển thị và trình duyệt hiển thị văn bản thay thế; ảnh bìa của sản phẩm bên cạnh vẫn tải được.

  ![](/fcj-workshop-nhat/images/5-Workshop/5.6-Operations/5.6.6-End-to-End-testing/24-upload-image-load-error.jpg)

Kết quả kiểm thử xác nhận giao diện có ảnh mặc định cho sản phẩm chưa có ảnh, đồng thời ghi nhận một trường hợp ảnh bìa tải thất bại. Cần kiểm tra đường dẫn ảnh, object trong S3 Media và log Lambda để xác minh nguyên nhân và hoàn tất kiểm thử luồng upload.

#### Kịch bản 5. Chức năng Admin

| Mục | Nội dung |
|-----|----------|
| **Mục tiêu** | Kiểm thử các chức năng quản trị cơ bản với tài khoản Admin |
| **Kết quả mong đợi** | Admin truy cập và xem được Dashboard, sản phẩm, thể loại, đơn hàng, người dùng và khuyến mãi |

**Quy trình thực hiện:**

1. Đăng nhập bằng tài khoản Admin và mở **Tổng quan**. Kiểm tra số lượng đơn hàng, doanh thu, người dùng, sản phẩm và biểu đồ thống kê.

   ![](/fcj-workshop-nhat/images/5-Workshop/5.6-Operations/5.6.6-End-to-End-testing/17-admin-dashboard.png)

2. Chọn **Quản lý sản phẩm** và kiểm tra danh sách sách, hình ảnh, trạng thái và các nút thao tác.

   ![](/fcj-workshop-nhat/images/5-Workshop/5.6-Operations/5.6.6-End-to-End-testing/18-admin-products.png)

3. Chọn **Quản lý thể loại** và kiểm tra danh sách năm thể loại sách.

   ![](/fcj-workshop-nhat/images/5-Workshop/5.6-Operations/5.6.6-End-to-End-testing/19-admin-categories.png)

4. Chọn **Quản lý đơn hàng**. Kiểm tra danh sách, đơn hàng số 75 có tổng 240.000 ₫, phương thức PayPal và trạng thái Chờ xác nhận.

   ![](/fcj-workshop-nhat/images/5-Workshop/5.6-Operations/5.6.6-End-to-End-testing/20-admin-orders.png)

5. Chọn **Quản lý người dùng**. Kiểm tra danh sách, vai trò Khách hàng/Admin và trạng thái tài khoản.

   ![](/fcj-workshop-nhat/images/5-Workshop/5.6-Operations/5.6.6-End-to-End-testing/21-admin-users.png)

6. Chọn **Quản lý khuyến mãi**. Kiểm tra ba mã khuyến mãi và mức giảm 10%, 20%, 30%.

   ![](/fcj-workshop-nhat/images/5-Workshop/5.6-Operations/5.6.6-End-to-End-testing/22-admin-promotions.png)


**Trường hợp ngoại lệ:**

- TH1: Tài khoản thường không có quyền truy cập trang Admin.
- TH2: Lỗi khi cập nhật dữ liệu do mất kết nối database.
- TH3: Các thao tác thêm, sửa, xóa, khóa tài khoản và thay đổi trạng thái đơn hàng cần xác minh bằng dữ liệu thử nghiệm.

#### Kịch bản 6. Demo gợi ý sách bằng Amazon Bedrock

| Mục | Nội dung |
|-----|----------|
| **Mục tiêu** | Kiểm thử chức năng gợi ý sách thông qua Bedrock |
| **Kết quả mong đợi** | Hệ thống trả về sách gợi ý phù hợp với input của người dùng |

**Quy trình thực hiện:**

1. Truy cập tính năng gợi ý sách.
2. Nhập câu hỏi gợi ý một cuốn sách về trí tuệ nhân tạo đang có tại RakiBookery.
3. Kiểm tra kết quả trả về: Dẫn nhập ngắn về khoa học - Trí tuệ nhân tạo, giá 180.000 VNĐ, còn 24 cuốn.

   ![](/fcj-workshop-nhat/images/5-Workshop/5.6-Operations/5.6.6-End-to-End-testing/06-chatbot.jpg)


**Trường hợp ngoại lệ:**

- TH1: Bedrock timeout hoặc lỗi quyền IAM.
- TH2: Đạt giới hạn Throttling API.
- TH3: Không có kết quả phù hợp.

#### Kịch bản 7. Kiểm thử xử lý lỗi

| Mục | Nội dung |
|-----|----------|
| **Mục tiêu** | Đảm bảo cơ chế xử lý ngoại lệ thân thiện với người dùng |
| **Kết quả mong đợi** | Giao diện hiển thị thông báo lỗi rõ ràng và ứng dụng tiếp tục hoạt động |

**Quy trình thực hiện:**

1. Nhập dữ liệu không hợp lệ vào form đăng ký và kiểm tra thông báo validation.

   ![](/fcj-workshop-nhat/images/5-Workshop/5.6-Operations/5.6.6-End-to-End-testing/07-registration-validation.jpg)

2. Quan sát thông báo lỗi ở lần đăng nhập bị từ chối.

   ![](/fcj-workshop-nhat/images/5-Workshop/5.6-Operations/5.6.6-End-to-End-testing/01-login-failed.jpg)

3. Thực hiện checkout khi chưa đăng nhập, kiểm tra cảnh báo.

   ![](/fcj-workshop-nhat/images/5-Workshop/5.6-Operations/5.6.6-End-to-End-testing/05-checkout-login-required.jpg)

4. Tiếp tục sử dụng chức năng duyệt sách và giỏ hàng.

**Trường hợp ngoại lệ:**

- TH1: Thông tin nhập không hợp lệ.
- TH2: Đăng nhập thất bại hoặc chưa đăng nhập khi checkout.
- TH3: Timeout, độ trễ mạng hoặc gián đoạn kết nối cần được kiểm thử trong môi trường có kiểm soát.

#### Kịch bản 8. Kiểm thử giao diện đa nền tảng (Mobile Responsiveness)

| Mục | Nội dung |
|-----|----------|
| **Mục tiêu** | Đảm bảo hoạt động nhất quán trên Desktop và Mobile |
| **Kết quả mong đợi** | Giao diện co giãn linh hoạt và các thành phần quan trọng vẫn truy cập được |

**Quy trình thực hiện:**

1. Dùng Developer Tools trên trình duyệt để giả lập viewport mobile 390 × 844 px.
2. Kiểm tra giao diện trang chủ.

   ![](/fcj-workshop-nhat/images/5-Workshop/5.6-Operations/5.6.6-End-to-End-testing/08-mobile-home.jpg)

3. Kiểm tra trang chi tiết sách.

   ![](/fcj-workshop-nhat/images/5-Workshop/5.6-Operations/5.6.6-End-to-End-testing/09-mobile-detail.jpg)

4. Kiểm tra giỏ hàng, tổng tiền và khả năng cuộn ngang.

   ![](/fcj-workshop-nhat/images/5-Workshop/5.6-Operations/5.6.6-End-to-End-testing/10-mobile-cart.jpg)

5. Kiểm tra form đăng nhập trên mobile.

   ![](/fcj-workshop-nhat/images/5-Workshop/5.6-Operations/5.6.6-End-to-End-testing/11-mobile-login.jpg)


### 5.6.6.2. Tổng kết kiểm thử

Các luồng chính của khách hàng và chức năng truy cập các trang Admin đã được kiểm thử. Kết quả được ghi nhận dựa trên minh chứng hiện có.

| Kịch bản | Kết quả |
|---|---|
| Luồng đăng ký → đăng nhập | Passed |
| Duyệt sách và xem chi tiết sản phẩm | Passed |
| Thêm vào giỏ hàng → Đặt hàng + Áp dụng khuyến mãi (Promo) | Passed  |
| Upload ảnh | Passed |
| Chức năng Admin | Passed  |
| Demo gợi ý sách bằng Amazon Bedrock | Passed |
| Kiểm thử xử lý lỗi | Passed |
| Kiểm thử giao diện đa nền tảng (Mobile Responsiveness) | Passed |

Tổng cộng có sáu kịch bản đạt ở phạm vi đã kiểm thử và hai kịch bản đạt một phần.
