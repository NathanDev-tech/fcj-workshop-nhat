---
title: "Week 13 Worklog"
date: 2026-10-26
weight: 13
chapter: false
pre: " <b> 1.13. </b> "
---

### Week 13 Objectives:

- Strengthen Python skills for AWS automation using a project-specific `.venv` and Boto3.
- Check AWS CLI, AWS identity, Region and credentials configuration before calling AWS services from Python.
- Practice creating and reading Amazon DynamoDB data with Python/Boto3.
- Build a DynamoDB Exporter mini project to read laptop records, handle pagination and export CSV/JSON files.
- Apply logging, error handling and the principle of keeping credentials out of source code.

### Tasks Completed During the Week:

| Day | Task | Start Date | Completion Date | References |
|:---:|---|:---:|:---:|---|
| 2 |  | 26/10/2026 | 26/10/2026 |  |
| 3 | Prepare a Python `.venv`, install Boto3, check AWS CLI, AWS identity and Region; resolve the Windows `aws` command conflict by invoking the official AWS CLI executable | 27/10/2026 | 27/10/2026 | [Boto3 Quickstart](https://docs.aws.amazon.com/boto3/latest/guide/quickstart.html), [Boto3 Credentials](https://docs.aws.amazon.com/boto3/latest/guide/credentials.html), [AWS CLI Installation](https://docs.aws.amazon.com/cli/latest/userguide/getting-started-install.html) |
| 4 | Review inventory data structures and CSV/JSON input/output organization for Python mini projects | 28/10/2026 | 28/10/2026 | [Python CSV](https://docs.python.org/3/library/csv.html), [Python JSON](https://docs.python.org/3/library/json.html) |
| 5 | Analyze DynamoDB Exporter requirements: use Scan, continue through `LastEvaluatedKey` and generate a summary report | 29/10/2026 | 29/10/2026 | [DynamoDB with Boto3](https://docs.aws.amazon.com/amazondynamodb/latest/developerguide/programming-with-python.html), [Boto3 Quickstart](https://docs.aws.amazon.com/boto3/latest/guide/quickstart.html) |
| 6 | Create the `LaptopInventory` table in Singapore Region, seed 5 sample laptop records with Boto3, verify them with Scan and build the DynamoDB Exporter source code for CSV/JSON output | 30/10/2026 | 30/10/2026 | [DynamoDB Scan](https://docs.aws.amazon.com/amazondynamodb/latest/developerguide/Scan.html), [Boto3 DynamoDB Table](https://docs.aws.amazon.com/boto3/latest/reference/services/dynamodb/table/scan.html) |

### Achievements During Week 13:

&emsp;◉ Tuesday:
<br>&emsp;&emsp;○ Created a project-specific Python `.venv` for `MiniProject2` and installed the Boto3 library.
<br>&emsp;&emsp;○ Checked AWS CLI configuration, credentials and the `ap-southeast-1` Region; used `aws sts get-caller-identity` to verify the active AWS identity.
<br>&emsp;&emsp;○ Troubleshot the Windows `aws` command, which initially invoked `D:\aws.exe` and returned a missing `_socket` module error; identified and directly invoked the official AWS CLI v2 executable in the per-user installation directory.
<br>&emsp;&emsp;○ Checked the configuration with `aws configure list` and confirmed that the installed AWS CLI v2 could run successfully.

&emsp;◉ Wednesday:
<br>&emsp;&emsp;○ Reviewed how to organize inventory data in CSV/JSON format and identified the fields required for the laptop use case.
<br>&emsp;&emsp;○ Prepared a separation between input data, processed data and output reports so the Python program would be easier to validate and extend.

&emsp;◉ Thursday:
<br>&emsp;&emsp;○ Analyzed the DynamoDB Exporter requirements: read table items, handle Scan results split across multiple pages and aggregate the data.
<br>&emsp;&emsp;○ Studied the role of `LastEvaluatedKey` and `ExclusiveStartKey` in continuing a read operation until no next page remains.
<br>&emsp;&emsp;○ Defined the two main output files as `laptop_export.csv` and `inventory_summary.json`.

&emsp;◉ Friday:
<br>&emsp;&emsp;○ Created the `LaptopInventory` DynamoDB table in `ap-southeast-1`, using `PAY_PER_REQUEST` billing mode and a String partition key named `laptop_id`.
<br>&emsp;&emsp;○ Wrote `seed_data.py` with Boto3 to insert five sample laptop records containing ID, brand, model, RAM, price and quantity. The script uses `attribute_not_exists(laptop_id)` to avoid overwriting existing items.
<br>&emsp;&emsp;○ Ran the seed script and received successful insert results for all five records. Then used the official AWS CLI v2 executable to scan the table and confirmed `Count = 5` and `ScannedCount = 5`.
<br>&emsp;&emsp;○ Built `dynamodb_exporter.py` with Python/Boto3 to connect to the table, read data using Scan, handle pagination through `LastEvaluatedKey`, write CSV with `csv.DictWriter` and generate a JSON report.
<br>&emsp;&emsp;○ Added logging, AWS error handling, `Decimal` conversion for JSON output and an `output` directory for generated files.


