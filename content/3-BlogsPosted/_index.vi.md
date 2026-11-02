---
title: "Các bài blog đã đăng"
date: 2024-01-01
weight: 1
chapter: false
pre: " <b> 3. </b> "
---

<!-- {{% notice warning %}}  
⚠️ **Lưu ý:** Các thông tin dưới đây chỉ nhằm mục đích tham khảo, vui lòng **không sao chép nguyên văn** cho bài báo cáo của bạn kể cả warning này.
{{% /notice %}} -->


###  [Blog 1 - ☁️ Tôi thử “phá” một kiến trúc AWS bằng cách bỏ từng service ra](3.1-Blog1/)


###  [Blog 2 - 🦆 Aurora PostgreSQL + DuckDB Nhúng: Truy Vấn Data Lake Không Cần ETL — Nhưng Đánh Đổi Gì?](3.2-Blog2/)
Blog này sẽ bóc tách tính năng Aurora PostgreSQL nhúng DuckDB để truy vấn Data Lake không cần ETL: Cơ chế hoạt động, bẫy bộ nhớ query_mem, nguy cơ dead tuple và bài toán đánh đổi chi phí thực tế."

###  [Blog 3 - 🐍 Boto3 + DynamoDB: Khi scan() Trả Về "Đủ" Nhưng Dữ Liệu Lại "Thiếu" Và Cơn Ác Mộng Serialization Decimal](3.3-Blog3/)
Blog này bóc tách cơ chế hoạt động thực tế bên dưới của Amazon DynamoDB và Python Boto3: Bản chất phân trang 1MB của Scan, hiện tượng trả về rỗng nhưng vẫn còn dữ liệu, lý do đằng sau lỗi TypeError Decimal khi serialize JSON và giải pháp chuẩn production.