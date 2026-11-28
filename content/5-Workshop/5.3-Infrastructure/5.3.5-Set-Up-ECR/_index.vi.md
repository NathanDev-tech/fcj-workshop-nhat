---
title : "Tạo ECR Repository + Dockerfile"
date : 2024-01-01 
weight : 5
chapter : false
pre : " <b> 5.3.5 </b> "
---

## Mục tiêu

Nhóm đóng gói ứng dụng PHP của RakiBookery thành image để có thể chạy cùng một bản ứng dụng trên máy phát triển và Amazon ECS. Sau khi kiểm tra website với MySQL, nhóm đưa image lên private repository Amazon ECR ở Singapore (`ap-southeast-1`).

## Luồng triển khai

**Mã nguồn + Dockerfile → Image → Container PHP chạy thử với MySQL → ECR → Task ECS Fargate**

Docker đọc Dockerfile để tạo image gồm PHP, Apache, mã nguồn và thư viện của ứng dụng. Container là một phiên bản được tạo từ image để chạy ứng dụng. Nhóm cấp cấu hình kết nối MySQL khi khởi động container để cùng một image có thể kết nối MySQL thử nghiệm ở máy phát triển hoặc RDS for MySQL trên AWS.

## 1. Đóng gói ứng dụng với Dockerfile

Dockerfile đặt ở thư mục gốc dự án, cùng cấp với `composer.json`, `composer.lock`, `src/` và `docker/`. Khi build với tham số `.`, Docker dùng thư mục này làm build context. Dùng `.dockerignore` để loại các tệp `.env`, `.env.*`, thư mục `.git` và bản dump `bookstoredb.sql` khỏi context. Ví dụ cấu hình dưới đây áp dụng cả với tệp môi trường trong các thư mục con.

~~~text
.git
**/.env
**/.env.*
**/bookstoredb.sql
~~~

Dockerfile của nhóm dùng hai giai đoạn trên image nền `php:8.2-apache`. Các đoạn dưới đây là những chỉ thị chính trong file dự án. [Tài liệu Docker về multi-stage build](https://docs.docker.com/build/building/multi-stage/)

**Builder.** Nhóm cài extension `curl` và `pdo_mysql`, sau đó lấy Composer từ image `composer:2` để cài thư viện vào `/var/www/vendor`. Docker chép các tệp Composer trước mã nguồn để có thể dùng lại lớp cài thư viện. `--no-dev` bỏ các gói phục vụ phát triển. Giữ `composer.lock` để lần build sau cài đúng phiên bản các gói đã khóa. [Tài liệu Composer về install](https://getcomposer.org/doc/03-cli.md#install-i)

~~~dockerfile
FROM php:8.2-apache AS builder
COPY --from=composer:2 /usr/bin/composer /usr/bin/composer
WORKDIR /var/www
COPY composer.json composer.lock* ./
RUN composer install --no-dev --optimize-autoloader --no-interaction --prefer-dist
~~~

**Runtime.** Giai đoạn cuối cài các extension cần lúc chạy và bật `mod_rewrite` cho các route như `/shop`. Docker chép thư viện từ builder vào `/var/www/vendor`, mã nguồn vào `/var/www/html` và cấu hình virtual host vào Apache. Khi container khởi động, `apache2-foreground` chạy Apache trên cổng `80`.

~~~dockerfile
COPY docker/apache/000-default.conf /etc/apache2/sites-available/000-default.conf
COPY --from=builder /var/www/vendor /var/www/vendor
COPY src/ /var/www/html/
~~~

Ở phiên bản Dockerfile này, `WORKDIR` là `/var/www/html`. Thư mục Apache phục vụ được khai báo ở `DocumentRoot` trong `docker/apache/000-default.conf`. Dockerfile còn gửi yêu cầu đến `http://localhost/` bằng chương trình `curl` để kiểm tra container khi chạy local. Nếu dùng health check của container trên ECS, khai báo kiểm tra trong task definition. Health check của ALB kiểm tra target qua mạng theo cấu hình riêng. [Tài liệu AWS về container health check trên ECS](https://docs.aws.amazon.com/AmazonECS/latest/developerguide/healthcheck.html)

## 2. Build và chạy thử trên máy phát triển

Chạy các lệnh trong Bash từ thư mục gốc dự án, với Docker ở chế độ Linux containers. Build image `linux/amd64` để dùng với task Fargate **Linux/X86_64**. Với Dockerfile hiện tại, `docker image inspect` cần hiển thị `linux/amd64 /var/www/html`. Ảnh bên dưới ghi lại lần build trước với thư mục làm việc `/var/www/html/public`.

~~~bash
docker build --platform linux/amd64 -t rakibookery:local . &&
docker image inspect rakibookery:local \
  --format '{{.Os}}/{{.Architecture}} {{.Config.WorkingDir}}'
~~~

![Lệnh build và nền tảng linux/amd64 trong báo cáo thử nghiệm](../../../../images/5-Workshop/5.3-Infrastructure/5.3.5-Set-Up-ECR/01-ecr-build-result.png)

### Chuẩn bị MySQL thử nghiệm

Dùng MySQL 8.0 và bản dump thử nghiệm của dự án. Tạo `.env.mysql.local` theo mẫu bên dưới, với mật khẩu riêng cho môi trường local. Bước này dùng tên network và container mới. Nếu đã có MySQL chứa dữ liệu thử nghiệm, dùng network và thông tin đăng nhập của container đó, rồi chuyển đến bước chạy PHP.

~~~dotenv
MYSQL_DATABASE=bookstoredb
MYSQL_USER=workshop_app
MYSQL_PASSWORD=<local-app-password>
MYSQL_ROOT_PASSWORD=<local-root-password>
~~~

~~~bash
NETWORK=rakibookery-workshop
docker network create "$NETWORK"

docker run -d --name rakibookery-mysql-local \
  --network "$NETWORK" --env-file .env.mysql.local mysql:8.0
~~~

Chạy lệnh dưới đây cho đến khi truy vấn `SELECT 1` trả kết quả. TCP qua `127.0.0.1` kiểm tra MySQL đã sẵn sàng nhận kết nối trong container.

~~~bash
docker exec rakibookery-mysql-local sh -c \
  'exec mysql --host=127.0.0.1 --user="$MYSQL_USER" --password="$MYSQL_PASSWORD" \
    --database="$MYSQL_DATABASE" --execute="SELECT 1"'
~~~

Lưu bản dump thử nghiệm thành `bookstoredb.sql` ở thư mục hiện tại. Bản dump dự án đã có lệnh chọn database `bookstoredb`. Nạp dữ liệu vào MySQL vừa khởi tạo.

~~~bash
SQL_DUMP="$PWD/bookstoredb.sql"
docker exec -i rakibookery-mysql-local sh -c \
  'exec mysql --user=root --password="$MYSQL_ROOT_PASSWORD"' < "$SQL_DUMP"
~~~

### Chạy và kiểm tra container PHP

Tạo tệp `.env.php.local` theo mẫu. Nếu đã dựng MySQL theo bước trên, dùng cùng mật khẩu của `MYSQL_USER`. Nếu dùng MySQL có sẵn, chỉnh `NETWORK` trong lệnh chạy PHP và các biến `DB_*` trong tệp để khớp với network và thông tin kết nối của MySQL đó. Hai container cần cùng Docker network để PHP tìm MySQL qua tên hoặc alias trong `DB_HOST`. Cổng `8080` trên máy phát triển được ánh xạ tới cổng `80` của Apache. [Tài liệu Docker về bridge network](https://docs.docker.com/engine/network/drivers/bridge/)

~~~dotenv
DB_HOST=rakibookery-mysql-local
DB_NAME=bookstoredb
DB_USER=workshop_app
DB_PASS=<local-app-password>
~~~

~~~bash
NETWORK=rakibookery-workshop
docker run -d --name rakibookery-web-local \
  --network "$NETWORK" --env-file .env.php.local \
  --publish 127.0.0.1:8080:80 rakibookery:local
~~~

Nhóm kiểm tra trang chủ, danh sách sách ở `/shop` và log của container. Kiểm tra danh sách có dữ liệu để xác nhận PHP đọc được MySQL. Nếu kiểm thử luồng tải ảnh, kiểm tra quyền ghi của `www-data` vào `public/uploads`. Dockerfile hiện gọi `chown` trước khi tạo thư mục, vì vậy thư mục mới có thể thuộc sở hữu của `root`.

~~~bash
curl -sS -o /dev/null -w '%{http_code}\n' http://127.0.0.1:8080/
curl -sS -o /dev/null -w '%{http_code}\n' http://127.0.0.1:8080/shop
docker inspect --format '{{.State.Health.Status}}' rakibookery-web-local
docker logs --tail 50 rakibookery-web-local
~~~

Báo cáo lần thử trước ghi nhận trang chủ, `/shop` và tệp tĩnh trả HTTP `200`, URL không tồn tại trả `404` và PHP kết nối được với MySQL. Sau khi thử xong, dùng `docker stop rakibookery-web-local rakibookery-mysql-local` nếu cả hai container được tạo cho buổi thử này. Để thử lại với cùng cấu hình, dùng `docker start` cho MySQL trước, chờ truy vấn `SELECT 1` thành công rồi khởi động container PHP.

![Kết quả HTTP và kết nối MySQL trong lần chạy thử của nhóm](../../../../images/5-Workshop/5.3-Infrastructure/5.3.5-Set-Up-ECR/02-local-http-test.png)

## 3. Đưa image đã kiểm thử lên Amazon ECR

Trong **Amazon ECR → Private repositories**, chọn Region Singapore và mở kho nhận image. Báo cáo của nhóm ghi nhận kho `rakibookery` với tag mutable, quét khi push và mã hóa `AES256`. Nếu chưa có kho trong tài khoản dùng để triển khai, chọn **Create repository** và tạo private repository trước khi push. Với tài khoản đang dùng cho ECS, kiểm tra kho và địa chỉ registry ở **View push commands** trước khi đẩy bản mới.

![Cấu hình kho ECR dùng trong lần triển khai của nhóm](../../../../images/5-Workshop/5.3-Infrastructure/5.3.5-Set-Up-ECR/03-ecr-repository-config.png)

Thay `<account-id>` bằng mã tài khoản của registry trong **View push commands** và đặt `REPOSITORY` theo đúng tên kho. Ví dụ dưới đây dùng tag `workshop-v1` để gắn với bản build đã kiểm thử. Ảnh từ lần triển khai trước ghi tag `latest`. Dùng AWS CLI lấy mật khẩu đăng nhập ECR, sau đó gắn tag và đẩy image. [Tài liệu AWS về đẩy Docker image lên ECR](https://docs.aws.amazon.com/AmazonECR/latest/userguide/docker-push-ecr-image.html)

~~~bash
REGION=ap-southeast-1
REGISTRY_ID='<account-id>'
REGISTRY="$REGISTRY_ID.dkr.ecr.$REGION.amazonaws.com"
REPOSITORY=rakibookery
TAG=workshop-v1
IMAGE_URI="$REGISTRY/$REPOSITORY:$TAG"

aws ecr get-login-password --region "$REGION" |
  docker login --username AWS --password-stdin "$REGISTRY" &&
docker tag rakibookery:local "$IMAGE_URI" &&
docker push "$IMAGE_URI"
~~~

![Docker đăng nhập thành công vào registry ECR](../../../../images/5-Workshop/5.3-Infrastructure/5.3.5-Set-Up-ECR/04-ecr-registry-login.png)

![Lần push image của nhóm và digest do ECR trả về](../../../../images/5-Workshop/5.3-Infrastructure/5.3.5-Set-Up-ECR/05-ecr-push-result.png)

Dùng `describe-images` với cùng mã registry để đối chiếu tag và digest, sau đó pull image. Trong lần triển khai ghi ở trên, kết quả push của tag `latest` trả về digest bắt đầu bằng `sha256:2364b729`. Dùng URI có tag phiên bản hoặc URI dạng `<registry>/<repository>@sha256:<digest>` trong trường **Image** của [task definition ở mục 5.4.1]({{< relref "/5-Workshop/5.4-Deployment/5.4.1-ECS-Cluster-Task-Definition/" >}}). [Tài liệu AWS CLI về describe-images](https://docs.aws.amazon.com/cli/latest/reference/ecr/describe-images.html)

~~~bash
aws ecr describe-images --region "$REGION" --registry-id "$REGISTRY_ID" \
  --repository-name "$REPOSITORY" \
  --image-ids imageTag="$TAG" --query 'imageDetails[0].{Tags:imageTags,Digest:imageDigest}' \
  --output json --no-cli-pager &&
docker pull --platform linux/amd64 "$IMAGE_URI"
~~~

![Kết quả kéo lại image từ kho ECR](../../../../images/5-Workshop/5.3-Infrastructure/5.3.5-Set-Up-ECR/07-ecr-pull-result.png)

## Kết luận

Nhóm đã đóng gói ứng dụng PHP bằng Dockerfile hai giai đoạn và kiểm thử website với MySQL ở máy phát triển. Kết quả HTTP và lần push, pull ghi trong báo cáo xác nhận image có thể chạy local và được lưu trên ECR. URI của bản image đã kiểm tra là đầu vào để khai báo container `app` của ECS.
