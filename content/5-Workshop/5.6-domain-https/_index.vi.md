---
title : "Tên miền và HTTPS"
date : 2024-01-01
weight : 6
chapter : false
pre : " <b> 5.6. </b> "
aliases:
  - /vi/5-workshop/5.6-cleanup/
---

#### Phạm vi triển khai

Cấu hình domain `sport.younglilliu.id.vn`, cấp certificate bằng ACM, gắn HTTPS listener vào ALB và kiểm thử website sau triển khai.

#### Nội dung

1. [Route 53](5.6.1-route53/)
2. [ACM và HTTPS](5.6.2-acm-https/)
3. [Kiểm thử truy cập](5.6.3-testing/)
4. [Khảo sát CloudFront](5.6.4-cloudfront/)

{{% notice info %}}
Hosted Zone và Alias record được hoàn tất trước khi yêu cầu certificate. HTTPS listener chỉ được tạo sau khi ACM xác thực domain; kiểm thử DNS và HTTPS được thực hiện sau cùng.
{{% /notice %}}
