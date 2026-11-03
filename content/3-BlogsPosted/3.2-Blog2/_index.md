---
title: "Blog 2"
date: 2026-10-15
weight: 2
chapter: false
pre: " <b> 3.2. </b> "
---

# 🦆 Aurora PostgreSQL + Embedded DuckDB: Querying the Data Lake Without ETL — But at What Cost?

## 📝 Introduction

In modern transactional systems, one architectural dilemma comes up constantly:

> **How do we join real-time operational data with years of historical analytics without turning our primary database into an expensive, bloated storage silo?**

The textbook answer has always been:

1. Keep live transactional data in **Aurora PostgreSQL**.
2. Offload cold history to an **Amazon S3 Data Lake** (in Parquet or Apache Iceberg format).
3. Build and maintain an **ETL pipeline** (Glue, Flink, Spark, or dbt) to sync aggregated history back into PostgreSQL whenever the application needs to join active orders with lifetime customer metrics.

Every data engineer knows the operational tax of this pattern: brittle pipeline jobs, data drift, sync latency, duplicate storage costs, and schema sync headaches.

On September 30, 2026, AWS introduced a game-changing feature: **Amazon Aurora PostgreSQL now supports direct querying of Apache Iceberg and Parquet in S3 via an embedded DuckDB columnar engine directly inside the PostgreSQL process.**

Your application uses the same connection string, the same Postgres driver, and familiar SQL queries. No external federated query engines, no custom APIs.

It sounds like a data engineer's dream. But in cloud architecture, there is no free lunch.

The question I asked was:

> **What happens under the hood when we replace our ETL pipelines with embedded DuckDB in Aurora, and what breaks when workloads scale?**

Let's break down the architecture layer by layer through practical experiments.

---

## 🏗️ Architecture / Context


Let's contrast the two architectural paradigms:

![Architecture Diagram](/fcj-workshop-nhat/images/AuroraPostgreeSQLwithDuckDB.png)

### Pattern A: Traditional ETL & Data Duplication
```text
Application
    ↓
Aurora PostgreSQL (Stores both hot orders & replicated historical tables)
    ↑
[ETL Pipeline (AWS Glue / EMR / Lambda / EventBridge)]
    ↑
Amazon S3 Data Lake (Parquet / Iceberg)
```

### Pattern B: Zero-ETL In-Process Lakehouse via Embedded DuckDB
```text
Application
    ↓ (Standard PostgreSQL SQL JOIN)
Aurora PostgreSQL Cluster
  ├── Native Table (e.g., `orders` in OLTP shared_buffers)
  └── Foreign Table (`aurora_analytics_server`)
        ↓
    Embedded DuckDB Engine (Vectorized execution & Column pruning)
        ↓
    Amazon S3 / S3 Tables / AWS Glue Data Catalog (Parquet & Iceberg)
```

In this architecture, historical data is exposed as a **PostgreSQL Foreign Table** backed by the `aurora_analytics` extension. When a query references both a native table and a foreign table, the Aurora Postgres planner delegates the analytical heavy lifting to DuckDB.

Let's run through the experiments and examine what happens when this architecture meets real-world pressures.

---

## 🔍 Experiment 1: 🔄 What Happens If We Eliminate the ETL Pipeline?

### The Promise
In a retail system, you need to display currently processing orders alongside a customer's lifetime purchase volume. 

With embedded DuckDB, you declare a foreign table pointing to an S3 URI or Glue Data Catalog:

```sql
CREATE FOREIGN TABLE analytics.ft_purchase_history (
    customer_id BIGINT,
    last_purchase_at TIMESTAMP,
    lifetime_order_count BIGINT
)
SERVER aurora_analytics_server
OPTIONS (
    location 's3://analytics-lakehouse/purchase-history/',
    format 'parquet',
    region 'ap-southeast-1'
);
```

Then query both within a single transaction:

```sql
SELECT
    o.order_id,
    o.customer_id,
    o.order_status,
    h.last_purchase_at,
    h.lifetime_order_count
FROM orders AS o
LEFT JOIN analytics.ft_purchase_history AS h
    ON h.customer_id = o.customer_id
WHERE o.order_status = 'PROCESSING';
```

### What You Gain
1. **Zero Data Movement:** No Glue ETL scripts, zero scheduling orchestration, and no double-paying for gigabytes or terabytes of duplicated storage inside Aurora.
2. **Freshness:** As soon as an upstream writer commits a new Parquet file or Iceberg snapshot to S3, foreign table queries see it immediately.
3. **Operational Simplicity:** Developers interact with a single Postgres endpoint using standard SQL dialects and familiar ORMs.

### The Catch
The data isn't missing; the **compute burden has simply moved**. Instead of a decoupled serverless Glue worker parsing Parquet files at 2:00 a.m., your primary Aurora database compute is now reading S3 objects, decompressing Snappy/ZSTD buffers, and constructing hash tables on demand.

### Verdict
> **"Eliminating ETL does not eliminate computational work — it transfers it directly onto your database instance."**

---

## 🔍 Experiment 2: ⚡ How Does Embedded DuckDB Optimize Reads (And When Does It Fail)?

### The Mechanics
DuckDB doesn't just scan S3 blindly. Aurora's implementation incorporates three performance pillars:

1. **Column Pruning:** If your Parquet file has 80 columns but your SQL only selects 2, only the byte ranges of those two columns are pulled from S3.
2. **Predicate Pushdown & Row-Group Skipping:** DuckDB reads Parquet file metadata and Iceberg manifest files. If your query filters by `date >= '2026-01-01'`, row groups whose metadata shows maximum values below that threshold are skipped entirely without downloading the data.
3. **On-Disk Read Cache:** Remote S3 chunks are cached on the Aurora instance's local SSD storage. Subsequent queries reading the same row groups hit local NVMe SSDs instead of S3 APIs.

### What Happens If Your Data Lake Layout Is Poor?
If your upstream data producers create millions of 100 KB files (the infamous **"Small Files Problem"**), or if data is not partitioned on frequently filtered columns:
- DuckDB sends tens of thousands of `s3:GetObject` HTTP requests.
- Latency spikes from milliseconds to tens of seconds due to request overhead.
- Predicate pushdown cannot prune row groups effectively.

Furthermore, **the on-disk read cache is ephemeral**. If your Aurora instance restarts, scales up/down, or fails over to a replica, the local SSD cache is wiped clean. Your initial queries after a maintenance event will hit cold S3 storage.

### Verdict
> **"DuckDB's query engine is extraordinarily fast, but it cannot compensate for an un-partitioned, fragmented Data Lake."**

---

## 🔍 Experiment 3: 💥 What Happens If the Analytical Query Runs Out of RAM?

### The Memory Reality in Aurora
PostgreSQL allocates the lion's share of instance RAM to `shared_buffers` (typically **65% to 75%** of total RAM) to cache native database pages for transactional OLTP speed.

Foreign table queries **do not use `shared_buffers`**. 

Instead, Aurora manages memory for DuckDB analytics using a dedicated parameter: `aurora_analytics.query_mem`.

```text
┌──────────────────────────────────────────────────────────────┐
│                  Aurora Instance Total RAM                   │
├──────────────────────────────────────┬───────────────────────┤
│    shared_buffers (~65% - 75%)       │  Remaining RAM        │
│   (OLTP tables, indexes, bufferpool) │  ├── OS / Backend     │
│                                      │  └── query_mem        │
└──────────────────────────────────────┴───────────────────────┘
```

Because `shared_buffers` dominates RAM, the memory budget available for large hash-joins and group-by aggregations is surprisingly tight.

### What happens when `query_mem` is exceeded?
1. **Disk Spilling:** DuckDB spills intermediate hash tables, sort buffers, and uncompressed columns onto the instance's **local storage**.
2. **I/O Contention:** The spilled temporary data shares the exact same physical NVMe SSD with the **on-disk read cache** and PostgreSQL temporary files.
3. **Thrashing & Query Aborts:** If local storage fills up or system-wide memory pressure triggers an Out-Of-Memory (OOM) threshold, Aurora cancels the query with an error or degrades severely in throughput.

### Verdict
> **"Foreign table queries are memory second-class citizens in an Aurora instance. Without careful `query_mem` sizing, heavy analytical queries will spill to disk and bottleneck your I/O."**

---

## 🔍 Experiment 4: 🛡️ Does Adding an Aurora Dedicated Reader Solve All Contention?

### The Natural Defense
"If analytical queries risk overwhelming our production writer, we will simply deploy an **Aurora Dedicated Reader instance** and configure custom endpoints!"

This is officially recommended by AWS. On a dedicated reader:
- You can tune `shared_buffers` down and increase `aurora_analytics.query_mem` up.
- You can assign promotion tier 15 (preventing it from ever becoming primary during failover).
- OLTP writes on the primary node are isolated from CPU spikes caused by DuckDB.

### The Hidden Trap: Dead Tuple Bloat & VACUUM Delay
Even on a dedicated reader, Aurora instances share the same underlying distributed storage (`Aurora Cluster Volume`) and PostgreSQL transaction catalog.

If a complex analytical query on your reader runs for 45 minutes:
1. The query holds open a transaction snapshot.
2. It sends feedback (`hot_standby_feedback`) to the primary writer instance.
3. The PostgreSQL autovacuum process on the writer **cannot remove dead tuples** that were deleted after the reader query's snapshot timestamp.
4. Result: Table bloat increases on your primary transactional database, index scans degrade, and disk consumption climbs.

### Mitigation
You must strictly enforce `statement_timeout` on the analytics reader and continuously monitor the CloudWatch metric `oldest_reader_feedback_xid_age`.

### Verdict
> **"A dedicated reader isolates CPU and memory, but long-running transactions can still silently cripple writer performance through PostgreSQL MVCC bloat."**

---

## 🔍 Experiment 5: 🚫 What Happens When You Try to Write or Use Complex Postgres Types?

Before redesigning your architecture around DuckDB foreign tables, you must test against the feature's strict boundaries:

### 1. Read-Only Enforcement
Foreign tables in Aurora Analytics are strictly read-only:
- `INSERT INTO ft_purchase_history ...` ❌ Fails.
- `UPDATE`, `DELETE`, `TRUNCATE`, and `COPY FROM` ❌ Fails.
If your application needs to write back or mutate lakehouse data, you still need Iceberg catalog tools, Spark, or Athena.

### 2. Missing Data Type Mappings
Not every PostgreSQL data type maps to Parquet/Iceberg primitives:
- `JSONB` / `JSON` ❌ Unsupported.
- `ARRAY`, `SERIAL` ❌ Unsupported.
- `UUID` (custom binary representations), `XML`, `INET`/`CIDR`, `RANGE`, `GEOMETRIC` ❌ Unsupported.
If your operational schema relies heavily on JSONB columns, you cannot easily project them into foreign tables without pre-flattening upstream.

### 3. Regional and Identifier Constraints
- **Single-Region Only:** A single query cannot federate S3 buckets across multiple AWS regions.
- **Identifier Limitations:** Column names derived from Parquet metadata cannot exceed 63 characters; case-insensitive collisions (e.g., `UserId` and `userid`) will cause schema initialization errors.
- **No System Columns:** System attributes like `ctid` and `xmin` are unavailable on foreign tables.

### Verdict
> **"Foreign tables are an analytical reading lens, not a general-purpose PostgreSQL storage replacement."**

---

## 🔍 Experiment 6: 💸 What Happens to the AWS Bill?

AWS documentation states that the `aurora_analytics` feature itself carries **no additional software license charge**. 

However, "zero feature cost" does not mean "zero infrastructure cost."

```text
Traditional ETL Model Costs:
  - AWS Glue Jobs / EMR Cluster time
  - Duplicate Aurora Storage ($0.10/GB-month)
  - Inter-service transfer

Direct DuckDB Query Model Costs:
  - Aurora ACU (Serverless v2) or Provisioned DB Instance Compute
  - Amazon S3 API Costs: `s3:GetObject` ($0.0004 per 1,000 requests) & `s3:ListBucket`
  - Aurora Local Storage consumption for disk spilling & cache
```

### The Cost Trap with Aurora Serverless v2
If dozens of users run ad-hoc lake queries simultaneously, Aurora Serverless v2 will rapidly scale up its ACUs (Aurora Capacity Units) to absorb the CPU/memory demand. Because ACU hourly rates are significantly higher than batch compute (like Spot-instance EMR or Glue Flex), unpredictable analytical queries can unexpectedly spike your Aurora invoice.

### Verdict
> **"Cost does not vanish; it shifts from batch processing line items (Glue/EMR) into database compute (Aurora ACUs) and S3 API request counts."**

---

## 📌 Summary — Architecture Thinking Framework

### The Common Pattern
Removing an ETL pipeline does not eliminate the work of data transformation, memory buffering, or filtering. It simply relocates that workload into the database engine.

When you bring the Data Lake into PostgreSQL:
- **Developer agility rises dramatically** (one query, zero sync lag).
- **Database reliability risk rises proportionally** (resource contention, disk spilling, snapshot holding).

### The Three-Question Framework

When deciding whether to adopt Aurora + DuckDB direct lake querying, ask:

**1. "What is the query shape and frequency?"**
- Point-lookups joining 100 active rows with filtered historical aggregates? 👉 **Adopt DuckDB Foreign Tables.**
- Heavy aggregate scans across 500 million rows multiple times per minute? 👉 **Keep a dedicated OLAP engine (Athena, Redshift, Snowflake, ClickHouse).**

**2. "Can our team tolerate schema strictness and read-only semantics?"**
- Does the lake schema change frequently? Are there unsupported types like `JSONB` or `ARRAY`? If yes, foreign tables will create schema maintenance friction.

**3. "Where is it cheaper to fail?"**
- An ETL pipeline failure results in stale analytics reports.
- An Aurora database failure results in an application outage.

### Decision Matrix

| Architectural Pattern | When It Thrives | Primary Trade-Off / Risk |
|---|---|---|
| **Traditional ETL Pipeline** | High-throughput, sub-second queries requiring pre-materialized tables in Aurora | High engineering overhead, data sync latency, duplicate storage |
| **Aurora + Embedded DuckDB** | Joining live transactional rows with cold S3 history; ad-hoc reporting; internal portals | Memory pressure on PostgreSQL, disk spilling, local cache invalidation on restart |
| **Dedicated OLAP (Athena / Redshift)** | Heavy multi-tenant analytics, complex BI dashboards, petabyte-scale scans | Multiple database endpoints, separate access credentials, higher architectural complexity |

### Closing Thought

A junior engineer looks at the new Aurora DuckDB release and says:

> **"Awesome! We can delete all our ETL pipelines and query everything directly from PostgreSQL."**

A staff architect looks at the exact same release and asks:

> **"If an un-indexed query scans a 500GB S3 table on our reader during peak Black Friday traffic, what happens to our primary instance's transaction logs and replication lag?"**

True architectural thinking begins when you look past the feature announcement and plan for the operational trade-offs underneath.
