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

Our goal is to use Amazon SNS for internal RakiBookery notifications. For example, a message can report a new order or an application error. We checked email delivery first, then connected Lambda to process messages as they arrive. We used the Singapore Region (`ap-southeast-1`) for this work.

## Prepare the topic, publisher, and subscribers

Use a test mailbox that the team can confirm. The topic is the common destination for messages. The publisher sends messages to the topic. The email address and the Lambda function subscribe to receive them. This lets us add another subscriber later without changing the publisher.

## Implementation

### Step 1. Create the `raki-noti-sns` topic

In **Amazon SNS → Topics → Create topic**, choose **Standard** and enter the name `raki-noti-sns`. We chose Standard because it supports direct email delivery and Lambda subscriptions. After you create the topic, copy the **Topic ARN**. Use it to create subscriptions or set up the function that sends messages. [AWS guide to SNS email subscriptions](https://docs.aws.amazon.com/sns/latest/dg/sns-email-notifications.html).

![Select Standard and name the topic raki-noti-sns](../../../images/5-Workshop/5.5-Serverless/5.5.1-SNS-Notification/01-topic-create.png)

![The raki-noti-sns topic in the SNS topic list](../../../images/5-Workshop/5.5-Serverless/5.5.1-SNS-Notification/02-topic-list.png)

### Step 2. Subscribe the team email address

Open the topic and choose **Create subscription**. Set **Protocol** to `Email` and enter the email address in **Endpoint**. We opened the email from AWS Notifications and chose **Confirm subscription**. We then checked for `Confirmed` in the console. The email address can now receive messages from the topic.

![Create a subscription with the Email protocol](../../../images/5-Workshop/5.5-Serverless/5.5.1-SNS-Notification/03-email-subscribe.png)

![AWS Notifications email asking the team to confirm the raki-noti-sns subscription](../../../images/5-Workshop/5.5-Serverless/5.5.1-SNS-Notification/04-email-confirmation-message.png)

![The email subscription status is Confirmed](../../../images/5-Workshop/5.5-Serverless/5.5.1-SNS-Notification/05-email-confirmed.png)

### Step 3. Connect Lambda to the topic

We created an **AWS Lambda** subscription and selected `raki-sns-handler` as the destination. When a new message arrives, SNS calls Lambda asynchronously, so it does not wait for the function to finish. The handler reads the subject and message and writes them to CloudWatch Logs. See the function setup in [section 5.5.2]({{< relref "/5-Workshop/5.5-Serverless/5.5.2-Lambda-Functions/" >}}).

The team notes also show a Lambda publisher. It reads the topic ARN from `SNS_TOPIC_ARN`, calls `sns.publish()`, and logs the `MessageId`. The publisher needs `sns:Publish` on the ARN of `raki-noti-sns`. Keep the publisher and the receiving handler separate so the handler does not send a message back to the topic that called it. [AWS guide to receiving SNS events in Lambda](https://docs.aws.amazon.com/lambda/latest/dg/with-sns.html).

![Select AWS Lambda and the raki-sns-handler endpoint](../../../images/5-Workshop/5.5-Serverless/5.5.1-SNS-Notification/06-lambda-subscribe.png)

![The Lambda subscription status is Confirmed](../../../images/5-Workshop/5.5-Serverless/5.5.1-SNS-Notification/07-lambda-confirmed.png)

![SNS topic showing both Email and Lambda subscriptions as Confirmed](../../../images/5-Workshop/5.5-Serverless/5.5.1-SNS-Notification/11-topic-confirmed-subscriptions.png)

## Tests and results

![Publish message button on the raki-noti-sns topic page](../../../images/5-Workshop/5.5-Serverless/5.5.1-SNS-Notification/08-publish-console.png)

1. **Send a different message for each protocol.** Open the topic and choose **Publish message**. Use the option for different messages by protocol and enter the content for `email`. The screenshot shows the test message in the inbox.
2. **Send the same message to all subscriptions.** We used the same message for each subscription and sent a test notification for order `#1001`. The screenshot shows the message in the inbox.
3. **Check the Lambda publisher.** For a test order, compare the time the function ran, the `MessageId` in the response or in CloudWatch Logs, and the email received. This is what we expect to check in a publisher test. The two email screenshots below show messages sent directly from the SNS console.

![Email with the test message for the Email protocol](../../../images/5-Workshop/5.5-Serverless/5.5.1-SNS-Notification/09-email-protocol-test.png)

![Test email for order #1001](../../../images/5-Workshop/5.5-Serverless/5.5.1-SNS-Notification/10-email-order-test.png)

## Conclusion

We confirmed that the Standard topic sent test emails and that we added the Lambda subscription. This checks the **SNS → email** flow. To verify the **application → publisher → SNS → email** flow, we need to match the publisher log to an email with the same test order ID. 