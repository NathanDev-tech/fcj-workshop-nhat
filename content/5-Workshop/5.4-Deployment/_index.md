---
title: "Deploy on AWS"
date: 2024-01-01
weight: 4
chapter: false
pre: " <b> 5.4. </b> "
---

## Objective

After we prepared the VPC, Amazon RDS, and Amazon ECR in section 5.3, our team started to deploy the RakiBookery bookstore website on AWS. On a local computer, we could open the website and check its data right away. On ECS Fargate, several things need to work before the catalog page can open. ECS must pull the image and start the container. The load balancer must forward the request, and PHP must connect to MySQL.

## Architecture overview

- **PHP website.** The browser sends a request to an Application Load Balancer (ALB). The ALB sends it through a target group to the PHP container on ECS Fargate. When CloudFront serves the whole website, it sends dynamic requests to the ALB origin.
- **Static files.** CloudFront can use Origin Access Control (OAC) to read static files from an S3 bucket. If we use our own domain, an Alias record in Route 53 points to the distribution.
- **Data.** The PHP container connects to Amazon RDS for MySQL over the private network. ECR provides the image. Secrets Manager provides the connection details when the task starts.

## Inputs from the infrastructure section

Before configuring ECS, we record the image URI and digest, task execution role, RDS secret ARN, VPC ID, and security group IDs. We use the same values when configuring the ALB and the ECS service. We also check which S3 bucket holds the files that CloudFront will serve.

## Deployment steps

We check each part of the deployment in order. First, we check if ECS can pull the image and start the container. Next, we check if we can reach the website through the ALB. Then we check static files through CloudFront and try a feature that reads from MySQL. If a step fails, this order helps us decide whether to check the container logs, the target health, or the database connection.

## Checks for each step

1. **Task startup.** A task using the new revision pulls the correct image and starts the PHP container. Check the task status and the log from the same task.
2. **Access through ALB.** The task IP is registered and passes the health check. Open a real page through the ALB DNS name.
3. **Access through CloudFront.** Open a static file and a PHP page. Try an action that needs a signed-in session. Check that the request reaches PHP with the data the app needs.
4. **Application data.** Open the catalog and save a test change. Confirm that the same change is present after loading the page again.

## Conclusion

This section brings together the main parts of the RakiBookery deployment. ECS Fargate runs the PHP containers, the ALB forwards web requests, and CloudFront serves static files and forwards dynamic requests. PHP uses the private network to connect to RDS. We check task startup, target health, website access, and database actions in this order to assess the deployment.

{{% children sort="Weight" /%}}