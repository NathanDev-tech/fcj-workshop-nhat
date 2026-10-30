---
title: "Blog 2"
date: 2026-10-15
weight: 2
chapter: false
pre: " <b> 3.2. </b> "
---

# 🦆 Aurora PostgreSQL + DuckDB nhúng: Truy vấn Data Lake không cần ETL — nhưng phải đánh đổi điều gì?

## 📝 Giới thiệu

Trong các hệ thống giao dịch hiện đại, một bài toán kiến trúc liên tục được đặt ra:

> **Làm thế nào để kết hợp dữ liệu vận hành theo thời gian thực với dữ liệu phân tích lịch sử trong nhiều năm mà không biến cơ sở dữ liệu chính thành một kho lưu trữ cồng kềnh và tốn kém?**

Câu trả lời theo cách tiếp cận truyền thống luôn là:

1. Giữ dữ liệu giao dịch đang hoạt động trong **Aurora PostgreSQL**.

2. Chuyển dữ liệu lịch sử ít được truy cập sang **Amazon S3 Data Lake** (ở định dạng Parquet hoặc Apache Iceberg).

3. Xây dựng và duy trì một **quy trình ETL** (Glue, Flink, Spark hoặc dbt) để đồng bộ dữ liệu lịch sử đã tổng hợp trở lại PostgreSQL mỗi khi ứng dụng cần kết hợp các đơn hàng đang hoạt động với chỉ số tích lũy trong toàn bộ vòng đời khách hàng.

Mọi kỹ sư dữ liệu đều hiểu những chi phí vận hành đi kèm mô hình này: các tác vụ pipeline dễ hỏng, dữ liệu sai lệch theo thời gian, độ trễ đồng bộ, chi phí lưu trữ trùng lặp và những vấn đề phát sinh khi phải đồng bộ schema.

Vào ngày 30 tháng 9 năm 2026, AWS giới thiệu một tính năng mang tính thay đổi cuộc chơi: **Amazon Aurora PostgreSQL hiện hỗ trợ truy vấn trực tiếp dữ liệu Apache Iceberg và Parquet trong S3 thông qua một bộ máy cột DuckDB được nhúng trực tiếp bên trong tiến trình PostgreSQL.**

Ứng dụng của bạn vẫn sử dụng cùng connection string, cùng Postgres driver và các câu truy vấn SQL quen thuộc. Không cần query engine liên kết bên ngoài, cũng không cần API tùy chỉnh.

Điều này nghe giống như giấc mơ của một kỹ sư dữ liệu. Nhưng trong kiến trúc đám mây, không có gì là miễn phí.

Câu hỏi tôi đặt ra là:

> **Điều gì diễn ra bên dưới khi chúng ta thay thế các pipeline ETL bằng DuckDB nhúng trong Aurora, và điều gì có thể gặp sự cố khi khối lượng công việc mở rộng?**

Hãy cùng phân tích kiến trúc theo từng lớp thông qua các thử nghiệm thực tế.

---

## 🏗️ Kiến trúc / Bối cảnh

Hãy so sánh hai mô hình kiến trúc:

![Sơ đồ kiến trúc](/fcj-workshop-nhat/images/AuroraPostgreeSQLwithDuckDB.png)

### Mô hình A: ETL truyền thống và sao chép dữ liệu

```text

Application

    ↓

Aurora PostgreSQL (Stores both hot orders & replicated historical tables)

    ↑

[ETL Pipeline (AWS Glue / EMR / Lambda / EventBridge)]

    ↑

Amazon S3 Data Lake (Parquet / Iceberg)

```

### Mô hình B: Lakehouse Zero-ETL trong tiến trình thông qua DuckDB nhúng

```text

Application

    ↓ (Standard PostgreSQL SQL JOIN)

Aurora PostgreSQL Cluster

  ├── Native Table (e.g., `orders` in OLTP shared_buffers)

  └── Foreign Table (`aurora_analytics_server`)

        ↓

    Embedded DuckDB Engine (Vectorized execution & Column pruning)

        ↓

    Amazon S3 / S3 Tables / AWS Glue Data Catalog (Parquet & Iceberg)

```

Trong kiến trúc này, dữ liệu lịch sử được hiển thị dưới dạng **PostgreSQL Foreign Table**, được hỗ trợ bởi extension `aurora_analytics`. Khi một truy vấn tham chiếu đồng thời đến bảng native và foreign table, bộ lập kế hoạch truy vấn của Aurora PostgreSQL sẽ chuyển phần xử lý phân tích nặng sang DuckDB.

Hãy lần lượt xem xét các thử nghiệm và tìm hiểu điều gì xảy ra khi kiến trúc này đối mặt với áp lực trong môi trường thực tế.

---

## 🔍 Thử nghiệm 1: 🔄 Điều gì xảy ra nếu chúng ta loại bỏ pipeline ETL?

### Lời hứa về lợi ích

Trong một hệ thống bán lẻ, bạn cần hiển thị các đơn hàng đang được xử lý cùng với tổng khối lượng mua hàng tích lũy trong toàn bộ vòng đời của một khách hàng. 

Với DuckDB nhúng, bạn khai báo một foreign table trỏ đến URI của S3 hoặc Glue Data Catalog:

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

Sau đó truy vấn cả hai trong cùng một transaction:

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

### Những lợi ích đạt được

1. **Không di chuyển dữ liệu:** Không cần script ETL của Glue, không cần điều phối lịch chạy và không phải trả phí hai lần cho hàng gigabyte hoặc terabyte dữ liệu lưu trữ trùng lặp bên trong Aurora.

2. **Tính cập nhật:** Ngay khi tiến trình ghi dữ liệu ở thượng nguồn commit một file Parquet mới hoặc một snapshot Iceberg lên S3, các truy vấn foreign table có thể nhìn thấy dữ liệu đó ngay lập tức.

3. **Đơn giản hóa vận hành:** Nhà phát triển tương tác với một Postgres endpoint duy nhất, sử dụng cú pháp SQL tiêu chuẩn và các ORM quen thuộc.

### Điểm đánh đổi

Dữ liệu không hề biến mất; **gánh nặng tính toán chỉ đơn giản là đã được chuyển đi nơi khác**. Thay vì một Glue worker serverless độc lập phân tích các file Parquet vào lúc 2:00 sáng, tài nguyên tính toán của cơ sở dữ liệu Aurora chính giờ đây phải đọc các object trong S3, giải nén các buffer Snappy/ZSTD và tạo các hash table theo nhu cầu.

### Kết luận

> **"Loại bỏ ETL không đồng nghĩa với loại bỏ công việc tính toán — nó chuyển trực tiếp công việc đó sang instance cơ sở dữ liệu của bạn."**

---

## 🔍 Thử nghiệm 2: ⚡ DuckDB nhúng tối ưu thao tác đọc như thế nào (và khi nào nó thất bại)?

### Cơ chế hoạt động

DuckDB không quét S3 một cách mù quáng. Cách triển khai của Aurora kết hợp ba trụ cột hiệu năng:

1. **Lược bỏ cột không cần thiết (Column Pruning):** Nếu file Parquet có 80 cột nhưng câu SQL chỉ chọn 2 cột, chỉ các dải byte chứa dữ liệu của hai cột đó được tải từ S3.

2. **Đẩy điều kiện lọc xuống nguồn (Predicate Pushdown) và bỏ qua Row Group:** DuckDB đọc metadata của file Parquet và các file manifest của Iceberg. Nếu truy vấn lọc theo `date >= '2026-01-01'`, những row group có metadata cho thấy giá trị lớn nhất thấp hơn ngưỡng đó sẽ được bỏ qua hoàn toàn mà không cần tải dữ liệu xuống.

3. **Bộ nhớ đệm đọc trên đĩa (On-Disk Read Cache):** Các khối dữ liệu từ S3 từ xa được cache trên bộ lưu trữ SSD cục bộ của Aurora instance. Những truy vấn tiếp theo đọc cùng row group sẽ truy cập SSD NVMe cục bộ thay vì gọi S3 API.

### Điều gì xảy ra nếu cách tổ chức Data Lake không tốt?

Nếu các hệ thống tạo dữ liệu ở thượng nguồn sinh ra hàng triệu file có kích thước 100 KB (vấn đề nổi tiếng **"Small Files Problem"**), hoặc dữ liệu không được partition theo những cột thường xuyên được dùng để lọc:

- DuckDB gửi hàng chục nghìn HTTP request `s3:GetObject`.

- Độ trễ tăng vọt từ mức mili giây lên hàng chục giây do chi phí xử lý request.

- Predicate pushdown không thể loại bỏ row group một cách hiệu quả.

Hơn nữa, **bộ nhớ đệm đọc trên đĩa chỉ tồn tại tạm thời**. Nếu Aurora instance khởi động lại, scale up/down hoặc failover sang một replica, cache trên SSD cục bộ sẽ bị xóa sạch. Những truy vấn đầu tiên sau một sự kiện bảo trì sẽ phải đọc dữ liệu từ S3 ở trạng thái cold cache.

### Kết luận

> **"Query engine của DuckDB có tốc độ rất cao, nhưng không thể bù đắp cho một Data Lake bị phân mảnh và không được partition hợp lý."**

---

## 🔍 Thử nghiệm 3: 💥 Điều gì xảy ra khi truy vấn phân tích hết RAM?

### Thực tế về bộ nhớ trong Aurora

PostgreSQL phân bổ phần lớn RAM của instance cho `shared_buffers` (thông thường chiếm **65% đến 75%** tổng RAM) để cache các trang dữ liệu native của cơ sở dữ liệu, phục vụ tốc độ giao dịch OLTP.

Các truy vấn foreign table **không sử dụng `shared_buffers`**. 

Thay vào đó, Aurora quản lý bộ nhớ dành cho các tác vụ phân tích của DuckDB bằng một tham số riêng: `aurora_analytics.query_mem`.

```text

┌──────────────────────────────────────────────────────────────┐

│                  Aurora Instance Total RAM                   │

├──────────────────────────────────────┬───────────────────────┤

│    shared_buffers (~65% - 75%)       │  Remaining RAM        │

│   (OLTP tables, indexes, bufferpool) │  ├── OS / Backend     │

│                                      │  └── query_mem        │

└──────────────────────────────────────┴───────────────────────┘

```

Vì `shared_buffers` chiếm phần lớn RAM, ngân sách bộ nhớ còn lại dành cho các phép hash-join lớn và các phép tổng hợp group-by bị giới hạn đáng kể.

### Điều gì xảy ra khi vượt quá `query_mem`?

1. **Tràn dữ liệu ra đĩa (Disk Spilling):** DuckDB ghi các hash table trung gian, sort buffer và các cột đã giải nén xuống **bộ lưu trữ cục bộ** của instance.

2. **Tranh chấp I/O (I/O Contention):** Dữ liệu tạm được ghi ra đĩa sử dụng chung chính SSD NVMe vật lý với **on-disk read cache** và các file tạm của PostgreSQL.

3. **Thrashing và hủy truy vấn (Query Aborts):** Nếu bộ lưu trữ cục bộ bị đầy hoặc áp lực bộ nhớ trên toàn hệ thống chạm ngưỡng Out-Of-Memory (OOM), Aurora sẽ hủy truy vấn bằng một thông báo lỗi hoặc khiến thông lượng suy giảm nghiêm trọng.

### Kết luận

> **"Các truy vấn foreign table không được ưu tiên về bộ nhớ trong một Aurora instance. Nếu không cấu hình `query_mem` cẩn thận, những truy vấn phân tích nặng sẽ tràn ra đĩa và trở thành nút thắt cổ chai I/O."**

---

## 🔍 Thử nghiệm 4: 🛡️ Thêm Aurora Dedicated Reader có giải quyết được mọi tranh chấp tài nguyên không?

### Giải pháp phòng vệ dễ nghĩ đến

"Nếu các truy vấn phân tích có nguy cơ làm quá tải writer của môi trường production, chúng ta chỉ cần triển khai một **Aurora Dedicated Reader instance** và cấu hình các endpoint tùy chỉnh!"

Đây là phương án được AWS chính thức khuyến nghị. Trên một dedicated reader:

- Bạn có thể giảm giá trị `shared_buffers` và tăng `aurora_analytics.query_mem`.

- Bạn có thể gán promotion tier 15 (ngăn instance này trở thành primary trong quá trình failover).

- Các thao tác ghi OLTP trên node primary được tách khỏi những đợt tăng đột biến CPU do DuckDB gây ra.

### Cạm bẫy tiềm ẩn: Dead Tuple Bloat và độ trễ VACUUM

Ngay cả trên dedicated reader, các Aurora instance vẫn chia sẻ cùng hệ thống lưu trữ phân tán bên dưới (`Aurora Cluster Volume`) và transaction catalog của PostgreSQL.

Nếu một truy vấn phân tích phức tạp trên reader chạy trong 45 phút:

1. Truy vấn giữ một transaction snapshot ở trạng thái mở.

2. Nó gửi phản hồi (`hot_standby_feedback`) đến primary writer instance.

3. Tiến trình autovacuum của PostgreSQL trên writer **không thể xóa các dead tuple** đã bị xóa sau thời điểm snapshot của truy vấn trên reader.

4. Kết quả: Table bloat tăng trên cơ sở dữ liệu giao dịch primary, các lượt quét index suy giảm hiệu năng và mức sử dụng đĩa tăng lên.

### Biện pháp giảm thiểu

Bạn phải áp dụng nghiêm ngặt `statement_timeout` trên analytics reader và liên tục theo dõi metric CloudWatch `oldest_reader_feedback_xid_age`.

### Kết luận

> **"Dedicated reader giúp cô lập CPU và bộ nhớ, nhưng các transaction chạy lâu vẫn có thể âm thầm làm suy giảm hiệu năng writer thông qua tình trạng MVCC bloat của PostgreSQL."**

---

## 🔍 Thử nghiệm 5: 🚫 Điều gì xảy ra khi bạn cố ghi dữ liệu hoặc sử dụng các kiểu dữ liệu Postgres phức tạp?

Trước khi thiết kế lại kiến trúc xoay quanh DuckDB foreign table, bạn cần kiểm tra các giới hạn nghiêm ngặt của tính năng này:

### 1. Bắt buộc chỉ đọc (Read-Only Enforcement)

Foreign table trong Aurora Analytics bị giới hạn nghiêm ngặt ở chế độ chỉ đọc:

- `INSERT INTO ft_purchase_history ...` ❌ Thất bại.

- `UPDATE`, `DELETE`, `TRUNCATE` và `COPY FROM` ❌ Thất bại.

Nếu ứng dụng cần ghi ngược hoặc thay đổi dữ liệu lakehouse, bạn vẫn cần các công cụ Iceberg catalog, Spark hoặc Athena.

### 2. Thiếu ánh xạ kiểu dữ liệu (Data Type Mappings)

Không phải kiểu dữ liệu PostgreSQL nào cũng có thể ánh xạ sang các kiểu dữ liệu nguyên thủy của Parquet/Iceberg:

- `JSONB` / `JSON` ❌ Không được hỗ trợ.

- `ARRAY`, `SERIAL` ❌ Không được hỗ trợ.

- `UUID` (biểu diễn nhị phân tùy chỉnh), `XML`, `INET`/`CIDR`, `RANGE`, `GEOMETRIC` ❌ Không được hỗ trợ.

Nếu schema vận hành phụ thuộc nhiều vào các cột JSONB, bạn không thể dễ dàng đưa chúng vào foreign table nếu không thực hiện flatten dữ liệu ở thượng nguồn trước.

### 3. Giới hạn về Region và định danh

- **Chỉ hỗ trợ một Region:** Một truy vấn đơn lẻ không thể liên kết các S3 bucket nằm ở nhiều AWS Region khác nhau.

- **Giới hạn định danh:** Tên cột lấy từ metadata Parquet không được vượt quá 63 ký tự; các tên bị trùng khi không phân biệt chữ hoa/chữ thường (ví dụ: `UserId` và `userid`) sẽ gây lỗi khởi tạo schema.

- **Không có system column:** Các thuộc tính hệ thống như `ctid` và `xmin` không khả dụng trên foreign table.

### Kết luận

> **"Foreign table là một góc nhìn phục vụ đọc và phân tích dữ liệu, không phải giải pháp thay thế lưu trữ PostgreSQL cho mọi mục đích."**

---

## 🔍 Thử nghiệm 6: 💸 Hóa đơn AWS sẽ thay đổi như thế nào?

Tài liệu AWS nêu rõ bản thân tính năng `aurora_analytics` **không phát sinh thêm phí giấy phép phần mềm**. 

Tuy nhiên, "không mất phí cho tính năng" không có nghĩa là "không mất chi phí hạ tầng".

```text

Traditional ETL Model Costs:

  - AWS Glue Jobs / EMR Cluster time

  - Duplicate Aurora Storage ($0.10/GB-month)

  - Inter-service transfer

Direct DuckDB Query Model Costs:

  - Aurora ACU (Serverless v2) or Provisioned DB Instance Compute

  - Amazon S3 API Costs: `s3:GetObject` ($0.0004 per 1,000 requests) & `s3:ListBucket`

  - Aurora Local Storage consumption for disk spilling & cache

```

### Cạm bẫy chi phí với Aurora Serverless v2

Nếu hàng chục người dùng đồng thời chạy các truy vấn lake theo nhu cầu, Aurora Serverless v2 sẽ nhanh chóng tăng ACU (Aurora Capacity Units) để đáp ứng nhu cầu CPU/bộ nhớ. Vì đơn giá ACU theo giờ cao hơn đáng kể so với tài nguyên tính toán theo lô (như EMR sử dụng Spot Instance hoặc Glue Flex), các truy vấn phân tích khó dự đoán có thể khiến hóa đơn Aurora tăng đột biến ngoài dự kiến.

### Kết luận

> **"Chi phí không biến mất; nó chuyển từ các khoản xử lý theo lô (Glue/EMR) sang tài nguyên tính toán cơ sở dữ liệu (Aurora ACU) và số lượng request đến S3 API."**

---

## 📌 Tổng kết — Khung tư duy kiến trúc

### Mô hình phổ biến

Loại bỏ một pipeline ETL không làm mất đi công việc chuyển đổi dữ liệu, đệm dữ liệu trong bộ nhớ hay lọc dữ liệu. Nó chỉ chuyển khối lượng công việc đó vào bên trong database engine.

Khi đưa Data Lake vào PostgreSQL:

- **Tính linh hoạt của nhà phát triển tăng đáng kể** (một truy vấn, không có độ trễ đồng bộ).

- **Rủi ro về độ tin cậy của cơ sở dữ liệu cũng tăng tương ứng** (tranh chấp tài nguyên, tràn dữ liệu ra đĩa, giữ snapshot).

### Khung đánh giá gồm ba câu hỏi

Khi quyết định có nên sử dụng truy vấn trực tiếp Data Lake bằng Aurora + DuckDB hay không, hãy tự hỏi:

**1. "Hình dạng và tần suất truy vấn là gì?"**

- Các truy vấn tra cứu từng điểm kết hợp 100 dòng đang hoạt động với dữ liệu lịch sử đã tổng hợp và lọc? 👉 **Sử dụng DuckDB Foreign Table.**

- Các lượt quét tổng hợp nặng trên 500 triệu dòng, lặp lại nhiều lần mỗi phút? 👉 **Tiếp tục sử dụng một OLAP engine chuyên dụng (Athena, Redshift, Snowflake, ClickHouse).**

**2. "Đội ngũ của chúng ta có thể chấp nhận schema nghiêm ngặt và ngữ nghĩa chỉ đọc không?"**

- Schema của Data Lake có thay đổi thường xuyên không? Có các kiểu dữ liệu không được hỗ trợ như `JSONB` hoặc `ARRAY` không? Nếu có, foreign table sẽ làm tăng khó khăn trong việc bảo trì schema.

**3. "Để xảy ra lỗi ở đâu thì ít tốn kém hơn?"**

- Pipeline ETL gặp lỗi sẽ khiến các báo cáo phân tích bị cũ.

- Aurora database gặp lỗi có thể khiến ứng dụng ngừng hoạt động.

### Ma trận ra quyết định

| Mô hình kiến trúc | Trường hợp hoạt động hiệu quả | Đánh đổi / Rủi ro chính |

|---|---|---|

| **Pipeline ETL truyền thống** | Truy vấn thông lượng cao, độ trễ dưới một giây và cần các bảng được materialize sẵn trong Aurora | Chi phí kỹ thuật và vận hành cao, độ trễ đồng bộ dữ liệu, lưu trữ trùng lặp |

| **Aurora + DuckDB nhúng** | Kết hợp các dòng giao dịch đang hoạt động với dữ liệu lịch sử ít truy cập trên S3; báo cáo theo nhu cầu; cổng thông tin nội bộ | Áp lực bộ nhớ trên PostgreSQL, tràn dữ liệu ra đĩa, cache cục bộ bị mất hiệu lực khi khởi động lại |

| **OLAP chuyên dụng (Athena / Redshift)** | Phân tích nặng cho nhiều tenant, dashboard BI phức tạp, quét dữ liệu ở quy mô petabyte | Nhiều database endpoint, thông tin xác thực truy cập riêng biệt, kiến trúc phức tạp hơn |

### Lời kết

Một kỹ sư junior nhìn vào bản phát hành Aurora DuckDB mới và nói:

> **"Tuyệt vời! Chúng ta có thể xóa tất cả pipeline ETL và truy vấn mọi thứ trực tiếp từ PostgreSQL."**

Một kiến trúc sư cấp cao nhìn vào chính bản phát hành đó và đặt câu hỏi:

> **"Nếu một truy vấn không có index quét một bảng S3 500 GB trên reader trong thời điểm lưu lượng Black Friday đạt đỉnh, điều gì sẽ xảy ra với transaction log và độ trễ replication của instance primary?"**

Tư duy kiến trúc thực sự bắt đầu khi bạn nhìn xa hơn thông báo về tính năng mới và lập kế hoạch cho những đánh đổi vận hành ẩn phía sau nó.