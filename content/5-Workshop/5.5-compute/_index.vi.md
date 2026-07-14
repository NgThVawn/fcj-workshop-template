---
title : "EC2 và cân bằng tải"
date : 2024-01-01
weight : 5
chapter : false
pre : " <b> 5.5. </b> "
aliases:
  - /vi/5-workshop/5.5-policy/
---

#### Phạm vi triển khai

Sau khi S3 chứa artifact và các thành phần IAM Role, RDS, Secrets Manager sẵn sàng, lớp compute mới được triển khai. Trình tự gồm: tạo EC2 seed để kiểm tra runtime và quyền truy cập, tạo AMI/Launch Template, tạo Target Group và Application Load Balancer, sau đó tạo Auto Scaling Group trong private app subnet.

{{% notice info %}}
Auto Scaling Group được tạo sau Target Group và ALB vì ASG cần gắn Target Group ngay trong quá trình cấu hình. Thứ tự này giúp các EC2 instance mới tự động được đăng ký và health check khi khởi tạo.
{{% /notice %}}

#### Nội dung

1. [EC2 Seed](5.5.1-ec2-seed/)
2. [Launch Template](5.5.2-launch-template/)
3. [Target Group và ALB](5.5.3-alb/)
4. [Auto Scaling Group](5.5.4-auto-scaling/)

{{% notice info %}}
Các trang được sắp xếp theo quan hệ phụ thuộc. EC2 Seed xác nhận runtime và quyền truy cập trước khi tạo AMI; Launch Template được hoàn tất trước Auto Scaling Group; Target Group và ALB phải sẵn sàng trước khi ASG đăng ký instance.
{{% /notice %}}
