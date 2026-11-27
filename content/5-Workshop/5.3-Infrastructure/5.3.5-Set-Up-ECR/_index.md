---
title : "Create ECR Repository + Dockerfile"
date : 2024-01-01 
weight : 5
chapter : false
pre : " <b> 5.3.5 </b> "
---

### Objective

Our goal is to build a Docker image for the RakiBookery PHP application and push it to a private Amazon ECR repository in Singapore (`ap-southeast-1`).

### What we need

Prepare the backend source code, Docker for Linux containers, the AWS CLI, and an identity with permission to push to the repository. We built the image for `linux/amd64`.

The Dockerfile puts PHP, Apache, and the required extensions in the image. This gives the development machine and Fargate the same application package. The image contains the code and dependencies. At runtime, ECS supplies the database settings from Secrets Manager.

### Deployment setup


1. Use `php:8.2-apache` as the base image. Install `pdo_mysql` and `curl` and turn on `mod_rewrite`. Set Apache to serve the `public` directory on port `80`.
2. Add a `.dockerignore` file so Docker leaves out `.env`, SQL data, and files that the build does not need. Build `rakibookery:local` and test it with MySQL 8.0 on a separate Docker network.
![Excerpt from PDF page 6 showing the build command and linux amd64 image result](../../../images/5-Workshop/5.3-Infrastructure/5.3.5-Set-Up-ECR/01-ecr-build-result.png)

![Excerpt from PDF page 8 showing HTTP tests and the local MySQL connection result](../../../images/5-Workshop/5.3-Infrastructure/5.3.5-Set-Up-ECR/02-local-http-test.png)

3. Create the private repository `rakibookery`. Our report shows that we chose **Mutable** tags, turned on scanning on push, and used `AES256` encryption. Sign in to the registry with a temporary ECR token through standard input. Tag the image as `latest`, push it, and pull it back to check that the image can be downloaded from ECR.

![Excerpt from PDF page 8 showing the private rakibookery repository settings in Singapore](../../../images/5-Workshop/5.3-Infrastructure/5.3.5-Set-Up-ECR/03-ecr-repository-config.png)

### Repeat the build and push

Run the following from the directory with the Dockerfile, using the approved AWS identity and a Bash shell, such as Git Bash or WSL. Set `ECR_REGISTRY` to the registry shown in **ECR → View push commands**. The `rakibookery` repository must exist before pushing.

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

![Excerpt from PDF page 10 showing Docker login to ECR and Login Succeeded](../../../images/5-Workshop/5.3-Infrastructure/5.3.5-Set-Up-ECR/04-ecr-registry-login.png)

![Excerpt from PDF page 10 showing the image tag, push command, and returned digest](../../../images/5-Workshop/5.3-Infrastructure/5.3.5-Set-Up-ECR/05-ecr-push-result.png)

Record the digest displayed by ECR. The `latest` tag shown in our setup can point to a different image after another push. Save the digest with the task revision used in a test so we can identify that build later. [AWS ECR push guide](https://docs.aws.amazon.com/AmazonECR/latest/userguide/docker-push-ecr-image.html).

### Results and checks

- In local tests, the home page, `/shop`, and static files returned HTTP `200`. A route that did not exist returned `404`. The report shows that the application connected to the test MySQL instance.
- The command `aws ecr describe-images` showed the `latest` and `manual-test` tags in the repository. Our report shows an OCI index digest that starts with `sha256:2364b729` and records a successful image pull from ECR.

The report records the `describe-images` command, OCI index digest, platform manifest, and attestation manifest. We use these values to check which image was pushed to ECR.

![Excerpt from PDF page 11 showing describe-images, manifests, and the ECR digest](../../../images/5-Workshop/5.3-Infrastructure/5.3.5-Set-Up-ECR/06-ecr-digest-check.png)

The pull test starts on page 11 and its result continues on page 12. The image below joins these two parts to show the full command and the result recorded in the report.

![Excerpts of the pull test from PDF pages 11 and 12](../../../images/5-Workshop/5.3-Infrastructure/5.3.5-Set-Up-ECR/07-ecr-pull-result.png)

### Conclusion

Our report shows that we built the image, tested it locally, pushed it to ECR, and pulled it back. We use the image URI with its tag or digest in the ECS task definition.