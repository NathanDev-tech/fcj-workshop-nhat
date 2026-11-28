---
title : "Testing, Operations, and Continuous Deployment"
date : 2024-01-01
weight : 6
chapter : false
pre : " <b> 5.6. </b> "
---

### Objectives

In this section, the RakiBookery project is operated and tested through an end-to-end workflow on AWS infrastructure to ensure the system runs stably, can scale automatically, is fully monitored, and supports the main user workflows.

The main objectives include:

* Provisioning and operating the ECS Service with Rolling Update.
* Configuring Auto Scaling for the Backend Service based on CPU utilization.
* Configuring Amazon CloudFront to distribute the Frontend and route API requests to the Backend through the Application Load Balancer.
* Setting up CloudWatch Dashboards, Metrics, and Alarms to monitor system status.
* Collecting and managing Application Logs, ALB Access Logs, and VPC Flow Logs.
* Configuring AWS Budgets, Cost Anomaly Detection, and cost alerts.
* Performing End-to-End Testing to verify the entire workflow from Frontend, CloudFront, ALB, ECS, and RDS to S3.
* Testing error scenarios and Responsive behavior on Mobile and Desktop devices.
* Cleaning up unused AWS resources to minimize unnecessary costs.

### Overview

The RakiBookery system operation process consists of the following main stages:

1. Provision the ECS Service using Fargate and Rolling Update.
2. Configure Auto Scaling for the Backend with a CPU threshold of 70%.
3. Configure CloudFront to distribute the Frontend and route API requests through the ALB.
4. Set up CloudWatch Dashboards, Logs, and Alarms for system monitoring.
5. Configure VPC Flow Logs, Cost Monitoring, and AWS Budget Alerts.
6. Perform End-to-End Testing for key flows such as registration, login, POI browsing, audio playback, tour booking, and payment.
7. Test error scenarios and Responsive behavior across multiple devices.
8. Clean up all AWS resources created during the workshop.


### Deployment Conclusion

After completing the configuration, monitoring, and testing processes, the RakiBookery system has been operated following a production-like model on AWS.

The main results achieved include:

* The ECS Service operates stably on Fargate with Rolling Update.
* The Backend can automatically scale from `2` to `6` tasks based on CPU utilization.
* CloudFront distributes the Frontend from S3 and routes API requests to the Backend through the ALB.
* The Application Load Balancer distributes traffic to the ECS Backend Service.
* The CloudWatch Dashboard provides monitoring of Metrics, Logs, and system status.
* CloudWatch Alarms help detect HTTP 5xx errors and operational anomalies at an early stage.
* VPC Flow Logs and Application Logs support troubleshooting and root cause analysis.
* AWS Budgets and Cost Anomaly Detection help control AWS usage costs.
* End-to-End Testing confirms that key functions such as registration, login, POI browsing, audio playback, tour booking, and sandbox payment operate correctly.
* The system is additionally tested against error scenarios and Responsive behavior across multiple devices.
* After completing the practice, unused AWS resources can be cleaned up to prevent unnecessary costs.

If the environment is no longer required, cleanup can be performed in an appropriate order, including unused ECS Services, Task Definitions, ALB, Target Groups, CloudFront Distribution, CloudWatch Alarms/Dashboards, Log Groups, and ECR Repositories.

### Infrastructure resources

{{% children sort="Weight" /%}}
