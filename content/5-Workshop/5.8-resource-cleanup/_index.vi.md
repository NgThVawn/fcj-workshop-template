---
title : "Dọn dẹp"
date : 2024-01-01
weight : 8
chapter : false
pre : " <b> 5.8. </b> "
aliases:
  - /vi/5-workshop/5.8-cleanup/
---

#### Phạm vi dọn dẹp

Sau khi hoàn tất triển khai, kiểm thử và lưu đầy đủ minh chứng, thu hồi tài nguyên của SportBooking theo đúng quan hệ phụ thuộc. Artifact bucket và lớp compute/network được xóa; upload bucket, final RDS snapshot và retained automated backup được giữ lại có chủ đích để bảo toàn dữ liệu cần phục hồi.

{{% notice warning %}}
Các thao tác xóa Auto Scaling Group, RDS, S3, VPC và IAM Role có thể làm mất dữ liệu hoặc gián đoạn hoàn toàn website. Chỉ thực hiện sau khi đã sao lưu dữ liệu cần thiết, hoàn tất báo cáo và xác nhận không còn sử dụng môi trường.
{{% /notice %}}

#### Thứ tự dọn dẹp

| Giai đoạn | Tài nguyên | Quan hệ phụ thuộc |
|---|---|---|
| 1 | Route 53, CloudWatch và SNS | Ngừng điều hướng người dùng; xóa alarm trước SNS topic. |
| 2 | ASG, ALB, Target Group và Launch Template | ASG được xóa trước Launch Template; ALB được xóa trước Target Group. |
| 3 | IAM, RDS, S3, Secrets Manager và ACM | EC2 đã dừng; ACM không còn được ALB sử dụng; CloudFront distribution chưa từng được tạo. |
| 4 | NAT Gateway, Elastic IP và VPC Endpoint | NAT Gateway đã chuyển sang `Deleted` trước khi Elastic IP được giải phóng. |
| 5 | Security Group, DB Subnet Group và VPC | ALB, EC2, RDS, endpoint và các tham chiếu Security Group đã được xóa. |
| 6 | Kiểm tra phần còn lại | Không còn snapshot, backup, log group, hosted zone hoặc policy ngoài kế hoạch lưu giữ. |

{{% notice info %}}
Trong toàn bộ quy trình, cần kiểm tra đúng AWS account và Region `us-east-1` trước mỗi thao tác. Chỉ chọn tài nguyên mang tên dự án SportBooking; không xóa tài nguyên mặc định hoặc tài nguyên dùng chung.
{{% /notice %}}

#### Chuẩn bị trước khi xóa

1. Xác định final RDS snapshot, retained automated backup và dữ liệu trong upload bucket cần giữ lại.
2. Ngừng Alias record `sport.younglilliu.id.vn` trỏ đến ALB trước khi xóa lớp cân bằng tải.
3. Xác nhận CloudFront distribution chưa được tạo, nên không có phụ thuộc CloudFront cần tháo gỡ.
4. Giữ public hosted zone vì domain vẫn tiếp tục được quản lý; chỉ các record không còn sử dụng được loại bỏ.
5. Ghi nhận rõ các tài nguyên lưu trữ được giữ lại để phân biệt với tài nguyên bị bỏ sót.

#### Nội dung

1. [Giám sát](5.8.1-monitoring/)
2. [Ứng dụng và cân bằng tải](5.8.2-application/)
3. [Dữ liệu và cấu hình](5.8.3-data/)
4. [NAT Gateway và VPC Endpoint](5.8.4-nat-endpoint/)
5. [Security Group và VPC](5.8.5-security-vpc/)
6. [Kiểm tra cuối](5.8.6-verification/)
