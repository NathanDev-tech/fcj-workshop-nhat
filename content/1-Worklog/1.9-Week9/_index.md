---
title: "Week 9 Worklog"
date: 2026-09-28
weight: 9
chapter: false
pre: " <b> 1.9. </b> "
---

### Week 9 Objectives:
- Understand EC2 Right-Sizing and the importance of selecting an EC2 instance size based on actual workload requirements.
- Learn how CloudWatch Agent can collect metrics for EC2 resource analysis.
- Understand the role of IAM Role when using CloudWatch Agent.
- Learn about EC2 Resource Optimization and AWS Compute Optimizer recommendations.
- Understand AWS Service Quotas and service limits.
- Learn how to check current quotas and understand the process of requesting a quota increase.
- Understand how IAM Policies can be used to control AWS resource usage.
- Practice restricting resource usage by Region, EC2 family, instance size and EBS volume.
- Relate Monitoring, Resource Optimization, IAM and Cost Management concepts in an AWS environment.
### Tasks completed this week:

### Tasks completed this week:

| Day | Task | Start Date | Completion Date | Reference Material |
|:---:|---|:---:|:---:|---|
| 2 |  | 28/09/2026 | 28/09/2026 |  |
| 3 |  | 29/09/2026 | 29/09/2026 |  |
| 4 | Studied EC2 Right-Sizing; created an IAM Role, attached the Role to EC2, installed and configured the CloudWatch Agent, collected metrics, and reviewed optimization recommendations from AWS Compute Optimizer | 30/09/2026 | 30/09/2026 | [EC2 Right-Sizing](https://000032.awsstudygroup.com/vi/) |
| 5 | Studied AWS Service Quotas; checked AWS service quotas, reviewed current limits, and learned the process of requesting a quota increase when additional capacity is required | 01/10/2026 | 01/10/2026 | [AWS Service Quotas](https://000063.awsstudygroup.com/vi/) |
| 6 | Studied Cost & Usage Management with IAM; created an IAM Group, User and Policy, practiced restricting permissions by Region, EC2 Family, Instance Size and EBS Volume, and tested the policy results | 02/10/2026 | 02/10/2026 | [Cost and Usage Management](https://000064.awsstudygroup.com/vi/) |

### Week 9 Achievements:

&emsp;◉ Wednesday:
<br>&emsp;&emsp;○ Studied the concept of EC2 Right-Sizing and the importance of selecting an EC2 instance size based on actual workload requirements.
<br>&emsp;&emsp;○ Became familiar with Amazon CloudWatch and learned how CloudWatch can be used to collect metrics for monitoring EC2 resources.
<br>&emsp;&emsp;○ Created an IAM Role for the CloudWatch Agent and learned how to provide the permissions required for the EC2 instance to send monitoring data.
<br>&emsp;&emsp;○ Attached the IAM Role to the EC2 instance and checked the permission configuration for the instance.
<br>&emsp;&emsp;○ Installed and configured the CloudWatch Agent on the EC2 instance to collect resource utilization information, including memory utilization data.
<br>&emsp;&emsp;○ Learned the relationship between EC2, IAM Role, CloudWatch Agent and monitoring data during the resource optimization process.

<br>&emsp;&emsp;○ Studied EC2 Resource Optimization and learned how collected metrics can support EC2 configuration analysis.
<br>&emsp;&emsp;○ Studied AWS Compute Optimizer and learned how its recommendations can support the selection of a more suitable EC2 configuration.
<br>&emsp;&emsp;○ Realized that EC2 optimization should be based on actual usage data rather than selecting an instance only from its default configuration.

&emsp;◉ Thursday:

<br>&emsp;&emsp;○ Studied AWS Service Quotas and the concept of quotas and service limits for AWS services.
<br>&emsp;&emsp;○ Explored the Service Quotas console and learned how to check quotas for different AWS services.
<br>&emsp;&emsp;○ Learned how to identify the current quota value and monitor resource limits within the AWS account.
<br>&emsp;&emsp;○ Studied the preparation steps before submitting a quota increase request.
<br>&emsp;&emsp;○ Learned the process of requesting a quota increase when resource requirements exceed the current limit.
<br>&emsp;&emsp;○ Learned that quota increase requests are reviewed by AWS and that not every request is necessarily approved in full.
<br>&emsp;&emsp;○ Gained a better understanding of the role of Service Quotas when preparing for and scaling AWS systems as resource usage grows.
<br>&emsp;&emsp;○ Related quota management concepts to deployment planning and system scalability in an AWS environment.

&emsp;◉ Friday:
<br>&emsp;&emsp;○ Studied the Cost and Usage Management workshop and the role of IAM in controlling AWS resource usage.
<br>&emsp;&emsp;○ Prepared the IAM Group, IAM User and IAM Policy according to the workshop structure.
<br>&emsp;&emsp;○ Learned how IAM Policies can be used to restrict resource usage based on specific conditions.
<br>&emsp;&emsp;○ Practiced restricting resource usage by Region.
<br>&emsp;&emsp;○ Practiced restricting EC2 resources by EC2 family to control which instance families users are allowed to use.
<br>&emsp;&emsp;○ Practiced restricting EC2 resources by instance size to control the sizes of resources that can be deployed.
<br>&emsp;&emsp;○ Practiced restricting the creation and usage of EBS volumes based on the configured IAM policies.
<br>&emsp;&emsp;○ Tested the effectiveness of IAM Policies by performing resource creation operations and observing whether the actions were allowed or denied.
<br>&emsp;&emsp;○ Gained a better understanding of how IAM can be used not only for access control but also for resource governance and cost management.
<br>&emsp;&emsp;○ Reviewed and cleaned up AWS resources after completing the workshop.