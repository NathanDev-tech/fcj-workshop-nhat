# Architecture Thinking: Khi Bỏ Từng Service Một — Bạn Học Được Gì?

*Một bài viết về cách học AWS theo chiều sâu — không phải bằng cách học thêm, mà bằng cách bỏ dần đi.*

---

## Học AWS kiểu gì cho đúng?

Hầu hết mọi người học AWS bằng cách ghi nhớ từng service làm gì. Route 53 là DNS. CloudFront là CDN. EC2 là máy chủ ảo. RDS là database được quản lý. CloudWatch là monitoring. Và cứ thế tiếp tục.

Cách học đó không sai — nhưng nó chỉ cho bạn biết *cái gì*, không phải *tại sao*.

Theo mình, kiến trúc tư duy thực sự bắt đầu khi bạn đặt câu hỏi ngược lại: **"Nếu tôi bỏ service này đi, điều gì xảy ra?"** Đó là lúc bạn bắt đầu hiểu giá trị thật sự của từng mảnh trong hệ thống — và đó cũng là cách bạn học architecture thinking thay vì chỉ thuộc service definition.

Bài viết này lấy kiến trúc production đầy đủ làm nền, rồi lần lượt tháo từng service ra — quan sát hệ thống thay đổi như thế nào, ai bị ảnh hưởng, và trade-off thực sự là gì.

---

## Kiến Trúc Gốc — "Production-Ready Architecture"

Đây là kiến trúc chúng ta sẽ phân tích. Mỗi service đều có lý do tồn tại:

**Luồng traffic từ User đến Database:**

```
User → Internet
  → Amazon Route 53 (DNS resolution)
  → Amazon CloudFront (CDN + TLS termination)
      └── AWS WAF (attaches to / protects CloudFront)
  → Internet Gateway (cổng vào VPC)
  → Application Load Balancer — ALB (Public Subnet)
      ← NAT Gateway (cùng Public Subnet, cho EC2 ra ngoài)
  → Amazon EC2 × 2 (Private Subnet, Auto Scaling Group)
  → Amazon RDS Multi-AZ (Private Subnet)

Monitoring toàn bộ hệ thống:
  → Amazon CloudWatch (Monitoring & Audit)
```

Mình gọi đây là **"production-ready architecture"** — đủ để phục vụ traffic thật, đủ để không bị đánh thức lúc 3 giờ sáng quá thường xuyên, và đủ bảo mật để không bị bypass bởi các attack cơ bản.

Bây giờ, hãy bắt đầu tháo dỡ từng lớp.

---

## Phần 1: Nếu Bỏ Internet Gateway Thì Sao?

Trước khi nói về CloudFront hay ALB, cần hiểu Internet Gateway (IGW) — vì đây là thứ nhiều người hay bỏ qua nhất trong kiến trúc.

### IGW làm gì?

IGW là **cổng duy nhất** kết nối VPC với internet. Không có IGW, VPC của bạn là một mạng nội bộ hoàn toàn cô lập — không có gì vào được, không có gì ra được.

Điều quan trọng mà nhiều người nhầm: **CloudFront không kết nối thẳng vào ALB**. CloudFront là edge service nằm bên ngoài VPC, và khi nó cần forward request đến ALB origin — nó đi qua internet bình thường, qua IGW, rồi mới vào đến ALB trong Public Subnet.

Luồng đúng:

```
CloudFront → [Internet] → Internet Gateway → ALB (Public Subnet trong VPC)
```

### Nếu bỏ IGW thì sao?

CloudFront không thể gọi vào ALB. Mọi request đến CloudFront sẽ nhận về lỗi 502 Bad Gateway — origin unreachable. Toàn bộ hệ thống offline với người dùng, dù EC2 và RDS vẫn đang chạy bình thường bên trong VPC.

Ngoài ra, NAT Gateway — thứ cho phép EC2 trong private subnet ra internet để cập nhật OS và gọi external API — cũng mất đường ra. EC2 hoàn toàn bị cô lập.

### Verdict

IGW không phải là "optional component". Nó là nền tảng bắt buộc. Không có IGW, public subnet không có ý nghĩa gì.

> **"Internet Gateway không làm gì fancy — nó chỉ là cánh cửa. Nhưng không có cánh cửa, căn nhà là nhà tù."**

---

## Phần 2: Nếu Bỏ NAT Gateway Thì Sao?

### NAT Gateway làm gì và tại sao nằm trong Public Subnet?

EC2 instances trong Private Subnet không có public IP. Đây là thiết kế đúng về bảo mật — EC2 không cần expose ra internet trực tiếp. Nhưng EC2 vẫn cần **đi ra** internet để:

- Cập nhật OS packages (yum update, apt-get)
- Gọi external APIs (payment gateway, third-party services)
- Pull Docker images từ Docker Hub
- Kết nối AWS services qua public endpoint

NAT Gateway đứng ở Public Subnet, có Elastic IP, và "đại diện" cho EC2 khi giao tiếp với internet. EC2 gửi request ra ngoài → NAT Gateway thay thế source IP bằng Elastic IP của nó → request đi ra ngoài qua IGW.

**Tại sao NAT Gateway phải ở Public Subnet?** Vì nó cần Elastic IP (public IP) để giao tiếp với internet, và chỉ Public Subnet mới có route đến IGW.

### Nếu bỏ NAT Gateway thì sao?

EC2 trong Private Subnet mất hoàn toàn outbound internet. Ngay lập tức:

- Không update được OS → security vulnerabilities tích lũy theo thời gian
- Không gọi được external API → nhiều tính năng app bị broken
- Không pull được Docker image mới → deployment bị chặn
- Không kết nối được một số AWS services (những service không có VPC Endpoint)

Hệ thống vẫn phục vụ được user request vào — ALB nhận traffic, forward cho EC2, EC2 xử lý và trả về. Nhưng bất kỳ tính năng nào cần EC2 **đi ra ngoài** đều fail silently.

### "Thì để EC2 trong Public Subnet cho xong"

Đây là giải pháp tệ hơn. EC2 trong Public Subnet có public IP, có nghĩa là attacker có thể tấn công thẳng vào EC2, bypass ALB và WAF. Bạn đánh đổi $32/tháng (chi phí NAT Gateway) lấy một bề mặt tấn công lớn hơn nhiều.

### Kiến trúc đúng trong sơ đồ

Trong sơ đồ bạn vẽ, NAT Gateway nằm trong Public Subnet và có mũi tên từ **EC2 Private Subnet ra ngoài qua NAT Gateway rồi qua IGW** — đây chính xác là luồng outbound của EC2. NAT Gateway không nhận traffic từ internet vào — nó chỉ cho EC2 đi ra.

### Verdict

NAT Gateway là "silent protector" — bạn không thấy nó làm gì cho đến khi nó không có mặt và mọi thứ bắt đầu vỡ.

> **"Bỏ NAT Gateway để tiết kiệm $32/tháng — rồi bạn dùng $300/giờ engineering time để debug tại sao EC2 không gọi được payment API."**

---

## Phần 3: Nếu Bỏ CloudFront Thì Sao?

### CloudFront làm gì trong kiến trúc này?

Nhìn vào sơ đồ, CloudFront nằm giữa Route 53 và Internet Gateway — và WAF được gắn trực tiếp vào nó (attaches to / protects). CloudFront đảm nhận nhiều việc cùng lúc:

**TLS termination tại edge.** User kết nối HTTPS đến CloudFront edge location gần nhất (Singapore, Tokyo, Sydney...). Certificate SSL được xử lý tại đây — không phải tại ALB trong region. Điều này có nghĩa là TLS handshake nhanh hơn vì edge gần user hơn.

**CDN caching.** Static assets (images, CSS, JS, fonts) được cache tại edge. Khi user request file đó lần thứ hai, CloudFront trả về từ cache — không cần đi vào ALB, không cần EC2 xử lý. Giảm latency từ 200ms xuống còn ~20ms cho cached content.

**Entry point duy nhất cho WAF.** Vì WAF được attach vào CloudFront, toàn bộ traffic đều phải đi qua WAF trước khi vào ALB. Không có đường tắt.

**Che giấu ALB origin.** User không biết ALB domain là gì. Họ chỉ thấy domain CloudFront. Kết hợp với ALB Security Group chỉ cho phép CloudFront prefix list — bạn có thể chắc chắn rằng request đến ALB đều đã qua CloudFront.

### Nếu bỏ CloudFront thì sao?

Luồng trở thành: Route 53 → IGW → ALB trực tiếp.

**WAF mất điểm bám.** WAF không thể đứng độc lập trong luồng traffic. Khi không có CloudFront, WAF phải attach vào ALB — điều này vẫn được, nhưng bạn mất lợi thế edge filtering. Request độc hại đi vào tận region rồi mới bị chặn — tốn bandwidth và tăng chi phí.

**Latency tăng cho global users.** User ở Hà Nội kết nối HTTPS đến ALB ở Singapore — TLS handshake phải đi toàn bộ đường đó. Với CloudFront, TLS handshake xảy ra tại Hanoi PoP — nhanh hơn đáng kể.

**Static content không được cache.** Mỗi request cho image hay CSS đều phải vào đến EC2. Tưởng tượng 10,000 users đồng thời request cùng 1 logo file — với CloudFront, chỉ có 1 request đến EC2, còn lại 9,999 request được serve từ cache.

**ALB domain bị expose.** Không có CloudFront làm proxy, ALB DNS name có thể bị discover qua nhiều cách (certificate transparency logs, network scanning). Attacker có thể bypass WAF bằng cách hit ALB trực tiếp.

### Verdict

CloudFront không chỉ là "CDN để tăng tốc". Trong kiến trúc này, nó là security layer, performance layer, và abstraction layer cùng một lúc.

> **"Bỏ CloudFront = bỏ khiên che + bỏ cache + expose origin. Ba cái giá đó cộng lại đắt hơn nhiều so với $10–50/tháng CloudFront charge."**

---

## Phần 4: Nếu Bỏ WAF Thì Sao?

### WAF làm gì và tại sao attach vào CloudFront?

Trong sơ đồ của bạn, WAF nằm bên dưới CloudFront với label "attaches to / protects" — đây là cách thể hiện đúng. WAF không đứng trong luồng traffic như một service riêng biệt. Nó là một Web ACL (Access Control List) được AWS gắn vào CloudFront distribution.

Mỗi request đến CloudFront sẽ được WAF inspect trước khi CloudFront quyết định forward hay block.

WAF với AWS Managed Rules bảo vệ bạn khỏi:
- **SQL Injection**: `'; DROP TABLE users; --` trong query parameters
- **Cross-Site Scripting (XSS)**: `<script>document.cookie</script>` trong form inputs
- **Known bad IPs**: IP của các botnet, scanner, known attackers
- **Request flooding**: rate limiting để chống DDoS ở Layer 7

### Nếu bỏ WAF thì sao?

**Ngắn hạn:** Hệ thống vẫn chạy bình thường. WAF không ảnh hưởng đến latency hay throughput của legitimate traffic.

**Trung hạn:** Bạn trở thành target dễ tấn công hơn. SQL Injection, XSS, và credential stuffing attack không cần kỹ năng cao — có sẵn tool tự động. Không có WAF, những request đó đi thẳng vào app.

**Dài hạn:** Một SQL Injection thành công có thể dump toàn bộ database. Một XSS attack thành công có thể steal session cookies của admin. Chi phí của một data breach — về tài chính, về reputation, về thời gian xử lý — vượt xa nhiều năm bill WAF.

### Sai lầm hay gặp: đặt WAF sai chỗ

Nhiều người khi mới học vẽ WAF như một block riêng biệt trong luồng traffic, ví dụ:

```
CloudFront → WAF → IGW → ALB   ← SAI, không thể thực hiện được
```

WAF không thể "đứng giữa" hai service như vậy. Trong AWS, WAF phải được **associate** vào một resource cụ thể: CloudFront, ALB, API Gateway, hoặc AppSync. Nó là một layer của resource đó, không phải một proxy riêng.

Cách đúng trong sơ đồ của bạn — WAF gắn vào CloudFront, bảo vệ từ bên trong — là cách thể hiện chính xác về mặt kỹ thuật.

### Verdict

WAF là một trong những service rẻ nhất so với giá trị nó mang lại. AWS Managed Rules tốn khoảng $5–20/tháng. Một lần bị SQL Injection thành công tốn bạn nhiều hơn thế gấp trăm lần.

> **"Không có WAF không có nghĩa là không bị tấn công. Nó chỉ có nghĩa là bạn không biết mình đang bị tấn công."**

---

## Phần 5: Nếu Bỏ ALB Thì Sao?

### ALB làm gì trong kiến trúc này?

ALB nằm trong Public Subnet, nhận traffic từ IGW (forwarded từ CloudFront), và phân phối xuống EC2 instances trong Private Subnet. Nó đảm nhận:

**Health-check routing.** ALB liên tục ping từng EC2 instance (ví dụ `/health` endpoint mỗi 30 giây). Khi EC2 trả về lỗi 3 lần liên tiếp, ALB ngay lập tức ngừng gửi traffic đến instance đó. Không cần operator can thiệp.

**SSL termination giữa CloudFront và EC2.** Kết nối CloudFront → ALB là HTTPS. ALB có thể terminate SSL ở đây và forward HTTP sang EC2 trong private subnet (acceptable vì đây là private network), hoặc forward HTTPS tiếp (end-to-end TLS).

**Connection draining.** Khi một EC2 bị mark để terminate (bởi ASG scale-in hay deployment), ALB ngừng gửi request mới vào instance đó, nhưng vẫn chờ các request đang xử lý hoàn thành trước khi drain. Zero-downtime deployment.

**Entry point duy nhất cho Private Subnet.** EC2 không có public IP — chỉ có ALB mới có thể tiếp cận chúng. Security Group của EC2 chỉ allow inbound từ ALB Security Group.

### Nếu bỏ ALB thì sao?

Khi không có ALB, bạn phải trỏ Route 53 hoặc CloudFront thẳng vào EC2. Hai lựa chọn:

**Option A — Route 53 round-robin DNS đến nhiều EC2 IP.** EC2 phải có public IP (hoặc Elastic IP). Security Group phải mở port 443 ra public. Khi EC2-1 crash lúc 2 giờ sáng, DNS TTL vẫn còn 60–300 giây — 50% users tiếp tục gửi request vào EC2 đã chết và nhận lỗi trong toàn bộ thời gian TTL đó.

**Option B — CloudFront origin trỏ vào 1 EC2 Elastic IP.** Không có load balancing thực sự. Mọi traffic vào 1 EC2. Khi EC2 đó chết — toàn bộ hệ thống offline.

Ngoài ra: EC2 phải có public IP, tức là bạn mất đi lớp bảo vệ "EC2 không thể bị access trực tiếp từ internet".

### Verdict

ALB tốn khoảng $18–25/tháng. Nhưng nó là tầng trung gian không thể thiếu giữa internet và private subnet.

> **"ALB không chỉ là load balancer — nó là người gác cổng của Private Subnet."**

---

## Phần 6: Nếu Bỏ Auto Scaling Thì Sao?

### Auto Scaling Group làm gì?

Trong sơ đồ, 2 EC2 instances nằm trong một Auto Scaling Group (ASG) — được thể hiện bằng dashed border bao quanh chúng. ASG liên tục:

- **Monitor health:** Nếu EC2 unhealthy (ALB health check fail), ASG terminate instance đó và launch instance mới thay thế
- **Scale out:** Khi CPU > 70% (CloudWatch alarm), ASG launch thêm EC2 instances
- **Scale in:** Khi tải giảm, ASG terminate bớt instance để tiết kiệm chi phí
- **Distribute across AZs:** ASG đảm bảo instances được phân bổ đều ở cả 2 Availability Zone

### Nếu bỏ Auto Scaling thì sao?

Bạn chạy 2 EC2 cố định — luôn luôn, bất kể traffic là bao nhiêu.

**Traffic spike.** Traffic tăng gấp 5 lần: CPU hit 100%, response time từ 200ms → 8 giây, users abandon. Với ASG: 2 minutes để detect → 3 minutes để launch thêm → traffic được xử lý bình thường.

**EC2 bị crash.** Instance die lúc 2 giờ sáng. Không có ASG: ai đó phải thức dậy, vào console, launch instance mới, đợi 3–5 phút, register vào ALB. Với ASG: toàn bộ việc đó xảy ra tự động trong vòng 3–5 phút mà không cần ai thức dậy.

**Lãng phí chi phí ban đêm.** 3 giờ sáng traffic gần bằng 0 nhưng bạn vẫn trả tiền cho 2 instance. ASG có thể scale down xuống 1 instance trong giờ thấp điểm.

**CloudWatch trở nên vô dụng hơn.** Auto Scaling phụ thuộc vào CloudWatch metrics. Nếu bỏ ASG, CloudWatch vẫn có giá trị cho observability, nhưng mất đi khả năng tự động hành động dựa trên metrics.

### Kịch bản thực tế: Black Friday

Bạn có e-commerce app. Traffic ngày thường: 1,000 concurrent users. Black Friday 0:00: traffic nhảy lên 10,000.

Với ASG: CPU alarm fire sau 2 phút → ASG launch thêm 8 instances trong 4 phút → hệ thống handle được → customers mua hàng bình thường.

Không có ASG: 2 EC2 cố định bị overload ngay lập tức → response time 8 giây → conversion rate sụp đổ → bạn mất doanh thu của đêm sale quan trọng nhất năm.

### Verdict

Không có Auto Scaling = bạn là Auto Scaling. Bạn phải luôn sẵn sàng để làm việc đó thủ công — kể cả lúc 2 giờ sáng.

> **"Auto Scaling không phải để tiết kiệm tiền. Nó để bạn ngủ ngon."**

---

## Phần 7: Nếu Bỏ RDS và Dùng Database Local Trên EC2 Thì Sao?

### RDS Multi-AZ trong sơ đồ

Trong sơ đồ, RDS nằm trong Private Subnet — không có public access, chỉ EC2 trong cùng VPC mới kết nối được. "Multi A-Z" nghĩa là AWS tự động tạo 1 Primary instance ở AZ1 và 1 Standby instance ở AZ2, với synchronous replication liên tục. Khi Primary fail, AWS promote Standby thành Primary trong vòng 60 giây — không cần operator can thiệp.

### Nếu thay RDS bằng database cài trên EC2 thì sao?

Bạn tiết kiệm được khoảng $80–100/tháng (RDS `db.t3.medium` Multi-AZ vs EC2 `t3.medium` tự cài Postgres).

Nhưng đây là những gì bạn mất:

**Mất backup tự động.** RDS backup daily, giữ 7 ngày, có thể restore về bất kỳ point-in-time nào trong retention period. Với Postgres trên EC2, bạn phải tự viết backup script, tự schedule, tự test restore. 90% người quên test restore cho đến ngày thực sự cần dùng.

**Mất Multi-AZ failover.** EC2 nằm trong 1 AZ. AZ down hoặc EC2 bị terminate — database offline, app offline. Không có automatic failover. Recovery time: vài giờ, tùy lần backup cuối là khi nào.

**Mất automated patching.** RDS tự patch trong maintenance window. Postgres trên EC2 sẽ dần lạc hậu — sau 1 năm, bạn đang chạy version có CVE công khai chưa được vá.

**"Noisy neighbor" problem.** Nếu app và DB cùng trên 1 EC2: app spike CPU → DB bị starved → query chậm → app timeout → cascade failure. Không có resource isolation.

**Bạn trở thành DBA.** Replication setup, vacuum scheduling, performance tuning, backup testing — tất cả là việc của bạn từ bây giờ.

### Kịch bản thực tế

Ai đó chạy `terraform destroy` nhầm. Hoặc EC2 bị spot instance reclaim. Nếu database nằm trên EC2 đó: toàn bộ data từ lần backup cuối đến lúc đó mất. Application và database down cùng lúc. Recovery time: 2–6 giờ.

Với RDS: data vẫn còn nguyên, standby promote lên trong 60 giây, app reconnect. Total impact: dưới 2 phút.

### Verdict

$80–100/tháng tiết kiệm được từ việc dùng EC2 DB sẽ không đủ để bù cho 1 lần incident mất data.

> **"RDS không cho bạn một database. Nó cho bạn một đội DBA — mà không cần tuyển dụng ai."**

---

## Phần 8: Nếu Bỏ CloudWatch Thì Sao?

### CloudWatch trong sơ đồ

CloudWatch nằm ngoài VPC trong box "Monitoring & Audit" riêng — đây là vị trí đúng. Nó không nằm trong luồng traffic chính, nhưng nó quan sát tất cả: EC2, ALB, RDS, và thậm chí cả NAT Gateway bandwidth.

CloudWatch là thứ khiến Auto Scaling hoạt động được. ASG không tự quyết định khi nào scale — nó dựa vào CloudWatch metric alarms. Khi bạn bỏ CloudWatch, bạn gián tiếp phá vỡ Auto Scaling.

### Nếu bỏ CloudWatch thì sao?

**Bạn bay mù.** CPU đang ở 95%? Bạn không biết cho đến khi server chết. Memory leak đang chạy ngầm? Bạn không biết cho đến khi OOM killer shutdown process. Deployment vừa rồi có làm tăng error rate không? Bạn không biết cho đến khi user complain.

**Auto Scaling mất "mắt".** Không có CloudWatch metrics → ASG không thể fire scaling policy → traffic tăng mà không có thêm EC2. Bỏ CloudWatch = bỏ Auto Scaling luôn.

**MTTD tăng từ phút lên giờ.** Mean Time to Detect — thời gian từ lúc có sự cố đến lúc ai đó biết — có thể tăng từ 2 phút (CloudWatch alarm fires) lên 2 giờ (user gửi email complain). Trong 2 giờ đó, bao nhiêu user đã trải qua lỗi?

**Không có audit trail.** Sau incident: "Cái config change lúc 14:30 có gây ra outage lúc 14:45 không?" Không có logs → không bao giờ biết.

### Về chi phí

CloudWatch basic metrics (CPU, network) là **miễn phí**. Custom metrics tốn $0.30/metric/tháng. Logs tốn $0.50/GB. Với một hệ thống production, toàn bộ CloudWatch cost thường dưới $15–20/tháng.

Không có trade-off thực sự ở đây. Lý do duy nhất để bỏ CloudWatch là không biết nó tồn tại.

### Verdict

CloudWatch là service duy nhất trong list này mà mình không thể nghĩ ra lý do hợp lý để bỏ đi.

> **"Bạn không thể cải thiện thứ bạn không đo được. Và bạn không thể sửa thứ bạn không thấy."**

---

## Phần 9: Tổng Kết — Architecture Thinking Framework

### Pattern chung

Sau khi đi qua từng service, mình nhận ra một pattern lặp lại: khi bỏ một service đi, vấn đề mà service đó giải quyết không biến mất — nó chỉ chuyển sang tay người khác, thường là Ops team, và thường là lúc 2 giờ sáng.

### Framework 3 câu hỏi

Khi đánh giá bất kỳ service nào trong kiến trúc:

**1. "Nếu service này fail, điều gì xảy ra?"** — Failure domain. 1 user? 50% users? Toàn bộ hệ thống? Bao lâu?

**2. "Nếu bỏ service này, ai phải làm việc đó thay?"** — Ownership. Ops team? Manual process? Không ai — và vấn đề trở nên vô hình?

**3. "Trade-off có xứng đáng không?"** — Value vs Cost. $32/tháng NAT Gateway vs 1 lần engineer debug 4 tiếng. $100/tháng RDS vs 1 lần data loss.

### Bảng tổng kết

| Bỏ service | Hệ thống vẫn chạy? | Tác động trực tiếp | Ai gánh thay |
|---|---|---|---|
| Internet Gateway | Không | Toàn bộ hệ thống offline | Không ai — không sửa được |
| NAT Gateway | Có (inbound OK) | EC2 mất outbound internet | Ops team debug tại sao app call API fail |
| CloudFront | Có (degraded) | Mất cache, mất edge TLS, expose ALB origin | User chịu latency cao hơn, WAF phải chuyển sang ALB |
| WAF | Có (unprotected) | Attack thẳng vào app | Security team xử lý breach |
| ALB | Có (fragile) | 50% users lỗi khi 1 EC2 crash | DNS TTL + Ops team manual failover |
| Auto Scaling | Có (rigid) | Traffic spike → server quá tải | Ops team scale tay lúc 2am |
| RDS → EC2 DB | Có (risky) | EC2 die = app + data offline cùng lúc | Bạn — bạn là DBA bây giờ |
| CloudWatch | Có (blind) | Sự cố không được phát hiện sớm | Không ai — vấn đề vô hình cho đến khi quá muộn |

### Closing thought

Kiến trúc không phải là việc dùng càng nhiều service càng tốt. Cũng không phải là cắt bỏ mọi thứ để tiết kiệm tiền.

Kiến trúc là việc **hiểu chính xác mỗi service đang bảo vệ bạn khỏi điều gì** — và đưa ra quyết định có ý thức về việc bạn có cần sự bảo vệ đó không, ở giai đoạn hiện tại của sản phẩm.

Một kỹ sư junior nhìn vào sơ đồ và hỏi: **"Service này làm gì?"**

Một kỹ sư senior nhìn vào sơ đồ và hỏi: **"Nếu thứ này biến mất lúc 2 giờ sáng, điều gì xảy ra — và ai bị đánh thức?"**

Câu trả lời cho câu hỏi thứ hai mới là thứ quyết định kiến trúc của bạn.

---

## Bonus: Câu Hỏi Tự Luyện

Áp dụng framework 3 câu hỏi cho những tình huống sau:

**"Nếu bỏ Route 53 và trỏ domain thẳng vào CloudFront IP thì sao?"**
Gợi ý: CloudFront IP có thể thay đổi. Không có DNS failover. Không có latency routing cho multi-region.

**"Nếu dùng 1 NAT Gateway cho cả 2 AZ thay vì mỗi AZ 1 cái thì sao?"**
Gợi ý: AZ1 down → EC2 ở AZ2 cũng mất outbound internet vì NAT GW ở AZ1. Chi phí tiết kiệm được là $32/tháng. Trade-off có xứng đáng không?

**"Nếu bỏ Multi-AZ trên RDS thì sao?"**
Gợi ý: Single-AZ RDS rẻ hơn ~50%. Khi AZ đó có sự cố (đã xảy ra với AWS thực tế), recovery time là bao lâu?

**"Nếu thay EC2 + ASG bằng AWS Lambda thì kiến trúc thay đổi như thế nào?"**
Gợi ý: Auto Scaling trở thành implicit — Lambda tự scale. Nhưng cold start latency xuất hiện, execution time limit 15 phút, RDS connection pooling trở thành vấn đề nghiêm trọng. Trade-off nào mới xuất hiện?

---

*Không có câu trả lời đúng hay sai cho những câu hỏi này. Chỉ có trade-off — và việc hiểu chúng là thứ phân biệt "người dùng được AWS" với "người thiết kế được hệ thống trên AWS".*

*Bài viết này dựa trên kiến trúc: User → Route 53 → CloudFront (+ WAF) → IGW → ALB → EC2 × 2 (ASG) → RDS Multi-AZ, với CloudWatch monitoring toàn bộ hệ thống.*
