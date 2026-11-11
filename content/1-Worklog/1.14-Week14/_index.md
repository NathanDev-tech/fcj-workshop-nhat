---
title: "Week 14 Worklog"
date: 2026-11-02
weight: 14
chapter: false
pre: " <b> 1.14. </b> "
---

### Week 14 Objectives:

- Strengthen my understanding of AWS Lambda with Python, including `lambda_handler`, `event`, `context` and logging.
- Practice reading and validating CSV files, identifying invalid records and troubleshooting input formatting errors.
- Complete the AWS Lambda connection to Amazon RDS MySQL and invoke an AI model through Amazon Bedrock for **RakiBookery**.
- Set up an Amazon Bedrock Knowledge Base containing information about the RakiBookery bookstore.
- Improve troubleshooting skills across input data, database connectivity, permissions and AI services.

### Tasks completed this week:

| Day | Task | Start Date | Completion Date | Reference Material |
|:---:|---|:---:|:---:|---|
| 2 |  | 02/11/2026 | 02/11/2026 |  |
| 3 | Review Python-based AWS Lambda handlers, `event`/`context`, logging and local JSON test events (hands-on completion has not been separately confirmed) | 03/11/2026 | Not confirmed | [Lambda Python Handler](https://docs.aws.amazon.com/lambda/latest/dg/python-handler.html), [Lambda Python Logging](https://docs.aws.amazon.com/lambda/latest/dg/python-logging.html) |
| 4 | Develop and validate a laptop CSV processing script; resolve a leading blank-line issue that prevents header detection | 04/11/2026 | 04/11/2026 | [Python CSV](https://docs.python.org/3/library/csv.html), [Python Decimal](https://docs.python.org/3/library/decimal.html) |
| 5 | Complete the AWS Lambda connection to Amazon RDS MySQL and successfully invoke AI through Amazon Bedrock for RakiBookery | 05/11/2026 | 05/11/2026 | [Lambda with RDS](https://docs.aws.amazon.com/lambda/latest/dg/services-rds.html), [Bedrock Inference API](https://docs.aws.amazon.com/bedrock/latest/userguide/inference-api.html) |
| 6 | Complete the setup of an Amazon Bedrock Knowledge Base with data containing information about the RakiBookery bookstore | 06/11/2026 | 06/11/2026 | [Knowledge Base Data Sync](https://docs.aws.amazon.com/bedrock/latest/userguide/kb-data-source-sync-ingest.html), [Knowledge Base Retrieval](https://docs.aws.amazon.com/bedrock/latest/userguide/kb-how-retrieval.html) |

### Week 14 Achievements:

&emsp;◉ Tuesday:
<br>&emsp;&emsp;○ Reviewed the structure of Python AWS Lambda functions, including `lambda_handler(event, context)`, event input and logging during request handling.
<br>&emsp;&emsp;○ Prepared a local practice flow using sample JSON events and a Python handler. **Successful execution of Tuesday's local exercise was not separately confirmed**, so this is recorded as preparation and learning rather than a verified run.

&emsp;◉ Wednesday:
<br>&emsp;&emsp;○ Practiced processing laptop records from `samples/laptops_week14.csv`, with the fields `laptop_id`, `brand`, `model`, `ram_gb`, `price` and `quantity`.
<br>&emsp;&emsp;○ Worked with CSV structure checks, invalid numeric values and duplicate laptop IDs. The six-row sample was designed to contain four valid rows and two invalid rows.
<br>&emsp;&emsp;○ Encountered `ValueError: CSV không có header` while running `day04_validate_csv.py`. An inspection of the file size and leading bytes showed that the file was not empty, but started with `\r\n`, indicating a blank line before the actual header.
<br>&emsp;&emsp;○ Identified why `csv.DictReader` failed to detect the intended header, addressed the leading blank line and completed Wednesday's exercise.

&emsp;◉ Thursday:
<br>&emsp;&emsp;○ Completed the connection between AWS Lambda and the Amazon RDS MySQL database used by **RakiBookery**.
<br>&emsp;&emsp;○ Continued troubleshooting the Lambda–RDS connection through relevant networking and access settings, distinguishing connectivity issues from code execution problems.
<br>&emsp;&emsp;○ Successfully invoked an AI model through Amazon Bedrock in the RakiBookery workflow, combining the bookstore data layer with AI capabilities.

&emsp;◉ Friday:
<br>&emsp;&emsp;○ Completed the setup of an **Amazon Bedrock Knowledge Base** containing data about the RakiBookery bookstore.
<br>&emsp;&emsp;○ Reviewed how a Knowledge Base can organize private source information and support information retrieval in bookstore-related AI features.
<br>&emsp;&emsp;○ Distinguished **Knowledge Base setup** from detailed retrieval-quality testing, application integration and end-to-end RAG validation; these additional steps have not been confirmed as completed.
