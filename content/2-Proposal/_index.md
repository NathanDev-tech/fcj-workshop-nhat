---
title: "Proposal"
date: 2026-09-09
weight: 2
chapter: false
pre: "<b>2. </b>"
---

# RakiBookery
## CI/CD Pipeline Automation on AWS
### 1. Project Overview

RakiBookery is an e-commerce platform specialized in online book management and sales services. The system integrates core functionalities including: product catalog, shopping cart, order management (Order), promotions (Promo), uploaded image resource processing, intelligent product recommendations, and order notification delivery. Given its nature of handling real-time purchase transactions and storing multimedia resources, the system requires deployment on an infrastructure with flexible scalability, high security, and ease of operation.

This proposal presents a solution for deploying the RakiBookery system on Amazon Web Services (AWS) in the Singapore Region (`ap-southeast-1`) following a Cloud-Native architecture, meeting requirements for high availability, multi-layered security, and operational automation. The goal of this solution is to build a reusable infrastructure that standardizes processes in alignment with DevOps principles in a Production environment.

- **Project Duration:** 4 Sprints / 4 weeks
- **Team Size:** 5 members (Product Owner: Văn An | Scrum Master: Trương Gia Hào | AWS Admin: Minh Nhật | DevOps & Cloud Engineers: Kiệt, Vũ Bảo)

---

### 2. Problem Statement

#### Current State
Prior to implementing this proposal, the RakiBookery project only existed as application source code (PHP Web Application & MySQL Database) running locally, facing several limitations:

- **Insufficient Security Hardening:** No enforcement of Multi-Factor Authentication (Force MFA Policy) for all IAM users, no application of the Least Privilege principle, and Database connection credentials stored in plain-text.
- **Single-tier Infrastructure, Lack of Network Segmentation:** The application and Database run together or have not been clearly separated into Private Subnets within a VPC.
- **Manual Processing Workflows:** Product image upload and processing has not been automated; the system lacks automatic notification mechanisms and AI-powered product recommendation support for users.

#### Objectives
- Standardize PHP application packaging using **Docker Images** and centrally manage them on **Amazon ECR**.
- Deploy the application as a Serverless Container with **Amazon ECS Fargate** combined with an **Application Load Balancer (ALB)**.
- Build a private **Amazon VPC** network infrastructure with separated Public/Private Subnets in the Singapore region (`ap-southeast-1`).
- Enforce **Force MFA Policy** activation, manage IAM Least Privilege, and store sensitive credentials securely on **AWS Secrets Manager**.
- Automate image upload event processing with **AWS Lambda**, send order alerts/notifications via **Amazon SNS**, and integrate AI book recommendations using **Amazon Bedrock**.
- Enhance PHP Web application security with **AWS WAF** at the CloudFront layer.

#### Solution
- Design and provision a Multi-Subnet VPC infrastructure and RDS MySQL in a Private Subnet.
- Package the PHP application with a Dockerfile and push it to Amazon ECR.
- Route user traffic through CloudFront CDN and Application Load Balancer to ECS Fargate.
- Centrally manage secrets with AWS Secrets Manager and protect authentication credentials.
- Establish a Serverless Event-driven workflow with S3 Event Trigger, Lambda, and SNS.
- Monitor the entire system with CloudWatch Dashboards, Alarms, and Log Groups.

#### Return on Investment
- **Cost Efficiency:** Using the Serverless model (ECS Fargate, Lambda) ensures payment only for actual resources consumed.
- **Security First:** Ensures data safety with Force MFA, Secrets Manager, Private Subnet database isolation, and AWS WAF to protect against PHP Web attacks.
- **Operational Excellence:** Minimizes manual operations in infrastructure management and media data processing.

---

### 3. Architecture Diagrams

#### Overall Architecture

To be added

#### Service Connection Architecture

To be added

#### Architecture Components

| Well-Architected Pillar | AWS Service | Role & Function in the System |
| --- | --- | --- |
| **Operational Excellence** | **Amazon ECR, Docker** | Write Dockerfile to package the PHP application, store and manage Container Image versions on ECR. |
| **Security** | **AWS IAM, AWS Secrets Manager, AWS WAF** | Enforce Force MFA Policy, apply IAM Least Privilege, secure DB credentials with Secrets Manager, attach AWS WAF to block PHP Web attacks. |
| **Reliability** | **Amazon VPC, Amazon RDS MySQL** | Set up VPC in Singapore (`ap-southeast-1`) with Public/Private Subnets and Internet Gateway; RDS MySQL runs in a Private Subnet with automatic Backup & Encryption enabled. |
| **Performance Efficiency** | **Amazon ECS Fargate, ALB, CloudFront** | Run the application as a Serverless Container on ECS Fargate in a Private Subnet, distribute traffic through ALB, and accelerate static/media content delivery via CloudFront CDN. |
| **Cost Optimization** | **Amazon S3, AWS Fargate** | Classify storage capacity across 3 dedicated S3 Buckets (Static Web, Media with Encryption & Versioning enabled, Logs); Fargate charges based on actual resource usage. |
| **Sustainability & Integration** | **AWS Lambda, Amazon SNS, Amazon Bedrock** | Lambda automatically processes images uploaded to S3 & sends messages; Amazon SNS sends order/error notifications; Amazon Bedrock experiments with intelligent book recommendations. |

---

### 4. Timeline & Milestones

| Sprint / Phase | Key Work Items |
| :--- | :--- |
| **SPRINT 1: Foundation & Infrastructure** | - **VPC Setup (`AWS-001`):** Create VPC, Internet Gateway, Public/Private Subnets, Route Tables, Security Groups.<br>- **RDS MySQL Provisioning (`AWS-002`):** Provision RDS MySQL in a Private Subnet, enable Backup/Encryption, import `bookstoredb.sql`.<br>- **S3 Buckets (`AWS-003`):** Create 3 S3 Buckets (Static Web, Media, Logs), configure Block Public Access & Encryption.<br>- **IAM & Secrets Manager (`IAM-001`):** Create IAM Users/Groups, configure Force MFA, store DB credentials in Secrets Manager.<br>- **ECR + Dockerfile (`AWS-004`):** Write PHP application Dockerfile, create ECR Repository, build & push image.<br>- **Order + Promo Fullstack (`BE-001`):** Update Order database schema and complete the Promotion feature. |
| **SPRINT 2: Compute & Core Services** | - **Fargate & Task Definition (`AWS-005`):** Initialize ECS Cluster (Fargate), create Task Definition connecting env/secrets from Secrets Manager.<br>- **ALB + Target Group (`AWS-006`):** Create Application Load Balancer, Target Group, and Health Check.<br>- **CloudFront & CDN (`AWS-007`):** Distribute static/media content from S3 & ALB through CloudFront Distribution.<br>- **Connect PHP App to RDS (`APP-001`):** Configure secret connection strings and test basic CRUD functionality. |
| **SPRINT 3: Serverless, Monitoring & Finalization** | - **SNS Notification (`AWS-008`):** Create SNS Topic to send order/error notifications via Email.<br>- **Lambda Functions (`AWS-009`):** Write Lambda functions to process images uploaded to S3 & send automatic notifications.<br>- **CloudWatch Monitoring (`OPS-001`):** Create CloudWatch Dashboard, configure Alarms (CPU, 5xx, RDS) and Log Groups.<br>- **Amazon Bedrock & AWS WAF (`BEDROCK-001`):** |
| **SPRINT 4: Testing & Documentation** | - **E2E Testing & Runbook (`QA-001` / `DOCS-001`):** Test the complete purchase flow, image upload, and package the operational documentation (Runbook). |

---

### 5. Estimated Budget

The system optimizes costs based on the **AWS Pay-As-You-Go** and **Serverless** model, maximizing the use of service free tiers in the Singapore Region (`ap-southeast-1`).

| AWS Service | Configuration & Role in Architecture |
| --- | --- |
| **Amazon ECS Fargate** | Runs the PHP backend container based on actual Task Definition resource requirements, optimizing idle costs. |
| **Amazon RDS MySQL** | Provision an RDS instance in a Private Subnet with appropriate storage capacity for `bookstoredb.sql`. |
| **Amazon S3 & CloudFront** | Store static files, media, and logs with 3 dedicated buckets; CloudFront CDN reduces origin bandwidth load. |
| **AWS Secrets Manager & ECR** | Securely store DB credentials (charged per secret/month) and store Docker Images on the ECR repository. |
| **AWS Lambda & Amazon SNS** | Charged per number of requests and execution time when triggering image processing/notification events. |
| **Amazon CloudWatch** | Configure log collection for Fargate/Lambda and set alerts based on cost/performance thresholds. |

---

### 6. Risk Assessment

#### Risk Matrix & Response Plan

| Identified Risk | Likelihood | Impact | Mitigation & Response Plan |
| --- | --- | --- | --- |
| **Exposure of Sensitive Information / DB Credentials** | Low | Very High | Store all credentials on Secrets Manager; enforce **Force MFA Policy** for all IAM Accounts. |
| **Database Connection Failure from Fargate** | Medium | High | Place RDS MySQL in a Private Subnet; configure Security Groups to only allow internal access from ECS Fargate Tasks. |
| **PHP Web Security Vulnerability Attack** | Medium | High | Integrate **AWS WAF** at the CloudFront layer to block common attack vectors targeting PHP applications. |
| **Errors During Image Upload Processing** | Medium | Medium | Decouple image processing logic into separate **AWS Lambda** functions running independently via S3 Event Trigger. |

---

### 7. Expected Outcomes

Upon completion of the RakiBookery project deployment, the system is expected to achieve the following outcomes:

- **Standardized Cloud-Native Infrastructure:** The entire RakiBookery Web E-commerce system operates stably and securely on AWS ECS Fargate and RDS MySQL in the Singapore region.
- **Automation & AI Integration:** Automate media resource processing workflows with Serverless Lambda, broadcast order alerts via SNS, and experiment with AI product recommendation features powered by Amazon Bedrock.
- **Security and Operational Best Practices:** Comply with Force MFA regulations, enforce IAM Least Privilege controls, encrypt S3/Secrets resources, and package a complete operational guide (Runbook) for the team.

---

### 8. References

[1]: [AWS Well-Architected Framework](https://aws.amazon.com/architecture/well-architected/)

[2]: [Amazon ECS Documentation](https://docs.aws.amazon.com/ecs/)

[3]: [AWS Secrets Manager Developer Guide](https://docs.aws.amazon.com/secretsmanager/)
