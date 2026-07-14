---
title : "Dữ liệu và phân quyền"
date : 2024-01-01
weight : 4
chapter : false
pre : " <b> 5.4. </b> "
aliases:
  - /vi/5-workshop/5.4-s3-onprem/
---

#### Phạm vi triển khai

Khởi tạo các dịch vụ nền trước khi triển khai EC2: tạo S3 bucket và upload artifact, tạo S3 Gateway Endpoint, triển khai RDS MySQL Multi-AZ, tạo Secrets Manager secret chứa cấu hình ứng dụng, sau đó cấp IAM Role cho EC2 trên các tài nguyên đã tồn tại.

#### Thứ tự thực hiện

1. [Tạo S3 bucket, upload artifact và cấu hình Gateway Endpoint](5.4.1-s3/)
2. [Tạo RDS MySQL private](5.4.2-rds/)
3. [Tạo Secrets Manager cho cấu hình ứng dụng](5.4.3-secrets/)
4. [Tạo IAM Role cho EC2](5.4.4-iam/)

{{% notice info %}}
Secret được tạo sau RDS để sử dụng đúng endpoint database trong `DB_URL`. IAM Role được tạo sau S3 và secret để policy tham chiếu các ARN đã tồn tại. Launch Template và EC2 chỉ được tạo khi artifact, RDS, secret và role đều sẵn sàng.
{{% /notice %}}

#### Kết quả đã ghi nhận

- S3 artifact bucket chứa jar trong prefix `releases/` và không public.
- S3 upload bucket dùng cho ảnh sân bóng, bật Block Public Access.
- S3 Gateway Endpoint được gắn với route table của private app subnet.
- IAM Role `sportbooking-ec2-role` có quyền SSM, đọc artifact S3, đọc/ghi upload S3 và đọc secret.
- RDS MySQL nằm trong private DB subnet, không public Internet.
- Secret `sportbooking/prod/app-env` chứa đủ cấu hình để EC2 tạo `/opt/sportbooking/app.env`.


