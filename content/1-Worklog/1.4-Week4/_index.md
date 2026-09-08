---
title: "Week 4 Worklog"
date: 2026-08-26
weight: 4
chapter: false
pre: " <b> 1.4. </b> "
---

### Week 4 objectives:

* Learn about the AWS Data Lake architecture and the roles of AWS services in data ingestion, storage, cataloging, processing and analysis.

### Tasks completed this week:

| Day | Tasks | Start date | Completion date | Reference Material |
|:---:|---|:---:|:---:|---|
| Wednesday | - Learned about the general architecture of a Data Lake on AWS.<br>&emsp;+ Learned the concept and purpose of a Data Lake.<br>&emsp;+ Identified the different types of data that can be stored in a Data Lake.<br>&emsp;+ Learned the data ingestion, storage, cataloging, processing and analysis workflow.<br>&emsp;+ Learned how Amazon S3 stores raw and processed data.<br>&emsp;+ Learned how AWS Glue Crawler scans data and detects schemas.<br>&emsp;+ Learned how AWS Glue Data Catalog manages metadata.<br>&emsp;+ Learned how Amazon Athena uses SQL to query data stored in Amazon S3.<br>&emsp;+ Studied the Ingest, Store, Catalog, Transform and Analyze layers.<br>&emsp;+ Learned how official AWS service icons are used in architecture diagrams.<br>&emsp;+ Created an S3 Data Lake architecture diagram using AWS service icons.<br>&emsp;+ Learned the basic methods for controlling costs when using Amazon S3, AWS Glue and Amazon Athena. | 26/08/2026 | 26/08/2026 | [AWS Cloud Journey](https://cloudjourney.awsstudygroup.com/)<br>[Amazon S3 User Guide](https://docs.aws.amazon.com/AmazonS3/latest/userguide/Welcome.html)<br>[Using AWS Glue Crawlers with Amazon Athena](https://docs.aws.amazon.com/athena/latest/ug/schema-crawlers.html) |
| Thursday | - Completed the AWS Data Lake on AWS workshop implementation.<br>&emsp;+ Reviewed the complete Data Lake workflow from data ingestion to data analysis.<br>&emsp;+ Practiced creating and managing Amazon S3 storage in a Data Lake architecture.<br>&emsp;+ Learned how Amazon Kinesis Data Firehose collects streaming data and delivers data into Amazon S3.<br>&emsp;+ Practiced configuring the data ingestion flow from Kinesis Data Firehose to Amazon S3.<br>&emsp;+ Learned how AWS Glue Crawler analyzes data stored in Amazon S3 and automatically discovers data schemas.<br>&emsp;+ Learned how AWS Glue Data Catalog manages metadata including tables, schemas and data locations.<br>&emsp;+ Used Amazon Athena to verify and query data stored in Amazon S3 using SQL.<br>&emsp;+ Understood the relationship between Amazon S3, Kinesis Data Firehose, AWS Glue and Amazon Athena in a Data Lake architecture.<br>&emsp;+ Reviewed all AWS resources created during the lab implementation.<br>&emsp;+ Performed cleanup and deleted all lab resources after completion to prevent unnecessary costs.<br>&emsp;+ Continuing with the AWS Certified Cloud Practitioner CLF-C02 course.<br>&emsp;+ Learned about AWS Global Infrastructure concepts including AWS Regions, Availability Zones and Edge Locations.<br>&emsp;+ Learned how AWS Global Infrastructure improves availability, scalability and reduces latency for cloud applications.<br>&emsp;+ Watched the AWS Certified Cloud Practitioner CLF-C02 Full Course by ExamPro to reinforce Cloud Practitioner knowledge and exam preparation. | 27/08/2026 | 27/08/2026 | [AWS Cloud Journey](https://cloudjourney.awsstudygroup.com/)<br>[Data Lake on AWS Workshop](https://000035.awsstudygroup.com/)<br>[AWS Certified Cloud Practitioner CLF-C02 Full Course - ExamPro](https://www.youtube.com/watch?v=7HKot-brXFE) |
| Friday | - Analyzed the Data Lake data created during the First Cloud Journey workshop using Amazon Athena SQL.<br>&emsp;+ Verified the AWS resource inventory and confirmed that the data pipeline created on Thursday was ready for analysis.<br>&emsp;+ Reviewed the relationship between Amazon S3, AWS Glue Data Catalog and Amazon Athena in the query workflow.<br>&emsp;+ Explored the Glue Catalog schema using Athena commands such as SHOW TABLES and DESCRIBE.<br>&emsp;+ Executed SQL queries to analyze data stored in Amazon S3 through Athena.<br>&emsp;+ Performed data quality checks including null value validation, duplicate UUID detection and activity type validation.<br>&emsp;+ Generated analytical results such as total events, unique devices, activity distribution and top tracks.<br>&emsp;+ Learned how Athena query cost depends on the amount of scanned data and how query optimization can reduce cost.<br>&emsp;+ Studied CTAS (CREATE TABLE AS SELECT) to create processed data in Parquet format for better query efficiency.<br>&emsp;+ Learned the difference between raw data and processed data in a Data Lake architecture.<br>&emsp;+ Performed AWS CloudTrail audit to understand how AWS records API activities and resource changes.<br>&emsp;+ Reviewed AWS Billing/Cost Management information after completing the lab.<br>&emsp;+ Cleaned up all AWS resources created during the workshop including Firehose, Glue resources, CloudFormation stack, IAM resources and S3 objects.<br>&emsp;+ Completed the Week 8 AWS Data Lake learning summary and documented the final results. | 28/08/2026 | 28/08/2026 | [Athena SQL, Audit, Cleanup and Summary Guide]<br>[AWS CloudTrail Documentation](https://docs.aws.amazon.com/awscloudtrail/latest/userguide/view-cloudtrail-events.html)<br>[Amazon Athena Documentation](https://docs.aws.amazon.com/athena/latest/ug/getting-started.html) |



### Week 4 learning outcomes:

&emsp;◉ Wednesday:
<br>&emsp;&emsp;○ Understood that a Data Lake is a centralized repository capable of storing structured, semi-structured and unstructured data.
<br>&emsp;&emsp;○ Understood the basic differences between raw data and processed data in a Data Lake architecture.
<br>&emsp;&emsp;○ Understood that Amazon S3 provides the central storage layer for raw data, processed data and query results.
<br>&emsp;&emsp;○ Understood that AWS Glue Crawler scans data files in Amazon S3, detects their structures and automatically creates table definitions.
<br>&emsp;&emsp;○ Understood that AWS Glue Data Catalog stores metadata such as table names, column names, data types, file formats and data locations.
<br>&emsp;&emsp;○ Understood that Glue Data Catalog manages data descriptions and does not directly store business data.
<br>&emsp;&emsp;○ Understood that Amazon Athena uses SQL to query data directly from Amazon S3 without requiring a database server.
<br>&emsp;&emsp;○ Understood the basic workflow: data is stored in Amazon S3, Glue Crawler detects the schema, Glue Data Catalog manages the metadata and Athena performs SQL queries.
<br>&emsp;&emsp;○ Learned how to read and explain a Data Lake diagram through the Ingest, Store, Catalog, Transform and Analyze layers.
<br>&emsp;&emsp;○ Learned how to use AWS service icons, connectors and functional areas to present a clear architecture diagram.
<br>&emsp;&emsp;○ Completed an S3 Data Lake architecture diagram using AWS service icons and English labels.
<br>&emsp;&emsp;○ Understood that Amazon Athena costs depend on the amount of data scanned and can be optimized by limiting the queried data.
<br>&emsp;&emsp;○ Understood that AWS Glue Crawler should only be run when the data structure needs to be updated to help control costs.

{{< figure src="images/Week8_Wednesday_S3_DataLake.png" title="Figure 1: AWS Data Lake Architecture with Amazon S3, AWS Glue and Amazon Athena" width="850" >}}


&emsp;◉ Thursday:
<br>&emsp;&emsp;○ Completed the AWS Data Lake on AWS workshop and understood the end-to-end workflow of building a simple Data Lake solution.
<br>&emsp;&emsp;○ Understood that Amazon S3 acts as the main storage layer for Data Lake architectures.
<br>&emsp;&emsp;○ Understood how Amazon Kinesis Data Firehose can be used to ingest streaming data into Amazon S3.
<br>&emsp;&emsp;○ Understood that AWS Glue Crawler discovers data structures and creates metadata definitions instead of storing the actual data.
<br>&emsp;&emsp;○ Understood that AWS Glue Data Catalog provides metadata management for analytics services.
<br>&emsp;&emsp;○ Understood that Amazon Athena can query data directly from Amazon S3 without managing database servers.
<br>&emsp;&emsp;○ Learned how different AWS services integrate together to create a complete Data Lake architecture.
<br>&emsp;&emsp;○ Successfully reviewed and removed all AWS resources created during the lab to prevent unnecessary costs.
<br>&emsp;&emsp;○ Continuing with the AWS Cloud Practitioner CLF-C02 preparation materials.
<br>&emsp;&emsp;○ Understood the basic concepts of AWS Global Infrastructure including Regions, Availability Zones and Edge Locations.
<br>&emsp;&emsp;○ Learned how AWS designs its global infrastructure to provide high availability, fault tolerance and low-latency access for applications.
<br>&emsp;&emsp;○ Improved understanding of how AWS infrastructure concepts support the design of reliable cloud solutions.

&emsp;◉ Friday:
<br>&emsp;&emsp;○ Successfully analyzed Data Lake data using Amazon Athena SQL after completing the AWS Data Lake workshop.
<br>&emsp;&emsp;○ Understood how Athena uses AWS Glue Data Catalog metadata to query data stored in Amazon S3.
<br>&emsp;&emsp;○ Learned how to explore table structures and schemas before writing SQL queries.
<br>&emsp;&emsp;○ Performed SQL-based data analysis including event counting, activity distribution analysis and identifying top tracks.
<br>&emsp;&emsp;○ Learned basic data quality validation techniques including checking null values, duplicate records and invalid categories.
<br>&emsp;&emsp;○ Understood that Athena query cost depends on scanned data volume and learned basic optimization methods.
<br>&emsp;&emsp;○ Learned how CTAS can be used to create processed datasets in Parquet format for better analytical performance.
<br>&emsp;&emsp;○ Understood the difference between raw data and processed data layers in a Data Lake architecture.
<br>&emsp;&emsp;○ Learned how CloudTrail can be used to audit AWS API activities and track resource changes.
<br>&emsp;&emsp;○ Reviewed AWS costs after the lab and successfully cleaned up all created resources.
<br>&emsp;&emsp;○ Completed the Week 8 summary and improved understanding of the complete AWS Data Lake workflow from ingestion to analysis.
