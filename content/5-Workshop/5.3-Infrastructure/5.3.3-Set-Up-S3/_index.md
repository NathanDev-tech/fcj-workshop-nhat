---
title : "Initialize and Configure S3"
date : 2024-01-01 
weight : 3
chapter : false
pre : " <b> 5.3.3 </b> "
---

### Objective

Our goal is to use Amazon S3 to store three types of RakiBookery data in Singapore (`ap-southeast-1`).

### What we need

Use the approved deployment identity from [section 5.3.1]({{< relref "/5-Workshop/5.3-Infrastructure/5.3.1-Set-Up-IAM/" >}}). When setting up a new environment, choose bucket names that are available when you create the buckets.

We use separate buckets because each type of file has a different use and needs different access. CloudFront reads static files through OAC. The image function reads uploads and writes thumbnails in the Media bucket. The Logs bucket stores logs from the sources that we turn on. Check the real bucket names in the project's Console before connecting any of these services.

### Deployment setup

1. In **S3 → General purpose buckets**, create `rakibookery-static-web-246111` for static files, `rakibookery-logs-246111` for logs, and `rakibookery-media-246111` for book covers, banners, and other media files.
2. Choose **General purpose**. Keep **ACLs disabled (Bucket owner enforced)** and **Block all public access** for all three buckets.

![Creating the Media bucket in Singapore](../../../images/5-Workshop/5.3-Infrastructure/5.3.3-Set-Up-S3/01-s3-media-creation.png)

![Static Web bucket name, Singapore Region, and ACLs disabled in the source report](../../../images/5-Workshop/5.3-Infrastructure/5.3.3-Set-Up-S3/02-s3-static-creation.png)

![Logs bucket name and ACLs disabled in the source report](../../../images/5-Workshop/5.3-Infrastructure/5.3.3-Set-Up-S3/03-s3-logs-creation.png)

3. Turn on **Bucket Versioning** for Media so earlier versions stay available when you replace an object.

![Block all public access and Versioning for the Media bucket](../../../images/5-Workshop/5.3-Infrastructure/5.3.3-Set-Up-S3/04-s3-media-versioning.png)

The bucket list from the CloudFront setup shows all three RakiBookery buckets in Singapore. Their names match the S3 report and the Media bucket used by the image function.

![The saved bucket list shows Static Web, Logs, and Media in Singapore](../../../images/5-Workshop/5.3-Infrastructure/5.3.3-Set-Up-S3/05-s3-project-bucket-list.png)

### Results and checks

The S3 report shows the creation forms for the three buckets and the Versioning option selected for Media. For the final check, review **Permissions** and **Properties**. After you connect the services, open a static file through CloudFront and check that uploaded images appear in Media. Check that logs appear in Logs after you set up the service that sends them.

The Lambda test document also shows uploaded files under `images/` in the Media bucket. This confirms that the bucket had stored objects at the time of the test.

![Uploaded image objects in the RakiBookery Media bucket](../../../images/5-Workshop/5.3-Infrastructure/5.3.3-Set-Up-S3/06-s3-media-uploaded-objects.png)

Use an approved identity to upload two test files in sequence with the same object key in the Media bucket. In **Objects**, turn on **Show versions** and check that S3 shows two version IDs. For a static file, check first that it exists in the bucket. Then request it through CloudFront after section 5.4.4. For the Logs bucket, check for a real log object after setting up the service that sends logs.


### Conclusion

We use each bucket for a different type of file. We keep public access blocked. With Versioning, we can recover an earlier version of a media file.