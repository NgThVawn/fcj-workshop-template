---
title : "Security Group"
date : 2024-01-01
weight : 2
chapter : false
pre : " <b> 5.3.2 </b> "
aliases:
  - /vi/5-workshop/5.3-s3-vpc/5.3.2-test-gwe/
---

#### Cấu hình triển khai

Ba Security Group `sportbooking-alb`, `sportbooking-app` và `sportbooking-rds` đã được tạo theo chuỗi truy cập: người dùng truy cập ALB; ALB chuyển tiếp đến EC2 app; EC2 app kết nối RDS MySQL.

#### ALB Security Group

| Hướng | Type | Port | Source/Destination | Vai trò |
|---|---|---|---|---|
| Inbound | HTTP | 80 | `0.0.0.0/0` | Cho người dùng truy cập HTTP và redirect sang HTTPS. |
| Inbound | HTTPS | 443 | `0.0.0.0/0` | Cho người dùng truy cập HTTPS. |
| Outbound | Custom TCP | 8080 | EC2 App SG | ALB forward traffic vào app. |

Kết quả đã ghi nhận: ALB SG không mở port `8080`, `3306` hoặc `22` public.

#### EC2 App Security Group

| Hướng | Type | Port | Source/Destination | Vai trò |
|---|---|---|---|---|
| Inbound | Custom TCP | 8080 | ALB Security Group | Chỉ ALB được gọi ứng dụng. |
| Inbound | SSH | Không mở | Không dùng | Quản trị bằng Session Manager thay SSH. |
| Outbound | HTTPS/All traffic | 443/All | Internet qua NAT hoặc AWS services | EC2 cần gọi S3, Secrets Manager, SSM và RDS. |

Kết quả đã ghi nhận: không có rule `0.0.0.0/0` vào port `8080`.

#### RDS Security Group

| Hướng/Cấu hình | Type | Port | Source | Vai trò |
|---|---|---|---|---|
| Inbound | MySQL/Aurora | 3306 | EC2 App Security Group | Chỉ EC2 app truy cập DB. |
| Public access | No | - | - | RDS không public Internet. |

Kết quả đã ghi nhận: RDS chỉ nhận kết nối từ EC2 App SG, không nhận từ public IP hoặc `0.0.0.0/0`.

#### Đối chiếu sau cấu hình

- Ghi nhận inbound rules của ALB SG, EC2 App SG và RDS SG.
- Kiểm tra source của rule EC2 là ALB SG, không phải CIDR public.
- Kiểm tra source của rule RDS là EC2 App SG.
- SSH public không được mở; EC2 được quản trị bằng Session Manager sau khi gắn IAM Role.

#### Quy trình thực hiện

Quy trình tạo Security Group được thực hiện theo chuỗi ALB, EC2 app và RDS để các rule tham chiếu luôn trỏ đến thành phần đã tồn tại.

**Quy trình thực hiện:**

1. **Mở danh sách Security Groups**

   Truy cập **EC2 > Security Groups**. Sau đó, chọn **Create security group**.

   ![Mở danh sách Security Groups](/images/5-Workshop/5.3-network/security-groups-01.png)

   <p class="image-caption"><em>Hình 1: Mở danh sách Security Groups</em></p>
   Security Group là firewall ở cấp tài nguyên. Quy trình triển khai cần tách SG cho ALB, EC2 app và RDS để áp dụng least privilege.


2. **Cấu hình Security Group cho ALB**

   Nhập tên `sportbooking-alb` và mô tả vai trò của ALB. Sau đó, chọn VPC `sportbooking-vpc`. Tiếp theo, thêm inbound rule HTTP port `80` và HTTPS port `443` từ `0.0.0.0/0`.

   ![Cấu hình Security Group cho ALB](/images/5-Workshop/5.3-network/security-groups-02.png)

   <p class="image-caption"><em>Hình 2: Cấu hình inbound cho Security Group của ALB</em></p>
   Hai rule public chỉ được đặt tại lớp ALB. Port ứng dụng `8080` và port cơ sở dữ liệu `3306` không được mở từ Internet.


3. **Hoàn tất tạo Security Group cho ALB**

   Rà soát outbound rule và thông tin định danh của Security Group. Sau đó, bổ sung tag phục vụ quản lý tài nguyên. Tiếp theo, chọn **Create security group**.

   ![Hoàn tất tạo Security Group cho ALB](/images/5-Workshop/5.3-network/security-groups-03.png)

   <p class="image-caption"><em>Hình 3: Rà soát và tạo Security Group cho ALB</em></p>
   Cấu hình được kiểm tra trước khi tạo để bảo đảm Security Group thuộc đúng VPC và chỉ có các rule cần thiết.


4. **Xác nhận Security Group đã tạo**

   Mở Security Group `sportbooking-alb` vừa tạo. Sau đó, kiểm tra tab **Inbound rules** và thông tin VPC.

   ![Xác nhận Security Group đã tạo](/images/5-Workshop/5.3-network/security-groups-04.png)

   <p class="image-caption"><em>Hình 4: Xác nhận Security Group sportbooking-alb đã được tạo</em></p>
   Thông tin sau khi tạo cho thấy `sportbooking-alb` thuộc đúng VPC và có hai inbound rule HTTP/HTTPS.


5. **Giới hạn outbound từ ALB đến ứng dụng**

   Mở tab **Outbound rules**. Sau đó, đặt protocol **Custom TCP**, port `8080`. Tiếp theo, chọn Security Group `sportbooking-app` làm destination và chọn **Save rules**.

   ![Giới hạn outbound từ ALB đến ứng dụng](/images/5-Workshop/5.3-network/security-groups-05.png)

   <p class="image-caption"><em>Hình 5: Giới hạn outbound của ALB đến port 8080 trên sportbooking-app</em></p>
   Sau bước này, ALB chỉ chuyển lưu lượng ứng dụng đến EC2 mang Security Group `sportbooking-app` qua port `8080`.

Quy trình **Create security group** được thực hiện tương tự cho hai lớp còn lại. `sportbooking-app` chỉ nhận inbound port `8080` từ `sportbooking-alb`; `sportbooking-rds` chỉ nhận inbound MySQL port `3306` từ `sportbooking-app`. Ba Security Group sau đó được sử dụng lần lượt cho ALB, Launch Template và RDS.




