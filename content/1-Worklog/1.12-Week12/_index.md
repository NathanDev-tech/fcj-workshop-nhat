---
title: "Week 12 Worklog"
date: 2026-10-19
weight: 12
chapter: false
pre: " <b> 1.12. </b> "
---

### Week 12 Objectives:

- Record the completed Amazon Bedrock setup through AWS Lambda for the RakiBookery project.
- Understand the role of AWS WAF in protecting web applications through Web ACLs and rule groups.
- Learn Amazon ElastiCache for Redis fundamentals, including caching, key/value, TTL, cache hits and cache misses.
- Explore Amazon CloudFront, origins, cache behaviors and delivery of static content stored in Amazon S3.
- Design a conceptual architecture for a laptop management web app, showing the frontend, API/backend and DynamoDB separately.
- Distinguish architecture design from actual AWS deployment and consider permissions and costs before creating resources.

### Tasks Completed During the Week:

| Day | Task | Start Date | Completion Date | References |
|:---:|---|:---:|:---:|---|
| 2 |  | 19/10/2026 | 19/10/2026 |  |
| 3 | Set up Amazon Bedrock through AWS Lambda for the RakiBookery project | 20/10/2026 | 20/10/2026 | [Amazon Bedrock](https://docs.aws.amazon.com/bedrock/), [AWS Lambda](https://docs.aws.amazon.com/lambda/latest/dg/welcome.html) |
| 4 | Study AWS WAF, Web ACLs, managed rule groups and custom rules at the learning and request-control design level | 21/10/2026 | 21/10/2026 | [AWS WAF Workshop](https://000026.awsstudygroup.com/vi/) |
| 5 | Study Amazon ElastiCache for Redis, caching, key/value, TTL, cache hits/misses and its supporting role alongside DynamoDB | 22/10/2026 | 22/10/2026 | [ElastiCache Workshop](https://000061.awsstudygroup.com/vi/), [Amazon ElastiCache](https://docs.aws.amazon.com/AmazonElastiCache/latest/dg/WhatIs.html) |
| 6 | Study CloudFront and design a conceptual architecture for a laptop management web app using S3, API/backend and DynamoDB | 23/10/2026 | 23/10/2026 | [CloudFront with S3 Workshop](https://000094.awsstudygroup.com/vi/), [Amazon CloudFront](https://docs.aws.amazon.com/AmazonCloudFront/latest/DeveloperGuide/Introduction.html) |

### Achievements During Week 12:

&emsp;◉ Tuesday:
<br>&emsp;&emsp;○ Completed the setup of Amazon Bedrock through AWS Lambda for the RakiBookery project.
<br>&emsp;&emsp;○ Strengthened my understanding of how an AWS Lambda function can act as an application processing layer when integrating an AWS AI service.

&emsp;◉ Wednesday:
<br>&emsp;&emsp;○ Studied AWS WAF and the role of a Web Application Firewall in controlling requests sent to a web application.
<br>&emsp;&emsp;○ Distinguished Web ACLs, managed rule groups and custom rules, and reviewed how requests can be allowed or blocked based on configured conditions.
<br>&emsp;&emsp;○ Noted that AWS WAF must be configured for the application's delivery entry point; reading a workshop is not evidence that WAF was deployed in a live environment.

&emsp;◉ Thursday:
<br>&emsp;&emsp;○ Studied Amazon ElastiCache for Redis and the purpose of caching frequently accessed data temporarily.
<br>&emsp;&emsp;○ Reviewed key/value, TTL, cache hit and cache miss, and understood that cached data does not replace the primary data source.
<br>&emsp;&emsp;○ Related ElastiCache to DynamoDB: DynamoDB remains the primary data store, while Redis may reduce repeated reads when the workload justifies it.
<br>&emsp;&emsp;○ Noted that an ElastiCache cluster should not be created only for experimentation before permissions and budget are confirmed, because it may incur charges.

&emsp;◉ Friday:

<br>&emsp;&emsp;○ Studied Amazon CloudFront, CDN, distributions, origins, cache behaviors, TTL and invalidation.
<br>&emsp;&emsp;○ Distinguished Amazon S3, which stores static frontend assets, from CloudFront, which distributes and caches content for users.
<br>&emsp;&emsp;○ Designed a conceptual architecture for a laptop management web app, showing the flow user → CloudFront/WAF → static frontend on S3 or API/backend → DynamoDB.
<br>&emsp;&emsp;○ Separated the frontend, API/backend and database components; the backend handles business logic and reads/writes data in DynamoDB.
<br>&emsp;&emsp;○ Completed the architecture diagram and notes describing AWS service responsibilities. The diagram is a proposed design, not evidence that CloudFront, WAF, S3 or the backend was deployed.

![AWS](/fcj-workshop-nhat/images/AWS_LAPTOP.png)
