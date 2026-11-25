---
title: "Setup CloudFront + Route 53"
date: 2024-01-01
weight: 4
chapter: false
pre: " <b> 5.4.4. </b> "
---

## Objective

Our team configured CloudFront for two kinds of RakiBookery content. Images, CSS, and JavaScript can be served from S3 and cached. Sign-in, cart, and order requests need to reach PHP on Fargate through the ALB. Route 53 adds a custom domain when the team has one to manage.

## Configuration from the team notes

**1. S3 origin for static files.** Distribution `RakiBookery-cloudfront` uses the S3 REST origin for `rakibookery-static-web-246111`. Keep **Block Public Access** enabled and use **Origin Access Control (OAC)**. The bucket policy grants `s3:GetObject` for the specified distribution ARN. [AWS OAC documentation](https://docs.aws.amazon.com/AmazonCloudFront/latest/DeveloperGuide/private-content-restricting-access-to-s3.html).

![The rakibookery-static-web-246111 bucket in the S3 console](../../../images/5-Workshop/5.4-Deployment/5.4.4-Cloudfont-and-R53/01-cloudfront-s3-bucket.png)

![S3 origin configuration and the option to allow CloudFront to access the private bucket](../../../images/5-Workshop/5.4-Deployment/5.4.4-Cloudfont-and-R53/02-cloudfront-private-s3-origin.png)

![The S3 origin shows an Origin Access Control in the Origins tab](../../../images/5-Workshop/5.4-Deployment/5.4.4-Cloudfont-and-R53/03-cloudfront-s3-oac.png)

**2. ALB origin for PHP.** Origin `RakiBookery-ALB-Origin` points to the DNS name of `RakiBookery-alb` in Singapore. The workshop notes specify **HTTP only** on port `80` because the recorded listener is `HTTP:80`. **Redirect HTTP to HTTPS** applies between the browser and CloudFront. Each connection has its own protocol settings.

![ALB origin using HTTP only on port 80](../../../images/5-Workshop/5.4-Deployment/5.4.4-Cloudfont-and-R53/04-cloudfront-alb-origin-http.png)

![RakiBookery-cloudfront with separate S3 and ALB origins](../../../images/5-Workshop/5.4-Deployment/5.4.4-Cloudfont-and-R53/05-cloudfront-two-origins.png)

**3. Behaviors based on application URLs.** The default `*` behavior routes to the ALB with `CachingDisabled`. Check that PHP receives the methods, cookies, query strings, and headers it uses. The notes specify `/assets/*` with `CachingOptimized` for S3 static files. The path pattern must match the website URL and the bucket object key, such as `assets/logo.png`.

![CachingDisabled policy for PHP requests to the ALB](../../../images/5-Workshop/5.4-Deployment/5.4.4-Cloudfont-and-R53/06-cloudfront-dynamic-no-cache.png)

![Creating the /assets/* behavior for S3](../../../images/5-Workshop/5.4-Deployment/5.4.4-Cloudfont-and-R53/07-cloudfront-static-assets.png)

![Both behaviors and their cache policies after saving](../../../images/5-Workshop/5.4-Deployment/5.4.4-Cloudfont-and-R53/08-cloudfront-two-behaviors.png)

**4. Optional Route 53 domain.** With a custom domain, use a public hosted zone, add the domain to the distribution, attach an ACM certificate in `us-east-1`, and create an A alias to CloudFront. Add an AAAA alias if IPv6 is enabled. The default `cloudfront.net` domain can serve for distribution tests. [AWS Route 53 and CloudFront documentation](https://docs.aws.amazon.com/Route53/latest/DeveloperGuide/routing-to-cloudfront-distribution.html) · [CloudFront certificate requirements](https://docs.aws.amazon.com/AmazonCloudFront/latest/DeveloperGuide/cnames-and-https-requirements.html).

## Verification and recorded results

Once the distribution is deployed, we request a static URL that matches an existing S3 object. For a `403`, we check OAC, the bucket policy, and the object key. After completing the ALB test in section 5.4.3, we open a PHP page through CloudFront and try a session-based action such as signing in or viewing the cart.

The CloudFront deployment notes document two origins and cache policies and describe the static file test. Record the static asset, dynamic website, and custom domain results individually, with a screenshot or HTTP response for each. The screenshots above document the origins and behaviors. The team notes do not include a Route 53 record or a successful browser test. [Team CloudFront notes](https://app.notion.com/p/CloudFront-Route-53-3f94cb9c2604806e8cb2ea5279aa1114?source=copy_link).
