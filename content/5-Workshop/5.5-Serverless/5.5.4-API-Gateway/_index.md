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

After the local Lambda handler tests in section 5.5.3, our team needs an HTTPS address for the RakiBookery website to send questions to the book assistant. API Gateway gets the request from the website, sends the JSON body to Lambda, and returns the result. We can test the API call first, then try it in a browser.

## Request flow

```text
RakiBookery website → HTTP API → Book assistant Lambda → Bedrock and data sources
RakiBookery website ← HTTP response from API Gateway ← Lambda result
```

## Test inputs and access

Prepare the assistant function, a sample question, and the browser origin of the RakiBookery site. Decide how callers can access the route before connecting it to the website. CORS controls what a browser can read across origins. If the route requires sign-in, configure an authorizer that checks the user's access. If a test route is open, run it within the test scope and set an appropriate rate limit.

## Step 1. Create the HTTP API and connect Lambda

In the Singapore Region (`ap-southeast-1`), open **API Gateway → Create API → HTTP API**. Choose the **Lambda** integration and select the assistant function from [section 5.5.3]({{< relref "/5-Workshop/5.5-Serverless/5.5.3-Bedrock-Demo/" >}}). Use payload format **2.0** for the Lambda proxy integration. The handler reads the HTTP method from `requestContext.http.method` and parses the JSON in `body`. Check the function's resource-based policy to confirm that API Gateway has `lambda:InvokeFunction` permission for the correct function. [AWS guide to Lambda proxy integration](https://docs.aws.amazon.com/apigateway/latest/developerguide/http-api-develop-integrations-lambda.html).

> **Screenshot to add.** Capture the HTTP API overview in Singapore and **Integrations** showing the target Lambda and payload format `2.0`.

## Step 2. Set up the route and stage

Use `POST /chat` as an example route and connect it to the Lambda integration. In **Stages**, check which stage gets requests and whether **Auto-deploy** is on. For the `$default` stage, add `/chat` directly to the invoke URL. For a named stage, copy the URL from the console and add the route path. Check that this matches the URL in the website setup. [AWS guide to HTTP API stages](https://docs.aws.amazon.com/apigateway/latest/developerguide/http-api-stages.html).

> **Screenshot to add.** Capture **Routes** showing the actual method and path. Capture **Stages** with the stage name and invoke URL.

## Step 3. Set up CORS for the website

If the website and API have different origins, set **Allow origins** to the website origin. For example, use the HTTPS address that users open through CloudFront. Allow the `POST` method and the `Content-Type` header so the browser can send JSON. If the application sends a token, allow the `Authorization` header too. Check that the authorizer matches the sign-in setup. When CORS is on for the HTTP API, API Gateway handles the `OPTIONS` preflight and returns the CORS headers set on the API. [AWS guide to CORS for HTTP APIs](https://docs.aws.amazon.com/apigateway/latest/developerguide/http-api-cors.html).

> **Screenshot to add.** Capture CORS settings with the website origin, allowed methods, and allowed headers.

For a first test, use the invoke URL from **Stages** and the actual route path. If you enable access logs for the stage, include the API Gateway request ID, route, and status. The `request_id` in the response body comes from Lambda and is used to find the function log. [AWS HTTP API access logging](https://docs.aws.amazon.com/apigateway/latest/developerguide/http-api-logging.html).

## Step 4. Test the API and the browser

Send a `POST` request to the route. Set the header to `Content-Type: application/json` and send this body.

```json
{
  "prompt": "Cho tôi thông tin liên hệ nhà sách."
}
```

- **Valid question.** Check the HTTP status and JSON response. If the handler finds the information and builds an answer, it should return `200` with `answer` and `request_id`. For this question, we compare the answer with the Knowledge Base document.
- **Website test.** Open the browser's **Network** panel and send a question from the website. Check the URL, method, HTTP status, and answer on the page. Use the response `request_id` to find the matching Lambda log as shown in [section 5.6.5]({{< relref "/5-Workshop/5.6-Operations/5.6.5-CloudWatch-Logs-Log-Insights/" >}}).

After testing with an API client, send the same question from the browser. Compare the JSON response with the answer on the page to check the path from the website to Lambda.

> **Screenshot to add.** Capture the successful API response with `answer` and `request_id`. Capture the browser's **Network** request, the answer on the page, and the matching Lambda log. Hide authorization headers and tokens.

## Conclusion

The Lambda code can read the API Gateway request body. The HTTP API setup shows how to connect the website to the assistant through a Lambda proxy integration, a route, a stage, and CORS. We use `POST /chat` as an example to compare the API URL, JSON response, and answer on the page.
