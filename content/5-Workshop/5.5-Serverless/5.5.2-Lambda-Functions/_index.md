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

Our team used Lambda to resize book images, receive and log SNS notifications, and read order totals from MySQL. The Lambda inventory shows `raki-image-processor`, `raki-sns-handler`, and `raki-stats-service` on Python 3.12 in Singapore.


![RakiBookery Lambda functions and Python 3.12 runtimes](../../../images/5-Workshop/5.5-Serverless/5.5.2-Lambda-Functions/01-lambda-inventory.png)


## Image processing with `raki-image-processor`

**1. Create the function and grant access.** Use Python 3.12 and package Pillow for the Lambda runtime and architecture. Grant the execution role read access to `images/`, write access to `thumbnails/` in the Media bucket, and access to write CloudWatch Logs.

![Creating raki-image-processor with Python 3.12](../../../images/5-Workshop/5.5-Serverless/5.5.2-Lambda-Functions/02-image-function.png)


![Execution role and logging permissions for raki-image-processor](../../../images/5-Workshop/5.5-Serverless/5.5.2-Lambda-Functions/03-image-permissions.png)

**2. Add the S3 trigger.** In **Configuration → Triggers → Add trigger**, select `rakibookery-media-246111`, the event `s3:ObjectCreated:Put`, and the prefix `images/`. Pillow creates an image within `300 × 300` pixels while preserving its aspect ratio. The function writes to `thumbnails/`. Separate prefixes keep output objects from triggering the function again. [AWS S3 trigger documentation](https://docs.aws.amazon.com/lambda/latest/dg/with-s3.html).

![Selecting the Media bucket and PUT event for the S3 trigger](../../../images/5-Workshop/5.5-Serverless/5.5.2-Lambda-Functions/04-s3-put-trigger.png)


![S3 trigger attached to raki-image-processor](../../../images/5-Workshop/5.5-Serverless/5.5.2-Lambda-Functions/05-s3-trigger-attached.png)

**3. Test with a book image.** We uploaded a JPG to `images/` and checked for the output under `thumbnails/`. The captures also show log streams in `/aws/lambda/raki-image-processor` for reviewing invocations by time.

![The test image uploaded to the images/ prefix](../../../images/5-Workshop/5.5-Serverless/5.5.2-Lambda-Functions/06-s3-source-image.png)


![CloudWatch log streams for raki-image-processor](../../../images/5-Workshop/5.5-Serverless/5.5.2-Lambda-Functions/07-image-log-streams.png)


![The generated thumbnail under the S3 thumbnails/ prefix](../../../images/5-Workshop/5.5-Serverless/5.5.2-Lambda-Functions/08-s3-generated-thumbnail.png)


## Receiving notifications with `raki-sns-handler`

**1. Connect the topic.** Choose **Add trigger → SNS** and select `raki-noti-sns` from section 5.5.1. The handler iterates over `Records`, reads `Sns.Subject` and `Sns.Message`, and writes a log. SNS invokes the handler on delivery and also delivers to the configured email subscription.

![Selecting raki-noti-sns when adding the SNS trigger](../../../images/5-Workshop/5.5-Serverless/5.5.2-Lambda-Functions/09-sns-trigger-setup.png)


![SNS trigger listed in the raki-sns-handler configuration](../../../images/5-Workshop/5.5-Serverless/5.5.2-Lambda-Functions/10-sns-trigger-attached.png)

**2. Test with a sample event.** We used **Test** in the console with JSON shaped like an SNS event. The capture shows a successful invocation, the sample notification in the log, and `statusCode: 200`. This verifies that the handler reads the sample payload. For live delivery, check the handler logs after publishing to the topic. [AWS SNS event format](https://docs.aws.amazon.com/lambda/latest/dg/with-sns.html).

![Successful Lambda test with a sample SNS event and statusCode 200](../../../images/5-Workshop/5.5-Serverless/5.5.2-Lambda-Functions/11-sns-handler-test.png)


## Reading statistics with `raki-stats-service`

**1. Prepare the database connection.** Use Python 3.12 and `PyMySQL`. Place the function on a network that can reach the RDS instance from section 5.3.4, and allow TCP `3306` from its security group on the database security group. Use a read-only MySQL account for the needed data, with credentials in Secrets Manager.

![Execution role and logging permissions for raki-stats-service](../../../images/5-Workshop/5.5-Serverless/5.5.2-Lambda-Functions/12-stats-execution-role.png)

**2. Check the statistics query.** The query in the report calculates `total_orders` and `total_revenue` from the `orders` table. Use a known dataset and compare both values with the MySQL result. The Lambda inventory confirms the function exists. The team report has no capture of a query result from this function. [AWS Lambda and RDS documentation](https://docs.aws.amazon.com/lambda/latest/dg/services-rds.html).

## Conclusion

The team evidence shows S3 input and thumbnail objects, image processing log streams, and the successful sample SNS event in Lambda. Live SNS invocation logs and a query result from the statistics function are still needed to verify those paths.
