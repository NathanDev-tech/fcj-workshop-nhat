---
title : "Design and Build Infrastructure on AWS"
date : 2024-01-01
weight : 3
chapter : false
pre : " <b> 5.3. </b> "
---

## Objective

In this project, our team set up the AWS infrastructure for the RakiBookery PHP application. We used a VPC for the network and Amazon RDS for MySQL for application data. We prepared S3 buckets for static files, media, and logs. We built a Docker image for the application and pushed it to ECR. We used IAM for access and Secrets Manager for login details.

## Architecture overview

The diagram shows the basic RakiBookery infrastructure. It also shows how these resources connect to ECS Fargate, the Application Load Balancer, and CloudFront in the system design.

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

- **Application traffic.** The Application Load Balancer receives requests in the public network. It sends them to the PHP container on ECS Fargate. The application connects to RDS for MySQL through the private network in the VPC.
- **Deployment.** When a task starts, ECS pulls the Docker image from ECR. It also gets the MySQL credentials from Secrets Manager based on the task definition.
- **Storage.** We use three S3 buckets for Static Web, Media, and Logs. CloudFront serves files from the Static Web bucket. The other two buckets store media and logs based on the settings in the application and the services that send logs.

## Design choices and outputs

- **Network.** We place the internet-facing ALB in public subnets. The design keeps the application tasks and RDS in private subnets. The next step needs the VPC ID, each subnet ID, and the security groups for the ALB, tasks, and database.
- **Access and data.** We give each service a role for its work. We prepare the database credentials in Secrets Manager and keep the S3 buckets private. The task definition needs the execution role and the secret ARN. CloudFront and Lambda use the matching buckets in sections 5.4 and 5.5.
- **Application package.** A container image in ECR lets the team deploy the same PHP build to Fargate. Record the image URI and digest so we can identify the exact build.
![Resource map with the four RakiBookery subnets](../../../../images/5-Workshop/5.3-Infrastructure/5.3.2-Set-Up-VPC/01-vpc-resource-map.png)

## Conclusion

This setup shows how we build a container image for PHP, connect it to the database, and manage access on AWS. To run the application on Fargate, we need the Docker image in ECR and the RDS connection details in Secrets Manager. We check the setup by starting the container, connecting to MySQL from Fargate, and opening a static file through CloudFront.
## Infrastructure resources

{{% children sort="Weight" /%}}
