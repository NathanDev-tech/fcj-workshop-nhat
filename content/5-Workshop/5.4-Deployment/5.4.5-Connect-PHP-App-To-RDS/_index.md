---
title: "Connect PHP app with RDS"
date: 2024-01-01
weight: 5
chapter: false
pre: " <b> 5.4.5. </b> "
---

## From a database probe to a website operation

Our RDS report includes a CloudWatch log showing `COUNT(*) = 20` from a standalone MySQL test task. That task connected to RDS and read data. The next step is to verify that the PHP container can use the connection during an actual website operation.

![MySQL query log from the standalone Fargate test task](../../../images/5-Workshop/5.3-Infrastructure/5.3.4-Set-Up-RDS/02-rds-fargate-query.png)

## Configuring the PHP connection

1. Verify that RDS MySQL has **Public access = No** and allows TCP `3306` from the authorized workload security group. Set `DB_HOST` to the RDS DNS endpoint and `DB_NAME` to `bookstoredb`.
2. Map the credentials in Secrets Manager to `DB_USER` and `DB_PASS`. Check the secret keys against the variable names PHP reads, then deploy a new ECS revision.
3. Open the catalog to test reads. Use test data to perform a write, reload the page, and verify the result. Record the request time and inspect PHP logs if the page shows a generic error.

## Evaluating the result

For connection timeouts, we check DNS and network access. For authentication errors, we check the account and secret. Application evidence should tie a browser operation to its corresponding log. The capture above documents the standalone network and MySQL check.

[AWS documentation](https://docs.aws.amazon.com/AmazonECS/latest/developerguide/secrets-envvar-secrets-manager.html).
