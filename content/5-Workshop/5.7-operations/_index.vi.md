---
title : "Vận hành"
date : 2024-01-01
weight : 7
chapter : false
pre : " <b> 5.7. </b> "
aliases:
  - /vi/5-workshop/5.7-operation/
---

#### Phạm vi triển khai

Phần này trình bày quy trình vận hành sau khi website đã truy cập được qua HTTPS: cập nhật artifact, cập nhật secret, kiểm tra database, cấu hình OAuth, tạo kênh cảnh báo bằng SNS, tạo CloudWatch alarm và ghi nhận các lỗi thường gặp.

#### Nội dung

1. [Cập nhật ứng dụng](5.7.1-updates/)
2. [Cảnh báo SNS](5.7.2-sns/)
3. [Giám sát CloudWatch](5.7.3-cloudwatch/)
4. [Xử lý sự cố](5.7.4-troubleshooting/)

{{% notice info %}}
Artifact và secret được cập nhật trước khi khởi chạy Instance Refresh. Kênh SNS phải ở trạng thái confirmed trước khi gắn vào CloudWatch alarm.
{{% /notice %}}
