---
title : "Tạo ECR Repository + Dockerfile"
date : 2024-01-01 
weight : 5
chapter : false
pre : " <b> 5.3.5 </b> "
---

### Mục tiêu

Đóng gói ứng dụng PHP của RakiBookery thành Docker image và đưa image lên private repository Amazon ECR tại Singapore (`ap-southeast-1`).

### Cần chuẩn bị

Cần mã nguồn backend, Docker chạy Linux container, AWS CLI và danh tính triển khai có quyền đẩy image lên repository. Nhóm dùng kiến trúc `linux/amd64` để tạo image.

Dockerfile đưa PHP, Apache và các extension cần thiết vào image. Nhờ vậy, máy phát triển và Fargate có thể chạy cùng một gói ứng dụng. Image chứa mã nguồn và các thư viện. Khi chạy, ECS lấy cấu hình cơ sở dữ liệu từ Secrets Manager.

### Cấu hình triển khai

1. Dùng image nền `php:8.2-apache`, cài `pdo_mysql` và `curl`, bật `mod_rewrite` và cấu hình Apache phục vụ thư mục `public` tại cổng `80`.
2. Thêm `.dockerignore` để loại `.env`, dữ liệu SQL và tệp không cần thiết khỏi build context. Build image `rakibookery:local` rồi kiểm tra với MySQL 8.0 trên Docker network riêng.

![Trích trang 6 của báo cáo ECR về lệnh build và kết quả image linux amd64](../../../../images/5-Workshop/5.3-Infrastructure/5.3.5-Set-Up-ECR/01-ecr-build-result.png)

![Trích trang 8 của báo cáo ECR về kết quả kiểm thử HTTP và kết nối MySQL local](../../../../images/5-Workshop/5.3-Infrastructure/5.3.5-Set-Up-ECR/02-local-http-test.png)

3. Tạo private repository `rakibookery` với tag mutability **Mutable**, scan on push bật và mã hóa `AES256` theo hồ sơ triển khai. Đăng nhập registry bằng token ECR tạm thời qua stdin. Gắn tag `latest` cho image, đẩy lên ECR (`push`), rồi kéo lại (`pull`) để kiểm tra.

![Trích trang 8 về cấu hình private repository rakibookery tại Singapore](../../../../images/5-Workshop/5.3-Infrastructure/5.3.5-Set-Up-ECR/03-ecr-repository-config.png)

### Thực hiện lại bước tạo và đẩy image

Chạy các lệnh sau tại thư mục chứa Dockerfile bằng danh tính AWS đã được duyệt và Bash, chẳng hạn Git Bash hoặc WSL. Đặt `ECR_REGISTRY` theo địa chỉ registry trong **ECR → View push commands**. Kho `rakibookery` cần được tạo trước khi đẩy image.

```bash
ECR_REGISTRY='<account-id>.dkr.ecr.ap-southeast-1.amazonaws.com'
IMAGE_URI="$ECR_REGISTRY/rakibookery:latest"

docker build --platform linux/amd64 -t rakibookery:local .
aws ecr get-login-password --region ap-southeast-1 |
  docker login --username AWS --password-stdin "$ECR_REGISTRY"
docker tag rakibookery:local "$IMAGE_URI"
docker push "$IMAGE_URI"
docker pull "$IMAGE_URI"
```

![Trích trang 10 về đăng nhập Docker vào ECR và kết quả Login Succeeded](../../../../images/5-Workshop/5.3-Infrastructure/5.3.5-Set-Up-ECR/04-ecr-registry-login.png)

![Trích trang 10 về tag image, push lên ECR và digest trả về](../../../../images/5-Workshop/5.3-Infrastructure/5.3.5-Set-Up-ECR/05-ecr-push-result.png)

Ghi lại digest hiển thị trong ECR. Tag `latest` trong cấu hình nhóm có thể trỏ sang image khác sau lần đẩy tiếp theo. Lưu digest cùng revision của task đã chạy thử để xác định lại đúng bản build. [AWS về đẩy image lên ECR](https://docs.aws.amazon.com/AmazonECR/latest/userguide/docker-push-ecr-image.html).

### Kết quả và kiểm tra

- Kiểm thử local trả HTTP `200` cho trang chủ, `/shop` và tài nguyên tĩnh. Route không tồn tại trả `404`. Báo cáo ghi nhận ứng dụng kết nối được với MySQL test.
- `aws ecr describe-images` xác nhận repository có các tag `latest` và `manual-test`. Báo cáo ghi nhận OCI index digest bắt đầu bằng `sha256:2364b729` và lần pull từ ECR thành công.

Phần xác minh trong tài liệu ghi lại lệnh `describe-images`, OCI index digest, platform manifest và attestation manifest. Nhóm dùng các giá trị này để đối chiếu image đã đẩy lên ECR.

![Trích trang 11 về describe-images, các manifest và digest trên ECR](../../../../images/5-Workshop/5.3-Infrastructure/5.3.5-Set-Up-ECR/06-ecr-digest-check.png)

Phần kiểm thử pull bắt đầu ở trang 11 và ghi kết quả tại trang 12. Hình dưới ghép hai phần liên tiếp này để thể hiện đầy đủ lệnh và kết quả được ghi trong báo cáo.

![Trích phần kiểm thử pull liên tiếp từ trang 11 và 12 của báo cáo ECR](../../../../images/5-Workshop/5.3-Infrastructure/5.3.5-Set-Up-ECR/07-ecr-pull-result.png)


### Kết luận

Theo báo cáo triển khai, nhóm đã tạo image, chạy thử tại máy phát triển, sau đó đẩy lên và kéo lại thành công từ ECR. URI kèm tag hoặc digest là đầu vào để khai báo image trong ECS task definition.