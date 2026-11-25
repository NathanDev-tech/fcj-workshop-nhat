---
title: "Deploy on AWS"
date: 2024-01-01
weight: 4
chapter: false
pre: " <b> 5.4. </b> "
---

## Objective

After preparing the VPC, Amazon RDS, and Amazon ECR in section 5.3, our team turned to deploying the RakiBookery bookstore. On a development machine, we can open the site and inspect its data directly. On ECS Fargate, loading the catalog depends on several steps: pulling the image, starting the container, routing the request, and connecting PHP to MySQL.

## Architecture overview

- **PHP website.** The browser sends requests to an Application Load Balancer (ALB), which forwards them through a target group to PHP containers on ECS Fargate. If CloudFront fronts the entire site, the ALB is an origin for dynamic requests.
- **Static files.** CloudFront can fetch static content from an S3 bucket through Origin Access Control (OAC). With a custom domain, a Route 53 alias points to the distribution.
- **Data.** PHP connects to RDS MySQL over the private network. ECR provides the image, and Secrets Manager provides the connection credentials when the task starts.

## Deployment sequence

We split deployment into checks that can be performed independently. First, verify that ECS pulls the image and starts the container. Then check the route through the ALB. Finally, check CloudFront assets and exercise features that query MySQL. This sequence helps us choose between container logs, target health, and database access when investigating an error.

{{% children sort="Weight" /%}}

## Conclusion

Section 5.3 provides evidence of an ECR image and a successful query from a standalone Fargate task to RDS. A website test requires additional application-level evidence, such as a browser capture of the catalog and the corresponding PHP task log.
