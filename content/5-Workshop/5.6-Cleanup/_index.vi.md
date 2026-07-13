---
title : "Route 53, ACM, HTTPS và kiểm thử"
date : 2024-01-01
weight : 6
chapter : false
pre : " <b> 5.6. </b> "
---

#### Phạm vi triển khai

Cấu hình domain `sport.younglilliu.id.vn`, cấp certificate bằng ACM, gắn HTTPS listener vào ALB và kiểm thử website sau triển khai.

#### Cấu hình Hosted Zone và Name Server

| Mục | Cấu hình đã ghi nhận |
|---|---|
| Hosted zone | `younglilliu.id.vn` |
| Subdomain app | `sport.younglilliu.id.vn` |
| PA Vietnam | 4 NS do Route 53 cấp đã thay thế toàn bộ NS cũ. |
| Lỗi đã đối chiếu | Sai chính tả giữa `youngliliu.id.vn` và `younglilliu.id.vn` làm DNS không phân giải và ACM giữ trạng thái pending. |

Delegation đã được kiểm tra bằng lệnh:

```powershell
nslookup -type=NS younglilliu.id.vn
```

#### Alias record đã tạo cho ALB

| Mục | Cấu hình |
|---|---|
| Record name | `sport.younglilliu.id.vn` |
| Type | A |
| Alias | Yes |
| Route traffic to | Application Load Balancer `sportbooking-alb-1198360694.us-east-1.elb.amazonaws.com` |
| Lý do dùng Alias | Route 53 Alias trỏ được tới ALB mà không cần IP tĩnh. |

Record đã được kiểm tra bằng lệnh:

```powershell
nslookup sport.younglilliu.id.vn
```

#### ACM certificate đã cấp

| Mục | Cấu hình |
|---|---|
| Certificate type | Public certificate |
| Domain name | `sport.younglilliu.id.vn` |
| Validation method | DNS validation |
| Region | `us-east-1` |
| Trạng thái đã ghi nhận | Issued |

CNAME validation record đã được tạo trong Route 53. Sau khi record xuất hiện trên DNS public, ACM chuyển từ `Pending validation` sang `Issued`.

{{% notice note %}}
Nếu ACM pending lâu, kiểm tra domain spelling, hosted zone có đúng public zone không và NS tại nhà cung cấp domain đã đổi sang Route 53 chưa.
{{% /notice %}}

#### HTTPS listener đã gắn vào ALB

| Mục | Kết quả/cấu hình |
|---|---|
| HTTPS listener 443 | Forward to `sportbooking-tg`, chọn ACM certificate `sport.younglilliu.id.vn`. |
| HTTP listener 80 | Redirect to HTTPS, port 443, status HTTP_301. |
| Security Group | ALB SG mở inbound 80 và 443 từ `0.0.0.0/0`. |

Kết quả đã ghi nhận:

- HTTP trả redirect 301.
- HTTPS trả 200 OK.
- Browser hiển thị website bằng domain chính, không dùng ALB DNS.

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

#### Quy trình thực hiện: Route 53, ACM và HTTPS

Quy trình công bố website bắt đầu bằng hosted zone và Name Server, tiếp tục với Alias record cùng DNS validation cho ACM, sau đó hoàn thiện listener HTTPS và kiểm thử domain.

**Quy trình thực hiện:**

1. **Bắt đầu tạo Hosted Zone**

   Truy cập **Route 53 > Hosted zones**. Sau đó, chọn **Create hosted zone**.

   ![Bắt đầu tạo Hosted Zone](/images/5-Workshop/5.6-Cleanup/route-53-01.png)

   <p class="image-caption"><em>Hình 1: Bắt đầu tạo Hosted Zone</em></p>
   Hosted Zone quản lý DNS cho domain dùng để truy cập website SportBooking.


2. **Nhập domain cho Hosted Zone**

   Nhập domain gốc `younglilliu.id.vn`. Sau đó, chọn type **Public hosted zone**.

   ![Nhập domain cho Hosted Zone](/images/5-Workshop/5.6-Cleanup/route-53-02.png)

   <p class="image-caption"><em>Hình 2: Nhập domain cho Hosted Zone</em></p>
   Public hosted zone cho phép Internet phân giải domain về ALB. Không tạo hosted zone cho sai chính tả domain.


3. **Tạo Hosted Zone**

   Kiểm tra domain name. Sau đó, chọn **Create hosted zone**.

   ![Tạo Hosted Zone](/images/5-Workshop/5.6-Cleanup/route-53-03.png)

   <p class="image-caption"><em>Hình 3: Tạo Hosted Zone</em></p>
   Sau khi tạo, AWS sinh ra bộ Name Server cần cấu hình ở nhà cung cấp domain.


4. **Ghi nhận Name Server**

   Mở hosted zone vừa tạo. Sau đó, ghi nhận 4 NS record do Route 53 cấp.

   ![Ghi nhận Name Server](/images/5-Workshop/5.6-Cleanup/route-53-04.png)

   <p class="image-caption"><em>Hình 4: Ghi nhận Name Server</em></p>
   Domain chỉ dùng Route 53 khi nhà cung cấp domain trỏ NS về 4 server này.


5. **Bắt đầu tạo record cho subdomain**

   Trong hosted zone, chọn **Create record**.

   ![Bắt đầu tạo record cho subdomain](/images/5-Workshop/5.6-Cleanup/route-53-05.png)

   <p class="image-caption"><em>Hình 5: Bắt đầu tạo record cho subdomain</em></p>
   Record `sport.younglilliu.id.vn` đã trỏ người dùng tới ALB thay vì sử dụng trực tiếp DNS name của ALB.


6. **Tạo Alias record về ALB**

   Nhập record name `sport`. Sau đó, chọn record type `A`. Tiếp theo, bật **Alias**. Đồng thời, chọn Application Load Balancer của dự án. Trước khi chuyển sang bước tiếp theo, chọn **Create records**.

   ![Tạo Alias record về ALB](/images/5-Workshop/5.6-Cleanup/route-53-06.png)

   <p class="image-caption"><em>Hình 6: Tạo Alias record về ALB</em></p>
   Alias record là cách đúng để trỏ domain về ALB vì ALB không có IP tĩnh cố định.


7. **Xác nhận record đã tạo**

   Kiểm tra danh sách records. Sau đó, xác nhận record `sport` xuất hiện cùng record NS/SOA.

   ![Xác nhận record đã tạo](/images/5-Workshop/5.6-Cleanup/route-53-07.png)

   <p class="image-caption"><em>Hình 7: Xác nhận record đã tạo</em></p>
   Nếu record chưa tạo hoặc tạo ở hosted zone sai domain, trình duyệt sẽ không phân giải được website.


8. **Bắt đầu request certificate ACM**

   Truy cập **AWS Certificate Manager**. Sau đó, chọn **Request a certificate**.

   ![Bắt đầu request certificate ACM](/images/5-Workshop/5.6-Cleanup/route-53-08.png)

   <p class="image-caption"><em>Hình 8: Bắt đầu request certificate ACM</em></p>
   ACM certificate dùng để bật HTTPS trên ALB.


9. **Chọn public certificate**

   Chọn **Request a public certificate**. Sau đó, chọn **Next**.

   ![Chọn public certificate](/images/5-Workshop/5.6-Cleanup/route-53-09.png)

   <p class="image-caption"><em>Hình 9: Chọn public certificate</em></p>
   Website public cần chứng chỉ public do ACM phát hành để browser tin cậy.


10. **Nhập domain cần cấp certificate**

   Nhập `sport.younglilliu.id.vn`. Sau đó, chọn DNS validation.

   ![Nhập domain cần cấp certificate](/images/5-Workshop/5.6-Cleanup/route-53-10.png)

   <p class="image-caption"><em>Hình 10: Nhập domain cần cấp certificate</em></p>
   Certificate phải khớp domain người dùng truy cập. DNS validation dễ tích hợp với Route 53.


11. **Rà soát và request certificate**

   Kiểm tra key algorithm và tag. Sau đó, chọn **Request**.

   ![Rà soát và request certificate](/images/5-Workshop/5.6-Cleanup/route-53-11.png)

   <p class="image-caption"><em>Hình 11: Rà soát và request certificate</em></p>
   Sai domain ở bước này sẽ khiến HTTPS không khớp hoặc ACM pending validation lâu.


12. **Mở certificate pending validation**

   Mở certificate vừa request. Sau đó, kiểm tra trạng thái validation.

   ![Mở certificate pending validation](/images/5-Workshop/5.6-Cleanup/route-53-12.png)

   <p class="image-caption"><em>Hình 12: Mở certificate pending validation</em></p>
   ACM cần record CNAME validation public trước khi chuyển sang Issued.


13. **Tạo DNS validation record**

   Chọn **Create records in Route 53**.

   ![Tạo DNS validation record](/images/5-Workshop/5.6-Cleanup/route-53-13.png)

   <p class="image-caption"><em>Hình 13: Tạo DNS validation record</em></p>
   Nút này tự tạo CNAME validation trong hosted zone tương ứng, giảm lỗi copy sai CNAME.


14. **Xác nhận tạo validation record**

   Kiểm tra record CNAME validation. Sau đó, chọn **Create records**.

   ![Xác nhận tạo validation record](/images/5-Workshop/5.6-Cleanup/route-53-14.png)

   <p class="image-caption"><em>Hình 14: Xác nhận tạo validation record</em></p>
   Validation record xác nhận quyền quản lý domain để ACM cấp certificate.


15. **Mở ALB để thêm HTTPS listener**

   Truy cập **EC2 > Load Balancers**. Sau đó, mở ALB SportBooking. Tiếp theo, chọn **Add listener**.

   ![Mở ALB để thêm HTTPS listener](/images/5-Workshop/5.6-Cleanup/route-53-15.png)

   <p class="image-caption"><em>Hình 15: Mở ALB để thêm HTTPS listener</em></p>
   Sau khi có certificate, cần listener 443 để browser truy cập HTTPS.


16. **Cấu hình HTTPS listener**

   Chọn protocol **HTTPS**. Sau đó, chọn port `443`. Tiếp theo, cấu hình forward đến Target Group `sportbooking-tg`.

   ![Cấu hình HTTPS listener](/images/5-Workshop/5.6-Cleanup/route-53-16.png)

   <p class="image-caption"><em>Hình 16: Cấu hình HTTPS listener</em></p>
   ALB terminate TLS ở port 443 rồi forward HTTP nội bộ tới app port 8080.


17. **Chọn certificate ACM**

   Chọn certificate `sport.younglilliu.id.vn`. Sau đó, chọn **Add**.

   ![Chọn certificate ACM](/images/5-Workshop/5.6-Cleanup/route-53-17.png)

   <p class="image-caption"><em>Hình 17: Chọn certificate ACM</em></p>
   Certificate đã ở trạng thái `Issued` trước khi được gắn vào HTTPS listener.


18. **Kiểm tra listener sau khi thêm**

   Mở tab **Listeners and rules**. Sau đó, xác nhận listener 443 forward đúng target group.

   ![Kiểm tra listener sau khi thêm](/images/5-Workshop/5.6-Cleanup/route-53-18.png)

   <p class="image-caption"><em>Hình 18: Kiểm tra listener sau khi thêm</em></p>
   Đây là bước xác minh HTTPS đã được bật ở ALB.


19. **Chỉnh HTTP listener sang redirect**

   Chọn listener HTTP:80. Sau đó, chọn **Edit listener**.

   ![Chỉnh HTTP listener sang redirect](/images/5-Workshop/5.6-Cleanup/route-53-19.png)

   <p class="image-caption"><em>Hình 19: Chỉnh HTTP listener sang redirect</em></p>
   HTTP listener được đổi từ forward sang redirect sau khi HTTPS listener hoạt động.


20. **Cấu hình redirect HTTP sang HTTPS**

   Chọn action **Redirect to URL**. Sau đó, đặt protocol HTTPS, port 443. Tiếp theo, chọn **Save changes**.

   ![Cấu hình redirect HTTP sang HTTPS](/images/5-Workshop/5.6-Cleanup/route-53-20.png)

   <p class="image-caption"><em>Hình 20: Cấu hình redirect HTTP sang HTTPS</em></p>
   Redirect 301 giúp tất cả truy cập HTTP tự chuyển sang HTTPS, hỗ trợ OAuth callback và secure cookie.


21. **Xác nhận listener 80/443**

   Kiểm tra listener HTTP và HTTPS. Sau đó, xác nhận 80 redirect, 443 forward target group.

   ![Xác nhận listener 80/443](/images/5-Workshop/5.6-Cleanup/route-53-21.png)

   <p class="image-caption"><em>Hình 21: Xác nhận listener 80/443</em></p>
   Trạng thái sau cấu hình gồm HTTP port `80` redirect và HTTPS port `443` forward đến Target Group.


22. **Kiểm thử website bằng domain**

   Mở `https://sport.younglilliu.id.vn`. Sau đó, kiểm tra trang SportBooking hiển thị.

   ![Kiểm thử website bằng domain](/images/5-Workshop/5.6-Cleanup/route-53-22.png)

   <p class="image-caption"><em>Hình 22: Kiểm thử website bằng domain</em></p>
   Bước cuối xác minh chuỗi DNS -> HTTPS -> ALB -> EC2 app hoạt động end-to-end.



#### Kết quả khảo sát CloudFront

Thử cấu hình CloudFront với ALB làm origin để đánh giá phương án đặt CDN trước website. AWS Console trả về lỗi yêu cầu xác minh tài khoản trước khi tạo tài nguyên CloudFront mới, vì vậy distribution không được tạo và CloudFront không tham gia luồng truy cập đã kiểm thử.

{{% notice info %}}
DNS `sportbooking-alb-2038054256.us-east-1.elb.amazonaws.com` trong ảnh CloudFront thuộc cấu hình ALB thử nghiệm trước đó. Cấu hình hoàn thiện sử dụng `sportbooking-alb-1198360694.us-east-1.elb.amazonaws.com` và Route 53 trỏ trực tiếp đến ALB này.
{{% /notice %}}

**Quy trình thực hiện:**

1. **Ghi nhận lỗi xác minh tài khoản khi tạo CloudFront**

   Mở màn hình **Create distribution** và chọn ALB làm origin thử nghiệm. Sau đó, rà soát cache settings và security protections. Tiếp theo, ghi nhận thông báo tài khoản phải được xác minh trước khi thêm tài nguyên CloudFront.

   ![Ghi nhận lỗi xác minh tài khoản khi tạo CloudFront](/images/5-Workshop/5.6-Cleanup/cloudfront-01.png)

   <p class="image-caption"><em>Hình 1: CloudFront không được tạo do tài khoản chưa hoàn tất xác minh</em></p>
   Việc khảo sát dừng tại màn hình này. Kiến trúc mục tiêu vẫn giữ CloudFront và WAF làm hướng mở rộng, không ghi nhận hai dịch vụ này là thành phần đã triển khai.




