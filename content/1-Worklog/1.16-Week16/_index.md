---
title: "Week 16 Worklog"
date: 2026-11-16
weight: 16
chapter: false
pre: " <b> 1.16. </b> "
---

### Week 16 Objectives:
- Draft and refine the **RakiBookery technical report and workshop documentation**, ensuring the content matches the project's AWS architecture and implementation sequence.
- Complete the **Amazon Bedrock Agent** setup for RakiBookery, building on the bookstore information Knowledge Base prepared in Week 14.
- Finish **Section 5.2 – Prerequisites** on Tuesday, 17 November 2026.
- Finish **Section 5.3 – Design and Build Infrastructure on AWS** on Wednesday, 18 November 2026, following all five subsections in the workshop outline.
- Standardize the documentation of prerequisites, IAM service permissions, implementation steps, and component relationships while distinguishing **writing the documentation** from **provisioning new AWS resources**.


### Tasks completed this week:

| Day | Task | Start Date | Completion Date | Reference Material |
|:---:|---|:---:|:---:|---|
| 2 | Review the RakiBookery workshop report, align the architecture-related chapters, and continue finalizing Amazon Bedrock Agent configuration. | 16/11/2026 | 16/11/2026 | [Amazon Bedrock Agents – Create and Configure](https://docs.aws.amazon.com/bedrock/latest/userguide/agents-create.html) |
| 3 | Write and complete **5.2 – Prerequisites**, including AWS preparation, the existing environment, and IAM permissions required by the services. | 17/11/2026 | 17/11/2026 | [AWS IAM Best Practices](https://docs.aws.amazon.com/IAM/latest/UserGuide/best-practices.html) |
| 4 | Complete **5.3 – Design and Build Infrastructure on AWS** covering IAM, VPC, S3, RDS, ECR repository, and Dockerfile; finalize the Bedrock Agent setup for RakiBookery. | 18/11/2026 | 18/11/2026 | [Amazon VPC](https://docs.aws.amazon.com/vpc/latest/userguide/what-is-amazon-vpc.html), [Amazon ECR](https://docs.aws.amazon.com/AmazonECR/latest/userguide/what-is-ecr.html), [Bedrock Agents](https://docs.aws.amazon.com/bedrock/latest/userguide/agents-create.html) |
| 5 |  | 19/11/2026 | 19/11/2026 |  |
| 6 |  | 20/11/2026 | 20/11/2026 |  |

### Week 16 Achievements:

&emsp;◉ **Monday:**

- Reviewed the RakiBookery workshop report and organized its sequence from project introduction through prerequisites and infrastructure design.
- Consolidated documentation of IAM, VPC, S3, RDS MySQL, Amazon ECR, and Dockerfile, connecting the infrastructure discussion with the earlier Lambda and Amazon Bedrock work.
- Continued configuring the **Amazon Bedrock Agent**, reviewing the components needed for a bookstore assistant and considering the project-specific Knowledge Base prepared in Week 14.
- Improved consistency across headings, technical descriptions, and implementation steps.

&emsp;◉ **Tuesday:**

- Wrote and completed **5.2 – Prerequisites** for the RakiBookery workshop report.
- Covered the AWS environment, selected Region, required permissions, existing resources, and settings to check before proceeding with infrastructure work.
- Emphasized **service-level IAM permissions through roles and policies**, rather than treating IAM user/group creation as a substitute for application permissions.
- Checked the prerequisites against the existing architecture to avoid instructions that unnecessarily recreate AWS resources.

&emsp;◉ **Wednesday:**

- **Completed Section 5.3 – Design and Build Infrastructure on AWS** in the report, matching the workshop outline:
  - **5.3.1 Initialize and Configure IAM:** service roles and required permissions.
  - **5.3.2 Initialize and Configure VPC:** subnet design, routing, and connectivity considerations.
  - **5.3.3 Initialize and Configure S3:** object storage for application-related files.
- Finalized the **Amazon Bedrock Agent** setup within the scope of the RakiBookery project, documenting its intended role and relationship to the bookstore data already prepared in the Knowledge Base.
- Reviewed technical vocabulary and the ordering of the workflow documentation to make it easier to follow.