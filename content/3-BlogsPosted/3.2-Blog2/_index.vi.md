---
title: "Blog 2"
date: 2026-10-15
weight: 2
chapter: false
pre: " <b> 3.2. </b> "
---

# 🦆 Aurora PostgreSQL + DuckDB Nhúng: Truy Vấn Data Lake Không Cần ETL — Nhưng Đánh Đổi Gì?

## 📝 Giới thiệu

Trong các hệ thống phần mềm thực tế, có một bài toán muôn thuở luôn khiến các kỹ sư đau đầu:

> **Làm sao để kết hợp dữ liệu giao dịch thời gian thực với hàng năm trời dữ liệu lịch sử mà không biến cơ sở dữ liệu vận hành thành một "kho lưu trữ khổng lồ" chậm chạp và đắt đỏ?**

Mô hình kinh điển mà chúng ta thường áp dụng:

1. Lưu dữ liệu giao dịch "nóng" (hot data) trên **Aurora PostgreSQL**.
2. Đẩy toàn bộ dữ liệu lịch sử "lạnh" (cold data) ra **Amazon S3 Data Lake** dưới định dạng Parquet hoặc Apache Iceberg.
3. Dựng và bảo trì một **pipeline ETL** (AWS Glue, Lambda, Spark hoặc dbt) để sao chép dữ liệu tổng hợp từ S3 ngược vào Aurora mỗi khi ứng dụng cần đối soát giữa đơn hàng mới với lịch sử mua hàng nhiều năm.

Nhưng ai từng vận hành pipeline cũng hiểu cái giá phải trả: pipeline dễ lỗi, dữ liệu bị lệch pha (sync lag), tốn tiền lưu trữ hai lần và đau đầu mỗi khi schema thay đổi.

Vào ngày 30/09/2026, AWS công bố một tính năng đột phá: **Amazon Aurora PostgreSQL chính thức hỗ trợ truy vấn trực tiếp Apache Iceberg và Parquet trên S3 thông qua engine DuckDB được nhúng trực tiếp bên trong tiến trình máy chủ PostgreSQL.**

Ứng dụng của bạn vẫn dùng endpoint cũ, driver cũ và cú pháp SQL PostgreSQL quen thuộc. Không cần thêm engine truy vấn liên kết (query federation), không cần viết API trung gian.

Nghe thì hoàn hảo, nhưng trong kiến trúc hệ thống, **không có bữa trưa nào miễn phí**.

Câu hỏi tôi đặt ra là:

> **Điều gì thực sự diễn ra bên dưới hệ thống khi chúng ta bỏ pipeline ETL để DuckDB nhúng gánh tải, và hệ thống sẽ "gãy" ở đâu khi lượng truy vấn tăng cao?**

Hãy cùng bóc tách từng lớp kiến trúc qua các thử nghiệm cụ thể.

---

## 🏗️ Kiến trúc & Bối cảnh

Hãy so sánh hai mô hình kiến trúc:

![Architecture Diagram](/fcj-workshop-nhat/images/AuroraPostgreeSQLwithDuckDB.png)

### Mô hình 1: Pipeline ETL truyền thống (Sao chép dữ liệu)
```text
Ứng dụng (Application)
    ↓
Aurora PostgreSQL (Chứa cả bảng vận hành lẫn bản sao dữ liệu lịch sử)
    ↑
[Pipeline ETL (AWS Glue / EMR / Lambda)]
    ↑
Amazon S3 Data Lake (Parquet / Iceberg)
```

### Mô hình 2: Truy vấn trực tiếp tại chỗ bằng DuckDB nhúng
```text
Ứng dụng (Application)
    ↓ (Câu lệnh SQL JOIN quen thuộc)
Cụm Aurora PostgreSQL
  ├── Bảng vận hành native (nằm trên shared_buffers của OLTP)
  └── Foreign Table (`aurora_analytics_server`)
        ↓
    Engine DuckDB nhúng (Vectorized execution, Column pruning)
        ↓
    Amazon S3 / S3 Tables / AWS Glue Data Catalog (Parquet & Iceberg)
```

Trong mô hình này, dữ liệu Data Lake được định nghĩa dưới dạng một **PostgreSQL Foreign Table** thông qua extension `aurora_analytics`. Khi một truy vấn kết hợp cả bảng native và foreign table, trình lập kế hoạch (planner) của PostgreSQL sẽ chuyển việc xử lý phân tích cho DuckDB.

Bây giờ, hãy thử nghiệm xem các thành phần này sẽ hoạt động ra sao và những đánh đổi nào xuất hiện.

---

## 🔍 Thử nghiệm 1: 🔄 Điều gì xảy ra khi xóa bỏ hoàn toàn pipeline ETL?

### Lời hứa hẹn từ tính năng mới
Giả sử hệ thống thương mại điện tử cần hiển thị các đơn hàng đang xử lý kèm theo tổng số đơn hàng trọn đời của khách hàng.

Với DuckDB nhúng, bạn chỉ cần tạo một Foreign Table trỏ tới S3:

```sql
CREATE FOREIGN TABLE analytics.ft_purchase_history (
    customer_id BIGINT,
    last_purchase_at TIMESTAMP,
    lifetime_order_count BIGINT
)
SERVER aurora_analytics_server
OPTIONS (
    location 's3://analytics-lakehouse/purchase-history/',
    format 'parquet',
    region 'ap-southeast-1'
);
```

Và thực hiện một câu lệnh `JOIN` thông thường:

```sql
SELECT
    o.order_id,
    o.customer_id,
    o.order_status,
    h.last_purchase_at,
    h.lifetime_order_count
FROM orders AS o
LEFT JOIN analytics.ft_purchase_history AS h
    ON h.customer_id = o.customer_id
WHERE o.order_status = 'PROCESSING';
```

### Lợi ích nhận được
1. **Bớt hẳn một hệ thống ETL:** Không còn script AWS Glue, không cần cronjob điều phối và không phải trả tiền lưu trữ trùng lặp hàng trăm GB dữ liệu lịch sử trong Aurora.
2. **Dữ liệu luôn mới nhất:** Ngay khi một file Parquet hoặc snapshot Iceberg mới được đẩy lên S3, câu truy vấn qua foreign table sẽ đọc được ngay.
3. **Đơn giản hóa cho lập trình viên:** Ứng dụng chỉ kết nối tới một endpoint duy nhất của Aurora với cú pháp SQL quen thuộc.

### Đánh đổi tiềm ẩn
Dữ liệu không tự nhiên chạy vào câu truy vấn. **Công việc tính toán chỉ bị dời chỗ, chứ không biến mất**. Thay vì một worker AWS Glue độc lập giải nén file Parquet lúc 2 giờ sáng, thì giờ đây chính CPU và RAM của cụm Aurora phải đọc S3, giải nén dữ liệu và dựng hash table ngay trong thời gian thực.

### Đánh giá (Verdict)
> **"Bỏ ETL không làm giảm khối lượng tính toán — nó chỉ chuyển thẳng gánh nặng đó lên máy chủ cơ sở dữ liệu của bạn."**

---

## 🔍 Thử nghiệm 2: ⚡ DuckDB tối ưu truy vấn thế nào (Và khi nào Data Lake phản chủ)?

### Cơ chế tối ưu của DuckDB nhúng
DuckDB không quét toàn bộ dữ liệu trên S3 một cách ngây thơ. Nó tận dụng 3 cơ chế:

1. **Column Pruning (Cắt tỉa cột):** Nếu file Parquet có 100 cột nhưng câu SQL chỉ gọi 2 cột, Aurora chỉ tải đúng dải byte của 2 cột đó qua network.
2. **Predicate Pushdown & Row-Group Skipping:** DuckDB đọc metadata của Parquet hoặc manifest của Iceberg. Nếu câu truy vấn có điều kiện `date >= '2026-01-01'`, những row group nào có giá trị lớn nhất nhỏ hơn mốc này sẽ bị bỏ qua hoàn toàn, không cần tải về.
3. **On-Disk Read Cache:** Các block dữ liệu tải từ S3 sẽ được lưu tạm trên ổ cứng SSD cục bộ của instance Aurora. Các truy vấn lần sau đọc trúng dữ liệu này sẽ chạy thẳng từ NVMe SSD thay vì gọi S3.

### Điều gì xảy ra nếu cách bố trí Data Lake bị kém?
Nếu Data Lake của bạn mắc phải lỗi kinh điển: **hàng triệu file nhỏ (Small Files Problem)** hoặc dữ liệu ghi không theo thứ tự phân vùng (partition):
- DuckDB sẽ phải phát sinh hàng chục ngàn request `s3:GetObject` nhỏ lẻ, khiến độ trễ tăng vọt từ mili-giây lên hàng chục giây.
- Predicate pushdown mất tác dụng vì row group nào cũng chứa dải dữ liệu phân tán.

Hơn nữa, **cache trên đĩa chỉ là tạm thời**. Khi instance Aurora khởi động lại, failover sang node khác hoặc mở rộng quy mô, bộ nhớ cache NVMe này sẽ bị xóa sạch. Các truy vấn đầu tiên sau đó sẽ chịu độ trễ đọc S3 "lạnh" hoàn toàn.

### Đánh giá (Verdict)
> **"DuckDB chạy rất nhanh, nhưng nó không thể cứu vãn một Data Lake phân mảnh và thiết kế phân vùng cẩu thả."**

---

## 🔍 Thử nghiệm 3: 💥 Điều gì xảy ra khi câu truy vấn ngốn hết RAM?

### Thực tế phân bổ bộ nhớ trong Aurora
Trong PostgreSQL truyền thống, phần lớn RAM của máy chủ được cấp cho `shared_buffers` (thường chiếm từ **65% đến 75%** tổng RAM) để cache các trang dữ liệu giao dịch OLTP.

Truy vấn foreign table thông qua DuckDB **hoàn toàn không dùng `shared_buffers`**.

Thay vào đó, Aurora kiểm soát bộ nhớ phân tích qua tham số riêng: `aurora_analytics.query_mem`.

```text
┌──────────────────────────────────────────────────────────────┐
│                    Tổng RAM của Instance                     │
├──────────────────────────────────────┬───────────────────────┤
│    shared_buffers (~65% - 75%)       │  Vùng RAM còn lại     │
│   (Bảng OLTP, Index, Bufferpool)     │  ├── OS / Backend     │
│                                      │  └── query_mem        │
└──────────────────────────────────────┴───────────────────────┘
```

Vì `shared_buffers` đã chiếm phần lớn tài nguyên, "chiếc bánh" bộ nhớ còn lại dành cho `query_mem` để DuckDB dựng hash-join và sort dữ liệu là khá khiêm tốn.

### Khi truy vấn vượt quá ngân sách `query_mem`:
1. **Disk Spilling:** DuckDB buộc phải ghi dữ liệu trung gian chưa tính xong xuống ổ đĩa cục bộ của instance.
2. **Tranh chấp I/O:** Vùng đĩa ghi tràn (spill) này dùng chung ổ cứng NVMe với **on-disk read cache** và các file tạm của PostgreSQL.
3. **Tụt giảm hiệu năng & Hủy truy vấn:** Truy vấn sẽ chậm lại trông thấy do tắc nghẽn I/O. Nếu hết dung lượng đĩa cục bộ hoặc gây áp lực OOM toàn hệ thống, Aurora sẽ hủy (abort) luôn câu truy vấn đó.

### Đánh giá (Verdict)
> **"Dữ liệu phân tích là 'công dân hạng hai' về mặt bộ nhớ trong Aurora. Nếu không cấu hình `query_mem` chuẩn xác, câu truy vấn sẽ tràn đĩa và gây nghẽn I/O cho cả hệ thống."**

---

## 🔍 Thử nghiệm 4: 🛡️ Tách một Aurora Reader riêng có giải quyết được mọi rủi ro?

### Giải pháp tự nhiên
"Nếu sợ tải phân tích ảnh hưởng đến node ghi (Writer), ta chỉ việc thêm một **Aurora Dedicated Reader** và cấu hình Custom Endpoint cho tác vụ analytics!"

Đây là khuyến nghị chính thức từ AWS:
- Reader này có thể hạ nhỏ `shared_buffers` xuống và nâng `query_mem` lên tối đa.
- Gán failover tier = 15 (để không bao giờ bị thăng cấp thành writer khi sự cố xảy ra).
- Bảo vệ CPU của writer khỏi bị DuckDB chiếm dụng.

### Cạm bẫy tiềm ẩn: Dead Tuple và nghẽn VACUUM
Dù là một instance độc lập, Reader vẫn dùng chung hệ thống lưu trữ phân tán (`Aurora Cluster Volume`) và chia sẻ snapshot giao dịch với Writer.

Nếu một truy vấn foreign table phức tạp trên Reader chạy mất 30 - 45 phút:
1. Truy vấn này giữ một snapshot giao dịch mở rất lâu.
2. Cơ chế `hot_standby_feedback` báo ngược về Writer rằng snapshot đó vẫn đang cần đọc dữ liệu cũ.
3. Quá trình Autovacuum trên Writer **không thể dọn dẹp các dead tuple** (bản ghi đã bị xóa/sửa) được tạo ra sau thời điểm snapshot đó.
4. Hậu quả: Bảng trên Writer bị phình to (table bloat), index phình to, hiệu năng ghi và quét index trên toàn hệ thống OLTP tụt dốc.

### Biện pháp xử lý
Bắt buộc phải cấu hình `statement_timeout` cho các phiên phân tích trên Reader và theo dõi sát sao chỉ số CloudWatch: `oldest_reader_feedback_xid_age`.

### Đánh giá (Verdict)
> **"Một Reader riêng tách biệt được CPU và RAM, nhưng một truy vấn chạy quá lâu vẫn có thể âm thầm 'bóp nghẹt' node Writer thông qua cơ chế MVCC của PostgreSQL."**

---

## 🔍 Thử nghiệm 5: 🚫 Giới hạn cứng: Khi nào DuckDB Foreign Table từ chối phục vụ?

Trước khi quyết định đập bỏ pipeline cũ, hãy chú ý đến những giới hạn cứng từ tài liệu AWS:

### 1. Chỉ đọc (Strictly Read-Only)
Foreign table của Aurora Analytics chỉ hỗ trợ đọc:
- `INSERT`, `UPDATE`, `DELETE`, `TRUNCATE` hay `COPY FROM` vào foreign table ❌ Đều bị chặn hoàn toàn.
Nếu ứng dụng cần cập nhật hay ghi kết quả ngược lại Data Lake, bạn vẫn cần các công cụ quản lý Iceberg hoặc Spark/Athena.

### 2. Không hỗ trợ các kiểu dữ liệu phức tạp
Một số kiểu dữ liệu thông dụng trong PostgreSQL **chưa có ánh xạ** sang Parquet/Iceberg trong tính năng này:
- `JSONB` / `JSON` ❌ Không hỗ trợ.
- Kiểu mảng (`ARRAY`), kiểu `SERIAL` ❌ Không hỗ trợ.
- Kiểu mạng (`INET`/`CIDR`), hình học (`GEOMETRIC`), `RANGE`, `XML` ❌ Không hỗ trợ.
Nếu bảng giao dịch của bạn thiết kế theo dạng hybrid semi-structured với nhiều cột JSONB, bạn không thể ánh xạ trực tiếp sang foreign table mà không làm phẳng (flatten) dữ liệu trước.

### 3. Rào cản Region và Tên cột
- **Không hỗ trợ truy vấn chéo Region:** Một câu SQL không thể cùng lúc đọc dữ liệu từ các S3 bucket nằm ở hai AWS Region khác nhau.
- **Quy tắc đặt tên:** Tên cột từ file Parquet không được quá 63 ký tự; không cho phép 2 cột chỉ khác nhau chữ hoa/thường (case-sensitive collision).
- **Không có system column:** Các thuộc tính hệ thống như `ctid` hay `xmin` không tồn tại trên foreign table.

### Đánh giá (Verdict)
> **"Foreign table là một 'ống kính đọc' dữ liệu phân tích, không phải là một giải pháp thay thế hoàn toàn cho kho lưu trữ của PostgreSQL."**

---

## 🔍 Thử nghiệm 6: 💸 Hóa đơn AWS sẽ biến động như thế nào?

AWS khẳng định bản thân tính năng `aurora_analytics` **không tính thêm phí bản quyền**.

Nhưng "không mất phí tính năng" không đồng nghĩa với "miễn phí hạ tầng".

```text
Chi phí mô hình ETL truyền thống:
  - Giờ chạy AWS Glue Job / EMR Cluster
  - Phí lưu trữ bản sao dữ liệu trong Aurora ($0.10/GB-tháng)
  - Phí truyền dữ liệu giữa các dịch vụ

Chi phí mô hình DuckDB trực tiếp:
  - Tài nguyên Compute của Aurora (ACU trên Serverless v2 hoặc cấu hình Provisioned)
  - Phí API S3: `s3:GetObject` ($0.0004 mỗi 1.000 request) & `s3:ListBucket`
  - Dung lượng ổ đĩa cục bộ trên instance cho cache và spill
```

### Cái bẫy chi phí với Aurora Serverless v2
Nếu nhiều người dùng đồng thời chạy các báo cáo phân tích nặng, Aurora Serverless v2 sẽ tự động scale-up số lượng ACU lên rất cao để gánh tải. Vì giá mỗi giờ của ACU đắt hơn nhiều so với việc chạy batch compute bằng Spot EMR hay Glue Flex, chi phí Aurora có thể tăng đột biến ngoài dự kiến.

### Đánh giá (Verdict)
> **"Chi phí không hề biến mất; nó chỉ chuyển dịch từ các dòng hóa đơn batch processing (Glue/EMR) sang compute của database (Aurora ACU) và số lượng S3 request."**

---

## 📌 Tổng kết — Khung tư duy kiến trúc (Architecture Thinking Framework)

### Quy luật chung
Khi loại bỏ pipeline ETL, bài toán xử lý dữ liệu không biến mất. Trọng trách đó chỉ chuyển từ pipeline ngoài sang thẳng bộ nhớ và CPU của cơ sở dữ liệu.

Đưa Data Lake vào trong lòng PostgreSQL:
- **Tăng tốc độ phát triển ứng dụng vượt bậc** (viết 1 câu SQL là xong, không chờ sync).
- **Gia tăng rủi ro vận hành cơ sở dữ liệu tương ứng** (tranh chấp tài nguyên, tràn đĩa, nguy cơ phình dead tuple).

### Khung 3 câu hỏi đánh giá trước khi triển khai

Khi cân nhắc sử dụng Aurora + DuckDB để truy vấn trực tiếp Data Lake, hãy tự hỏi:

**1. "Mẫu truy vấn và tần suất của hệ thống là gì?"**
- Cần lookup vài chục dòng giao dịch kết hợp với lịch sử đã lọc kỹ? 👉 **Nên dùng DuckDB Foreign Table.**
- Cần quét aggregate hàng trăm triệu dòng liên tục mỗi phút cho dashboard? 👉 **Hãy giữ lại engine OLAP chuyên dụng (Athena, Redshift, ClickHouse, Snowflake).**

**2. "Schema dữ liệu có phụ thuộc vào JSONB hay kiểu phức tạp không?"**
- Nếu dữ liệu S3 biến động liên tục và chứa JSONB hoặc mảng phức tạp, bạn sẽ tốn rất nhiều công bảo trì schema foreign table.

**3. "Chấp nhận sự cố xảy ra ở đâu?"**
- Pipeline ETL lỗi: Báo cáo phân tích bị trễ dữ liệu vài giờ.
- Database Aurora kiệt quệ tài nguyên: **Toàn bộ ứng dụng và việc ghi đơn hàng sập theo.**

### Bảng so sánh tổng hợp

| Mô hình kiến trúc | Khi nào nên dùng | Đánh đổi / Rủi ro lớn nhất |
|---|---|---|
| **Pipeline ETL truyền thống** | Tần suất đọc cực cao, đòi hỏi sub-second latency trên bảng vật chất hóa sẵn trong Aurora | Tốn công vận hành pipeline, dữ liệu bị trễ, tốn tiền lưu trữ 2 lần |
| **Aurora + DuckDB nhúng** | Ghép đơn hàng nóng với lịch sử lạnh trên S3; báo cáo ad-hoc nội bộ; ứng dụng monolithic gọn gàng | Áp lực bộ nhớ RAM (`query_mem`), tràn đĩa cục bộ, nguy cơ dead tuple trên writer |
| **OLAP chuyên dụng (Athena / Redshift)** | BI dashboard cho hàng ngàn người; quét petabyte dữ liệu; tổng hợp số liệu phức tạp | Phải quản lý nhiều endpoint kết nối, phân quyền phức tạp, kiến trúc cồng kềnh hơn |

### Lời kết

Một kỹ sư mới vào nghề nhìn thấy tính năng mới của Aurora sẽ hào hứng nói:

> **"Tuyệt vời! Từ nay ta có thể xóa hết mọi pipeline ETL và truy vấn mọi thứ trực tiếp từ PostgreSQL."**

Một kỹ sư trưởng (Principal Architect) giàu kinh nghiệm nhìn vào sơ đồ đó sẽ hỏi:

> **"Nếu một câu query quét trúng bảng S3 nặng 500GB lúc nửa đêm đúng đợt sale Black Friday, điều gì sẽ xảy ra với replication lag và transaction log của node chính?"**

Tư duy kiến trúc thực sự chỉ bắt đầu khi bạn nhìn thấu qua những lời quảng cáo tính năng để thấy rõ những đánh đổi vận hành nằm bên dưới.
