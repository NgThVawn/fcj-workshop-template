---
title : "Tổng quan kiến trúc"
date : 2024-01-01
weight : 1
chapter : false
pre : " <b> 5.1. </b> "
---

#### Kiến trúc dự án

Kiến trúc mục tiêu của SportBooking được thiết kế theo mô hình nhiều lớp trên AWS, phân tách rõ lớp truy cập, lớp ứng dụng, lớp dữ liệu và lớp giám sát. Hệ thống sử dụng hai Availability Zone để tăng khả năng sẵn sàng cho EC2 và RDS.

![Kiến trúc mục tiêu của hệ thống SportBooking trên AWS](/images/5-Workshop/5.1-Workshop-overview/sportbooking-architecture.png)

<p class="image-caption"><em>Hình 1: Kiến trúc mục tiêu của hệ thống SportBooking trên AWS</em></p>

#### Luồng xử lý trên sơ đồ

1. Người dùng gửi yêu cầu đến domain `sport.younglilliu.id.vn`.
2. Lớp DNS và biên tiếp nhận yêu cầu trước khi chuyển lưu lượng vào Application Load Balancer. Trong phạm vi đã triển khai, Route 53 trỏ trực tiếp đến ALB; CloudFront và AWS WAF được giữ ở mức kiến trúc mở rộng.
3. Nhánh S3 tĩnh trên sơ đồ mô tả phương án tách frontend thành static website khi mở rộng. Bản triển khai hiện tại sử dụng S3 cho artifact và ảnh upload của ứng dụng.
4. ALB chuyển request đến Target Group trên port `8080` và chỉ phân phối lưu lượng đến target vượt qua health check.
5. Auto Scaling Group duy trì các EC2 instance tại private app subnet thuộc hai Availability Zone.
6. Ứng dụng kết nối đến RDS MySQL Multi-AZ thông qua endpoint duy nhất; standby được AWS quản lý và không phục vụ truy vấn trực tiếp.
7. EC2 truy cập S3 qua S3 Gateway VPC Endpoint để lấy artifact và đọc/ghi ảnh mà không cần đưa lưu lượng S3 qua Internet.
8. CloudWatch thu thập metric vận hành; alarm gửi sự kiện đến SNS và SNS chuyển thông báo đến email đã xác nhận.

#### Kết quả triển khai đã ghi nhận

- Website truy cập qua domain `https://sport.younglilliu.id.vn`.
- EC2 chạy ứng dụng Spring Boot trong private subnet, không gắn public IP.
- RDS MySQL Multi-AZ nằm trong private DB subnet và chỉ nhận kết nối từ EC2 app.
- File jar release và ảnh upload lưu trong S3, không public bucket trực tiếp.
- Cấu hình nhạy cảm được đọc từ Secrets Manager bằng IAM Role.
- ALB xử lý HTTPS, redirect HTTP sang HTTPS và kiểm tra health check `/actuator/health`.

{{% notice info %}}
Sơ đồ thể hiện kiến trúc mục tiêu đầy đủ. CloudFront, AWS WAF và frontend static S3 chưa nằm trong đường truy cập đã kiểm thử của báo cáo; đường truy cập thực tế là Route 53 → ALB → Target Group → EC2. IAM Role, Secrets Manager và ACM đã được triển khai nhưng được lược khỏi sơ đồ để giữ luồng chính dễ theo dõi.
{{% /notice %}}

#### Thông tin tài nguyên

| Tài nguyên | Giá trị trong bài triển khai |
|---|---|
| Region | `us-east-1` |
| Domain triển khai thử nghiệm | `https://sport.younglilliu.id.vn` |
| ALB DNS | `sportbooking-alb-1198360694.us-east-1.elb.amazonaws.com` |
| Artifact bucket | `sportbooking-artifacts-3stars-us-east-1` |
| Artifact object | `releases/SportBooingSystem-0.0.1-SNAPSHOT.jar` |
| Upload bucket | `sportbooking-uploads-3stars-us-east-1` |
| Secret ID | `sportbooking/prod/app-env` |
| RDS endpoint | `sportbooking-mysql.cudgu8qk8z19.us-east-1.rds.amazonaws.com` |
| Database | `sport_booking` |
| App port | `8080` |
| Health check path | `/actuator/health` |

#### Dịch vụ AWS sử dụng

| Dịch vụ | Mục đích |
|---|---|
| Amazon VPC | Chia public subnet, private app subnet và private DB subnet. |
| NAT Gateway | Cho EC2 private outbound Internet khi cần tải package hoặc gọi dịch vụ AWS chưa có endpoint. |
| Security Group | Giới hạn traffic theo nguyên tắc least privilege. |
| IAM Role | Cho EC2 gọi AWS API mà không lưu access key trên máy. |
| Amazon S3 | Lưu jar release và file upload của website. |
| Secrets Manager | Tách cấu hình nhạy cảm khỏi source code và jar. |
| Amazon RDS for MySQL | Cung cấp database managed có backup/snapshot. |
| EC2 Launch Template | Định nghĩa AMI, role, security group và User Data bootstrap. |
| Auto Scaling Group | Duy trì số lượng EC2 và hỗ trợ Instance Refresh khi deploy jar mới. |
| Target Group | Health check EC2 app trước khi nhận traffic. |
| Application Load Balancer | Nhận request public, terminate TLS và forward vào app. |
| Route 53 + ACM | Cấu hình domain và HTTPS. |
| Systems Manager Session Manager | Truy cập shell vào private EC2 mà không mở SSH. |
| Amazon CloudWatch | Quan sát metric/log và tạo alarm cho hệ thống. |
| Amazon SNS | Gửi thông báo khi alarm CloudWatch chuyển trạng thái. |
| Amazon CloudFront + AWS WAF | Kiến trúc mở rộng đã được đánh giá, chưa kích hoạt trong đường truy cập chính. |


