---
title : "Kiểm tra cuối"
date : 2024-01-01
weight : 6
chapter : false
pre : " <b> 5.8.6 </b> "
---

| Dịch vụ | Trạng thái đã ghi nhận |
|---|---|
| EC2 / Auto Scaling | Không còn `sportbooking-asg`, EC2 instance, Launch Template hoặc EBS volume ngoài kế hoạch lưu giữ. |
| Elastic Load Balancing | Không còn `sportbooking-alb` và `sportbooking-tg`. |
| RDS | Không còn DB instance và DB Subnet Group; snapshot/backup giữ lại đã được ghi nhận. |
| S3 | Artifact bucket đã xóa; `sportbooking-uploads-3stars-us-east-1` được giữ có chủ đích. |
| Secrets Manager | Secret ở trạng thái scheduled for deletion với ngày xóa xác định. |
| CloudWatch / SNS | Không còn alarm, subscription và topic của SportBooking; kiểm tra thêm log group nếu đã cấu hình CloudWatch Agent. |
| VPC | Không còn NAT Gateway, Elastic IP, VPC Endpoint, Security Group tùy chỉnh và `sportbooking-vpc`. |
| IAM | Không còn `sportbooking-ec2-role`; customer managed policy và instance profile của dự án đã được xử lý. |
| ACM / Route 53 | Certificate đã xóa; Alias record và CNAME validation không còn nếu không sử dụng; hosted zone chỉ được giữ khi domain vẫn hoạt động. |
| CloudFront | Không có distribution cần xóa vì bước tạo CloudFront đã dừng tại lỗi xác minh tài khoản. |

{{% notice info %}}
Việc dọn dẹp chỉ được xem là hoàn tất khi danh sách tài nguyên thực tế khớp với kế hoạch lưu giữ. Các snapshot, backup, S3 object, CloudWatch log group, Route 53 hosted zone và customer managed policy còn tồn tại phải có lý do và thời hạn lưu cụ thể.
{{% /notice %}}

Sau bước này, quy trình triển khai SportBooking đã hoàn tất từ xây dựng kiến trúc, triển khai dịch vụ, kiểm thử, vận hành đến thu hồi tài nguyên AWS.
