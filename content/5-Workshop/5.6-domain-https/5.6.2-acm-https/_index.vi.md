---
title : "ACM và HTTPS"
date : 2024-01-01
weight : 2
chapter : false
pre : " <b> 5.6.2 </b> "
---

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

#### Quy trình thực hiện

1. **Bắt đầu request certificate ACM**

   Truy cập **AWS Certificate Manager**. Sau đó, chọn **Request a certificate**.

   ![Bắt đầu request certificate ACM](/images/5-Workshop/5.6-domain-https/route-53-08.png)

   <p class="image-caption"><em>Hình 1: Bắt đầu request certificate ACM</em></p>
   ACM certificate dùng để bật HTTPS trên ALB.


2. **Chọn public certificate**

   Chọn **Request a public certificate**. Sau đó, chọn **Next**.

   ![Chọn public certificate](/images/5-Workshop/5.6-domain-https/route-53-09.png)

   <p class="image-caption"><em>Hình 2: Chọn public certificate</em></p>
   Website public cần chứng chỉ public do ACM phát hành để browser tin cậy.


3. **Nhập domain cần cấp certificate**

   Nhập `sport.younglilliu.id.vn`. Sau đó, chọn DNS validation.

   ![Nhập domain cần cấp certificate](/images/5-Workshop/5.6-domain-https/route-53-10.png)

   <p class="image-caption"><em>Hình 3: Nhập domain cần cấp certificate</em></p>
   Certificate phải khớp domain người dùng truy cập. DNS validation dễ tích hợp với Route 53.


4. **Rà soát và request certificate**

   Kiểm tra key algorithm và tag. Sau đó, chọn **Request**.

   ![Rà soát và request certificate](/images/5-Workshop/5.6-domain-https/route-53-11.png)

   <p class="image-caption"><em>Hình 4: Rà soát và request certificate</em></p>
   Sai domain ở bước này sẽ khiến HTTPS không khớp hoặc ACM pending validation lâu.


5. **Mở certificate pending validation**

   Mở certificate vừa request. Sau đó, kiểm tra trạng thái validation.

   ![Mở certificate pending validation](/images/5-Workshop/5.6-domain-https/route-53-12.png)

   <p class="image-caption"><em>Hình 5: Mở certificate pending validation</em></p>
   ACM cần record CNAME validation public trước khi chuyển sang Issued.


6. **Tạo DNS validation record**

   Chọn **Create records in Route 53**.

   ![Tạo DNS validation record](/images/5-Workshop/5.6-domain-https/route-53-13.png)

   <p class="image-caption"><em>Hình 6: Tạo DNS validation record</em></p>
   Nút này tự tạo CNAME validation trong hosted zone tương ứng, giảm lỗi copy sai CNAME.


7. **Xác nhận tạo validation record**

   Kiểm tra record CNAME validation. Sau đó, chọn **Create records**.

   ![Xác nhận tạo validation record](/images/5-Workshop/5.6-domain-https/route-53-14.png)

   <p class="image-caption"><em>Hình 7: Xác nhận tạo validation record</em></p>
   Validation record xác nhận quyền quản lý domain để ACM cấp certificate.


8. **Mở ALB để thêm HTTPS listener**

   Truy cập **EC2 > Load Balancers**. Sau đó, mở ALB SportBooking. Tiếp theo, chọn **Add listener**.

   ![Mở ALB để thêm HTTPS listener](/images/5-Workshop/5.6-domain-https/route-53-15.png)

   <p class="image-caption"><em>Hình 8: Mở ALB để thêm HTTPS listener</em></p>
   Sau khi có certificate, cần listener 443 để browser truy cập HTTPS.


9. **Cấu hình HTTPS listener**

   Chọn protocol **HTTPS**. Sau đó, chọn port `443`. Tiếp theo, cấu hình forward đến Target Group `sportbooking-tg`.

   ![Cấu hình HTTPS listener](/images/5-Workshop/5.6-domain-https/route-53-16.png)

   <p class="image-caption"><em>Hình 9: Cấu hình HTTPS listener</em></p>
   ALB terminate TLS ở port 443 rồi forward HTTP nội bộ tới app port 8080.


10. **Chọn certificate ACM**

   Chọn certificate `sport.younglilliu.id.vn`. Sau đó, chọn **Add**.

   ![Chọn certificate ACM](/images/5-Workshop/5.6-domain-https/route-53-17.png)

   <p class="image-caption"><em>Hình 10: Chọn certificate ACM</em></p>
   Certificate đã ở trạng thái `Issued` trước khi được gắn vào HTTPS listener.


11. **Kiểm tra listener sau khi thêm**

   Mở tab **Listeners and rules**. Sau đó, xác nhận listener 443 forward đúng target group.

   ![Kiểm tra listener sau khi thêm](/images/5-Workshop/5.6-domain-https/route-53-18.png)

   <p class="image-caption"><em>Hình 11: Kiểm tra listener sau khi thêm</em></p>
   Đây là bước xác minh HTTPS đã được bật ở ALB.


12. **Chỉnh HTTP listener sang redirect**

   Chọn listener HTTP:80. Sau đó, chọn **Edit listener**.

   ![Chỉnh HTTP listener sang redirect](/images/5-Workshop/5.6-domain-https/route-53-19.png)

   <p class="image-caption"><em>Hình 12: Chỉnh HTTP listener sang redirect</em></p>
   HTTP listener được đổi từ forward sang redirect sau khi HTTPS listener hoạt động.


13. **Cấu hình redirect HTTP sang HTTPS**

   Chọn action **Redirect to URL**. Sau đó, đặt protocol HTTPS, port 443. Tiếp theo, chọn **Save changes**.

   ![Cấu hình redirect HTTP sang HTTPS](/images/5-Workshop/5.6-domain-https/route-53-20.png)

   <p class="image-caption"><em>Hình 13: Cấu hình redirect HTTP sang HTTPS</em></p>
   Redirect 301 giúp tất cả truy cập HTTP tự chuyển sang HTTPS, hỗ trợ OAuth callback và secure cookie.


14. **Xác nhận listener 80/443**

   Kiểm tra listener HTTP và HTTPS. Sau đó, xác nhận 80 redirect, 443 forward target group.

   ![Xác nhận listener 80/443](/images/5-Workshop/5.6-domain-https/route-53-21.png)

   <p class="image-caption"><em>Hình 14: Xác nhận listener 80/443</em></p>
   Trạng thái sau cấu hình gồm HTTP port `80` redirect và HTTPS port `443` forward đến Target Group.
