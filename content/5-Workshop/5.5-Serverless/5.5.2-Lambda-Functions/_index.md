---
title: "Lambda functions"
date: 2024-01-01
weight: 2
chapter: false
pre: " <b> 5.5.2. </b> "
aliases:
  - "/5-workshop/5.5-operations/5.5.2-lambda-functions/"
---

## Objective

Our goal is to use Lambda for three tasks in RakiBookery. The functions create thumbnails, receive and log SNS messages, and read order statistics from MySQL. The project list shows three functions in Singapore using Python 3.12. They are `raki-image-processor`, `raki-sns-handler`, and `raki-stats-service`.

![RakiBookery Lambda functions using Python 3.12](../../../images/5-Workshop/5.5-Serverless/5.5.2-Lambda-Functions/01-lambda-inventory.png)

## Settings shared by the three functions

For each function, check the handler name, runtime, execution role, and log group. Choose a code package or layer that matches the runtime and architecture when a library is needed. An execution role controls the AWS APIs the function can call. S3 and SNS use a separate resource-based policy to invoke their target functions.

## Image function `raki-image-processor`

**1. Create the function and give it permissions.** Create a Lambda function with the Python 3.12 runtime. Package Pillow for the selected runtime and CPU architecture. The execution role needs permission to read objects under `images/`, write thumbnails under `thumbnails/` in the Media bucket, and write to CloudWatch Logs.

![Create raki-image-processor with Python 3.12](../../../images/5-Workshop/5.5-Serverless/5.5.2-Lambda-Functions/02-image-function.png)

![Execution role and log permissions for raki-image-processor](../../../images/5-Workshop/5.5-Serverless/5.5.2-Lambda-Functions/03-image-permissions.png)

**2. Add an S3 trigger.** Go to **Configuration → Triggers → Add trigger**. Select a Media bucket in the same Region as the function, the **PUT** event (`s3:ObjectCreated:Put`), and the `images/` prefix. The function uses Pillow to make an image no larger than `300 × 300` pixels. It keeps the original proportions and saves the result under `thumbnails/`. This output prefix is outside the trigger filter, so a thumbnail does not start the function again. [AWS guide to S3 triggers](https://docs.aws.amazon.com/lambda/latest/dg/with-s3.html)

The screenshots below show the setup with the `rakibookery-prod-media-987989461846` bucket. For the Media bucket in [section 5.3.3]({{< relref "/5-Workshop/5.3-Infrastructure/5.3.3-Set-Up-S3/" >}}), select `rakibookery-prod-media-987989461846`. Check that the bucket can invoke the function and that the execution role can read and write objects in the right bucket.

![Select the Media bucket and the PUT event for the S3 trigger](../../../images/5-Workshop/5.5-Serverless/5.5.2-Lambda-Functions/04-s3-put-trigger.png)

![The S3 trigger is connected to raki-image-processor](../../../images/5-Workshop/5.5-Serverless/5.5.2-Lambda-Functions/05-s3-trigger-attached.png)

**3. Test with a book image.** In the test shown below, we uploaded a JPG under `images/` in `rakibookery-prod-media-987989461846` and checked for the matching object under `thumbnails/`. We use a log stream in `/aws/lambda/raki-image-processor` to check when the function ran.

![Test image uploaded under images/](../../../images/5-Workshop/5.5-Serverless/5.5.2-Lambda-Functions/06-s3-source-image.png)

![CloudWatch log streams for raki-image-processor](../../../images/5-Workshop/5.5-Serverless/5.5.2-Lambda-Functions/07-image-log-streams.png)

![The small image appears under thumbnails/ in S3](../../../images/5-Workshop/5.5-Serverless/5.5.2-Lambda-Functions/08-s3-generated-thumbnail.png)

![CloudWatch log group for raki-image-processor from the Lambda source document](../../../images/5-Workshop/5.5-Serverless/5.5.2-Lambda-Functions/14-image-log-group.png)

To test the Media bucket in section 5.3.3, use **PUT** to upload a JPG with a new key under `images/`. Check the matching object in `thumbnails/` and confirm that its dimensions fit the `300 × 300` limit. Use the upload time and the object key to find the matching Lambda log.


## Notification function `raki-sns-handler`

**1. Connect the topic.** Choose **Add trigger → SNS** and select `raki-noti-sns` from section 5.5.1. The handler goes through `Records`, reads `Sns.Subject` and `Sns.Message`, and writes them to a log. SNS calls the handler when a message arrives. It also sends email through the subscription we set up.

![Select raki-noti-sns when adding the SNS trigger](../../../images/5-Workshop/5.5-Serverless/5.5.2-Lambda-Functions/09-sns-trigger-setup.png)

![The SNS trigger appears in the raki-sns-handler settings](../../../images/5-Workshop/5.5-Serverless/5.5.2-Lambda-Functions/10-sns-trigger-attached.png)

![Execution role and CloudWatch Logs permissions for raki-sns-handler](../../../images/5-Workshop/5.5-Serverless/5.5.2-Lambda-Functions/11-sns-handler-log-permissions.png)

**2. Test with a sample event.** We used **Test** in the console with JSON in the same format as an SNS event. The screenshot shows that the test ran successfully. It also shows the test message in the log and `statusCode: 200` in the response. This confirms that the handler can read the sample payload. To check a real SNS call, publish to the topic and check the handler logs. [AWS guide to the SNS event format](https://docs.aws.amazon.com/lambda/latest/dg/with-sns.html).

![Successful Lambda test with a sample SNS event and statusCode 200](../../../images/5-Workshop/5.5-Serverless/5.5.2-Lambda-Functions/12-sns-handler-test.png)


## Statistics function `raki-stats-service`

**1. Prepare the database connection.** Use Python 3.12 and the `PyMySQL` library. Configure the Lambda VPC connection so the function can reach the RDS instance in section 5.3.4. In the RDS security group, add an inbound rule on TCP port `3306` with the Lambda security group as the source. Use a MySQL account that can only read the data it needs. Get the login details from Secrets Manager.

![Execution role and log permissions for raki-stats-service](../../../images/5-Workshop/5.5-Serverless/5.5.2-Lambda-Functions/13-stats-execution-role.png)

**2. Check the statistics query.** The query in the report gets `total_orders` and `total_revenue` from the `orders` table. In a test, use data you know and compare both values with the MySQL results. The Lambda list shows that `raki-stats-service` exists. The report does not yet have a screenshot of the query results from this function. [AWS guide to connecting Lambda to RDS](https://docs.aws.amazon.com/lambda/latest/dg/services-rds.html).

Before running the statistics query, compare the table and column names in the sample code with the imported schema. The source document uses `orders`, while the database import screenshot shows an `order` table. Use the actual schema names when preparing the function, then compare its response with the same SQL query.

Before the statistics test, check that the function can reach Secrets Manager from its VPC through an endpoint or another working network path. Call the function with a test event and compare the returned order count and total with the same SQL query in the same schema. Use a test dataset with a known total so you can spot filters or missing data.


## Conclusion

Our screenshots show the source image and the small image in S3, the log stream for the image function, and the sample event test for the SNS handler. When we put together the AWS test results, we will add the log from a real SNS call and the RDS query results from the statistics function.
