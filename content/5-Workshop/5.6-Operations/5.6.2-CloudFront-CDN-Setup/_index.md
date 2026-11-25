---
title : "CloudFront + CDN Setup"
date : 2024-01-01
weight : 2
chapter : false
pre : " <b> 5.6.2 </b> "
---

### 5.6.2.1. Create a CloudFront Distribution

Open **CloudFront Console → Distributions** and select **Create distribution**, then fill in the following settings:

**Distribution name:** Enter `raki-prod-php-edge-cdn`.

**Description – optional:** Leave blank or enter `rakibookery-prod PHP on ALB, static on S3`.

**Distribution type:** Keep **Single website or app** selected.

**Domain (Route 53 managed domain – optional):** Leave blank because the project uses the default `*.cloudfront.net` URL provided by AWS.

![134447.png](/fcj-workshop-nhat/images/5-Workshop/5.6-Operations/5.6.2-CloudFront-CDN-Setup/134447.png)

### 5.6.2.2. Configure the S3 Origin and OAC

**Origin type:** Select **Amazon S3**.

**S3 origin:** Select the bucket `rakibookery-prod-static-98798946184613 ap-southeast-1.amazonaws.com`.

**Origin path – optional:** Leave blank; do not enter `/src/public` because the frontend is stored in the public directory of the bucket.

**Allow private S3 bucket access to CloudFront:** Keep **Allow private S3 bucket access to CloudFront – Recommended** selected. This is the **Origin Access Control (OAC)** feature, which allows CloudFront to read the private bucket and prevents users from accessing S3 directly.

**Origin settings:** Keep **Use recommended origin settings**.

**Cache settings:** Keep **Use recommended cache settings tailored to serving S3 content**.

![134701.png](/fcj-workshop-nhat/images/5-Workshop/5.6-Operations/5.6.2-CloudFront-CDN-Setup/134701.png)

### 5.6.2.3. Adjust the ALB Origin

After creation, go to **Distributions**, select the distribution just created, open the **Origins** tab, and edit the linked Elastic Load Balancing origin. Set *Protocol* to **HTTP only** to match the current ALB/API setup and avoid communication errors or `400 Bad Request` responses caused by a protocol mismatch.

![134711.png](/fcj-workshop-nhat/images/5-Workshop/5.6-Operations/5.6.2-CloudFront-CDN-Setup/134711.png)

![135129.png](/fcj-workshop-nhat/images/5-Workshop/5.6-Operations/5.6.2-CloudFront-CDN-Setup/135129.png)

Wait until the distribution status is **Enabled** and the update is complete, then open the actual deployed URL.
