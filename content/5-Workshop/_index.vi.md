---
title: "Workshop"
date: 2024-01-01
weight: 5
chapter: false
pre: " <b> 5. </b> "
---

# Triển khai website đặt sân bóng đá SportBooking lên AWS

### [Video demo các chức năng của website](<https://drive.google.com/file/d/1-0sFCHzJM-VBLlGwRa8S51IZn340tMVN/view?usp=drive_link>)

#### Phạm vi báo cáo

Báo cáo này trình bày quá trình đã triển khai ứng dụng **SportBookingSystem** lên AWS theo mô hình production cơ bản. Website được công bố qua domain HTTPS; lưu lượng đi qua Route 53 và Application Load Balancer; ứng dụng Spring Boot chạy trên EC2 trong private subnet; dữ liệu nghiệp vụ lưu tại RDS MySQL; artifact và ảnh upload lưu trong S3; cấu hình nhạy cảm được quản lý bằng Secrets Manager.

Nội dung được sắp xếp theo thứ tự tài nguyên đã được khởi tạo: chuẩn bị source code; xây dựng mạng và bảo mật; tạo S3 và tải artifact; tạo RDS; lưu cấu hình trong Secrets Manager; cấp quyền IAM; triển khai EC2, ALB và Auto Scaling Group; cấu hình DNS/HTTPS; thiết lập giám sát; cuối cùng thu hồi tài nguyên.

{{% notice warning %}}
Các ảnh minh chứng không hiển thị DB password, OAuth client secret, token, access key hoặc secret value. Báo cáo chỉ giữ lại tên tài nguyên, Region, trạng thái, port, nguồn Security Group và các cấu hình không nhạy cảm.
{{% /notice %}}

#### Kiến trúc triển khai

| Thành phần | Vai trò |
|---|---|
| Route 53 | Quản lý DNS cho `sport.younglilliu.id.vn` và trỏ record Alias về ALB. |
| AWS Certificate Manager | Cấp chứng chỉ TLS để bật HTTPS trên ALB. |
| Application Load Balancer | Nhận HTTP/HTTPS public, redirect HTTP sang HTTPS và forward request vào Target Group. |
| Auto Scaling Group + EC2 | Chạy file jar Spring Boot trong private app subnet và thay instance khi rollout. |
| Amazon RDS for MySQL | Lưu dữ liệu ứng dụng trong private DB subnet. |
| Amazon S3 | Lưu artifact jar release và file ảnh upload của website. |
| Secrets Manager | Lưu DB URL, DB username/password, domain, cấu hình S3 và OAuth. |
| IAM Role + SSM | Cho EC2 đọc S3/Secrets và quản trị bằng Session Manager, không cần mở SSH. |
| CloudWatch + SNS | Theo dõi trạng thái vận hành và gửi cảnh báo khi metric/alarm vượt ngưỡng. |
| CloudFront và AWS WAF | Thành phần trong kiến trúc mục tiêu; đã được khảo sát nhưng chưa đưa vào luồng truy cập chính của phạm vi triển khai này. |

#### Nội dung

1. [Tổng quan kiến trúc](5.1-Workshop-overview/)
2. [Chuẩn bị môi trường triển khai](5.2-Prerequiste/)
3. [Cấu hình mạng và Security Group](5.3-S3-vpc/)
4. [S3, RDS, Secrets Manager và IAM](5.4-S3-onprem/)
5. [EC2, Application Load Balancer và Auto Scaling Group](5.5-Policy/)
6. [Route 53, ACM, HTTPS và kiểm thử](5.6-Cleanup/)
7. [Vận hành, giám sát, cảnh báo và xử lý lỗi](5.7-Operation/)
8. [Dọn dẹp tài nguyên](5.8-Cleanup/)


