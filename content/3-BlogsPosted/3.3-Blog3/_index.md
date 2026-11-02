---
title: "Blog 3"
date: 2026-10-30
weight: 3
chapter: false
pre: " <b> 3.3. </b> "
---

# 🐍 Boto3 and DynamoDB: When the "Seemingly Simple" Scan Becomes an Architectural Trap

## 📝 Introduction: A Tale from a "10-Minute" Task

When working with data services on AWS using Python, **Boto3** and **Amazon DynamoDB** are practically the most familiar duo for every cloud engineer. The initial scenario is usually deceptively simple:

> *"I have a DynamoDB table storing a product catalog. The requirement is straightforward: write a quick Python script to extract data, filter products by certain criteria, and export the report to JSON or CSV format."*

For a developer accustomed to SQL or relational databases, the natural mindset is:
1. Open a connection to the table via the Boto3 SDK.
2. Call the `scan()` function with a conditional filter expression.
3. Receive the result set and write it directly to a file using Python's standard `json.dump()`.

Everything sounds like an elementary exercise that should take less than 10 minutes. But when deploying the script to a real environment with thousands of records, the system immediately throws two buckets of cold water at the developer:

* **The Disappearing Data Shock (Pagination Pitfall):** The table contains thousands of rows, and you know for certain that records matching your filter exist, yet the scan operation returns an empty array `[]`. The script terminates early and misses the vast majority of the data.
* **The Serialization Nightmare:** The JSON serialization call crashes immediately with a familiar exception:  
  `TypeError: Object of type Decimal is not JSON serializable`.

Why does an apparently elementary data retrieval task harbor such bizarre traps? What are DynamoDB and Boto3 doing beneath the distributed storage layer that "Hello World" tutorials rarely mention?

This post won't just dump hundreds of lines of boilerplate code for you to copy-paste. Instead, we will dissect the **underlying architectural realities**: from the 1MB physical pagination limit and the design philosophy behind the `Decimal` data type, to the costly illusion of using `FilterExpression` in production environments.

---

## 🏗️ Context & Under the Hood Mechanics

To understand why these behaviors occur, we need to examine how Boto3 communicates with DynamoDB storage partitions:

![aws-architecture](/fcj-workshop-nhat/images/aws_blog3.png)

### 1. Overall Data Extraction Flow Architecture

The data extraction journey starts from the client request, traverses IAM authentication, request routing, physical partition access, and output normalization.

### 2. Internal Pagination Lifecycle of the DynamoDB Engine

The critical misunderstanding for many developers lies in the **order of operations** between reading raw data and applying filters on the DynamoDB server.

> ⚠️ **The Golden Rule:** DynamoDB executes steps **1 & 2 (Read raw data from physical disk storage)** BEFORE executing step **3 (Filter data in memory)**. This exact separation is the root cause of the first pagination pitfall.

---

## 🔍 Deep Dive 1: The 1MB Physical Limit and the "Empty Scan with Data Remaining" Paradox

### The 1MB Hard Limit
In every single network round-trip, DynamoDB imposes a hard limit: **it will scan a maximum of 1 MB of raw data** from storage partitions.

Imagine a realistic scenario:
* Your table is 10 MB in size and contains around 20,000 products.
* You want to find high-end laptops priced above $3,000, and these items happen to reside in the second half of the table.
* When you call `table.scan(FilterExpression=...)`, DynamoDB starts reading the first 1 MB of raw data from the physical partitions.
* Only after reading that 1 MB does it evaluate the conditional filter.
* **The outcome:** Not a single laptop in that first 1 MB has a price greater than $3,000.
* **What gets returned to you:**
  - `Items` list: **Empty** (`[]`)
  - `Count`: **0**
  - Scanned records `ScannedCount`: **~2,000 records**
  - Accompanied by a special field: `LastEvaluatedKey` (the cursor pointer marking where the 1 MB read stopped).

### The Fatal Trap in Application Logic
Most developers, upon seeing an empty `Items` list returned, have an instinctive reaction:

> *"If no elements were returned, it means there is no more data or no matching records exist. Time to break the loop!"*

This is the most critical mistake. The script aborts after the very first call, completely ignoring the remaining 9 MB of data. The user receives an empty report, even though dozens of matching products exist in the system.

### Stateless Cursor Pagination Mindset
In the distributed NoSQL world, pagination does not operate with `page=1, page=2` or SQL `OFFSET 50`. It operates entirely on **positional cursors**.

Golden rules to remember:
1. **The presence of `LastEvaluatedKey` is the sole criterion** for determining whether the table has more data to evaluate.
2. The number of elements in the `Items` array is completely independent of whether additional data remains. `Items` can be empty, but as long as `LastEvaluatedKey` is present, you must continue issuing requests with `ExclusiveStartKey` set to that cursor.
3. The loop is only complete when the `LastEvaluatedKey` field is absent from the DynamoDB response.

---

## 🔍 Deep Dive 2: The `Decimal` Nightmare — When Precision Standards Clash with JSON Serialization

Once past the pagination trap and having retrieved the full dataset, developers confidently call the export function:
`json.dump(items, file)` — only to be greeted by an immediate crash:

```text
TypeError: Object of type Decimal is not JSON serializable
```

Why does a standard library like Boto3 convert simple integers and floats into Python's complex `Decimal` object?

### The Design Philosophy Behind Boto3's `Decimal` Type
In DynamoDB, the `Number` data type supports up to **38 digits of precision** (covering both integers and decimals).

Conversely, standard `float` in Python (and most modern programming languages) conforms to the **IEEE-754** standard (64-bit floating point). This format is notorious for binary rounding errors:
* In Python, a simple addition like `0.1 + 0.2` results in `0.30000000000000004` instead of a clean `0.3`.

Now imagine this: DynamoDB is storing bank account balances, cryptocurrency values, or stock trading prices. If Boto3 automatically cast DynamoDB Numbers into standard `float`, rounding errors would accumulate and lead to severe financial discrepancies.

Therefore, the Boto3 SDK design team made an architecturally airtight decision: **Map all DynamoDB Number types directly to Python's `decimal.Decimal` objects**.

### Conflict with the Standard JSON Serializer
The problem arises because Python's default `json` module is built to only support primitive data types: strings, integers, floats, booleans, dicts, lists, and `None`. When encountering a `Decimal` object, the default serializer has no built-in instruction on how to represent it in JSON text, triggering a `TypeError`.

### Architectural Mitigation Approaches
To resolve this without compromising data integrity, you have three design options:

1. **Custom Encoder at the Serialization Layer (Recommended):**  
   Create a class inheriting from Python's `json.JSONEncoder`. Inside, check if the object is an instance of `Decimal`, and convert it to `int` (if it has no fractional part) or `float` at the moment of serialization. This preserves full in-memory precision throughout your business logic, converting only when producing external output.
2. **Use an Advanced Serializer (such as `simplejson`):**  
   Instead of the standard library, use alternative packages that natively recognize and serialize `Decimal` objects to JSON numbers without custom wrapper code.
3. **Data Sanitization Pipeline Before Handoff:**  
   If the data retrieved from DynamoDB needs to be forwarded to other downstream services (e.g., returned via FastAPI, pushed to a Celery queue, or stored in MongoDB), the safest approach is running it through a recursive normalization function to sanitize all `Decimal` values into native Python numbers before leaving the database integration layer.

---

## 🔍 Deep Dive 3: The `FilterExpression` Illusion and the True Cost of `Scan` in Production

One of the most widespread and financially destructive misconceptions in DynamoDB is:

> *"I provided conditions in `FilterExpression`, and the server only returned the 5 records I need, so I only pay read capacity for those 5 records."*

This illusion is completely at odds with how DynamoDB actually operates.

### The Raw Reality of Read Capacity Units (RCUs)
DynamoDB charges you based on the **raw volume of data read from physical disk partitions**, not the filtered payload returned to your client over the wire:

* Suppose your table holds **10 GB** of data.
* You execute a `Scan` with a `FilterExpression` to search for products under brand X (which only account for **50 MB** of the table).
* DynamoDB iterates through all storage partitions, reads the entire **10 GB of raw data**, applies the filter in server memory, and streams back just 50 MB.
* **The AWS Bill:** You are billed for RCUs consumed across the full **10 GB read**, down to the last byte!

```text
┌─────────────────────────────────────────────────────────────┐
│                    DYNAMODB STORAGE PARTITIONS              │
│ [Partition 1: 2.5GB] [Partition 2: 2.5GB] [Partition 3: ...]│
└─────────────────────────────────────────────────────────────┘
                                ▲
                         table.scan(...)
              (Reads and consumes RCUs for entire 10GB)
                                │
                      ┌─────────────────────┐
                      │  FilterExpression   │
                      │  (Discards 9.95GB)  │
                      └─────────────────────┘
                                ▼
                      Only returns 50MB to Client!
```

### Two Operational Hazards of Scan Abuse

1. **Cost Explosion:**  
   If your table is configured in **On-Demand** mode (`PAY_PER_REQUEST`), running a scan across a multi-gigabyte table in an automated script can cost tens of dollars per execution. If that script runs via a cron job every 15 minutes, your end-of-month cloud invoice will be a catastrophic surprise.
2. **Table Throttling and Application Outages (Throttling Blast Radius):**  
   If the table runs in **Provisioned** mode, read capacity is fixed per second (e.g., 100 RCU/sec). An aggressive table-wide scan will instantly deplete the partition's available throughput. Consequently, production web applications or user-facing APIs trying to perform single-item reads or writes will be rejected with `ProvisionedThroughputExceededException`. An internal reporting script just took down your core customer service!

### When Is `Scan` Acceptable, and What Are the Standard Alternatives?

* **When is Scan acceptable?**  
  Only for very small lookup/configuration tables (under a few megabytes), in non-production environments (Dev/Staging), or during scheduled off-peak maintenance batch jobs equipped with strict rate-limiting.
* **Alternative 1: Primary Key Redesign and `Query` Operations**  
  In DynamoDB, `Query` is the most efficient operation because it jumps directly to the target partition using the Partition Key and reads only the relevant range defined by the Sort Key. Zero extraneous bytes are read.
* **Alternative 2: Global Secondary Indexes (GSIs)**  
  If you frequently need to filter by a non-primary key attribute (such as category or status), define a GSI with that attribute as the Partition Key. You can then execute a targeted `Query` against the index instead of scanning the whole table.
* **Alternative 3: DynamoDB Export to Amazon S3 (Zero-RCU Impact)**  
  When you need to extract an entire large table (tens of GBs or TBs) for analytics, reporting, or data lake ingestion, never write a custom Boto3 scan loop. Leverage AWS's native **DynamoDB Export to S3** feature. It reads directly from internal storage snapshots and writes to S3 as Parquet or JSON **without consuming a single RCU** from the active table and with zero impact on production traffic.

---

## ⚖️ Summary & Architectural Takeaways

Working with cloud services and distributed databases requires more than memorizing SDK function calls or writing a quick local script. Understanding the internal storage and execution model is what separates amateur code from production-grade engineering:

| Dimension | Common Assumption | DynamoDB Architectural Reality |
|---|---|---|
| **Pagination** | Break the loop when the `Items` array is empty | Paginate via `LastEvaluatedKey`. An empty `Items` list does not mean the scan is finished. |
| **Numeric Types** | Expect native integers or floats | Boto3 enforces `Decimal` to guarantee 38-digit precision, requiring explicit serialization handling. |
| **Data Filtering** | Assume `FilterExpression` reduces read capacity costs | RCUs are charged for 100% of raw data read from disk prior to filtering. Filters only reduce network bandwidth. |
| **Bulk Extraction** | Use sequential `Scan` loops to dump tables | Use indexed `Query`, or switch to `DynamoDB Export to S3` for cost efficiency and zero-RCU production impact. |

> *"Understanding the distributed storage mechanisms under the hood matters far more than memorizing SDK syntax. When you respect how cloud services are engineered, your solutions become resilient, scalable, and cost-effective."*

Hopefully, this deep dive provides you with the architectural clarity needed to design, operate, and extract data from Amazon DynamoDB reliably—without unwelcome runtime exceptions or surprise cloud bills!