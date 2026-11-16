---
title : "Connect PHP app with RDS"
date : 2024-01-01
weight : 5
chapter : false
pre : " <b> 5.4.5 </b> "
---


# Connecting a PHP Application on Amazon ECS to Amazon RDS

This workshop guide provides step-by-step instructions on configuring environment variables from AWS Secrets Manager, importing database schemas and records into a private Amazon RDS instance using a one-off ECS Fargate task, resolving MySQL case-sensitivity issues on Linux, and verifying end-to-end connectivity through Application Load Balancer (ALB) and Amazon CloudFront.

---

## Part 1: Configuring Environment Variables & Secrets Manager for ECS

The PHP application retrieves database credentials via standard environment variables (`DB_HOST`, `DB_NAME`, `DB_USER`, `DB_PASS`). We will map the keys from AWS Secrets Manager to these environment variable names inside the ECS Task Definition.

### Step 1: Create a New Revision and Update Environment Variable Keys in ECS

1. Navigate to **Amazon ECS** $\rightarrow$ **Task definitions** $\rightarrow$ select **RakiBookery-task**.
2. Click **Create new revision**.
   [img1]
3. Scroll down to **Container - 1** (Main) under the **Environment variables** section.
4. Keep the existing `ValueFrom` ARNs (pointing to AWS Secrets Manager) intact, and update the **Key** names according to the mapping table below:

| Current Key | Change To |
| :--- | :--- |
| `host` | `DB_HOST` |
| `dbname` | `DB_NAME` |
| `username` | `DB_USER` |
| `password` | `DB_PASS` |

> **Note:** Leave other secrets (`engine`, `port`, `dbInstanceIdentifier`), Image URI, and port mappings unchanged.
   [img2]
   [img3]
5. Scroll to the bottom of the page and click **Create** to generate a new revision (e.g., `RakiBookery-task:3`).
6. Verify the updated keys under the **Environment and secrets** tab in the container details.
   [img4]

### Step 2: Update the Database Name in AWS Secrets Manager

By default, the `dbname` field in a newly generated RDS secret may be set to `mysql`. We need to update it to match our application's database name (`bookstoredb`).

1. Open **AWS Secrets Manager** $\rightarrow$ select the secret **fcj/sprint1/rds/mysql**.
2. In the **Secret value** section, click **Edit**.
   [img5]
3. Switch to the **Plaintext** tab, locate `"dbname"`, and change its value from `"mysql"` to `"bookstoredb"`.
4. Click **Save**.
   [img6]

### Step 3: Update the ECS Service to Apply Changes

1. Navigate to **Amazon ECS** $\rightarrow$ **Clusters** $\rightarrow$ **RakiBookery-cluster** $\rightarrow$ **Services** tab.
2. Select **RakiBookery-task-service-01** and click **Update service**.
   [img7]
3. Under **Deployment configuration**:
   - **Task definition revision:** Select the latest revision (e.g., `3 (LATEST)`).
   - **Desired tasks:** Set to `1` (if currently `0`).
   - Check **Force new deployment** whenever you update secret values in Secrets Manager without creating a new task definition revision.
   [img8]
4. Click **Update** and wait for the new task to reach the **Running** state.
   [img9]

---

## Part 2: Importing Data into Private RDS via an ECS Fargate Task

To maintain security best practices, our RDS instance remains in a private subnet with port `3306` closed to the public Internet. Instead of exposing RDS publicly, we will package `bookstoredb.sql` into a temporary Docker image and run a **one-off ECS Fargate task** within the same VPC to execute the SQL import.

### Step 1: Prepare Local Docker Environment

1. Ensure WSL 2 and **Docker Desktop** (configured with the WSL 2 backend) are installed and running on your machine.
2. Open **PowerShell** and verify Docker status:
   ```powershell
   docker --version
   docker info
   ```

### Step 2: Create a Dockerfile and Build the SQL Import Image

1. Navigate to your project root directory and create a `db-import` folder:
   ```powershell
   cd D:\Documents\AWS_FCAJ\RakiBookery
   mkdir db-import
   ```
2. Create a `Dockerfile` inside `db-import`:
   ```powershell
   @'
   FROM mysql:8.0
   WORKDIR /import
   COPY bookstoredb.sql /import/bookstoredb.sql
   '@ | Set-Content -Encoding ascii .\db-import\Dockerfile
   ```
3. Copy `bookstoredb.sql` into the `db-import` directory and verify its contents:
   ```powershell
   Copy-Item .\bookstoredb.sql .\db-import\bookstoredb.sql
   Get-ChildItem .\db-import
   ```
   [img10]
4. Build the Docker image tagged as `rakibookery-db-import:latest`:
   ```powershell
   docker build -t rakibookery-db-import:latest .\db-import
   ```

### Step 3: Push the Import Image to Amazon ECR

1. Authenticate Docker with Amazon ECR (adjust Account ID and Region if needed):
   ```powershell
   aws ecr get-login-password --region ap-southeast-1 | docker login --username AWS --password-stdin 147956724787.dkr.ecr.ap-southeast-1.amazonaws.com
   ```
   [img11]
2. Tag the image with `db-import` so it does not overwrite your web application image:
   ```powershell
   docker tag rakibookery-db-import:latest 147956724787.dkr.ecr.ap-southeast-1.amazonaws.com/rakibookery:db-import
   docker images
   ```
   [img12]
3. Push the image to ECR:
   ```powershell
   docker push 147956724787.dkr.ecr.ap-southeast-1.amazonaws.com/rakibookery:db-import
   ```
   [img13]
4. Open **AWS Console $\rightarrow$ Amazon ECR $\rightarrow$ rakibookery** to verify that the `db-import` tag is listed.
   [img14]

### Step 4: Create a Dedicated Task Definition for DB Import

1. Go to **Amazon ECS $\rightarrow$ Task definitions $\rightarrow$ Create new task definition**.
   [img15]
2. Configure basic task settings:
   - **Task definition family:** `RakiBookery-db-import-task`
   - **Launch type:** `AWS Fargate`
   - **OS/Architecture:** `Linux/X86_64`
   - **CPU / Memory:** `0.25 vCPU` / `0.5 GB`
   - **Task role:** Leave empty (`None`)
   - **Task execution role:** `ecsTaskExecutionRole`
   [img16]
3. Configure **Container - 1**:
   - **Name:** `db-import`
   - **Image URI:** Browse and select `rakibookery:db-import` from ECR.
   [img17]
   [img18]
   [img19]
   - **Port mappings:** Click **Remove** to delete port `80` (not required for a background batch task).
   [img20]
4. Add **Environment variables** referencing Secrets Manager (`DB_HOST`, `DB_NAME`, `DB_USER`, `DB_PASS`).
   [img21]
5. Enable **Logging** (CloudWatch Logs) with log group `/ecs/RakiBookery-db-import-task`.
   [img22]
6. Under **Docker configuration**, set **Entry point** to `sh,-c` and **Command** to import `/import/bookstoredb.sql` into RDS:
   ```bash
   mysql -h "$DB_HOST" -P 3306 -u "$DB_USER" -p"$DB_PASS" "$DB_NAME" < /import/bookstoredb.sql
   ```
   *(You can subsequently create a new revision with a `SELECT` verification query to confirm the imported row counts).*
7. Click **Create** to save the Task Definition.
   [img23]

### Step 5: Run the One-off Task to Execute Import and Check Logs

1. Navigate to **Amazon ECS $\rightarrow$ Clusters $\rightarrow$ RakiBookery-cluster $\rightarrow$ Tasks tab $\rightarrow$ Run new task**.
   [img24]
2. Select **Task definition family** `RakiBookery-db-import-task` (`Latest` revision) and **Launch type** `Fargate`.
   [img25]
   [img26]
3. Under **Networking**:
   - **VPC:** `RakiBookery-vpc-01`
   - **Subnets:** Select the same 2 subnets used by your web ECS Service.
   - **Security group:** Select `RakiBookery-fargate-sg` (already permitted on port `3306` in the RDS Security Group).
   - **Public IP:** `Turned on` (allows the Fargate task to pull images from ECR and fetch secrets if no NAT Gateway/VPC Endpoints exist).
   [img27]
4. Click **Create**. The task will execute the SQL import and transition to **Stopped** upon completion.
5. Click on the completed Task ID $\rightarrow$ **Logs** tab to inspect the output and confirm that all tables (`book`, `category`, `user`, etc.) and records have been populated.
   [img28]
   [img29]
   [img30]
   [img31]

---

## Part 3: Resolving MySQL Table Name Case-Sensitivity & Updating PHP Code

When running MySQL on Linux (Amazon RDS), table names are **case-sensitive** by default (`lower_case_table_names=0`). While the SQL dump creates lowercase table names (`book`, `category`, `order`, etc.), several PHP model queries reference capitalized table names (`Book`, `Order`, etc.), causing `Table doesn't exist` errors.

### Step 1: Standardize Table Names to Lowercase in PHP Models

Inspect `src/app/models/` and update all SQL queries to use lowercase table names across these four files:
- `src/app/models/Book.php`
- `src/app/models/Order.php`
- `src/app/models/Address.php`
- `src/app/models/StatisticsModel.php`

### Step 2: Build and Push the Updated PHP Application Image

1. From the project root directory (`D:\Documents\AWS_FCAJ\RakiBookery`), build a new Docker image with a distinct tag:
   ```powershell
   docker build -t rakibookery:rds-casefix-20261030 .
   docker images rakibookery
   ```
   [img32]
2. Log in to ECR, tag the image, and push it to the repository:
   ```powershell
   aws ecr get-login-password --region ap-southeast-1 | docker login --username AWS --password-stdin 147956724787.dkr.ecr.ap-southeast-1.amazonaws.com
   docker tag rakibookery:rds-casefix-20261030 147956724787.dkr.ecr.ap-southeast-1.amazonaws.com/rakibookery:rds-casefix-20261030
   docker push 147956724787.dkr.ecr.ap-southeast-1.amazonaws.com/rakibookery:rds-casefix-20261030
   ```
   [img33]
   [img34]
   [img35]
3. Verify that the `rds-casefix-20261030` tag appears in Amazon ECR.
   [img36]

### Step 3: Deploy the Updated Image to the ECS Service

1. Go to **Amazon ECS $\rightarrow$ Task definitions $\rightarrow$ RakiBookery-task $\rightarrow$ Create new revision**.
   [img37]
2. Update the main container's **Image URI** to `147956724787.dkr.ecr.ap-southeast-1.amazonaws.com/rakibookery:rds-casefix-20261030`.
   [img38]
3. Keep all Secrets Manager environment variables (`DB_HOST`, `DB_NAME`, `DB_USER`, `DB_PASS`) and IAM roles unchanged, then click **Create** (e.g., creating `RakiBookery-task:5`).
   [img39]
   [img40]
   [img41]
   [img42]
4. Navigate to **Clusters $\rightarrow$ RakiBookery-cluster $\rightarrow$ Services $\rightarrow$ RakiBookery-task-service-01 $\rightarrow$ Update service**.
   [img43]
5. Select the `Latest` **Task definition revision** (`5`) and click **Update**. Wait for the rolling deployment to complete.
   [img44]
   [img45]

---

## Part 4: Verifying Connectivity & End-to-End Application Functionality

### Step 1: Verify Target Group Health & Test via ALB

1. Open **EC2 $\rightarrow$ Target Groups $\rightarrow$ RakiBookery-tg** and check the **Targets** tab to confirm the registered task status is **Healthy**.
2. Open **EC2 $\rightarrow$ Load Balancers $\rightarrow$ RakiBookery-alb**, copy the **DNS name**, and open `http://<ALB-DNS-name>` in your browser.
   [img46]
   [img47]

### Step 2: Verify via CloudFront & Upload Missing Static Assets (CSS/JS) if Needed

1. Open **CloudFront $\rightarrow$ Distributions $\rightarrow$ RakiBookery-cloudfront**, copy the **Distribution domain name** (`xxxxx.cloudfront.net`), and open it in a browser.
   [img48]
2. If styles or scripts fail to load (pressing `F12` $\rightarrow$ `Network` tab shows `403` errors for `.css`/`.js` files):
   [img49]
   [img50]
   [img51]
   [img52]
   - Check the CloudFront **Origins** tab to identify the S3 static asset bucket (e.g., `rakibookery-static-web-246111`).
   [img53]
   [img54]
   - Open **Amazon S3**, navigate to that bucket, and upload your local `client/` directory (containing `css/` and `js/`) into the `assets/` prefix so the structure matches `assets/client/css/...` and `assets/client/js/...`.
   [img55]
   [img56]
   [img57]
   [img58]
   [img59]
   [img60]
   [img61]
   [img62]
   [img63]
   - Hard-refresh the CloudFront URL (`Ctrl + Shift + R`) to bypass browser cache and confirm full styling.
   [img64]
   [img65]
   [img66]

### Step 3: Validate Live Database Queries on the Website

Once the website loads with full styling and database connectivity, test the following database-driven features:
1. **Product Listing:** Confirm that all 20 books load on the homepage with titles, authors, categories, and prices fetched from RDS.
   [img67]
2. **Search & Category Filtering:** Search by book title (e.g., `midnight`, `harry`) and filter by category.
   [img68]
   [img69]
3. **Price Range Filtering:** Adjust the price slider or enter `From` / `To` values and click **Search**.
   [img70]
   [img71]
4. **Sorting:** Test sorting by `Price: Low to High` and `Price: High to Low`.
   [img72]
   [img73]

> **Post-Workshop Cleanup:** Once verification is complete, quit **Docker Desktop** on your local machine (right-click the Docker tray icon $\rightarrow$ **Quit Docker Desktop**) to free up local system resources.
> [img74]
> [img75]