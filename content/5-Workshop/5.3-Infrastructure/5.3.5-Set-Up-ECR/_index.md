---
title : "Create ECR Repository + Dockerfile"
date : 2024-01-01 
weight : 5
chapter : false
pre : " <b> 5.3.5 </b> "
---

### Objective

Packages RakiBookery’s PHP application as a Docker image and pushes it to an Amazon ECR private repository in Singapore (`ap-southeast-1`).

### Prerequisites

Prepare the backend source, Docker for Linux containers, AWS CLI, and an identity authorized to push to the repository. The team built the image for `linux/amd64`.

### Deployment configuration

1. Use `php:8.2-apache` as the base image, install `pdo_mysql` and `curl`, enable `mod_rewrite`, and configure Apache to serve the `public` directory on port `80`.
2. Use `.dockerignore` to exclude `.env`, SQL data, and files outside the build scope. Build `rakibookery:local` and test with MySQL 8.0 on a dedicated Docker network.
3. Create the `rakibookery` private repository. The deployment report records **Mutable** tags, scan on push enabled, and `AES256` encryption. Authenticate using a temporary ECR token over stdin, tag the image `latest`, push it, and pull it again for verification.

### Results and verification

- Local tests returned HTTP `200` for the home page, `/shop`, and static assets. A missing route returned `404`. The report records a successful connection to the test MySQL container.
- `aws ecr describe-images` confirmed the `latest` and `manual-test` tags. The report records an OCI index digest starting with `sha256:2364b729` and a successful image pull.

![Deployment report excerpt showing describe-images and ECR digest verification](../../../images/5-Workshop/5.3-Infrastructure/ecr-image-report-p11.png)

### Conclusion

The deployment report records a successful build, local test, push, and pull. The image URI with its tag or digest is the input for the ECS task definition.