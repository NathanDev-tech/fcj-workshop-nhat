---
title: "Setup API Gateway"
date: 2024-01-01
weight: 4
chapter: false
pre: " <b> 5.5.4. </b> "
aliases:
  - "/5-workshop/5.5-operations/5.5.4-api-gateway/"
---

## Objective

After testing the Lambda handler locally in section 5.5.3, our team needed an HTTPS endpoint for the RakiBookery website to send questions to the book assistant. API Gateway receives the request, forwards the JSON body to Lambda, and returns the result to the website. This lets us verify the API call before testing it in a browser.

## Request flow

```text
RakiBookery website → HTTP API → Book assistant Lambda → Bedrock and data sources
RakiBookery website ← API Gateway HTTP response ← Lambda result
```

## Step 1. Create the HTTP API and connect Lambda

In Singapore (`ap-southeast-1`), open **API Gateway → Create API → HTTP API**. Choose a **Lambda** integration and select the assistant function from [section 5.5.3]({{< relref "/5-Workshop/5.5-Serverless/5.5.3-Bedrock-Demo/" >}}). Use payload format **2.0** for the proxy integration. The handler can read the method from `requestContext.http.method` and decode the `body` field. Check the function's resource-based policy for API Gateway invoke permission. [AWS Lambda proxy integration documentation](https://docs.aws.amazon.com/apigateway/latest/developerguide/http-api-develop-integrations-lambda.html).

<!-- Image 01 when available: API overview and Region. -->
<!-- Image 02 when available: Lambda target and integration payload format. -->

## Step 2. Configure the route and stage

Use `POST /chat` as the example route and attach the Lambda integration. In **Stages**, confirm which stage receives requests and whether **Auto-deploy** is enabled. For the `$default` stage, append `/chat` to the invoke URL. For a named stage, use the stage URL displayed in the console and append the route path. Compare the final URL with the website configuration. [AWS HTTP API stages documentation](https://docs.aws.amazon.com/apigateway/latest/developerguide/http-api-stages.html).

<!-- Image 03 when available: route method and integration. -->
<!-- Image 04 when available: stage and invoke URL. -->

## Step 3. Configure CORS for the website

If the website and API use different origins, set **Allow origins** to the website origin in use, such as the HTTPS address users open through CloudFront. Allow `POST` and the `Content-Type` header so the browser can send JSON. If the application sends a token, also allow the `Authorization` header and verify the authorizer against the project's sign-in flow. With CORS enabled on the HTTP API, API Gateway handles the `OPTIONS` preflight and sets the CORS headers from its configuration. [AWS HTTP API CORS documentation](https://docs.aws.amazon.com/apigateway/latest/developerguide/http-api-cors.html).

<!-- Image 05 when available: CORS origin, method, and headers. -->

## Step 4. Verify the API and browser request

Send a `POST` request with the `Content-Type: application/json` header and the following body.

```json
{
  "prompt": "Cho tôi thông tin liên hệ nhà sách."
}
```

- **Valid question.** Check the HTTP status and JSON body. After successful retrieval and generation, the handler is expected to return `200` with `answer` and `request_id`. Compare the answer with the source document in the Knowledge Base.
- **Missing question.** Send `{}` to the same route. The handler is expected to return `400` with `BAD_REQUEST`. This checks request delivery to Lambda and the return of its validation error.
- **Website request.** Open the browser's **Network** panel and send a question from the website. Check the URL, method, status, and displayed response. If a direct request works and the browser reports an error, inspect the preflight request and origin. For Lambda errors, use the response `request_id` to inspect CloudWatch as described in [section 5.6.5]({{< relref "/5-Workshop/5.6-Operations/5.6.5-CloudWatch-Logs-Log-Insights/" >}}).

<!-- Image 06 when available: responses for a valid question and a missing prompt. -->
<!-- Image 07 when available: browser Network request or displayed answer. -->

## Conclusion

The project Lambda handles an API Gateway request body. The example `POST /chat` route documents the configuration and verification flow. Confirm the deployment using the actual route, stage, and response from the RakiBookery environment.
