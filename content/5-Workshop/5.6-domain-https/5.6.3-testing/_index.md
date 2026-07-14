---
title : "Access Testing"
date : 2024-01-01
weight : 3
chapter : false
pre : " <b> 5.6.3 </b> "
---

#### Test DNS and HTTPS with PowerShell

```powershell
nslookup -type=NS younglilliu.id.vn
nslookup sport.younglilliu.id.vn

curl.exe -I http://sport.younglilliu.id.vn
curl.exe -I https://sport.younglilliu.id.vn
curl.exe -I https://sport.younglilliu.id.vn/auth/login
curl.exe -I https://sport.younglilliu.id.vn/auth/register
curl.exe -I https://sport.younglilliu.id.vn/actuator/health
```

Recorded results:

- `http://sport.younglilliu.id.vn` returns `HTTP/1.1 301` with a `Location: https://...` header.
- `https://sport.younglilliu.id.vn` returns `HTTP/1.1 200`.
- The session cookie includes secure attributes when served through HTTPS.

#### Test a New EC2 Instance

Connect to EC2 through Session Manager:

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

Recorded results:

- The `sportbooking` service is `active (running)`.
- The local health check returns OK.
- The application environment contains the HTTPS domain, S3 storage settings, and `us-east-1` Region.
- The JAR hash on EC2 matches the hash of the JAR downloaded from S3.

#### Test Procedure

1. **Test the website with the domain**

   Open `https://sport.younglilliu.id.vn` and confirm that the SportBooking page loads.

   ![Test the website with the domain](/images/5-Workshop/5.6-domain-https/route-53-22.png)

   <p class="image-caption"><em>Figure 1: Test the website with the domain</em></p>
   This final step validates the complete DNS -> HTTPS -> ALB -> EC2 application path.
