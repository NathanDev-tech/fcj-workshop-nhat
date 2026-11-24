---
title : "Initialize and Configure S3"
date : 2024-01-01 
weight : 3
chapter : false
pre : " <b> 5.3.3 </b> "
---

### Objective

Work provisions Amazon S3 storage for three types of RakiBookery data in Singapore (`ap-southeast-1`).

### Prerequisites

Use the deployment identity reviewed in [section 5.3.1]({{< relref "/5-Workshop/5.3-Infrastructure/5.3.1-Set-Up-IAM/" >}}). When recreating the setup, use bucket names available in the selected namespace.

### Deployment configuration

1. In **S3 → General purpose buckets**, create `rakibookery-prod-static-987989461846` for static assets, `rakibookery-prod-logs-987989461846 ` for logs, and `rakibookery-prod-media-987989461846` for book covers, banners, and other media.
2. Select **General purpose**, **ACLs disabled (Bucket owner enforced)**, and **Block all public access** for all three buckets.

![Media bucket creation in Singapore](../../../images/5-Workshop/5.3-Infrastructure/s3-media-creation.png)

3. Enable **Bucket Versioning** for Media to retain earlier versions when objects are overwritten.

![Block all public access and Versioning selected for Media](../../../images/5-Workshop/5.3-Infrastructure/s3-media-versioning.png)

### Results and verification

The report documents the three bucket configurations and Media Versioning. Verify **Permissions** and **Properties** during acceptance. After integration, open a static file through CloudFront, verify image uploads to Media, and confirm that log objects appear after configuring the delivery service.

### Conclusion

The three buckets support separate data flows. Public access stays blocked and Versioning makes earlier media objects recoverable.