---
title : "Setup CloudFront + Route 53"
date : 2024-01-01
weight : 4
chapter : false
pre : " <b> 5.4.4 </b> "
---

# Setup CloudFront + Route 53

## Step 1 — Check S3 Static Web (AWS-003)

1. Go to **Amazon S3**.
2. Locate the bucket containing the static files for the RakiBookery website and verify that CSS, JS, and image files have been uploaded.
3. Keep **Block Public Access** ENABLED. There is no need to enable the *S3 Static Website Hosting* feature if you plan to use CloudFront with an S3 REST origin and OAC.

![Check S3 Static Web](your_image_url_1)

## Step 2 — Create CloudFront Distribution

1. Open the **AWS CloudFront Console**.
2. Click **Create distribution**.
3. If AWS prompts for a distribution type, choose the one intended for websites/web applications.
4. In the **Origin** section, select your S3 bucket `rakibookery-static-web-246111`.
5. Under **Origin access**, select **Origin access control (OAC)** or the recommended option to access a private S3 bucket.

**Important Configurations:**

| Field | Value |
| --- | --- |
| **Origin** | S3 Static Web bucket |
| **Origin access** | Origin Access Control (OAC) |
| **Viewer protocol policy** | Redirect HTTP to HTTPS |
| **Price class** | Choose according to your budget |
| **WAF** | Not required for this practice |

> *Note:* AWS may ask you to create a new OAC; if so, select **Create new OAC**.

![CloudFront Distribution Config](your_image_url_2)

## Step 3 — Grant CloudFront permission to read S3

After creating the Distribution, open the prompt guiding you to update the S3 Bucket Policy.
1. If AWS provides a **Copy policy** button, copy that policy.
2. Go to **S3** → Select your **Bucket** → **Permissions** → **Bucket policy** → **Edit**.
3. Paste the policy allowing the CloudFront Distribution to read objects using the `s3:GetObject` action.

> *Note:* The policy must strictly limit access by the specific Distribution ARN. Do not grant public read access to everyone.

## Step 4 — Add ALB as the second Origin

### 4.1. Add ALB Origin
1. Go to the AWS CloudFront Console.
2. Select the newly created `RakiBookery-cloudfront` Distribution.
3. Select the **Origins** tab.
4. Click **Create origin**.

![Add ALB Origin](your_image_url_3)

### 4.2. Fill in ALB Origin Configuration
On the Create origin screen, apply the following values:

| Field | Value |
| --- | --- |
| **Origin type** (if applicable) | Elastic Load Balancer |
| **Origin domain** | `RakiBookery-alb-1141293019.ap-southeast-1.elb.amazonaws.com` |
| **Origin name / ID** | RakiBookery-ALB-Origin |
| **Origin path** | *Leave blank* |
| **Protocol** | HTTP only |
| **HTTP port** | 80 |
| **HTTPS port** | *Keep default if not used* |
| **Origin Shield** | Disabled |
| **Connection attempts / timeout**| *Keep default* |

> **Why choose HTTP only?** Your ALB currently only has an `HTTP:80` listener and lacks `HTTPS:443`. If you select HTTPS only, CloudFront will fail to connect to the ALB with the current setup. This is acceptable for practice purposes. For real-world production deployments, HTTPS should be used between CloudFront and ALB.

![Detailed ALB Origin Config](your_image_url_4)

### 4.3. Create Origin
Click **Create origin**. After creation, the Origins tab should list two origins:

| Origin | Purpose |
| --- | --- |
| **S3 `rakibookery-static-web-246111`** | Serve static files (CSS, JS, images) |
| **RakiBookery-ALB-Origin** | PHP Website/Application on Fargate |


## Step 5 — Configure Cache Behaviors

Navigate to CloudFront → Distribution → **Behaviors**. Configure them based on the example below:

| Path pattern | Origin | Cache policy |
| --- | --- | --- |
| `/assets/*` | S3 Static Web | CachingOptimized |
| `Default (*)` | RakiBookery-alb | CachingDisabled |

> **Behavior Configuration Notes:**
> * For the default behavior routing to ALB, ensure you select the appropriate HTTP methods for a PHP application (e.g., `GET, HEAD, OPTIONS, PUT, POST, PATCH, DELETE` as needed).
> * Configure forwarding for cookies, query strings, and headers according to your app's requirements; using the managed origin request policy `AllViewer` is a good starting point.
> * **Important:** The path `/assets/*` is only correct if the actual URL structure of your website uses it and the corresponding S3 objects have matching keys. If the application uses `/css/*`, `/js/*`, or `/images/*`, you must create separate behaviors reflecting those actual paths.
> * Never apply static caching policies to dynamic HTML, login pages, or shopping carts.

![Create Cache Behavior](your_image_url_5)
![Configure S3 Cache Behavior](your_image_url_6)

## Step 6 — Test CloudFront

1. Wait for the Distribution status to transition to **Deployed**.
2. AWS will provide a domain resembling: `dxxxxxxxxxxxxx.cloudfront.net`
3. Try accessing an existing static file from S3, for example:
   `https://dxxxxxxxxxxxxx.cloudfront.net/assets/logo.png`
   *(Replace the domain and path with your actual values).*

If the file returns an `HTTP 200` status, CloudFront is successfully distributing static content. If you encounter a `403 AccessDenied` error, double-check your OAC configuration, S3 Bucket Policy, and object keys.

![Test CloudFront Access](your_image_url_7)

## Step 7 — Route 53 (Optional)

*Execute this step only if your team possesses a custom domain.*

1. Create or use an existing Hosted Zone in **Route 53**.
2. Request and configure an HTTPS certificate via **ACM** (it must be in the `us-east-1` region for CloudFront).
3. Attach your custom domain to the Distribution configuration.
4. Create a **DNS Alias A record** (you can also add an AAAA record for IPv6) pointing to your CloudFront distribution.

If your team does not have a custom domain, using the default `cloudfront.net` domain is perfectly sufficient for the optional segment of the AWS-007 lab.