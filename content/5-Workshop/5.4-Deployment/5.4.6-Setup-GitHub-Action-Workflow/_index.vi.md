---
title: "Cấu hình GitHub Action Workflow"
date: 2024-01-01
weight: 6
chapter: false
pre: " <b> 5.4.6 </b> "
---

### Chuẩn bị mã nguồn và cấu trúc dự án

Đảm bảo dự án đã có mã nguồn chạy được và tuân thủ cấu trúc thư mục tiêu chuẩn sau:

```text
RakiBookery/
├── .github/workflows/
│   └── deploy.yml
├── src/                            
│   ├── app/
│   ├── config/
│   ├── public/
│   └── routes/
├── docker/
├── Dockerfile 
├── composer.json
├── composer.lock
└── docker-compose.yml                
```

### Các kiểm tra bắt buộc trước khi tiếp tục:

- `Dockerfile` tồn tại và build được locally (PHP 8.2 Apache, multi-stage)
- `composer.json` đã khai báo đúng autoload PSR-4 cho `App\` và `Config\`
- Thư mục `src/public/` là document root của Apache
- File `.github/workflows/deploy.yml` đã có trong kho lưu trữ GitHub

### Lấy file Workflow GitHub Actions:

Truy cập kho mã nguồn GitHub của dự án và tải/kiểm tra file workflow:
- **Đường dẫn**: `.github/workflows/deploy.yml`
- **URL tham chiếu**: [GitHub Actions deploy.yml](https://github.com/HaoWasabi/RakiBookery/blob/main/.github/workflows/deploy.yml)

### Chi tiết các Jobs trong Pipeline

Pipeline bao gồm 6 giai đoạn (Jobs) chính:

#### Job 1: `lint` — PHP Lint & Static Analysis
| Mục | Chi tiết |
| :--- | :--- |
| **Trigger** | Mọi lệnh Push/PR có thay đổi thuộc `src/**`, `Dockerfile`, `composer.json`, `tests/**`, `e2e/**` |
| **Runner** | `ubuntu-latest` |
| **Môi trường** | PHP 8.2 với Composer v2, extensions: `curl`, `pdo_mysql`, `mbstring`, `xml`, `zip` |
| **Syntax Check** | `php -l` — quét toàn bộ file `.php` trong `src/`, lỗi cú pháp sẽ block pipeline |
| **Static Analysis** | `phpstan analyse src --level=5` (bỏ qua nếu chưa cài PHPStan) |
| **Code Style** | `phpcs --standard=PSR12 src` (bỏ qua nếu chưa cài PHPCS) |
| **Caching** | Composer cache theo hash của `composer.lock` |


#### Job 2: `test` — PHP Unit Tests (PHPUnit)
| Mục | Chi tiết |
| :--- | :--- |
| **Phụ thuộc** | Yêu cầu Job `lint` phải Pass |
| **Runner** | `ubuntu-latest` |
| **Môi trường** | PHP 8.2 với extension `pcntl`, `APP_ENV=testing` |
| **Testing** | `vendor/bin/phpunit --colors=always --testdox` (bỏ qua nếu chưa cài PHPUnit) |
| **Caching** | Composer cache theo hash của `composer.lock` |


#### Job 3: `e2e-tests` — Playwright E2E Tests
| Mục | Chi tiết |
| :--- | :--- |
| **Phụ thuộc** | Yêu cầu Job `test` phải Pass |
| **Runner** | `ubuntu-latest` |
| **Môi trường** | PHP 8.2 + Node.js 22, Playwright Chromium headless |
| **App Server** | PHP built-in server chạy tại `127.0.0.1:8080` trỏ vào `src/public/` |
| **Phạm vi** | Chạy các test kịch bản trọng yếu từ thư mục `e2e/` với `BASE_URL=http://127.0.0.1:8080` |
| **Artifact** | Báo cáo Playwright được lưu trữ 7 ngày trên GitHub Actions |


#### Job 4: `build-and-push` — Build & Push Docker Image lên ECR
| Mục | Chi tiết |
| :--- | :--- |
| **Phụ thuộc** | Yêu cầu cả ba Jobs `lint`, `test`, `e2e-tests` đều phải Pass |
| **Điều kiện** | Chỉ kích hoạt khi push trực tiếp vào nhánh `main` |
| **Xác thực AWS** | Sử dụng OIDC để Assume Role vào `secrets.AWS_ROLE_ARN` |
| **Image Tag** | Gắn đồng thời tag `latest` và `sha-<7_ký_tự_commit>` |
| **Caching** | Sử dụng GitHub Actions Cache (`type=gha`) để tối ưu thời gian build |
| **Output** | Xuất `image` (URI đầy đủ kèm SHA tag) để Job tiếp theo sử dụng |


#### Job 5: `deploy` — Triển khai lên Amazon ECS
| Mục | Chi tiết |
| :--- | :--- |
| **Phụ thuộc** | Yêu cầu Job `build-and-push` thành công |
| **Điều kiện** | Chỉ kích hoạt khi push vào nhánh `main` |
| **Environment** | `production` với `url` trỏ đến `APP_URL` (ALB endpoint) |
| **Chiến lược** | Render Task Definition mới → Rolling Update trên ECS Service, chờ service ổn định (`wait-for-service-stability: true`) |
| **Xác thực AWS** | OIDC Assume Role (tương tự Job 4) |


#### Job 6: `smoke-tests` — Kiểm tra sau Triển khai
| Mục | Chi tiết |
| :--- | :--- |
| **Phụ thuộc** | Yêu cầu Job `deploy` hoàn tất |
| **Điều kiện** | Chỉ chạy khi push vào nhánh `main` |
| **Khởi động** | Chờ 20 giây để service ổn định trước khi kiểm tra |
| **Health Check** | Kiểm tra các trang: `/`, `/books`, `/login`, `/cart`, `/about` |
| **Tiêu chí PASS** | Mã HTTP trả về < 500 và khác `000` (tức không timeout) |

---

### Bảng Tổng hợp Điều kiện Kích hoạt (Triggers)

| Sự kiện (Event) | Nhánh (Branches) | Các Jobs được thực thi |
| :--- | :--- | :--- |
| **Push** | `main` | Tất cả 6 jobs: `lint` → `test` → `e2e-tests` → `build-and-push` → `deploy` → `smoke-tests` |
| **Push** | `develop`, `feature/**` | Chỉ 3 jobs kiểm thử: `lint`, `test`, `e2e-tests` |
| **Pull Request** | Gửi vào `main` | Chỉ 3 jobs kiểm thử: `lint`, `test`, `e2e-tests` |

> **Lưu ý:** Pipeline chỉ kích hoạt khi có thay đổi tại các path: `src/**`, `docker/**`, `Dockerfile`, `composer.json`, `composer.lock`, `tests/**`, `e2e/**`, `.github/workflows/deploy.yml`.

---

### Cơ chế Bảo mật của Pipeline

| Lớp bảo mật | Mô tả chi tiết |
| :--- | :--- |
| **OIDC Federation** | GitHub Actions xác thực với AWS bằng token tạm thời (`id-token: write`), không lưu Access Key tĩnh |
| **Least Privilege** | GitHub Actions Role chỉ có quyền ECR push + ECS deploy, không cấp quyền Admin |
| **Environment Protection** | Job `deploy` sử dụng GitHub Environment `production`, hỗ trợ cấu hình approval trước khi triển khai |
| **Multi-stage Build** | Image production (stage `runtime`) chỉ chứa PHP runtime + Apache, không chứa Composer hay build tools |
| **Apache Non-root** | Container chạy bằng user `www-data` (mặc định của Apache), không dùng root |
| **Secret Management** | `AWS_ROLE_ARN` lưu trong GitHub Secrets; biến cấu hình ECS lưu trong GitHub Variables (`vars.*`) |
| **Branch Protection** | Chỉ push/merge vào nhánh `main` mới kích hoạt build, push ECR và deploy production |