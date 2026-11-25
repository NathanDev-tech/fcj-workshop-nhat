---
title: "Week 15 Worklog"
date: 2026-11-09
weight: 15
chapter: false
pre: " <b> 1.15. </b> "
---

### Week 15 Objectives:
- Explore **AWS Step Functions** for serverless workflow orchestration, including `Pass`, `Task`, `Choice`, `Succeed`, and `Fail` states.
- Understand `Retry`, `Catch`, state input/output, and execution history for controlled failure handling.
- Study **Amazon Cognito** authentication, focusing on User Pools, App Clients, sign-up, user confirmation, and sign-in.
- Learn the fundamentals of **AWS AppSync** and **GraphQL**, including schemas, queries, mutations, resolvers, and DynamoDB data sources.
- Study three separate official FCJ workshops while keeping practice resources isolated from the running RakiBookery application.

### Tasks completed this week:

| Day | Task | Start Date | Completion Date | Reference Material |
|:---:|---|:---:|:---:|---|
| 2 |  | 09/11/2026 | 09/11/2026 |  |
| 3 | Review AWS Step Functions foundations, Amazon States Language, basic state types, and branching workflow design. | 10/11/2026 | 10/11/2026 | [FCJ 000047 – AWS Step Functions](https://000047.awsstudygroup.com/vi/) |
| 4 | Study Task states, state input/output, `Retry`/`Catch`, and how to inspect workflow execution history. | 11/11/2026 | 11/11/2026 | [FCJ – Service Orchestration](https://000047.awsstudygroup.com/vi/3-serviceorchestration/), [FCJ – Error Handling](https://000047.awsstudygroup.com/vi/5-errorhandling/) |
| 5 | Review Cognito User Pools and App Clients, the sign-up/confirmation/sign-in flow, and the distinction between User Pools and Identity Pools. | 12/11/2026 | 12/11/2026 | [FCJ 000081 – Amazon Cognito](https://000081.awsstudygroup.com/vi/), [Create User Pool](https://000081.awsstudygroup.com/vi/2-create-user-pool/) |
| 6 | Explore AWS AppSync, GraphQL schemas, queries, mutations, and DynamoDB resolvers for reading and writing data. | 13/11/2026 | 13/11/2026 | [FCJ 000086 – AWS AppSync](https://000086.awsstudygroup.com/), [DynamoDB Resolvers](https://000086.awsstudygroup.com/2-tutorial-dynamodb-resolvers/) |

### Week 15 Achievements:

&emsp;◉ **Tuesday:**

- Reviewed service orchestration and the role of state machines when a single request passes through several operations.
- Studied the basic Amazon States Language structure, `Pass`, `Choice`, `Task`, `Succeed`, and `Fail` states, and transition rules.
- Mapped the concepts to a hypothetical book-request validation workflow for RakiBookery, while keeping this learning exercise separate from its production Lambda functions.

&emsp;◉ **Wednesday:**

- Examined how Task states coordinate Lambda functions and how `InputPath`, `Parameters`, and `ResultPath` affect the data passed between states.
- Studied retry policies for transient failures and catch handlers for controlled error paths.
- Reviewed execution history as a way to investigate state-level failures and understand why orchestration should be separate from business logic.

&emsp;◉ **Thursday:**

- Explored how Amazon Cognito supports authentication and user administration for web and mobile applications.
- Reviewed the purpose of User Pools and App Clients, including account registration, verification, sign-in, and issued authentication tokens.
- Distinguished User Pools from Identity Pools and noted the importance of keeping passwords, client secrets, and JWTs out of repositories and report screenshots.

&emsp;◉ **Friday:**

- Compared GraphQL with REST, particularly the ability to request specific fields through a GraphQL `Query`.
- Studied GraphQL `Mutation`, schema definitions, and AppSync resolvers connecting API operations to DynamoDB.
- Reviewed the FCJ examples for creating, reading, and updating data, and identified the components required for a working GraphQL API: schema, data source, resolver, authorization, and test queries.