---
title: "SNS Notification"
date: 2024-01-01
weight: 1
chapter: false
pre: " <b> 5.5.1. </b> "
aliases:
  - "/5-workshop/5.5-operations/5.5.1-sns-notification/"
---

## Objective

Our team used Amazon SNS to deliver internal RakiBookery notifications for events such as new orders or application errors. We checked email delivery first, then connected Lambda to process messages. These resources are in Singapore (`ap-southeast-1`).

## Implementation

### Step 1 Create the `raki-noti-sns` topic

In **Amazon SNS → Topics → Create topic**, select **Standard** and name the topic `raki-noti-sns`. We selected Standard to support direct email and Lambda subscriptions. After creation, record the **Topic ARN** for subscriptions and publisher configuration. [AWS SNS email subscription documentation](https://docs.aws.amazon.com/sns/latest/dg/sns-email-notifications.html).


![Selecting Standard and naming the topic raki-noti-sns](../../../images/5-Workshop/5.5-Serverless/5.5.1-SNS-Notification/01-topic-create.png)


![The raki-noti-sns topic in the SNS console](../../../images/5-Workshop/5.5-Serverless/5.5.1-SNS-Notification/02-topic-list.png)


### Step 2 Subscribe the team email address

Open the topic and choose **Create subscription**. Set **Protocol** to `Email` and enter the recipient as the **Endpoint**. We opened the AWS Notifications email, selected **Confirm subscription**, and checked for status `Confirmed` in the console. The endpoint can then receive messages from the topic.


![Creating a subscription using the Email protocol](../../../images/5-Workshop/5.5-Serverless/5.5.1-SNS-Notification/03-email-subscribe.png)


![The email subscription shows Confirmed](../../../images/5-Workshop/5.5-Serverless/5.5.1-SNS-Notification/04-email-confirmed.png)


### Step 3 Connect Lambda to the topic

We configured an **AWS Lambda** subscription for `raki-sns-handler`. On delivery, SNS invokes the function to read and log the subject and body. [Section 5.5.2]({{< relref "/5-Workshop/5.5-Serverless/5.5.2-Lambda-Functions/" >}}) documents the receiving function.

For publishing, the team notes show a Lambda example that reads the ARN from `SNS_TOPIC_ARN`, calls `sns.publish()`, and logs the `MessageId`. Grant a publisher `sns:Publish` for the `raki-noti-sns` ARN. Keep the publisher separate from the receiving handler so it cannot republish an event to the topic that triggered it. [AWS documentation on SNS-triggered Lambda functions](https://docs.aws.amazon.com/lambda/latest/dg/with-sns.html).


![Selecting AWS Lambda and raki-sns-handler as the endpoint](../../../images/5-Workshop/5.5-Serverless/5.5.1-SNS-Notification/05-lambda-subscribe.png)


![The Lambda subscription shows Confirmed](../../../images/5-Workshop/5.5-Serverless/5.5.1-SNS-Notification/06-lambda-confirmed.png)


## Tests and results


![Publish message on the raki-noti-sns topic page](../../../images/5-Workshop/5.5-Serverless/5.5.1-SNS-Notification/07-publish-console.png)


1. **Protocol-specific payload.** From the topic, choose **Publish message**, use a protocol-specific payload, and set a value for `email`. The inbox capture shows the sample email payload.
2. **Identical payload.** We sent a test notification for order `#1001` using the same payload for every subscription. The inbox capture records receipt.
3. **Lambda publisher.** For a test order, match the invocation time, the `MessageId` in the response or CloudWatch Logs, and the received email. This is the expected publisher result. The email captures below document messages published directly from the SNS console.


![Email received with the protocol-specific test payload](../../../images/5-Workshop/5.5-Serverless/5.5.1-SNS-Notification/08-email-protocol-test.png)


![Email received for test order #1001](../../../images/5-Workshop/5.5-Serverless/5.5.1-SNS-Notification/09-email-order-test.png)


## Conclusion

The Standard topic delivered test messages to email, and the Lambda subscription was configured. This verifies **SNS → email**. Verifying **application → publisher → SNS → email** requires a publisher log and email from the same invocation.
