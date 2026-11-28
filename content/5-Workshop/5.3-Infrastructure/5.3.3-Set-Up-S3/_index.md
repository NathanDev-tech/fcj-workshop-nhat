---
title : "Initialize and Configure S3"
date : 2024-01-01 
weight : 3
chapter : false
pre : " <b> 5.3.3 </b> "
---

### Objective

We set up Amazon S3 in Singapore (`ap-southeast-1`) to store static files, images, and logs for RakiBookery. We use three separate buckets so each type of data can have the access it needs.

## Architecture overview

- **Media** (`rakibookery-prod-media-987989461846`) stores bookstore images. The bucket uses Versioning to keep earlier versions when we replace an object.
- **Static** (`rakibookery-prod-static-987989461846`) stores static files for the website. CloudFront can read the files through Origin Access Control (OAC) and a bucket policy. The origin setup is in [section 5.4.4]({{< relref "/5-Workshop/5.4-Deployment/5.4.4-Cloudfont-and-R53/" >}}).
- **Logs** (`rakibookery-prod-logs-987989461846`) is for log files. Files appear after we configure the service that sends the logs and give it permission to write.

## 1. Create the buckets

Use a deployment identity with the permissions described in [section 5.3.1]({{< relref "/5-Workshop/5.3-Infrastructure/5.3.1-Set-Up-IAM/" >}}). In **Amazon S3 → General purpose buckets → Create bucket**, select **Asia Pacific (Singapore)**, **General purpose**, and **Global namespace** as shown in the screenshots. Enter each bucket name. If you repeat the setup, use a name that is available in the selected namespace. [AWS guide to bucket naming rules](https://docs.aws.amazon.com/AmazonS3/latest/userguide/bucketnamingrules.html)

Under **Object Ownership**, keep **ACLs disabled (Bucket owner enforced)** so access is managed through policies. Keep **Block all public access** on to block direct public access. CloudFront can read the Static bucket through the bucket policy after integration. [AWS guide to Object Ownership](https://docs.aws.amazon.com/AmazonS3/latest/userguide/about-object-ownership.html)

### Media bucket

Enter `rakibookery-prod-media-987989461846`. The creation screen shows the Singapore Region and **ACLs disabled**. Before saving the Media bucket, select **Bucket Versioning → Enable** as shown in step 2.

![Creating the Media bucket in Singapore with ACLs disabled](../../../images/5-Workshop/5.3-Infrastructure/5.3.3-Set-Up-S3/01-s3-media-creation.png)

### Static bucket

Enter `rakibookery-prod-static-987989461846` and keep **ACLs disabled**. After creating the bucket, use this name when checking the CloudFront S3 origin.

![Static bucket name and Object Ownership on the Create bucket screen](../../../images/5-Workshop/5.3-Infrastructure/5.3.3-Set-Up-S3/02-s3-static-creation.png)

### Logs bucket

Enter `rakibookery-prod-logs-987989461846` and keep **ACLs disabled** as shown. This bucket is the storage destination. Set up log delivery in the service that creates the logs.

![Logs bucket name and Object Ownership on the Create bucket screen](../../../images/5-Workshop/5.3-Infrastructure/5.3.3-Set-Up-S3/03-s3-logs-creation.png)

## 2. Enable and check Versioning for Media

On the Media bucket creation screen, **Block all public access** is selected and **Bucket Versioning** is set to **Enable**.

![Block all public access and Enable Versioning selected when creating the Media bucket](../../../images/5-Workshop/5.3-Infrastructure/5.3.3-Set-Up-S3/04-s3-media-versioning.png)

## 3. Check the result

The **General purpose buckets** list shows all three buckets, Media, Static, and Logs, in `ap-southeast-1`.

![Media, Static, and Logs buckets listed in Singapore](../../../images/5-Workshop/5.3-Infrastructure/5.3.3-Set-Up-S3/05-s3-project-bucket-list.png)

After creating the bucket, we open **Media → Properties → Bucket Versioning**. The console shows **Enabled**, which confirms the setting was saved. When we upload new content with the same object key, S3 creates a new version and keeps the earlier one. [AWS guide to S3 Versioning](https://docs.aws.amazon.com/AmazonS3/latest/userguide/Versioning.html)

![Media bucket Properties showing Bucket Versioning as Enabled](../../../images/5-Workshop/5.3-Infrastructure/5.3.3-Set-Up-S3/06-s3-media-versioning-enabled.png)

- **Media.** To check Versioning, upload two files with different content to the same test key, such as `versioning-check/sample.txt`. Turn on **Show versions** in **Objects** and check for two version IDs. See [section 5.5.2]({{< relref "/5-Workshop/5.5-Serverless/5.5.2-Lambda-Functions/" >}}) for the image processing trigger.
- **Static.** After connecting the origin, open a static file through the CloudFront URL to check read access.
- **Logs.** After setting up log delivery, open the Logs bucket and look for an object under the selected prefix.

## Conclusion

We created the three RakiBookery S3 buckets in Singapore. The **Properties** screen confirms that Versioning is on for Media. Each bucket stores a different type of data and supports the setup for CloudFront, image processing, or log delivery.
