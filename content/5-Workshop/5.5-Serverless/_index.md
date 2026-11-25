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

During my project, I worked with the team on features that support the RakiBookery storefront. The site needs thumbnails when administrators upload book covers. We also wanted to send test notifications by email and build an assistant that could answer store questions and suggest books. This section describes how we integrated those features and checked the results for each request flow.

## Architecture overview

The PHP website runs on ECS Fargate and keeps its bookstore data in RDS MySQL. For image processing, S3 emits an event when a file is added under `images/`. Lambda processes the image and saves it under `thumbnails/`. Using separate prefixes keeps the thumbnail from triggering another invocation. For notifications, SNS delivers messages to confirmed email subscribers and to a Lambda handler that logs them.

For the book assistant, the design uses API Gateway as the HTTP entry point to Lambda. Depending on the question, Lambda retrieves store documents from the Knowledge Base or reads the in-stock catalog from RDS to provide relevant data to Nova Micro. Guardrails check the input and output. A policy question needs a synced store document, while questions about current prices and stock depend on MySQL data.

## Implementation sections

{{% children sort="Weight" /%}}

## Results and lessons learned

Our team received a test email published directly from the SNS console. After uploading a test image to S3, we found a thumbnail under `thumbnails/` and log streams for the image function. The SNS handler also processed a sample event using **Test** in the Lambda console. Full notification verification still requires a live invocation from SNS and a test of an application notification sent through the publisher.

For the assistant, we synced documents to the Knowledge Base and passed 16 local tests. One AWS run recorded an input Guardrail check, and the recommendation request stopped at MySQL login with error `1045`. The API Gateway section documents configuration and verification steps. We still need to match a website request to the deployed route and its browser response.

I learned to trace a request through several services. For thumbnails, I could compare the source image, Lambda log, and output object. For the assistant, I used the error code and `request_id` to locate the failed step before changing configuration.

## Conclusion

The team collected evidence of SNS email delivery and Lambda thumbnail generation. Testing the assistant from the website requires resolving MySQL authentication and checking the deployed API Gateway configuration. [Section 5.6 Operations]({{< relref "/5-Workshop/5.6-Operations/" >}}) describes how we monitor these paths and test them as part of the full system.
