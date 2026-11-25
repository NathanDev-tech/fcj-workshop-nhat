---
title : "Design and Build Infrastructure on AWS"
date : 2024-01-01
weight : 3
chapter : false
pre : " <b> 5.3. </b> "
---

## Objectives

In project, the RakiBookery team establishes the AWS foundation for its PHP application. The scope covers VPC networking, application data in RDS for MySQL, separate storage purposes in S3, and the Docker image in ECR. IAM and Secrets Manager manage access and deployment credentials.

## Architecture overview

The diagram shows the Sprint 1 infrastructure and how it connects to ECS Fargate, the Application Load Balancer, and CloudFront during service integration. The connections represent the system design.

{{< mermaid >}}
graph TD
    Client["Users"] --> CF["Amazon CloudFront"]
    CF --> S3["Amazon S3<br/>Static Web"]
    Client --> ALB
    subgraph Amazon VPC
        ALB["Application Load Balancer<br/>Public subnet"] --> APP["PHP - ECS Fargate<br/>Private subnet"]
        APP -->|MySQL TCP 3306| DB["Amazon RDS for MySQL<br/>Private subnet"]
    end
    ECR["Amazon ECR<br/>Docker image"] --> APP
    SM["AWS Secrets Manager<br/>MySQL credentials"] --> APP
{{< /mermaid >}}

- **Application traffic.** The Application Load Balancer receives requests in the public network layer and forwards them to PHP containers on ECS Fargate. The application connects to RDS for MySQL over the private network within the VPC.
- **Deployment.** When a task starts, ECS pulls the Docker image from ECR and reads MySQL credentials from Secrets Manager according to the task definition.
- **Storage.** S3 provides three buckets for Static Web, Media, and Logs. The Static Web bucket supplies content for CloudFront delivery. The other buckets hold media and logs according to the application and log delivery configuration.

## Infrastructure work items

{{% children sort="Weight" /%}}

## Conclusion

The infrastructure defines how the PHP application is packaged, connects to its database, and receives permissions on AWS. The ECR image and the RDS connection settings in Secrets Manager are direct inputs to Fargate integration. Deployment validation covers container startup, MySQL connectivity from Fargate, and static content delivery through CloudFront.