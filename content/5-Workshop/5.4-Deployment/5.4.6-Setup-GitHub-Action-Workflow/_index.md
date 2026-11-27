---
title: "Setup GitHub Action Workflow"
date: 2024-01-01
weight: 6
chapter: false
pre: " <b> 5.4.6 </b> "
---

### Prepare Source Code and Project Structure

Ensure the project has working source code and follows the standard directory structure below:

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

### Mandatory Checks Before Proceeding:

- `Dockerfile` exists and builds successfully locally (PHP 8.2 Apache, multi-stage)
- `composer.json` has correct PSR-4 autoload declarations for `App\` and `Config\`
- `src/public/` is the Apache document root
- `.github/workflows/deploy.yml` is present in the GitHub repository

### Retrieve the GitHub Actions Workflow File:

Access the project's GitHub repository and review the workflow file:
- **Path**: `.github/workflows/deploy.yml`
- **Reference URL**: [GitHub Actions deploy.yml](https://github.com/HaoWasabi/RakiBookery/blob/main/.github/workflows/deploy.yml)

### Pipeline Job Details

The pipeline consists of 6 main stages (Jobs):

#### Job 1: `lint` — PHP Lint & Static Analysis
| Item | Details |
| :--- | :--- |
| **Trigger** | Any Push/PR with changes to `src/**`, `Dockerfile`, `composer.json`, `tests/**`, `e2e/**` |
| **Runner** | `ubuntu-latest` |
| **Environment** | PHP 8.2 with Composer v2, extensions: `curl`, `pdo_mysql`, `mbstring`, `xml`, `zip` |
| **Syntax Check** | `php -l` — scans all `.php` files in `src/`; syntax errors will block the pipeline |
| **Static Analysis** | `phpstan analyse src --level=5` (skipped if PHPStan is not installed) |
| **Code Style** | `phpcs --standard=PSR12 src` (skipped if PHPCS is not installed) |
| **Caching** | Composer cache keyed by hash of `composer.lock` |


#### Job 2: `test` — PHP Unit Tests (PHPUnit)
| Item | Details |
| :--- | :--- |
| **Depends On** | Requires Job `lint` to pass |
| **Runner** | `ubuntu-latest` |
| **Environment** | PHP 8.2 with `pcntl` extension, `APP_ENV=testing` |
| **Testing** | `vendor/bin/phpunit --colors=always --testdox` (skipped if PHPUnit is not installed) |
| **Caching** | Composer cache keyed by hash of `composer.lock` |


#### Job 3: `e2e-tests` — Playwright E2E Tests
| Item | Details |
| :--- | :--- |
| **Depends On** | Requires Job `test` to pass |
| **Runner** | `ubuntu-latest` |
| **Environment** | PHP 8.2 + Node.js 22, Playwright Chromium headless |
| **App Server** | PHP built-in server running at `127.0.0.1:8080` pointing to `src/public/` |
| **Scope** | Runs critical path tests from the `e2e/` directory with `BASE_URL=http://127.0.0.1:8080` |
| **Artifact** | Playwright report retained for 7 days on GitHub Actions |


#### Job 4: `build-and-push` — Build & Push Docker Image to ECR
| Item | Details |
| :--- | :--- |
| **Depends On** | Requires all three Jobs `lint`, `test`, and `e2e-tests` to pass |
| **Condition** | Only triggered on a direct push to the `main` branch |
| **AWS Auth** | Uses OIDC to assume role via `secrets.AWS_ROLE_ARN` |
| **Image Tags** | Tags both `latest` and `sha-<7_char_commit>` simultaneously |
| **Caching** | Uses GitHub Actions Cache (`type=gha`) to speed up builds |
| **Output** | Exports `image` (full URI with SHA tag) for the next Job to consume |


#### Job 5: `deploy` — Deploy to Amazon ECS
| Item | Details |
| :--- | :--- |
| **Depends On** | Requires Job `build-and-push` to succeed |
| **Condition** | Only triggered on a push to the `main` branch |
| **Environment** | `production` with `url` pointing to `APP_URL` (ALB endpoint) |
| **Strategy** | Render new Task Definition → Rolling Update on ECS Service, waits for stability (`wait-for-service-stability: true`) |
| **AWS Auth** | OIDC Assume Role (same as Job 4) |


#### Job 6: `smoke-tests` — Post-Deploy Smoke Tests
| Item | Details |
| :--- | :--- |
| **Depends On** | Requires Job `deploy` to complete |
| **Condition** | Only runs on a push to the `main` branch |
| **Warm-up** | Waits 20 seconds for the service to stabilize before checking |
| **Health Check** | Checks the following pages: `/`, `/books`, `/login`, `/cart`, `/about` |
| **Pass Criteria** | HTTP status code < 500 and not `000` (i.e., no timeout) |

---

### Trigger Summary

| Event | Branches | Jobs Executed |
| :--- | :--- | :--- |
| **Push** | `main` | All 6 jobs: `lint` → `test` → `e2e-tests` → `build-and-push` → `deploy` → `smoke-tests` |
| **Push** | `develop`, `feature/**` | Test jobs only: `lint`, `test`, `e2e-tests` |
| **Pull Request** | targeting `main` | Test jobs only: `lint`, `test`, `e2e-tests` |

> **Note:** The pipeline only triggers when changes are made to: `src/**`, `docker/**`, `Dockerfile`, `composer.json`, `composer.lock`, `tests/**`, `e2e/**`, `.github/workflows/deploy.yml`.

---

### Pipeline Security Mechanisms

| Security Layer | Details |
| :--- | :--- |
| **OIDC Federation** | GitHub Actions authenticates with AWS using short-lived tokens (`id-token: write`); no static Access Keys are stored |
| **Least Privilege** | The GitHub Actions Role only has permissions for ECR push + ECS deploy; no Admin access granted |
| **Environment Protection** | The `deploy` Job uses the GitHub Environment `production`, supporting approval configuration before deployment |
| **Multi-stage Build** | The production image (stage `runtime`) contains only PHP runtime + Apache; Composer and build tools are excluded |
| **Apache Non-root** | Container runs as `www-data` (the default Apache user), not root |
| **Secret Management** | `AWS_ROLE_ARN` is stored in GitHub Secrets; ECS configuration variables are stored in GitHub Variables (`vars.*`) |
| **Branch Protection** | Only pushes/merges to the `main` branch trigger the build, ECR push, and production deployment |
