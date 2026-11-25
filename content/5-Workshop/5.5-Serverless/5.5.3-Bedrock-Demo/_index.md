---
title : "Integrate Bedrock"
date : 2024-01-01
weight : 3
chapter : false
pre : " <b> 5.5.3 </b> "
---

## Objective

Our team built the RakiBookery assistant to help customers use the website, look up store information, and choose books. Prices and stock come from MySQL so they reflect the current catalog. Store contact details, policies, and website guidance come from documents in the Knowledge Base.

## Architecture overview

```text
Question → Python 3.12 Lambda → Input Guardrail → Intent routing
                                  ├─ Store questions → Agent Runtime Retrieve → Knowledge Base
                                  └─ Book requests   → RDS MySQL → In-stock catalog
                        Relevant data → Nova Micro Converse → Output check → Response
```

Lambda uses **Bedrock Agent Runtime** to call `Retrieve` on the Knowledge Base. It orchestrates the two data paths and calls `Converse` through **Bedrock Runtime** to generate the response. [Amazon Bedrock Retrieve](https://docs.aws.amazon.com/bedrock/latest/APIReference/API_agent-runtime_Retrieve.html) · [Amazon Bedrock Converse](https://docs.aws.amazon.com/bedrock/latest/APIReference/API_runtime_Converse.html).

## Implementation

### Step 1 Prepare the Knowledge Base and S3 source

We prepared reference documents and their metadata in S3. The managed Knowledge Base uses `managedSearchConfiguration` and filters on `document_type`. Store and website questions use `store_info` documents. Reference material marked `environment=demo` is presented as example information. If retrieval finds no suitable passages, the handler reports that the information is unavailable.

![Knowledge Base enabled in Lambda](../../../images/5-Workshop/5.5-Serverless/5.5.3-Bedrock-Demo/01-knowledge-base-config.png)

The project notes record the Knowledge Base as `Available` and one completed sync with **two documents, two metadata files, and zero errors**. To verify retrieval from Lambda, we check for `KB retrieved` in the logs and compare the answer with the source document.

### Step 2 Configure Lambda and Nova Micro

![Nova Micro profile and Region in the Lambda settings](../../../images/5-Workshop/5.5-Serverless/5.5.3-Bedrock-Demo/02-nova-micro-config.png)

We switched to **Amazon Nova Micro** with the `apac.amazon.nova-micro-v1:0` profile, called from Singapore (`ap-southeast-1`). Lambda signs requests using its execution role. The relevant permissions are `bedrock:Retrieve` for the Knowledge Base, `bedrock:InvokeModel` for the authorized profile and models, and `bedrock:ApplyGuardrail` for the configured Guardrail. MySQL credentials are kept in Secrets Manager.

### Step 3 Connect the RDS catalog for recommendations

![Lambda MySQL settings reviewed during troubleshooting, with connection values redacted](../../../images/5-Workshop/5.5-Serverless/5.5.3-Bedrock-Demo/03-rds-connection-config.png)

Lambda connects to MySQL in the VPC and reads the `bookstoredb` schema. We checked the project SQL, used `book.Name`, and joined the `category` table through `CategoryID`. Only active, in-stock books are included in the catalog sent to the model. The model returns up to three IDs, and Lambda validates them and uses the SQL results for titles, prices, and stock. Setting `BOOK_KB_ENABLED` can add context from book documents.

### Step 4 Apply Guardrails to input and output

![Guardrail version 1 in the Lambda settings](../../../images/5-Workshop/5.5-Serverless/5.5.3-Bedrock-Demo/04-guardrail-config.png)

The configuration notes record Guardrail version `1`. The handler calls `ApplyGuardrail` before looking up data and again for the content the user will see, including fallback book lists. If the Guardrail intervenes, the handler returns `GUARDRAIL_BLOCKED`. We prepared an ordinary question and a test keyword covered by the published rule to exercise both cases. [Amazon Bedrock ApplyGuardrail](https://docs.aws.amazon.com/bedrock/latest/APIReference/API_runtime_ApplyGuardrail.html).

## Testing and troubleshooting

We check a store question, a book recommendation, and a configured block case separately. We can use each `request_id` to find the corresponding CloudWatch logs. For recommendations, we compare book IDs and prices with RDS. For store questions, we check the number of retrieved passages and compare the response with its S3 source.

![AWS Lambda log excerpt showing the Guardrail check and MySQL error 1045](../../../images/5-Workshop/5.5-Serverless/5.5.3-Bedrock-Demo/05-guardrail-mysql-diagnostic.png)

A Lambda diagnostic run on AWS recorded an input Guardrail result of `action=NONE`. In the same invocation, MySQL returned `1045` during login. The following excerpt retains the fields relevant to the diagnosis.

```text
Guardrail source=INPUT action=NONE
MYSQL_CONNECTION_DIAGNOSTIC {"mysql_error": 1045, "stage": "connect", "attempt": 1}
MYSQL_CONNECTION_DIAGNOSTIC {"mysql_error": 1045, "stage": "connect", "attempt": 2}
```

Error `1045` shows that Lambda received a response from MySQL and authentication was rejected. We need to match the secret credentials to the active instance before repeating the recommendation case. All **16 local tests passed**, covering metadata filters, book ID validation, quota handling, and output Guardrails.

![Local test output confirming 16 passing tests](../../../images/5-Workshop/5.5-Serverless/5.5.3-Bedrock-Demo/06-local-test-result.png)

## Conclusion

We prepared and synced the Knowledge Base source and recorded an input Guardrail check on AWS. The book recommendation path needs a rerun after the MySQL authentication issue is resolved. AWS `Retrieve` and `Converse` results need confirmation from per-case logs and responses. Section 5.5.4 covers connecting the assistant to the website through API Gateway.
