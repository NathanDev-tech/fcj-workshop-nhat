---
title: "Blog 3"
date: 2026-10-30
weight: 3
chapter: false
pre: " <b> 3.3. </b> "
---

# 🐍 Boto3 và DynamoDB: Khi câu lệnh Scan "tưởng chừng đơn giản" trở thành cái bẫy kiến trúc

## 📝 Mở đầu: Câu chuyện từ một tác vụ "chỉ mất 10 phút"

Khi làm việc với các dịch vụ dữ liệu trên AWS bằng Python, **Boto3** và **Amazon DynamoDB** gần như là cặp bài trùng quen thuộc nhất đối với mọi kỹ sư đám mây. Kịch bản khởi đầu thường rất nhẹ nhàng:

> *"Tôi có một bảng DynamoDB lưu trữ danh mục sản phẩm. Yêu cầu đặt ra rất đơn giản: viết một script Python ngắn để trích xuất dữ liệu, lọc các sản phẩm theo điều kiện và xuất ra báo cáo định dạng JSON hoặc CSV."*

Với một lập trình viên đã quen thuộc với SQL hoặc các cơ sở dữ liệu quan hệ, tư duy tự nhiên sẽ là:
1. Mở kết nối đến bảng qua SDK Boto3.
2. Gọi hàm quét dữ liệu `scan()` kèm theo một bộ lọc điều kiện.
3. Nhận danh sách kết quả và ghi trực tiếp ra file bằng thư viện `json.dump()`.

Mọi thứ nghe như một bài tập cơ bản chỉ mất chưa đầy 10 phút. Nhưng khi đưa script vào chạy trên môi trường thực tế với dữ liệu hàng nghìn bản ghi, hệ thống lập tức "dội hai gáo nước lạnh" vào mặt người phát triển:

* **Cú sốc dữ liệu biến mất (Pagination Pitfall):** Bảng có hàng ngàn dòng, bạn biết chắc chắn có dữ liệu thỏa mãn điều kiện lọc, nhưng kết quả trả về từ lệnh scan lại là một mảng rỗng `[]`. Script lập tức dừng lại và bỏ sót phần lớn dữ liệu phía sau.
* **Cơn ác mộng Serialization:** Lệnh ghi file JSON sụp đổ ngay lập tức với lỗi ngoại lệ quen thuộc:  
  `TypeError: Object of type Decimal is not JSON serializable`.

Tại sao một tác vụ đọc dữ liệu tưởng như sơ đẳng lại ẩn chứa những cái bẫy kỳ lạ như vậy? DynamoDB và Boto3 đã làm gì bên dưới tầng lưu trữ phân tán mà các tài liệu "Hello World" ít khi đề cập tới?

Bài viết này sẽ không cung cấp hàng trăm dòng code mẫu để bạn copy-paste. Thay vào đó, chúng ta sẽ cùng bóc tách **bản chất kiến trúc bên dưới**: từ cơ chế phân trang vật lý 1MB, triết lý đằng sau kiểu dữ liệu `Decimal`, cho đến ảo tưởng tai hại về chi phí khi sử dụng `FilterExpression` trong môi trường Production.

---

## 🏗️ Bối cảnh & Cơ chế vận hành bên dưới (Under the Hood)

Để hiểu rõ tại sao những hành vi trên lại xảy ra, chúng ta cần nhìn vào cách thức Boto3 giao tiếp với các phân vùng lưu trữ (partitions) của DynamoDB qua hai sơ đồ kiến trúc sau:

### 1. Kiến trúc luồng trích xuất dữ liệu tổng thể

Sơ đồ dưới đây mô tả hành trình từ lúc client gửi yêu cầu trích xuất dữ liệu, đi qua các lớp xác thực IAM, phân tuyến request, truy cập phân vùng vật lý và chuẩn hóa đầu ra:

{{< mermaid align="center" >}}
flowchart LR
    subgraph ClientEnv ["Môi trường thực thi (Local / EC2 / Lambda)"]
        Script["Python Automation Script"]
        Boto3["AWS SDK for Python (Boto3)"]
        Encoder["Data Transformation Layer"]
        Script --> Boto3
        Boto3 --> Encoder
    end

    subgraph AWS_Security ["Bảo mật & Quyền hạn"]
        IAM["AWS IAM Role / STS<br/>Quyền: dynamodb:Scan"]
    end

    subgraph AWS_Cloud ["Dịch vụ Amazon DynamoDB"]
        Router["Request Router<br/>(Hệ thống định tuyến phân tán)"]
        subgraph Partitions ["Storage Partitions (Các phân vùng vật lý)"]
            P1["Partition 1<br/>(Dải Hash A)"]
            P2["Partition 2<br/>(Dải Hash B)"]
            P3["Partition 3<br/>(Dải Hash C)"]
        end
    end

    subgraph OutputDestination ["Đích xuất dữ liệu"]
        JSON_File["Báo cáo JSON"]
        CSV_File["Báo cáo CSV"]
    end

    Boto3 -.->|"1. Xác thực IAM"| IAM
    Boto3 -->|"2. Gửi HTTPS Scan Request"| Router
    Router --> P1 & P2 & P3
    P1 & P2 & P3 -->|"3. Đọc dữ liệu thô (Tối đa 1MB)"| Router
    Router -->|"4. Trả về Response (Items + LastKey)"| Boto3
    Encoder -->|"5. Chuẩn hóa dữ liệu & Xuất file"| JSON_File & CSV_File
{{< /mermaid >}}

---

### 2. Vòng đời phân trang nội bộ của DynamoDB Engine

Điểm mấu chốt gây hiểu lầm cho nhiều lập trình viên nằm ở **thứ tự xử lý** giữa việc đọc dữ liệu thô và việc áp dụng bộ lọc trên máy chủ DynamoDB:

{{< mermaid align="center" >}}
flowchart TD
    Start(["Bắt đầu: Gửi lệnh Scan"]) --> ReadStorage["1. Đọc dữ liệu từ đĩa vật lý (Storage Partitions)"]
    
    subgraph DynamoDB_Engine ["Bên trong DynamoDB Engine"]
        ReadStorage --> Check1MB{"2. Đã đọc đủ 1MB<br/>hoặc chạm mốc cuối bảng?"}
        Check1MB -- "Chưa đủ" --> ReadStorage
        Check1MB -- "Đủ 1MB / Hết bảng" --> ApplyFilter["3. Áp dụng FilterExpression trên bộ nhớ đệm<br/>(Loại bỏ item không khớp)"]
        ApplyFilter --> PrepareResp["4. Đóng gói kết quả:<br/>- Items: Các bản ghi thỏa mãn<br/>- LastEvaluatedKey: Con trỏ vị trí tiếp theo"]
    end

    PrepareResp --> ReceiveResp["Client nhận phản hồi"]
    ReceiveResp --> CheckLastKey{"Response có chứa<br/>LastEvaluatedKey?"}
    
    CheckLastKey -- "CÓ: Dữ liệu vẫn còn!" --> UpdateKey["Gán ExclusiveStartKey = LastEvaluatedKey"]
    UpdateKey --> ReadStorage
    
    CheckLastKey -- "KHÔNG: Đã duyệt toàn bộ bảng" --> Finish(["Hoàn thành quét toàn bộ"])

    classDef highlight fill:#ff9900,stroke:#333,stroke-width:2px,color:#000;
    class CheckLastKey,ApplyFilter highlight;
{{< /mermaid >}}

> ⚠️ **Quy luật bất biến:** DynamoDB thực hiện bước **1 & 2 (Đọc dữ liệu thô từ đĩa cứng)** trước khi thực hiện bước **3 (Lọc dữ liệu trên bộ nhớ)**. Sự tách biệt này chính là nguồn gốc của cái bẫy phân trang đầu tiên.

---

## 🔍 Bóc tách 1: Giới hạn vật lý 1MB và nghịch lý "Scan rỗng nhưng dữ liệu vẫn còn"

### Giới hạn cứng 1MB (1MB Hard Limit)
Trong mỗi một network round-trip, DynamoDB đặt ra một giới hạn cứng: **chỉ quét tối đa 1 MB dữ liệu thô** từ các phân vùng lưu trữ.

Hãy hình dung một kịch bản thực tế:
* Bảng của bạn có kích thước 10 MB, chứa khoảng 20.000 sản phẩm.
* Bạn muốn tìm những chiếc laptop cao cấp có giá trên 3.000 USD, và những sản phẩm này tình cờ nằm rải rác ở nửa sau của bảng.
* Khi bạn gọi `table.scan(FilterExpression=...)`, DynamoDB bắt đầu đọc 1 MB dữ liệu đầu tiên từ phân vùng vật lý.
* Sau khi đọc xong 1 MB này, nó mới áp dụng bộ lọc điều kiện. 
* **Tình huống xảy ra:** Không có chiếc laptop nào trong 1 MB đầu tiên này có giá trên 3.000 USD.
* **Kết quả trả về cho bạn:**
  - Danh sách `Items`: **Rỗng** (`[]`)
  - Số lượng `Count`: **0**
  - Số lượng bản ghi đã quét `ScannedCount`: **khoảng 2.000 bản ghi**
  - Đi kèm một trường đặc biệt: `LastEvaluatedKey` (con trỏ lưu vị trí dừng lại của lần đọc 1 MB vừa rồi).

### Cái bẫy chết người trong logic code
Đa số lập trình viên khi thấy danh sách `Items` trả về rỗng sẽ có phản xạ tự nhiên:

> *"Nếu không có phần tử nào trả về, nghĩa là đã hết dữ liệu hoặc không có bản ghi nào thỏa mãn điều kiện. Dừng vòng lặp thôi!"*

Đây chính là sai lầm nghiêm trọng nhất. Script ngắt ngay sau lần gọi đầu tiên và bỏ sót hoàn toàn 9 MB dữ liệu còn lại ở phía sau. Người dùng sẽ nhận được một bản báo cáo rỗng, trong khi thực tế trong hệ thống vẫn có hàng chục sản phẩm thỏa mãn điều kiện.

### Tư duy phân trang con trỏ không trạng thái (Stateless Cursor Pagination)
Trong thế giới NoSQL phân tán, phân trang không hoạt động theo kiểu `page=1, page=2` hay `OFFSET 50` như SQL truyền thống. Nó hoạt động hoàn toàn dựa trên **con trỏ vị trí (Cursor)**.

Nguyên tắc vàng cần ghi nhớ:
1. **Sự hiện diện của `LastEvaluatedKey` là tiêu chí duy nhất** để xác định bảng còn dữ liệu cần duyệt tiếp hay không.
2. Số lượng phần tử trong mảng `Items` hoàn toàn độc lập với việc còn dữ liệu hay không. `Items` có thể rỗng, nhưng chừng nào còn `LastEvaluatedKey`, bạn vẫn phải tiếp tục gửi request kế tiếp kèm theo tham số `ExclusiveStartKey` được gán bằng con trỏ đó.
3. Vòng lặp chỉ được phép kết thúc khi trường `LastEvaluatedKey` không còn xuất hiện trong phản hồi từ DynamoDB.

---

## 🔍 Bóc tách 2: Cơn ác mộng `Decimal` — Khi chuẩn số học xung đột với chuẩn truyền tải JSON

Sau khi vượt qua bẫy phân trang và lấy về đầy đủ dữ liệu, lập trình viên thường tự tin gọi hàm xuất file:
`json.dump(items, file)` — và ngay lập tức gặp lỗi:

```text
TypeError: Object of type Decimal is not JSON serializable
```

Tại sao một thư viện chuẩn như Boto3 lại biến các số nguyên và số thực đơn giản thành kiểu đối tượng `Decimal` phức tạp của Python?

### Triết lý thiết kế đằng sau kiểu `Decimal` của Boto3
Trong cơ sở dữ liệu DynamoDB, kiểu dữ liệu `Number` hỗ trợ độ chính xác lên đến **38 chữ số** (bao gồm cả số nguyên lẫn số thập phân).

Ngược lại, kiểu số thực `float` trong ngôn ngữ Python (và hầu hết các ngôn ngữ lập trình hiện đại) tuân theo chuẩn **IEEE-754** (64-bit floating point). Chuẩn này nổi tiếng với các sai số làm tròn số học do cách máy tính biểu diễn số nhị phân:
* Trong Python, phép tính cộng đơn giản `0.1 + 0.2` sẽ cho ra kết quả `0.30000000000000004` thay vì con số `0.3` tròn trĩnh.

Bây giờ hãy thử tưởng tượng: DynamoDB đang lưu trữ số dư tài khoản ngân hàng, giá trị tiền mã hóa hoặc đơn giá chứng khoán. Nếu Boto3 tự động ép kiểu dữ liệu từ DynamoDB sang `float`, các sai số làm tròn số học sẽ tích lũy và gây ra sự chênh lệch tài chính nghiêm trọng. 

Vì vậy, các kỹ sư thiết kế SDK Boto3 đã đưa ra một quyết định an toàn tuyệt đối về mặt kiến trúc: **Ánh xạ toàn bộ kiểu Number trong DynamoDB sang đối tượng `decimal.Decimal` của Python**.

### Xung đột với bộ phân tích JSON tiêu chuẩn
Vấn đề nảy sinh từ việc thư viện `json` mặc định của Python được xây dựng để chỉ hỗ trợ các kiểu dữ liệu nguyên thủy cơ bản: chuỗi, số nguyên, số thực, boolean, từ điển, danh sách và giá trị rỗng (`None`). Khi gặp một đối tượng `Decimal`, bộ parser mặc định hoàn toàn không biết phải chuyển đổi nó sang định dạng chuỗi JSON như thế nào, dẫn đến ngoại lệ `TypeError`.

### Các hướng tiếp cận ở tầng kiến trúc
Để giải quyết bài toán này mà không phá vỡ tính toàn vẹn của dữ liệu, bạn có ba lựa chọn về mặt thiết kế:

1. **Xây dựng Custom Encoder tại tầng Serialization (Khuyên dùng):**  
   Tạo một lớp kế thừa từ `json.JSONEncoder` của Python. Trong đó, kiểm tra nếu đối tượng gặp phải là `Decimal`, hãy chuyển đổi nó thành `int` (nếu là số nguyên) hoặc `float` (nếu là số thập phân) ngay tại thời điểm chuẩn bị ghi ra file. Cách này giữ cho toàn bộ dữ liệu trong bộ nhớ vẫn giữ nguyên độ chính xác, chỉ chuyển đổi khi thực sự cần xuất bản.
2. **Sử dụng bộ Parser nâng cao (như thư viện `simplejson`):**  
   Thay vì dùng thư viện chuẩn, sử dụng các thư viện thay thế hỗ trợ sẵn việc nhận diện và tự động chuyển đổi đối tượng `Decimal` sang số học JSON mà không cần viết thêm mã xử lý.
3. **Phẳng hóa và làm sạch dữ liệu trước khi chuyển giao (Data Sanitization Pipeline):**  
   Nếu dữ liệu lấy về từ DynamoDB cần được chuyển tiếp sang các hệ thống khác (như trả về qua FastAPI, đưa vào hàng đợi Celery, hoặc ghi vào MongoDB), cách an toàn nhất là chạy dữ liệu qua một hàm làm sạch đệ quy để chuẩn hóa toàn bộ kiểu `Decimal` thành kiểu số nguyên thủy trước khi dữ liệu rời khỏi tầng giao tiếp cơ sở dữ liệu.

---

## 🔍 Bóc tách 3: Ảo tưởng về `FilterExpression` và cái giá đắt đỏ của thao tác `Scan` trong Production

Một trong những nhận thức sai lầm phổ biến và tai hại nhất về mặt tài chính trong DynamoDB là:

> *"Tôi đã truyền điều kiện vào `FilterExpression`, server chỉ trả về đúng 5 bản ghi tôi cần, nên tôi chỉ phải trả chi phí đọc cho 5 bản ghi đó."*

Đây là một ảo tưởng hoàn toàn trái ngược với thực tế vận hành của DynamoDB.

### Sự thật trần trụi về đơn vị đọc (Read Capacity Unit - RCU)
Cách tính phí của DynamoDB dựa trên **lượng dữ liệu thô mà nó phải đọc từ đĩa cứng vật lý**, chứ không phải lượng dữ liệu mà nó trả về cho bạn qua mạng:

* Giả sử bảng của bạn có dung lượng **10 GB**.
* Bạn chạy lệnh `Scan` với `FilterExpression` để tìm kiếm các sản phẩm thuộc nhãn hàng X (vốn chỉ chiếm tổng cộng **50 MB** trong bảng).
* DynamoDB sẽ duyệt qua toàn bộ các phân vùng lưu trữ, đọc đủ **10 GB dữ liệu thô**, sau đó áp dụng bộ lọc trên RAM của server và chỉ trả về 50 MB dữ liệu cho bạn.
* **Hóa đơn AWS tính phí:** Bạn vẫn phải trả chi phí tiêu thụ RCU cho toàn bộ **10 GB dữ liệu đã đọc**, không thiếu một byte nào!

```text
┌─────────────────────────────────────────────────────────────┐
│                    DYNAMODB STORAGE PARTITIONS              │
│ [Partition 1: 2.5GB] [Partition 2: 2.5GB] [Partition 3: ...]│
└─────────────────────────────────────────────────────────────┘
                               ▲
                        table.scan(...)
              (Đọc và tiêu tốn RCU toàn bộ 10GB)
                               │
                     ┌─────────────────────┐
                     │  FilterExpression   │
                     │  (Vứt bỏ 9.95GB data)│
                     └─────────────────────┘
                               ▼
                     Chỉ trả về 50MB cho Client!
```

### Hai hiểm họa vận hành khi lạm dụng Scan

1. **Bùng nổ chi phí (Cost Explosion):**  
   Nếu bảng của bạn đang chạy ở chế độ **On-Demand** (`PAY_PER_REQUEST`), mỗi lần một tác vụ tự động hóa chạy lệnh scan trên một bảng hàng chục gigabyte, chi phí cho mỗi lượt quét có thể lên tới hàng chục USD. Nếu script đó được cài đặt chạy định kỳ mỗi 15 phút qua cron job, hóa đơn đám mây cuối tháng sẽ là một cú sốc thực sự.
2. **Nghẽn bảng và làm sập ứng dụng chính (Throttling Blast Radius):**  
   Nếu bảng đang chạy ở chế độ **Provisioned**, dung lượng đọc được cấp phát cố định cho mỗi giây (ví dụ: 100 RCU/giây). Một câu lệnh scan quét dữ liệu ồ ạt sẽ ngay lập tức "vét cạn" toàn bộ hạn mức đọc của phân vùng. Kết quả là các ứng dụng web hoặc API phục vụ khách hàng trực tiếp đang cố gắng đọc/ghi dữ liệu đơn lẻ sẽ bị từ chối với lỗi `ProvisionedThroughputExceededException`. Một script báo cáo nội bộ đã vô tình làm sập toàn bộ hệ thống phục vụ người dùng cuối!

### Khi nào được phép dùng `Scan` và các giải pháp thay thế chuẩn mực?

* **Khi nào được phép dùng Scan?**  
  Chỉ nên dùng Scan cho các bảng cấu hình rất nhỏ (dưới vài megabyte), trong môi trường phát triển (Dev/Staging), hoặc các batch job bảo trì chạy vào khung giờ thấp điểm ban đêm với cơ chế giới hạn tốc độ (rate-limiting) nghiêm ngặt.
* **Giải pháp thay thế 1: Thiết kế lại khóa chính và sử dụng `Query`**  
  Trong DynamoDB, `Query` là câu lệnh hiệu quả nhất vì nó nhảy thẳng đến phân vùng chính xác dựa trên Partition Key và chỉ đọc đúng phạm vi dữ liệu dựa trên Sort Key. Không có một byte dữ liệu thừa nào bị đọc.
* **Giải pháp thay thế 2: Xây dựng Global Secondary Index (GSI)**  
  Nếu bạn thường xuyên cần lọc dữ liệu theo một thuộc tính không phải khóa chính (ví dụ: lọc sản phẩm theo danh mục), hãy tạo GSI với Partition Key là thuộc tính đó. Khi đó bạn có thể dùng `Query` trên Index thay vì phải `Scan` toàn bộ bảng.
* **Giải pháp thay thế 3: Tính năng DynamoDB Export to Amazon S3 (Zero-RCU)**  
  Nếu bạn cần trích xuất toàn bộ bảng dữ liệu lớn (hàng chục GB hoặc hàng TB) phục vụ cho nhu cầu phân tích, báo cáo hoặc sao lưu Data Lake, tuyệt đối không nên viết script Boto3 để scan. Hãy sử dụng tính năng **Export to S3** tích hợp sẵn của AWS. Cơ chế này đọc trực tiếp từ tầng snapshot nội bộ của DynamoDB, xuất dữ liệu thẳng sang S3 dưới dạng Parquet hoặc JSON mà **hoàn toàn không tiêu tốn dù chỉ một đơn vị RCU** của bảng live và không ảnh hưởng đến người dùng ứng dụng.

---

## ⚖️ Đúc kết & Bài học tư duy kiến trúc

Làm việc với các dịch vụ điện toán đám mây và cơ sở dữ liệu phân tán không chỉ đơn thuần là việc nhớ tên hàm SDK hay viết được vài dòng script chạy được trên máy local. Những chi tiết kiến trúc ngầm định bên dưới chính là ranh giới giữa một đoạn mã nghiệp dư và một hệ thống chuẩn Production:

| Khía cạnh | Tư duy thông thường | Bản chất kiến trúc DynamoDB |
|---|---|---|
| **Phân trang dữ liệu** | Kiểm tra mảng kết quả `Items` rỗng để dừng lại | Phân trang dựa trên con trỏ `LastEvaluatedKey`. `Items` rỗng không có nghĩa là đã duyệt hết bảng. |
| **Kiểu dữ liệu số** | Mong đợi số nguyên/số thực nguyên thủy | Boto3 bắt buộc dùng `Decimal` để bảo vệ độ chính xác 38 chữ số, đòi hỏi cơ chế chuyển đổi tại tầng serialize. |
| **Lọc dữ liệu** | Nghĩ rằng `FilterExpression` giúp giảm chi phí đọc | Phí RCU tính trên 100% dữ liệu thô đọc từ đĩa trước khi lọc. Filter chỉ giúp giảm băng thông mạng gửi về client. |
| **Chiến lược đọc lớn** | Dùng vòng lặp `Scan` để kéo toàn bộ bảng về | Dùng `Query` trên Index, hoặc chuyển sang tính năng `DynamoDB Export to S3` để đạt hiệu quả chi phí và zero-RCU impact. |

> *"Hiểu được cơ chế lưu trữ phân tán bên dưới nắp capo quan trọng hơn nhiều việc thuộc lòng cú pháp thư viện. Khi bạn tôn trọng cách thức dịch vụ đám mây vận hành, bạn sẽ viết ra những đoạn mã vừa bền bỉ, vừa tiết kiệm chi phí cho doanh nghiệp."*

Hy vọng những phân tích chuyên sâu trong bài viết này sẽ giúp bạn tự tin hơn khi thiết kế, vận hành và trích xuất dữ liệu từ Amazon DynamoDB mà không còn phải giật mình trước những lỗi ngoại lệ hay những hóa đơn đám mây bất ngờ!