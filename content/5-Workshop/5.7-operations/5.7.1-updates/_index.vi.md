---
title : "Cập nhật ứng dụng"
date : 2024-01-01
weight : 1
chapter : false
pre : " <b> 5.7.1 </b> "
---

#### Quy trình cập nhật artifact

| Bước | Việc làm | Lệnh/màn hình |
|---|---|---|
| 1 | Đóng gói JAR tại máy cục bộ | `.\mvnw.cmd clean package -DskipTests` |
| 2 | Ghi đè artifact trên S3 | `aws s3 cp target\SportBooingSystem-0.0.1-SNAPSHOT.jar s3://sportbooking-artifacts-3stars-us-east-1/releases/SportBooingSystem-0.0.1-SNAPSHOT.jar --region us-east-1` |
| 3 | Khởi chạy Instance Refresh | ASG > Instance refresh > Start > Use current Auto Scaling group configuration |
| 4 | Chờ target chuyển sang Healthy | Target Group > Targets > Healthy |
| 5 | Kiểm thử lại domain | `curl.exe -I https://sport.younglilliu.id.vn` và trình duyệt |

#### Quy trình cập nhật secret

| Bước | Việc làm | Chi tiết |
|---|---|---|
| 1 | Chỉnh sửa secret | Secrets Manager > `sportbooking/prod/app-env` > Edit. |
| 2 | Bảo vệ dữ liệu nhạy cảm | Không ghi `DB_PASSWORD` hoặc OAuth secrets vào ảnh. |
| 3 | Triển khai lại EC2 | Instance Refresh tạo instance mới và đọc phiên bản secret hiện tại. |
| 4 | Kiểm tra `app.env` | Chỉ dùng `grep` với các key không nhạy cảm trên EC2 mới. |
| 5 | Kiểm thử chức năng | Đăng ký, đăng nhập, OAuth và upload ảnh. |

{{% notice warning %}}
Thay đổi secret không tự động cập nhật vào EC2 đang chạy nếu ứng dụng chỉ đọc secret trong lúc boot. Sau khi sửa secret, cần restart service hoặc chạy Instance Refresh để instance mới đọc cấu hình mới.
{{% /notice %}}

#### Kiểm tra database và dữ liệu khởi tạo

Sau khi EC2 app hoạt động, kết nối RDS từ EC2 bằng Session Manager:

```bash
sudo -i
mariadb -h sportbooking-mysql.cudgu8qk8z19.us-east-1.rds.amazonaws.com -P 3306 -u sportbooking_ad -p
```

Schema được kiểm tra bằng:

```sql
USE sport_booking;
SHOW TABLES;
```

Khi xác nhận bảng `membership_levels` đã tồn tại nhưng chưa có dữ liệu, chạy câu lệnh idempotent sau:

```sql
INSERT INTO membership_levels
  (name, min_bookings, discount_rate, description, badge_color)
VALUES
  ('NONE', 0, 0.0000, 'Thanh vien thuong', '#888888'),
  ('SILVER', 10, 0.0500, 'Bac - giam 5%', '#C0C0C0'),
  ('GOLD', 30, 0.0700, 'Vang - giam 7%', '#FFD700'),
  ('DIAMOND', 100, 0.1000, 'Kim cuong - giam 10%', '#B9F2FF')
ON DUPLICATE KEY UPDATE
  min_bookings = VALUES(min_bookings),
  discount_rate = VALUES(discount_rate),
  description = VALUES(description),
  badge_color = VALUES(badge_color);
```

{{% notice info %}}
Giá trị `SPRING_JPA_HIBERNATE_DDL_AUTO=update` đã được sử dụng trong đợt triển khai này. Flyway hoặc Liquibase chưa nằm trong phạm vi thực hiện và được xác định là hạng mục hoàn thiện quản lý phiên bản schema.
{{% /notice %}}

#### Cấu hình OAuth đã cập nhật sau khi có HTTPS

Google OAuth:

| Mục | Giá trị cấu hình |
|---|---|
| Authorized JavaScript origins | `https://sport.younglilliu.id.vn` |
| Authorized redirect URIs | `https://sport.younglilliu.id.vn/login/oauth2/code/google` |
| Lỗi thường gặp | `redirect_uri_mismatch` khi redirect URI không trùng tuyệt đối. |
| Cách xử lý | Thêm đúng HTTPS callback, dùng đúng `younglilliu` và không dùng ALB DNS. |

Facebook OAuth:

| Mục | Giá trị cấu hình |
|---|---|
| App domain | `sport.younglilliu.id.vn` |
| Valid OAuth Redirect URI | `https://sport.younglilliu.id.vn/login/oauth2/code/facebook` |
| Lỗi thường gặp | Facebook yêu cầu secure connection khi dùng HTTP hoặc ALB DNS. |
| Cách xử lý | Sử dụng HTTPS domain qua ACM và ALB listener 443. |
