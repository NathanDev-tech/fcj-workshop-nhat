---
title: "Connect PHP app with RDS"
date: 2024-01-01
weight: 5
chapter: false
pre: " <b> 5.4.5. </b> "
---

## Objective

The CloudWatch log from the MySQL test task shows `COUNT(*) = 20`. This shows that the test task could connect to RDS and read the data. Our goal here is to check the connection from the PHP container when a user reads or changes data on the website.

![CloudWatch log from a Fargate MySQL test task showing a count of 20 records from RDS](../../../images/5-Workshop/5.4-Deployment/5.4.5-Connect-PHP-App-To-RDS/01-rds-fargate-query.png)

## Set up the PHP connection

**1. Check the network settings.** Check that RDS for MySQL has **Publicly accessible = No**. In the RDS security group, check the inbound rule that allows TCP port `3306` from the PHP task security group. Use the RDS DNS endpoint for `DB_HOST` and `bookstoredb` for `DB_NAME`.

![RDS source screenshot showing the endpoint, port 3306, subnet group, and Publicly accessible No](../../../images/5-Workshop/5.4-Deployment/5.4.5-Connect-PHP-App-To-RDS/02-rds-private-network.png)

**2. Set the database variables.** Use the login details in Secrets Manager for `DB_USER` and `DB_PASS`. Check that the secret keys and the variables in the PHP code match. Then deploy the new revision to the ECS service.

**3. Check the deployed revision.** Wait for the ECS service deployment to finish and the tasks to run normally. Confirm that the active task uses the revision with the new settings before testing from the browser.

The MySQL test task's **Logs** tab shows the same task ID as the CloudWatch log above. This links the query result to the database connectivity test. The steps below check database actions from the PHP website.

![ECS Logs showing the MySQL test task ID and COUNT result of 20](../../../images/5-Workshop/5.4-Deployment/5.4.5-Connect-PHP-App-To-RDS/03-ecs-mysql-test-log.png)

## Run a read test and a write test

Use the same task definition revision in the ECS service and the approved test account for both tests. For the read test, pick a sample book and compare its name and price on the catalog page with the database values. For the write test, use a permitted action with test data, such as editing a test book. Refresh the page and check the changed value. If you can access MySQL from an approved client, confirm the saved row using the test book's ID.

> **Screenshot to add.** Capture the running PHP task's ID and revision, then the catalog page with the test book's ID, name, and price. For the write test, capture the changed value after refreshing the page and the matching row in RDS when a database client is available. Include the task's application log for the same test time. Use sample data and hide account details.

## Check connection errors

If the connection times out, our team checks DNS and the network path. If MySQL rejects the login, we check the MySQL user and the secret values passed to the container. To show that the application works, save the browser action and the log for that request.

[AWS guide to using Secrets Manager with ECS](https://docs.aws.amazon.com/AmazonECS/latest/developerguide/secrets-envvar-secrets-manager.html).

## Conclusion

The query returned `COUNT(*) = 20`, which confirms that the Fargate test task connected to RDS and read data. For the PHP application, the connection depends on the RDS endpoint, the security group rules, and the login values from Secrets Manager. A read test and a saved change after page refresh are needed to confirm the database connection from the website.
