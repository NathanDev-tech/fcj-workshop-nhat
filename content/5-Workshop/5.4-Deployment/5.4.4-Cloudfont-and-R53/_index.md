---
title: "Setup CloudFront + Route 53"
date: 2024-01-01
weight: 4
chapter: false
pre: " <b> 5.4.4. </b> "
---

## Objective

Our goal is to use CloudFront for two kinds of content on RakiBookery. CloudFront can get images, CSS, and JavaScript from S3 and cache them. Requests to sign in, use the cart, or place an order go through the ALB to the PHP website on Fargate. We use Route 53 to add a custom domain if the team has a domain to manage.

## Prepare the two origins

First open the PHP website through the ALB as described in section 5.4.3. Prepare a static object with a known key in the private S3 bucket. Use the S3 REST origin so Origin Access Control (OAC) can sign requests to the bucket. The bucket name in our CloudFront screenshots belongs to that configuration. Check the actual bucket name before repeating the setup. [CloudFront deployment document](https://app.notion.com/p/CloudFront-Route-53-3f94cb9c2604806e8cb2ea5279aa1114?source=copy_link).

## Create the distribution and set the origins

In CloudFront, choose **Create distribution**. We use the name `RakiBookery-cloudfront` and choose **Single website configuration**. Then select the private S3 bucket as the first origin.

![Source screenshot showing the CloudFront distribution name and Single website configuration](../../../images/5-Workshop/5.4-Deployment/5.4.4-Cloudfont-and-R53/01-cloudfront-distribution-name.png)

**1. Use an S3 origin for static files.** The `RakiBookery-cloudfront` distribution uses `rakibookery-static-web-246111` as an S3 REST origin. We kept **Block Public Access** on and used **Origin Access Control (OAC)**. The bucket policy gives the CloudFront service principal `s3:GetObject` permission. The `AWS:SourceArn` condition limits this access to the distribution's ARN. [AWS guide to Origin Access Control](https://docs.aws.amazon.com/AmazonCloudFront/latest/DeveloperGuide/private-content-restricting-access-to-s3.html).

![The rakibookery-static-web-246111 bucket in the S3 console](../../../images/5-Workshop/5.4-Deployment/5.4.4-Cloudfont-and-R53/02-cloudfront-s3-bucket.png)

![S3 origin settings and the option to let CloudFront read the private bucket](../../../images/5-Workshop/5.4-Deployment/5.4.4-Cloudfont-and-R53/03-cloudfront-private-s3-origin.png)

![The Origins tab shows Origin Access Control for the S3 origin](../../../images/5-Workshop/5.4-Deployment/5.4.4-Cloudfont-and-R53/04-cloudfront-s3-oac.png)

**2. Use the ALB origin for the PHP website.** The `RakiBookery-ALB-Origin` origin points to the DNS name of `RakiBookery-alb` in Singapore. This setup uses **HTTP only** on port `80` because the listener is `HTTP:80`. The **Redirect HTTP to HTTPS** setting applies to requests from the browser to CloudFront. We set the protocol for the CloudFront-to-ALB connection separately.

![The ALB origin uses HTTP only on port 80](../../../images/5-Workshop/5.4-Deployment/5.4.4-Cloudfont-and-R53/05-cloudfront-alb-origin-http.png)

![RakiBookery-cloudfront has an S3 origin and an ALB origin](../../../images/5-Workshop/5.4-Deployment/5.4.4-Cloudfont-and-R53/06-cloudfront-two-origins.png)

**3. Set cache behaviors for the website URLs.** The default `*` behavior sends requests to the ALB and uses `CachingDisabled`. The source form allows all seven HTTP methods and uses **Redirect HTTP to HTTPS** for viewer requests. We check that these choices match the PHP routes we test. For static files, we use `/assets/*` to send requests to S3 with `CachingOptimized`. This path pattern must match the URL that the website uses and the object key in S3, such as `assets/logo.png`.

![Source screenshot showing the default ALB behavior, HTTPS redirect, and allowed HTTP methods](../../../images/5-Workshop/5.4-Deployment/5.4.4-Cloudfont-and-R53/07-cloudfront-dynamic-methods.png)

![The CachingDisabled policy for PHP requests to the ALB](../../../images/5-Workshop/5.4-Deployment/5.4.4-Cloudfont-and-R53/08-cloudfront-dynamic-no-cache.png)

![Create the /assets/* behavior for S3](../../../images/5-Workshop/5.4-Deployment/5.4.4-Cloudfont-and-R53/09-cloudfront-static-assets.png)

![The two behaviors and their cache policies after saving](../../../images/5-Workshop/5.4-Deployment/5.4.4-Cloudfont-and-R53/10-cloudfront-two-behaviors.png)

**4. Add a custom domain in Route 53.** If our team has a custom domain, we use a public hosted zone and add the domain to the distribution. Add an ACM certificate from `us-east-1` and create an Alias A record that points to CloudFront. If the distribution uses IPv6, also add an Alias AAAA record. We can use the default `cloudfront.net` domain to test the distribution. [AWS guide to routing from Route 53 to CloudFront](https://docs.aws.amazon.com/Route53/latest/DeveloperGuide/routing-to-cloudfront-distribution.html) · [AWS CloudFront certificate requirements](https://docs.aws.amazon.com/AmazonCloudFront/latest/DeveloperGuide/cnames-and-https-requirements.html).

For the dynamic behavior, `CachingDisabled` turns off caching. The source screenshot also selects `AllViewerExceptHostHeader` as the origin request policy. This forwards viewer cookies, query strings, and headers except the viewer's `Host` header. CloudFront sets the origin request's Host header to the origin domain name. Check that PHP receives the session data it needs. Allow the HTTP methods used by the app, including `POST` for forms. For `/assets/*`, check that the behavior points to S3 and that the path matches the object key. [AWS origin request policies](https://docs.aws.amazon.com/AmazonCloudFront/latest/DeveloperGuide/controlling-origin-requests.html) · [AWS managed origin request policies](https://docs.aws.amazon.com/AmazonCloudFront/latest/DeveloperGuide/using-managed-origin-request-policies.html).

![Bucket policy](../../../../images/5-Workshop/5.4-Deployment/5.4.4-Cloudfont-and-R53/11-s3-bucket-policy.png)

## Test the setup and record the results

After CloudFront deploys the distribution, our team tests a static file URL for an object that exists in S3. If the response is `403`, we check OAC, the bucket policy, and the object key. After the test in section 5.4.3, we try to open a PHP page through the CloudFront domain and test an action that uses a session, such as signing in or viewing the cart. After signing in, check the redirect URL and confirm that the browser still uses the CloudFront domain.

Record the results for static files, the PHP website, and the custom domain separately. Compare the HTTP status, the returned content, and the domain used after signing in. These checks help us find whether a problem is in S3 access, the cache behavior, or the PHP application.


## Conclusion

We configured CloudFront with an S3 origin for static files and an ALB origin for the PHP website. The cache behaviors separate static and dynamic requests. OAC controls access to the private S3 bucket, while `CachingDisabled` prevents CloudFront from caching PHP responses. A custom domain can be added through Route 53 when a domain and a valid ACM certificate are available.

