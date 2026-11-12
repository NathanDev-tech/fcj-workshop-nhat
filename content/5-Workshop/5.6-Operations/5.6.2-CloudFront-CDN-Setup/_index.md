---
title : "CloudFront + CDN Setup"
date : 2024-01-01
weight : 2
chapter : false
pre : " <b> 5.6.2 </b> "
---

### 5.5.2.1. Create a CloudFront Distribution

Open the CloudFront console, go to **Distributions**, and select **Create distribution**. Then configure the following settings:

**Distribution name:** Enter `raki-frontend-cdn`.

**Description – optional:** Leave this blank or enter `CloudFront CDN for RakiBookery Frontend and API`.

**Distribution type:** Keep **Single website or app** selected.

**Domain (Route 53 managed domain – optional):** Leave this blank because the project uses the default `*.cloudfront.net` URL provided by AWS.

![image011.png](/images/5-Workshop/5.5-Raki-Operations/image011.png)

### 5.5.2.3. Configure the S3 Origin and OAC

**Origin type:** Select **Amazon S3**.

**S3 origin:** Select the `raki-frontend-dev.s3.ap-southeast-1.amazonaws.com` bucket.

**Origin path – optional:** Leave this blank; do not enter `/path`, because the frontend is stored in the bucket root.

**Allow private S3 bucket access to CloudFront:** Keep **Allow private S3 bucket access to CloudFront – Recommended** selected. This enables **Origin Access Control (OAC)**, allowing CloudFront to read the private bucket while preventing direct user access to S3.

**Origin settings:** Keep **Use recommended origin settings**.

**Cache settings:** Keep **Use recommended cache settings tailored to serving S3 content**.

![image013.png](/images/5-Workshop/5.5-Raki-Operations/image013.png)

### 5.5.2.4. Adjust the ALB Origin

After creating the distribution, open **Distributions**, select the newly created distribution, open the **Origins** tab, and edit the linked Elastic Load Balancing origin. Set *Protocol* to **HTTP only** to match the current ALB/API configuration and avoid communication issues or `400 Bad Request` responses caused by a protocol mismatch.

![image015.png](/images/5-Workshop/5.5-Raki-Operations/image015.png)

Wait until the distribution status is **Enabled** and deployment is complete, then open the deployed URL.

![image017.png](/images/5-Workshop/5.5-Raki-Operations/image017.png)