---
title : "Route 53"
date : 2024-01-01
weight : 1
chapter : false
pre : " <b> 5.6.1 </b> "
---

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

#### Quy trình thực hiện

Quy trình công bố website bắt đầu bằng hosted zone và Name Server, tiếp tục với Alias record cùng DNS validation cho ACM, sau đó hoàn thiện listener HTTPS và kiểm thử domain.

**Quy trình thực hiện:**

1. **Bắt đầu tạo Hosted Zone**

   Truy cập **Route 53 > Hosted zones**. Sau đó, chọn **Create hosted zone**.

   ![Bắt đầu tạo Hosted Zone](/images/5-Workshop/5.6-domain-https/route-53-01.png)

   <p class="image-caption"><em>Hình 1: Bắt đầu tạo Hosted Zone</em></p>
   Hosted Zone quản lý DNS cho domain dùng để truy cập website SportBooking.


2. **Nhập domain cho Hosted Zone**

   Nhập domain gốc `younglilliu.id.vn`. Sau đó, chọn type **Public hosted zone**.

   ![Nhập domain cho Hosted Zone](/images/5-Workshop/5.6-domain-https/route-53-02.png)

   <p class="image-caption"><em>Hình 2: Nhập domain cho Hosted Zone</em></p>
   Public hosted zone cho phép Internet phân giải domain về ALB. Không tạo hosted zone cho sai chính tả domain.


3. **Tạo Hosted Zone**

   Kiểm tra domain name. Sau đó, chọn **Create hosted zone**.

   ![Tạo Hosted Zone](/images/5-Workshop/5.6-domain-https/route-53-03.png)

   <p class="image-caption"><em>Hình 3: Tạo Hosted Zone</em></p>
   Sau khi tạo, AWS sinh ra bộ Name Server cần cấu hình ở nhà cung cấp domain.


4. **Ghi nhận Name Server**

   Mở hosted zone vừa tạo. Sau đó, ghi nhận 4 NS record do Route 53 cấp.

   ![Ghi nhận Name Server](/images/5-Workshop/5.6-domain-https/route-53-04.png)

   <p class="image-caption"><em>Hình 4: Ghi nhận Name Server</em></p>
   Domain chỉ dùng Route 53 khi nhà cung cấp domain trỏ NS về 4 server này.


5. **Bắt đầu tạo record cho subdomain**

   Trong hosted zone, chọn **Create record**.

   ![Bắt đầu tạo record cho subdomain](/images/5-Workshop/5.6-domain-https/route-53-05.png)

   <p class="image-caption"><em>Hình 5: Bắt đầu tạo record cho subdomain</em></p>
   Record `sport.younglilliu.id.vn` đã trỏ người dùng tới ALB thay vì sử dụng trực tiếp DNS name của ALB.


6. **Tạo Alias record về ALB**

   Nhập record name `sport`. Sau đó, chọn record type `A`. Tiếp theo, bật **Alias**. Đồng thời, chọn Application Load Balancer của dự án. Trước khi chuyển sang bước tiếp theo, chọn **Create records**.

   ![Tạo Alias record về ALB](/images/5-Workshop/5.6-domain-https/route-53-06.png)

   <p class="image-caption"><em>Hình 6: Tạo Alias record về ALB</em></p>
   Alias record là cách đúng để trỏ domain về ALB vì ALB không có IP tĩnh cố định.


7. **Xác nhận record đã tạo**

   Kiểm tra danh sách records. Sau đó, xác nhận record `sport` xuất hiện cùng record NS/SOA.

   ![Xác nhận record đã tạo](/images/5-Workshop/5.6-domain-https/route-53-07.png)

   <p class="image-caption"><em>Hình 7: Xác nhận record đã tạo</em></p>
   Nếu record chưa tạo hoặc tạo ở hosted zone sai domain, trình duyệt sẽ không phân giải được website.
