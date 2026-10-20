---
title: "Week 11 Worklog"
date: 2026-10-12
weight: 11
chapter: false
pre: " <b> 1.11. </b> "
---

### Week 11 Objectives:

- Use AWS CLI to check AWS identity, Region and profile configuration.
- Review Amazon DynamoDB fundamentals and perform basic CRUD operations.
- Explore AWS Toolkit for Visual Studio Code and Amazon Q in a development workflow.
- Build a Python/Boto3 mini project that manages laptop inventory data in DynamoDB.
- Apply secure credential-handling practices and review resource cleanup after hands-on work.

### Tasks Completed During the Week:

| Day | Task | Start Date | Completion Date | References |
|:---:|---|:---:|:---:|---|
| 2 | Review Week 10 topics, check tools and prepare the AWS/Python practice environment | 12/10/2026 | 12/10/2026 | [AWS Cloud Journey](https://cloudjourney.awsstudygroup.com/vi/) |
| 3 | Learn AWS CLI, identity, Region and profile checks; explore command-line interaction with AWS | 13/10/2026 | 13/10/2026 | [AWS CLI Workshop](https://000011.awsstudygroup.com/vi/) |
| 4 | Study Amazon DynamoDB tables, items, attributes and partition keys; practice data read/write/update/delete operations | 14/10/2026 | 14/10/2026 | [Amazon DynamoDB Workshop](https://000060.awsstudygroup.com/vi/) |
| 5 | Explore AWS Toolkit for VS Code, AWS Explorer and credential methods approved for the account | 15/10/2026 | 15/10/2026 | [AWS Toolkit for VS Code](https://000087.awsstudygroup.com/vi/) |
| 6 | Complete the AWS Laptop Inventory mini project using Python, Boto3 and DynamoDB | 16/10/2026 | 16/10/2026 | [DynamoDB Workshop](https://000060.awsstudygroup.com/vi/), [Boto3 Credentials](https://docs.aws.amazon.com/boto3/latest/guide/credentials.html) |

### Achievements During Week 11:

&emsp;◉ Monday:
<br>&emsp;&emsp;○ Reviewed the main topics from Week 10 and prepared the environment for the next hands-on exercises.
<br>&emsp;&emsp;○ Checked the development tools and considered cost and resource-cleanup requirements before using AWS resources.

&emsp;◉ Tuesday:
<br>&emsp;&emsp;○ Studied AWS CLI and its role in checking and operating AWS resources from a terminal.
<br>&emsp;&emsp;○ Practiced commands for checking tool versions, profile/Region configuration and the current AWS identity.
<br>&emsp;&emsp;○ Reviewed secure credential practices, including keeping access keys out of source code and using the authentication method approved for the account.

&emsp;◉ Wednesday:

<br>&emsp;&emsp;○ Studied Amazon DynamoDB, its NoSQL data model and the roles of tables, items, attributes and partition keys.
<br>&emsp;&emsp;○ Practiced the concepts behind create, read, update and delete operations, and compared the use cases of Query and Scan.
<br>&emsp;&emsp;○ Explored how Python connects to DynamoDB through the AWS SDK for Python (Boto3).

&emsp;◉ Thursday:
<br>&emsp;&emsp;○ Explored AWS Toolkit for Visual Studio Code and AWS Explorer to inspect supported AWS resources from the development environment.
<br>&emsp;&emsp;○ Reviewed Region selection and the use of the appropriate profile/credentials when connecting VS Code to AWS.
<br>&emsp;&emsp;○ Learned how Amazon Q can help explain code and offer development suggestions, while recognizing that its output must be reviewed before use.

&emsp;◉ Friday:
<br>&emsp;&emsp;○ Completed the AWS Laptop Inventory mini project with Python, Boto3 and Amazon DynamoDB.
<br>&emsp;&emsp;○ Created the `LaptopInventory` table with a String partition key named `laptop_id` and connected to it from the Python application.
<br>&emsp;&emsp;○ Implemented functions to add a laptop, list records, find a laptop by ID, update price/stock and delete a record.
<br>&emsp;&emsp;○ Added basic input validation, a confirmation step before deletion and handling for AWS errors through `ClientError`.
<br>&emsp;&emsp;○ Reviewed resource cleanup, keeping credentials outside the source code and preparing a project structure suitable for a future GitHub upload.
