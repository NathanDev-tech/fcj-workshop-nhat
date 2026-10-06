---

title: "Week 7 Worklog"

date: 2026-08-17

weight: 7

chapter: false

pre: " <b> 1.7. </b> "

---

### Week 7 Objectives:

- Understand AWS Lambda and the Serverless model.
- Review AWS Cloud Practitioner CLF-C02 topics with ExamPro.
- Understand Amazon SQS and message queue concepts.
- Get started with Grafana and understand its basic monitoring and visualization capabilities.
- Learn how IAM Users, IAM Groups and IAM Policies can be used to manage access to AWS resources.
- Learn how AWS Secrets Manager can be used to securely store sensitive project credentials and configuration information.
- Practice AWS services through hands-on workshops and relate the knowledge to practical AWS environments.
- Apply AWS knowledge to the first project, RakiBookery.

### Tasks completed this week:

| Day | Task | Start Date | Completion Date | Reference Material |
|:---:|---|:---:|:---:|---|
| 2 | Studied AWS Lambda and Serverless computing; configured IAM Role, created a Lambda Function, and practiced automating EC2 Start/Stop operations to support cost optimization | 14/09/2026 | 14/09/2026 | [Optimize EC2 cost with Lambda](https://000022.awsstudygroup.com/) |
| 3 | Reviewed AWS Certified Cloud Practitioner CLF-C02 with ExamPro; studied Amazon SQS, Queue, Message, Producer and Consumer concepts, and practiced sending and receiving messages | 15/09/2026 | 15/09/2026 | [ExamPro CLF-C02](https://www.exampro.co/clf-c02) |
| 4 | Studied Getting Started with Grafana Basic; practiced deploying and accessing Grafana on EC2, worked with IAM Users and Groups, attached appropriate IAM Policies, and configured AWS Secrets Manager access for the RakiBookery project | 16/09/2026 | 16/09/2026 | [Getting started with Grafana basic](https://000029.awsstudygroup.com/) |
| 5 |  | 17/09/2026 | 17/09/2026 |  |
| 6 |  | 18/09/2026 | 18/09/2026 |  |

### Week 7 Achievements:

&emsp;◉ Monday:
<br>&emsp;&emsp;○ Studied AWS Lambda and the Serverless computing model.
<br>&emsp;&emsp;○ Learned how AWS Lambda executes code without requiring direct server management.
<br>&emsp;&emsp;○ Reviewed the event-driven execution model of Lambda Functions.
<br>&emsp;&emsp;○ Created and configured an IAM Role for Lambda and learned how permissions are provided to the function.
<br>&emsp;&emsp;○ Created a Lambda Function and configured it to perform operational tasks on EC2 instances.
<br>&emsp;&emsp;○ Practiced using Lambda to automate EC2 Start and Stop operations.
<br>&emsp;&emsp;○ Checked the execution result of the Lambda Function and reviewed how the function interacts with EC2.
<br>&emsp;&emsp;○ Learned how Lambda can reduce the need for continuously running infrastructure and support EC2 cost optimization.

&emsp;◉ Tuesday:
<br>&emsp;&emsp;○ Reviewed AWS Certified Cloud Practitioner CLF-C02 topics using ExamPro.
<br>&emsp;&emsp;○ Reviewed Application Integration, Messaging, Serverless, Security, Shared Responsibility Model and AWS service use cases.
<br>&emsp;&emsp;○ Studied Amazon SQS and its role as a message queue service.
<br>&emsp;&emsp;○ Learned the concepts of Queue, Message, Producer and Consumer.
<br>&emsp;&emsp;○ Learned how a Producer sends messages to a Queue and how a Consumer retrieves and processes messages.
<br>&emsp;&emsp;○ Studied how Amazon SQS can separate application components and support asynchronous message processing.
<br>&emsp;&emsp;○ Practiced creating and working with an SQS Queue.
<br>&emsp;&emsp;○ Practiced sending messages to the Queue and receiving messages for processing.
<br>&emsp;&emsp;○ Reviewed the role of Standard Queue in scalable message processing and understood that message ordering is not absolutely guaranteed.

&emsp;◉ Wednesday:
<br>&emsp;&emsp;○ Studied the Getting Started with Grafana Basic workshop and became familiar with Grafana and its basic interface.
<br>&emsp;&emsp;○ Learned the basic purpose of Grafana in monitoring and data visualization.
<br>&emsp;&emsp;○ Practiced preparing an EC2 environment for Grafana deployment and reviewed the basic steps required to access Grafana.
<br>&emsp;&emsp;○ Practiced connecting to the EC2 instance and checked the configuration required for the Grafana environment.
<br>&emsp;&emsp;○ Worked with IAM Users and IAM Groups to organize access for team members in the RakiBookery project.
<br>&emsp;&emsp;○ Created IAM Users for project members and organized the users into an appropriate IAM Group.
<br>&emsp;&emsp;○ Attached suitable IAM Policies to the IAM Group to provide the permissions required for each project role.
<br>&emsp;&emsp;○ Reviewed the relationship between IAM Users, IAM Groups and IAM Policies when managing access to AWS resources.
<br>&emsp;&emsp;○ Configured AWS Secrets Manager for the RakiBookery project to securely store sensitive credentials and configuration information.
<br>&emsp;&emsp;○ Configured appropriate IAM permissions so authorized project users could access the required secrets without exposing sensitive values directly in application code.
<br>&emsp;&emsp;○ Learned the importance of separating credentials from source code and using centralized secret management for project security.
<br>&emsp;&emsp;○ Practiced accessing the Grafana environment and reviewed the role of port 3000 when connecting to the Grafana web interface.
<br>&emsp;&emsp;○ Built a foundation for further learning about monitoring, visualization, IAM access management and secure configuration management in AWS.
