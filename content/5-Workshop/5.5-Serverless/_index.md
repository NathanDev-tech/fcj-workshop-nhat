---
title: "Serverless & AI"
date: 2024-01-01
weight: 5
chapter: false
pre: " <b> 5.5. </b> "
aliases:
  - "/5-workshop/5.5-operations/"
---

## Objective

While working on the project, I helped my team add features to the RakiBookery bookstore website. When an admin uploads a book image, the site needs a small image for the catalog. We also needed to send test emails and build an assistant to answer store questions or help customers find books. This section shows how we added these features and checked each flow.

## Architecture overview

The PHP website runs on Amazon ECS with Fargate and stores bookstore data in Amazon RDS for MySQL. When an image is uploaded with a PUT request and its object key starts with `images/`, the S3 event calls the image Lambda function. The function saves the thumbnail under `thumbnails/`. The output prefix is outside the trigger filter, so the thumbnail does not start the function again. For notifications, an SNS topic sends messages to confirmed email subscribers and calls a Lambda function to log them.

For the book assistant, we designed API Gateway as the HTTP entry point to Lambda. Based on the question, Lambda finds store information in the Knowledge Base or reads the list of books in stock from RDS. It sends the data for that question to Nova Micro. Guardrails check the input and the output. For example, a question about a store policy needs a synced document. A question about price or stock needs MySQL data.

## Why we use separate event flows

SNS delivers one published message to its subscribers. Lambda processes events such as new uploads without keeping an extra server running. The book assistant uses HTTP because the website needs a response to each question. These flows have different ways to prove success. For an event, find the output object, log, or email. For the assistant, trace the HTTP response back to the Lambda call and its data source.

## Work order and checks

**Notifications with SNS.** We created a topic and sent a test message to a confirmed email address. This helps us check email delivery before adding the application as a source. When we connect the publisher, we use a test message with an ID. We can match the ID in the publish log and the email.

**Event processing with Lambda.** After creating a function, we test it with a sample event in the console. Then we test it with an event from the real service. For book images, we match the file in S3 with the log for the same run. For the statistics function, we compare the result with a direct RDS query to check the numbers.

**Book assistant.** We prepare the documents and metadata in S3, then sync the Knowledge Base before testing retrieval. We use separate prompts for store information and book suggestions. We test the Guardrail on its own so we can see its result before looking for errors in later data or model calls.

**API Gateway tests.** The test starts with a JSON request sent directly to the HTTP API route. Then we will call the same route from the website. We will check the URL, HTTP status, displayed response, and `request_id` if Lambda returns one. These details help us find whether a failure is in the browser, API Gateway, or the handler.

## Results and lessons learned

We received a test email sent directly from the SNS console. After we uploaded a test image to S3, we found a smaller image under `thumbnails/` and a log stream for the image function. The SNS handler also read a sample event when we used **Test** in the Lambda console. To check the full notification flow, we still need a real call from SNS to Lambda and a message from the application publisher.

We synced documents to the Knowledge Base and passed 16 local tests for the book assistant. In one AWS test, the log showed that a Guardrail checked the question. The book suggestion request then stopped at the MySQL login with error `1045`. The API Gateway section has setup and test steps. We still need to check the real route and the browser response for a request from the website.

I learned how to follow a request through several services. For example, when a small image appears, I can compare the original image, the Lambda log, and the new S3 object. For the book assistant, I can use the error code and `request_id` to find the failed step before changing the setup.

## Evidence for the final demo

For each flow, keep one test input, the result, and an ID or timestamp that links them. Use an order code for SNS, an object key for image processing, and a Lambda `request_id` for the assistant. When a case fails, record the last step that worked and the change to test next.

## Conclusion

Our evidence shows that SNS sent a test email and Lambda created a thumbnail. To continue the website test for the assistant, we need to fix the MySQL login issue and check the API Gateway setup. [Section 5.6 Operations]({{< relref "/5-Workshop/5.6-Operations/" >}}) shows how we monitor and test these flows together.

## Implementation sections

{{% children sort="Weight" /%}}
