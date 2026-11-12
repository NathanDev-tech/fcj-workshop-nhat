---
title : "Các bước chuẩn bị"
date : 2024-01-01 
weight : 2
chapter : false
pre : " <b> 5.2. </b> "
---

#### IAM permissions
Để triển khai và dọn dẹp tài nguyên trong workshop, tài khoản AWS cần có quyền IAM phù hợp.

Trong dự án RakiBookery, chúng ta sử dụng IAM User dành cho tất cả thành viên trong nhóm, thuộc IAM Group RakiBookery. Tài khoản đã bật MFA và được gắn chính sách AdministratorAccess thông qua IAM Group; ngoài ra, policy IAMUserChangePassword được gắn trực tiếp vào User. Vì vậy, nhóm không cần gắn thêm policy mẫu của bài lab gốc.

Chính sách AdministratorAccess đang sử dụng có nội dung như sau:
```
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": "*",
      "Resource": "*"
    }
  ]
}
```

#### Khởi tạo tài nguyên bằng CloudFormation

Trong lab này, chúng ta sử dụng Asia Pacific (Singapore) region (ap-southeast-1).

Để chuẩn bị cho môi trường làm workshop, chúng ta deploy CloudFormation template sau (click link): [PrivateLinkWorkshop ](https://us-east-1.console.aws.amazon.com/cloudformation/home?region=us-east-1#/stacks/quickcreate?templateURL=https://s3.us-east-1.amazonaws.com/reinvent-endpoints-builders-session/Nested.yaml&stackName=PLCloudSetup). Để nguyên các lựa chọn mặc định.

![create stack](/fcj-workshop-nhat/images/5-Workshop/5.2-Prerequisite/cloudfomation_stack1.png)

+ Đặt tên cloudformation: RakiBookery-5-2-S3-Gateway
+ Chọn Create stack

![create stack](/fcj-workshop-nhat/images/5-Workshop/5.2-Prerequisite/cloudfomation_stack2.png)

Quá trình triển khai CloudFormation cần khoảng 15 phút để hoàn thành.

![complete](/fcj-workshop-nhat/images/5-Workshop/5.2-Prerequisite/cloudfomation_success.png)

+ VPCs đã được tạo

![vpcs](/fcj-workshop-nhat/images/5-Workshop/5.2-Prerequisite/vpc.png)

+ 3 EC2s đã được tạo

![EC2](/fcj-workshop-nhat/images/5-Workshop/5.2-Prerequisite/ec2.png)