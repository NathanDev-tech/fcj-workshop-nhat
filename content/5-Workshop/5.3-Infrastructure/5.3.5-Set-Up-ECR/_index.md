---
title : "Create ECR Repository + Dockerfile"
date : 2024-01-01 
weight : 5
chapter : false
pre : " <b> 5.3.5 </b> "
---

### Objective
We package the RakiBookery PHP application as an image so we can run the same build on a development machine and Amazon ECS. After testing the website with MySQL, we push the image to a private Amazon ECR repository in Singapore (`ap-southeast-1`).

## Deployment flow

**Source code + Dockerfile → Image → Local PHP container test with MySQL → ECR → ECS Fargate task**

Docker reads the Dockerfile to build an image with PHP, Apache, our source code, and its libraries. A container is an instance of this image. We set the MySQL connection details when the container starts. This lets the same image connect to a local test database or to RDS for MySQL on AWS.

## 1. Package the application with the Dockerfile

The Dockerfile is in the project root with `composer.json`, `composer.lock`, `src/`, and `docker/`. The dot (`.`) in the build command tells Docker to use this folder as the build context. Use `.dockerignore` to leave `.env`, `.env.*`, `.git`, and the `bookstoredb.sql` dump out of the context. The example below also excludes environment files in subfolders.

~~~text
.git
**/.env
**/.env.*
**/bookstoredb.sql
~~~

Our Dockerfile uses two stages based on `php:8.2-apache`. The blocks below show the main instructions from the project file. [Docker guide to multi-stage builds](https://docs.docker.com/build/building/multi-stage/)

**Builder.** We install the `curl` and `pdo_mysql` extensions and get Composer from the `composer:2` image. Composer installs libraries into `/var/www/vendor`. Docker copies the Composer files before the application code so it can reuse the library layer. `--no-dev` skips packages for development. Keep `composer.lock` so future builds use the same package versions. [Composer guide to the install command](https://getcomposer.org/doc/03-cli.md#install-i)

~~~dockerfile
FROM php:8.2-apache AS builder
COPY --from=composer:2 /usr/bin/composer /usr/bin/composer
WORKDIR /var/www
COPY composer.json composer.lock* ./
RUN composer install --no-dev --optimize-autoloader --no-interaction --prefer-dist
~~~

**Runtime.** The final stage installs the PHP extensions that the app needs and turns on `mod_rewrite` for routes such as `/shop`. Docker copies the libraries from the builder into `/var/www/vendor`, the source code into `/var/www/html`, and the virtual host settings into Apache. When the container starts, `apache2-foreground` runs Apache on port `80`.

~~~dockerfile
COPY docker/apache/000-default.conf /etc/apache2/sites-available/000-default.conf
COPY --from=builder /var/www/vendor /var/www/vendor
COPY src/ /var/www/html/
~~~

In this Dockerfile, `WORKDIR` is `/var/www/html`. The folder that Apache serves is set by `DocumentRoot` in `docker/apache/000-default.conf`. The Dockerfile also uses the `curl` program to check `http://localhost/` when running locally. To use a container health check on ECS, add it to the task definition. The ALB checks the target over the network using its own settings. [AWS guide to ECS container health checks](https://docs.aws.amazon.com/AmazonECS/latest/developerguide/healthcheck.html)

## 2. Build and test the image locally

Run the commands in Bash from the project root with Docker set to Linux containers. Build a `linux/amd64` image for a **Linux/X86_64** Fargate task. With the current Dockerfile, `docker image inspect` should show `linux/amd64 /var/www/html`. The screenshot below shows an earlier build with `/var/www/html/public` as the working directory.

~~~bash
docker build --platform linux/amd64 -t rakibookery:local . &&
docker image inspect rakibookery:local \
  --format '{{.Os}}/{{.Architecture}} {{.Config.WorkingDir}}'
~~~

![Build command and linux/amd64 platform from our test report](../../../images/5-Workshop/5.3-Infrastructure/5.3.5-Set-Up-ECR/01-ecr-build-result.png)

### Prepare MySQL for the test

Use MySQL 8.0 and the project's test dump. Create `.env.mysql.local` using the example below, with passwords for local use. This step uses new names for the network and container. If you already have MySQL with test data, use its network and login details and go to the PHP container step.

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

Run the command below until `SELECT 1` returns a result. Using TCP on `127.0.0.1` checks that MySQL is ready to accept connections inside its container.

~~~bash
docker exec rakibookery-mysql-local sh -c \
  'exec mysql --host=127.0.0.1 --user="$MYSQL_USER" --password="$MYSQL_PASSWORD" \
    --database="$MYSQL_DATABASE" --execute="SELECT 1"'
~~~

Save the test dump as `bookstoredb.sql` in the current folder. The project dump already selects the `bookstoredb` database. Import the data into the new MySQL container.

~~~bash
SQL_DUMP="$PWD/bookstoredb.sql"
docker exec -i rakibookery-mysql-local sh -c \
  'exec mysql --user=root --password="$MYSQL_ROOT_PASSWORD"' < "$SQL_DUMP"
~~~

### Run and check the PHP container

Create `.env.php.local` using the example below. If you started MySQL in the previous step, use the same password for `MYSQL_USER`. If you already have MySQL, set `NETWORK` in the PHP command and the `DB_*` values in the file to match that database. Both containers need to use the same Docker network so PHP can find MySQL by the name or alias in `DB_HOST`. Port `8080` on the development machine maps to port `80` in Apache. [Docker guide to bridge networks](https://docs.docker.com/engine/network/drivers/bridge/)

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

We check the home page, the book list at `/shop`, and the container logs. Check that the book list contains data to confirm that PHP can read from MySQL. If you test image uploads, check that `www-data` can write to `public/uploads`. The Dockerfile runs `chown` before it creates this directory, so a new directory can be owned by `root`.

~~~bash
curl -sS -o /dev/null -w '%{http_code}\n' http://127.0.0.1:8080/
curl -sS -o /dev/null -w '%{http_code}\n' http://127.0.0.1:8080/shop
docker inspect --format '{{.State.Health.Status}}' rakibookery-web-local
docker logs --tail 50 rakibookery-web-local
~~~

In the earlier test, the home page, `/shop`, and static files returned HTTP `200`. A missing page returned `404`, and PHP connected to MySQL. When the test is complete, use `docker stop rakibookery-web-local rakibookery-mysql-local` if both containers were created for this test. To test again with the same settings, use `docker start` for MySQL first. Wait for `SELECT 1` to succeed before starting the PHP container.

![HTTP and MySQL connection results from our local test](../../../images/5-Workshop/5.3-Infrastructure/5.3.5-Set-Up-ECR/02-local-http-test.png)

## 3. Push the tested image to Amazon ECR

In **Amazon ECR → Private repositories**, select the Singapore Region and open the repository for the image. Our report shows that `rakibookery` used mutable tags, scan on push, and `AES256` encryption. If the deployment account has no repository yet, choose **Create repository** and create a private repository before pushing. For the account used by ECS, check the repository and registry address in **View push commands** before pushing a new build.

![ECR repository settings from our deployment report](../../../images/5-Workshop/5.3-Infrastructure/5.3.5-Set-Up-ECR/03-ecr-repository-config.png)

Replace `<account-id>` with the registry account ID from **View push commands** and set `REPOSITORY` to the repository name. The example below uses `workshop-v1` to identify the tested build. The earlier deployment images show the `latest` tag. Use the AWS CLI to get an ECR password, then tag and push the image. [AWS guide to pushing Docker images to ECR](https://docs.aws.amazon.com/AmazonECR/latest/userguide/docker-push-ecr-image.html)

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

![Successful Docker login to the ECR registry](../../../images/5-Workshop/5.3-Infrastructure/5.3.5-Set-Up-ECR/04-ecr-registry-login.png)

![Our image push and the digest returned by ECR](../../../images/5-Workshop/5.3-Infrastructure/5.3.5-Set-Up-ECR/05-ecr-push-result.png)

Use `describe-images` with the same registry ID to check the tag and digest, then pull the image. In the deployment shown above, the push for tag `latest` returned a digest starting with `sha256:2364b729`. Use a URI with a version tag or in the form `<registry>/<repository>@sha256:<digest>` in the **Image** field of the [task definition in section 5.4.1]({{< relref "/5-Workshop/5.4-Deployment/5.4.1-ECS-Cluster-Task-Definition/" >}}). [AWS CLI guide to describe-images](https://docs.aws.amazon.com/cli/latest/reference/ecr/describe-images.html)

~~~bash
aws ecr describe-images --region "$REGION" --registry-id "$REGISTRY_ID" \
  --repository-name "$REPOSITORY" \
  --image-ids imageTag="$TAG" --query 'imageDetails[0].{Tags:imageTags,Digest:imageDigest}' \
  --output json --no-cli-pager &&
docker pull --platform linux/amd64 "$IMAGE_URI"
~~~

![Result of pulling the image back from ECR](../../../images/5-Workshop/5.3-Infrastructure/5.3.5-Set-Up-ECR/07-ecr-pull-result.png)

## Conclusion

We packaged the PHP application with a two-stage Dockerfile and tested the website with MySQL on a development machine. The HTTP results and the push and pull in the report show that the image ran locally and was stored in ECR. We use the verified image URI to set up the ECS `app` container.
