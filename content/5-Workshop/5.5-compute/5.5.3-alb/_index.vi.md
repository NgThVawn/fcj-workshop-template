---
title : "Target Group và ALB"
date : 2024-01-01
weight : 3
chapter : false
pre : " <b> 5.5.3 </b> "
---

#### Target Group và Application Load Balancer

Target Group kiểm tra health của EC2 app trên port `8080`. Application Load Balancer là điểm truy cập public, nhận request HTTP/HTTPS và forward vào Target Group.

| Thành phần | Cấu hình |
|---|---|
| Target type | Instances |
| Target Group protocol/port | HTTP : `8080` |
| Health check path | `/actuator/health` |
| ALB scheme | Internet-facing |
| ALB subnets | 2 public subnets ở 2 AZ |
| ALB Security Group | Mở inbound 80/443 từ Internet |
| Listener ban đầu | HTTP:80 forward `sportbooking-tg` |

Kết quả đã ghi nhận: Target Group có health check đúng path, ALB ở trạng thái active và có DNS name để kiểm thử trước khi trỏ domain.

#### Quy trình thực hiện: Target Group và ALB

Target Group được tạo và kiểm tra health check trước khi ALB public chuyển lưu lượng đến ứng dụng; website sau đó được kiểm thử qua DNS của ALB.

**Quy trình thực hiện:**

1. **Bắt đầu tạo Target Group**

   Truy cập **EC2 > Target Groups**. Sau đó, chọn **Create target group**.

   ![Bắt đầu tạo Target Group](/images/5-Workshop/5.5-compute/alb-01.png)

   <p class="image-caption"><em>Hình 1: Bắt đầu tạo Target Group</em></p>
   Target Group là nơi ALB gửi request tới EC2 app và kiểm tra health.


2. **Chọn target type và protocol**

   Chọn target type **Instances**. Sau đó, nhập tên target group. Tiếp theo, chọn protocol HTTP và port `8080`.

   ![Chọn target type và protocol](/images/5-Workshop/5.5-compute/alb-02.png)

   <p class="image-caption"><em>Hình 2: Chọn target type và protocol</em></p>
   Spring Boot app chạy ở port 8080 trên EC2. ALB nhận public 80/443 nhưng forward nội bộ tới 8080.


3. **Chọn VPC và protocol version**

   Chọn VPC SportBooking. Sau đó, giữ protocol version HTTP1 nếu app không yêu cầu khác.

   ![Chọn VPC và protocol version](/images/5-Workshop/5.5-compute/alb-03.png)

   <p class="image-caption"><em>Hình 3: Chọn VPC và protocol version</em></p>
   Target Group phải cùng VPC với EC2 app để đăng ký target được.


4. **Cấu hình health check**

   Nhập health check path `/actuator/health`. Sau đó, giữ protocol HTTP.

   ![Cấu hình health check](/images/5-Workshop/5.5-compute/alb-04.png)

   <p class="image-caption"><em>Hình 4: Cấu hình health check</em></p>
   Health check giúp ALB chỉ gửi traffic tới instance app đang hoạt động. `/actuator/health` phản ánh trạng thái Spring Boot tốt hơn `/`.


5. **Chuyển sang bước đăng ký target**

   Kiểm tra target selection và attribute. Sau đó, chọn **Next**.

   ![Hoàn tất cấu hình Target Group](/images/5-Workshop/5.5-compute/alb-05.png)

   <p class="image-caption"><em>Hình 5: Hoàn tất cấu hình Target Group và chuyển sang đăng ký target</em></p>
   Sau khi hoàn tất protocol và health check, quy trình chuyển sang đăng ký EC2 seed để kiểm thử ALB trước khi tạo ASG.


6. **Chọn EC2 seed làm target ban đầu**

   Chọn instance `sportbooking-seed`. Sau đó, nhập port `8080`. Tiếp theo, chọn **Include as pending below**.

   ![Chọn instance target nếu đăng ký thủ công](/images/5-Workshop/5.5-compute/alb-06.png)

   <p class="image-caption"><em>Hình 6: Chọn sportbooking-seed làm target trên port 8080</em></p>
   EC2 seed được đăng ký để kiểm tra Target Group và ALB trước; sau đó ASG tiếp quản việc đăng ký các instance do Auto Scaling Group tạo ra.


7. **Rà soát target**

   Kiểm tra `sportbooking-seed`, port `8080` và trạng thái `Running` trong danh sách pending. Sau đó, chọn **Create target group**.

   ![Rà soát target](/images/5-Workshop/5.5-compute/alb-07.png)

   <p class="image-caption"><em>Hình 7: Rà soát target</em></p>
   Sai port target là nguyên nhân phổ biến khiến health check fail.


8. **Xác nhận tạo Target Group**

   Rà soát health check `/actuator/health` và target `sportbooking-seed`. Sau đó, chọn **Create target group**.

   ![Xác nhận tạo Target Group](/images/5-Workshop/5.5-compute/alb-08.png)

   <p class="image-caption"><em>Hình 8: Xác nhận tạo Target Group</em></p>
   Đây là bước cuối để Target Group sẵn sàng gắn vào ALB hoặc ASG.


9. **Xác nhận đăng ký target pending**

   AWS Console hiển thị hộp thoại xác nhận pending target; chọn **Continue**.

   ![Xác nhận đăng ký target pending](/images/5-Workshop/5.5-compute/alb-09.png)

   <p class="image-caption"><em>Hình 9: Xác nhận đăng ký target pending</em></p>
   AWS cảnh báo vì target đang pending health check. Sau vài phút và app chạy ổn, trạng thái mới chuyển healthy.


10. **Kiểm tra Target Group**

   Mở Target Group vừa tạo. Sau đó, kiểm tra tab Targets và health check.

   ![Kiểm tra Target Group](/images/5-Workshop/5.5-compute/alb-10.png)

   <p class="image-caption"><em>Hình 10: Kiểm tra Target Group</em></p>
   Trạng thái Healthy là điều kiện quan trọng trước khi đưa domain về ALB.


11. **Bắt đầu tạo Load Balancer**

   Truy cập **EC2 > Load Balancers**. Sau đó, chọn **Create load balancer**.

   ![Bắt đầu tạo Load Balancer](/images/5-Workshop/5.5-compute/alb-11.png)

   <p class="image-caption"><em>Hình 11: Bắt đầu tạo Load Balancer</em></p>
   Load Balancer là entry point public cho website, thay vì cho người dùng truy cập trực tiếp EC2 private.


12. **Chọn Application Load Balancer**

   Chọn **Application Load Balancer**. Sau đó, chọn **Create**.

   ![Chọn Application Load Balancer](/images/5-Workshop/5.5-compute/alb-12.png)

   <p class="image-caption"><em>Hình 12: Chọn Application Load Balancer</em></p>
   ALB phù hợp web HTTP/HTTPS, hỗ trợ listener, rule, target group và TLS termination.


13. **Nhập cấu hình cơ bản ALB**

   Nhập tên `sportbooking-alb`. Sau đó, chọn scheme **Internet-facing**. Tiếp theo, chọn IP address type IPv4.

   ![Nhập cấu hình cơ bản ALB](/images/5-Workshop/5.5-compute/alb-13.png)

   <p class="image-caption"><em>Hình 13: Nhập cấu hình cơ bản ALB</em></p>
   Internet-facing ALB nhận traffic từ người dùng. EC2 app vẫn private phía sau.


14. **Chọn VPC và public subnets**

   Chọn VPC SportBooking. Sau đó, chọn 2 public subnet ở 2 AZ.

   ![Chọn VPC và public subnets](/images/5-Workshop/5.5-compute/alb-14.png)

   <p class="image-caption"><em>Hình 14: Chọn VPC và public subnets</em></p>
   ALB cần public subnet để nhận request Internet và đa AZ để tăng availability.


15. **Chọn Security Group và listener**

   Chọn ALB Security Group mở 80/443. Sau đó, tạo listener HTTP:80 ban đầu. Tiếp theo, cấu hình forward đến Target Group `sportbooking-tg`.

   ![Chọn Security Group và listener](/images/5-Workshop/5.5-compute/alb-15.png)

   <p class="image-caption"><em>Hình 15: Chọn Security Group và listener</em></p>
   ALB SG là điểm public duy nhất. Listener forward request vào app qua Target Group.


16. **Rà soát và tạo ALB**

   Kiểm tra summary: VPC, subnets, SG, listener, target group. Sau đó, chọn **Create load balancer**.

   ![Rà soát và tạo ALB](/images/5-Workshop/5.5-compute/alb-16.png)

   <p class="image-caption"><em>Hình 16: Rà soát và tạo ALB</em></p>
   VPC, hai public subnet, Security Group, listener và Target Group đã được rà soát trước khi tạo ALB.


17. **Xác nhận ALB tạo thành công**

   Mở ALB vừa tạo. Sau đó, ghi nhận DNS name `sportbooking-alb-1198360694.us-east-1.elb.amazonaws.com`.

   ![Xác nhận ALB tạo thành công](/images/5-Workshop/5.5-compute/alb-17.png)

   <p class="image-caption"><em>Hình 17: Xác nhận ALB tạo thành công</em></p>
   DNS name của ALB dùng để test trước khi cấu hình Route 53 Alias.


18. **Kiểm tra listener và security**

   Mở tab listener/security. Sau đó, kiểm tra HTTP listener forward đúng Target Group.

   ![Kiểm tra listener và security](/images/5-Workshop/5.5-compute/alb-18.png)

   <p class="image-caption"><em>Hình 18: Kiểm tra listener và security</em></p>
   Nếu listener sai action, browser có thể truy cập ALB nhưng không tới được app.


19. **Kiểm thử website qua ALB**

   Mở ALB DNS hoặc domain đã trỏ. Sau đó, kiểm tra trang chủ SportBooking hiển thị.

   ![Kiểm thử website qua ALB](/images/5-Workshop/5.5-compute/alb-19.png)

   <p class="image-caption"><em>Hình 19: Kiểm thử website qua ALB</em></p>
   Đây là bằng chứng end-to-end: user -> ALB -> Target Group -> EC2 app -> RDS/S3.
