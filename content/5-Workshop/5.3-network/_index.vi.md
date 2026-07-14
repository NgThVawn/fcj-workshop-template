---
title : "Mạng và bảo mật"
date : 2024-01-01
weight : 3
chapter : false
pre : " <b> 5.3. </b> "
aliases:
  - /vi/5-workshop/5.3-s3-vpc/
---

#### Phạm vi triển khai

Xây dựng lớp mạng cho SportBooking gồm VPC, subnet, Internet Gateway, NAT Gateway, route table và Security Group. Các tài nguyên mạng được tạo trước storage, database và compute để những dịch vụ ở các phần sau có sẵn VPC, subnet và rule bảo mật để sử dụng.

#### Nội dung

- [Tạo VPC, subnet và route table](5.3.1-vpc-routing/)
- [Cấu hình Security Group](5.3.2-security-groups/)

#### Mô hình mạng

| Thành phần | Cấu hình |
|---|---|
| VPC | `sportbooking-vpc`, CIDR `10.20.0.0/16`. |
| Availability Zones | 2 AZ để phân tán ALB, EC2 và RDS. |
| Public subnet | 2 subnet public cho ALB và NAT Gateway. |
| Private app subnet | 2 subnet private cho EC2 app trong Auto Scaling Group. |
| Private DB subnet | 2 subnet private cho RDS DB subnet group. |
| NAT Gateway | 2 NAT Gateway, mỗi AZ một tài nguyên. |

Kết quả đã ghi nhận: ALB là thành phần public duy nhất nhận request từ Internet; EC2 app và RDS không được public.


