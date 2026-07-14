---
title : "Kiểm thử truy cập"
date : 2024-01-01
weight : 3
chapter : false
pre : " <b> 5.6.3 </b> "
---

#### Kiểm thử DNS và HTTPS bằng PowerShell

```powershell
nslookup -type=NS younglilliu.id.vn
nslookup sport.younglilliu.id.vn

curl.exe -I http://sport.younglilliu.id.vn
curl.exe -I https://sport.younglilliu.id.vn
curl.exe -I https://sport.younglilliu.id.vn/auth/login
curl.exe -I https://sport.younglilliu.id.vn/auth/register
curl.exe -I https://sport.younglilliu.id.vn/actuator/health
```

Kết quả đã ghi nhận:

- `http://sport.younglilliu.id.vn` trả `HTTP/1.1 301` và có `Location: https://...`.
- `https://sport.younglilliu.id.vn` trả `HTTP/1.1 200`.
- Cookie session có thuộc tính bảo mật sau khi chạy qua HTTPS.

#### Kiểm thử trên EC2 mới

Truy cập EC2 bằng Session Manager:

```bash
sudo -i
systemctl status sportbooking --no-pager
curl -I http://127.0.0.1:8080/
curl http://127.0.0.1:8080/actuator/health
grep -n -E "APP_BASE_URL|APP_UPLOADS_BASE_URL|SERVER_FORWARD_HEADERS_STRATEGY|APP_STORAGE_TYPE|APP_STORAGE_S3_BUCKET|APP_STORAGE_S3_KEY_PREFIX|AWS_REGION|AWS_DEFAULT_REGION|AWS_S3_REGION" /opt/sportbooking/app.env
sha256sum /opt/sportbooking/app.jar
aws s3 cp s3://sportbooking-artifacts-3stars-us-east-1/releases/SportBooingSystem-0.0.1-SNAPSHOT.jar /tmp/latest-app.jar --region us-east-1
sha256sum /opt/sportbooking/app.jar /tmp/latest-app.jar
```

Kết quả đã ghi nhận:

- Service `sportbooking` ở trạng thái `active (running)`.
- Health check local trả OK.
- App env có domain HTTPS, storage S3 và region `us-east-1`.
- Hash jar trên EC2 giống hash jar tải từ S3.

#### Quy trình kiểm thử

1. **Kiểm thử website bằng domain**

   Mở `https://sport.younglilliu.id.vn`. Sau đó, kiểm tra trang SportBooking hiển thị.

   ![Kiểm thử website bằng domain](/images/5-Workshop/5.6-domain-https/route-53-22.png)

   <p class="image-caption"><em>Hình 1: Kiểm thử website bằng domain</em></p>
   Bước cuối xác minh chuỗi DNS -> HTTPS -> ALB -> EC2 app hoạt động end-to-end.
