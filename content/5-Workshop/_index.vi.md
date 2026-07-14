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

1. [Kiến trúc](5.1-architecture/)
2. [Chuẩn bị](5.2-preparation/)
3. [Mạng và bảo mật](5.3-network/)
4. [Dữ liệu và phân quyền](5.4-data-access/)
5. [EC2 và cân bằng tải](5.5-compute/)
6. [Tên miền và HTTPS](5.6-domain-https/)
7. [Vận hành](5.7-operations/)
8. [Dọn dẹp](5.8-resource-cleanup/)


