---
title : "Introduction"
date : 2024-01-01 
weight : 1 
chapter : false
pre : " <b> 5.1. </b> "
---

# Cloud & DevOps Problem: Deploying the System on AWS Cloud, Building Container + Serverless Infrastructure and Monitoring

To meet the requirements of flexible deployment, scalability, security, and partial automation of operational processes, the project deploys the **RakiBookery** application (an online book management/sales system) on the Amazon Web Services (AWS) platform. The architecture is organized into main flows: the user request processing flow, the serverless processing flow (Lambda + Bedrock), and the monitoring – notification flow.

### CI/CD Flow (Pipeline Flow)
The continuous integration and deployment pipeline is designed to: fully automate the entire process from when a developer pushes code updates to when the new version of the application is deployed to the production environment. Specifically, when a Developer pushes code to the GitHub repository, GitHub Actions is triggered to start the pipeline. The pipeline uses the OIDC (OpenID Connect) authentication mechanism to integrate with AWS Security Token Service (STS) and issue temporary credentials, replacing the use of static Access Key/Secret Key in order to enhance security. After successful authentication, the pipeline builds a Docker image of the application and pushes the image to Amazon Elastic Container Registry (ECR). Subsequently, Amazon ECS Fargate automatically pulls the latest image from ECR and deploys the new version of the application without any manual intervention.

- **Flow:** Developer → GitHub → GitHub Actions → OIDC Authentication → AWS STS → Amazon ECR → Amazon ECS Fargate.

### User Flow (Request Flow)

The request processing flow is built on a content distribution model combined with load balancing to ensure access performance, fault tolerance, and security. When a user sends an access request, the request is first received by Amazon CloudFront (CDN). For static frontend assets, CloudFront retrieves them directly from the Amazon S3 Static Bucket. For dynamic requests, CloudFront forwards them to the Application Load Balancer (ALB). The ALB distributes traffic to the running ECS Fargate tasks. The PHP containers connect to Amazon RDS MySQL (placed in a Private Subnet) to perform read/write data operations. At the same time, media (book images, uploaded files) are stored and retrieved from the Amazon S3 Media Bucket.

- **Flow:** User → CloudFront → (S3 Static or ALB) → ECS Fargate → RDS MySQL (Private Subnet) + S3 Media

### Serverless & AI Flow (Lambda + Bedrock)

The serverless flow is designed to handle auxiliary tasks automatically without the need to maintain a continuously running server. When a user uploads an image to the S3 Media Bucket, the S3 event automatically triggers AWS Lambda to perform image processing (resize, optimize, etc.). Lambda can also be invoked to send order notifications or important events via Amazon SNS. Additionally, the system integrates Amazon Bedrock (via Lambda or API Gateway) to implement a simple book recommendation feature based on a Knowledge Base. API Gateway serves to securely expose Lambda and Bedrock endpoints with CORS support.

- **Flow:** S3 Media Upload → Lambda (image processing / send notifications) → SNS → Email
or API Gateway → Lambda → Amazon Bedrock (book recommendations).

### Monitoring & Notification Flow (Observability)

To ensure stable system operation and early detection of incidents, the project deploys a centralized monitoring mechanism based on AWS management services. All system logs (from ECS Fargate, Lambda, Application) and metrics (CPU, Memory, 5xx, RDS, etc.) are collected and centralized in Amazon CloudWatch. The CloudWatch Dashboard provides an overview of system health. Key Alarms are configured; when a threshold is exceeded, CloudWatch triggers Amazon SNS to send alert emails to the administration team, supporting timely response.

- **Flow:** ECS / Lambda / RDS → CloudWatch (Logs + Metrics + Alarm) → Amazon SNS → Email Alert.

---

## System Architecture

### Overall Architecture

![AWS Diagram](/fcj-workshop-nhat/images/5-Workshop/5.1-Workshop-overview/aws_diagram.jpg)

The RakiBookery architecture is divided into the following main layers:

#### 1. Presentation Layer
- Amazon CloudFront
- Amazon S3 Static Website
- (Optional) Route 53

The static frontend is served through CloudFront to reduce latency and improve access speed.

#### 2. Application Layer
- Application Load Balancer (ALB)
- Amazon ECS Cluster (Fargate)
- PHP Application (Docker container)

The PHP application runs on ECS Fargate, eliminating the need to manage EC2 instances. ALB handles load balancing and health checks.

#### 3. Data Layer
- Amazon RDS MySQL (Private Subnet)
- Amazon S3 (Static + Media + Logs)
- Secrets Manager (stores DB credentials)

RDS is placed in a private subnet with encryption and backup enabled. Secrets Manager securely stores database passwords.

#### 4. Serverless & AI Layer
- AWS Lambda (image processing, notifications, simple statistics)
- Amazon SNS
- Amazon Bedrock (book recommendation demo)
- API Gateway

#### 5. Networking & Security Layer
- VPC (Public / Private Subnet)
- Internet Gateway + (NAT Gateway if needed)
- Security Groups
- IAM (Least Privilege) + Secrets Manager

#### 6. Monitoring Layer
- Amazon CloudWatch (Dashboard, Logs, Alarms)
- Amazon SNS

---

### ECS Deployment Architecture

![ECS Deploy Diagram](/fcj-workshop-nhat/images/5-Workshop/5.1-Workshop-overview/aws_diagram2.jpg)

The system uses an **ECS Cluster (Fargate)** with:
- **Task Definition** configured with the image from ECR, environment variables, and secrets from Secrets Manager.
- PHP app container connecting to RDS and S3.
- ALB Target Group + Health Check routing traffic to running tasks.

---

## AWS Components Used

| Service              | Purpose |
|----------------------|---------|
| **IAM**              | Manage identity and permissions following the Least Privilege principle. Create Users/Groups/Roles/Policies for the team and services. |
| **VPC**              | Build a virtual private network with Public/Private Subnets, Route Tables, and Security Groups. |
| **Internet Gateway** | Connect the VPC to the Internet for the Public Subnet (ALB, CloudFront origin, etc.). |
| **ECS Fargate**      | Run the PHP app container without managing EC2 instances. |
| **ECR**              | Store Docker images of the PHP application. |
| **ALB**              | HTTP/HTTPS load balancing, Health Checks, and routing to ECS tasks. |
| **RDS MySQL**        | Managed relational database (bookstoredb), placed in a Private Subnet with encryption and backup. |
| **S3**               | Store Static Website, Media (book images), and Logs. With encryption + versioning (Media). |
| **CloudFront**       | CDN for distributing static and dynamic content, reducing latency. |
| **Secrets Manager**  | Securely store DB passwords and other sensitive credentials. |
| **Lambda**           | Process S3 image uploads, send notifications, and perform simple statistics. |
| **SNS**              | Send email notifications for order events or alarms. |
| **CloudWatch**       | Collect Logs and Metrics, create Dashboards and Alarms. |
| **API Gateway**      | Expose Lambda and Bedrock endpoints. |
| **Amazon Bedrock**   | Demo book recommendation feature (Knowledge Base + Guardrails). |

---

**Actual Deployment Notes:**
- Region: **ap-southeast-1 (Singapore)**
- Team: Văn An, Minh Nhật, Kiệt, Vũ Bảo, Gia Hào
- Timeline: 4 Sprints (Foundation → Compute → Serverless/Monitoring → Finalization & Docs)
- Key tasks completed/in progress: VPC, RDS MySQL, S3 (3 buckets), IAM + Secrets Manager, ECR + Dockerfile PHP, ECS Fargate, ALB, CloudFront, Lambda, SNS, CloudWatch, Bedrock demo, API Gateway.
