---
title : "Auto Scaling Group"
date : 2024-01-01
weight : 4
chapter : false
pre : " <b> 5.5.4 </b> "
---

#### Auto Scaling Group

Auto Scaling Group dùng Launch Template để tạo EC2 app trong private app subnets và gắn các instance vào Target Group đã tạo ở bước trước.

| Mục | Cấu hình |
|---|---|
| Launch Template | `sportbooking-lt`, version `Default (1)`. |
| Subnet | 2 private app subnets ở 2 AZ. |
| Desired capacity | `2`. |
| Min/Max | Min `2`, Max `4`. |
| Target Group | `sportbooking-tg`. |
| Health check | EC2 và Elastic Load Balancing; grace period `300` giây. |
| Scaling policy | Target tracking theo Average CPU utilization, target `60`, warmup `300` giây. |

{{% notice info %}}
Desired capacity và minimum capacity đều được đặt bằng `2` để ASG duy trì hai instance trên hai Availability Zone. Maximum capacity `4` tạo khoảng mở rộng cho target tracking policy khi CPU trung bình vượt mục tiêu.
{{% /notice %}}

#### Quy trình thực hiện: Auto Scaling Group

Auto Scaling Group sử dụng Launch Template đã hoàn thiện, triển khai instance trên các private app subnet và tự đăng ký chúng vào Target Group để kiểm tra trạng thái healthy.

**Quy trình thực hiện:**

1. **Bắt đầu tạo Auto Scaling Group**

   Truy cập **EC2 > Auto Scaling Groups**. Sau đó, chọn **Create Auto Scaling group**.

   ![Bắt đầu tạo Auto Scaling Group](/images/5-Workshop/5.5-compute/auto-scaling-group-01.png)

   <p class="image-caption"><em>Hình 1: Bắt đầu tạo Auto Scaling Group</em></p>
   ASG duy trì số lượng EC2 app, tự thay instance lỗi và hỗ trợ rollout bằng Instance Refresh.


2. **Chọn Launch Template**

   Nhập tên ASG `sportbooking-asg`. Sau đó, chọn Launch Template `sportbooking-lt`. Tiếp theo, chọn version `Default (1)`.

   ![Chọn Launch Template](/images/5-Workshop/5.5-compute/auto-scaling-group-02.png)

   <p class="image-caption"><em>Hình 2: Chọn Launch Template</em></p>
   ASG tạo instance dựa trên Launch Template. Chọn sai version có thể khiến EC2 dùng User Data/AMI cũ.


3. **Xác nhận template version**

   Kiểm tra AMI, instance type, key pair, security group trong summary. Sau đó, chọn **Next**.

   ![Xác nhận template version](/images/5-Workshop/5.5-compute/auto-scaling-group-03.png)

   <p class="image-caption"><em>Hình 3: Xác nhận template version</em></p>
   Đây là điểm kiểm tra trước khi ASG tạo instance thật trong subnet private.


4. **Chọn VPC và private app subnets**

   Chọn VPC SportBooking. Sau đó, chọn các private app subnet ở nhiều AZ.

   ![Chọn VPC và private app subnets](/images/5-Workshop/5.5-compute/auto-scaling-group-04.png)

   <p class="image-caption"><em>Hình 4: Chọn VPC và private app subnets</em></p>
   EC2 app không cần public IP. Đặt trong private subnet giúp chỉ ALB gọi được app.


5. **Cấu hình capacity nâng cao**

   Chọn chiến lược phân bố **Balanced best effort**. Sau đó, giữ Capacity Reservation preference ở giá trị **Default**. Tiếp theo, chọn **Next**.

   ![Cấu hình capacity nâng cao](/images/5-Workshop/5.5-compute/auto-scaling-group-05.png)

   <p class="image-caption"><em>Hình 5: Cấu hình capacity nâng cao</em></p>
   Quy trình triển khai ưu tiên cấu hình đơn giản và ổn định. Trong môi trường production, có thể sử dụng mixed instances hoặc Spot Instance để tối ưu chi phí.


6. **Gắn load balancing**

   Chọn tích hợp với load balancer. Sau đó, gắn ASG vào Target Group hiện có `sportbooking-tg`.

   ![Gắn load balancing](/images/5-Workshop/5.5-compute/auto-scaling-group-06.png)

   <p class="image-caption"><em>Hình 6: Gắn load balancing</em></p>
   ASG cần đăng ký EC2 vào Target Group để ALB forward traffic và kiểm tra health.


7. **Cấu hình health checks**

   Bật Elastic Load Balancing health checks. Sau đó, đặt health check grace period là `300` giây. Tiếp theo, chọn **Next**.

   ![Cấu hình health checks](/images/5-Workshop/5.5-compute/auto-scaling-group-07.png)

   <p class="image-caption"><em>Hình 7: Cấu hình health checks</em></p>
   Nếu grace period quá ngắn, ASG có thể thay instance khi app chưa kịp khởi động.


8. **Cấu hình group size và scaling**

   Đặt desired capacity là `2`. Sau đó, đặt minimum capacity `2` và maximum capacity `4`.

   ![Cấu hình group size và scaling](/images/5-Workshop/5.5-compute/auto-scaling-group-08.png)

   <p class="image-caption"><em>Hình 8: Cấu hình group size và scaling</em></p>
   Hai instance được duy trì làm mức nền, đồng thời ASG có thể mở rộng tối đa bốn instance.


9. **Cấu hình target tracking policy**

   Chọn **Target tracking scaling policy**. Sau đó, chọn metric **Average CPU utilization**, target value `60` và instance warmup `300` giây.

   ![Cấu hình target tracking policy](/images/5-Workshop/5.5-compute/auto-scaling-group-09.png)

   <p class="image-caption"><em>Hình 9: Cấu hình target tracking theo CPU trung bình 60%</em></p>
   Chính sách này điều chỉnh desired capacity trong giới hạn từ hai đến bốn instance dựa trên CPU trung bình của Auto Scaling Group.


10. **Hoàn tất các tùy chọn của Auto Scaling Group**

   Giữ deletion protection ở giá trị **None (default)** và không sử dụng placement group. Sau đó, rà soát các tùy chọn monitoring, warmup và instance maintenance. Tiếp theo, chọn **Next**.

   ![Hoàn tất các tùy chọn của Auto Scaling Group](/images/5-Workshop/5.5-compute/auto-scaling-group-10.png)

   <p class="image-caption"><em>Hình 10: Rà soát tùy chọn bảo trì và chuyển sang bước tiếp theo</em></p>
   Các tùy chọn hoàn tất trước khi chuyển sang cấu hình notification và tag.


11. **Bỏ qua notification khi tạo ASG**

   Notification chưa được thêm tại bước tạo ASG. Chọn **Next**.

   ![Bỏ qua hoặc thêm notification](/images/5-Workshop/5.5-compute/auto-scaling-group-11.png)

   <p class="image-caption"><em>Hình 11: Bỏ qua hoặc thêm notification</em></p>
   Kênh SNS được tạo riêng ở phần 5.7 và dùng làm action cho CloudWatch alarm.


12. **Thêm tag cho ASG**

   Thêm tag như `Project=SportBooking`. Sau đó, chọn **Next**.

   ![Thêm tag cho ASG](/images/5-Workshop/5.5-compute/auto-scaling-group-12.png)

   <p class="image-caption"><em>Hình 12: Thêm tag cho ASG</em></p>
   Tag giúp lọc tài nguyên, quản lý chi phí và nhận biết instance thuộc dự án.


13. **Rà soát và tạo ASG**

   Rà soát lại Launch Template, subnets, target group, capacity và tag. Sau đó, chọn **Create Auto Scaling group**.

   ![Rà soát và tạo ASG](/images/5-Workshop/5.5-compute/auto-scaling-group-13.png)

   <p class="image-caption"><em>Hình 13: Rà soát và tạo ASG</em></p>
   Launch Template, private subnet, Target Group, capacity và tag đã được rà soát trước khi tạo ASG.


14. **Kiểm tra ASG sau khi tạo**

   Mở ASG vừa tạo. Sau đó, kiểm tra instance, desired capacity, launch template và health.

   ![Kiểm tra ASG sau khi tạo](/images/5-Workshop/5.5-compute/auto-scaling-group-14.png)

   <p class="image-caption"><em>Hình 14: Kiểm tra ASG sau khi tạo</em></p>
   Đây là bằng chứng compute layer đã chạy và sẵn sàng nhận traffic qua Target Group/ALB.



#### Quy trình Instance Refresh

Trong lần cập nhật phiên bản ứng dụng, build lại file jar và upload đè object release trên S3:

```powershell
.\mvnw.cmd clean package -DskipTests

aws s3 cp target\SportBooingSystem-0.0.1-SNAPSHOT.jar `
  s3://sportbooking-artifacts-3stars-us-east-1/releases/SportBooingSystem-0.0.1-SNAPSHOT.jar `
  --region us-east-1
```

Sau đó, mở **EC2 > Auto Scaling Groups > sportbooking-asg > Instance refresh > Start** và sử dụng:

- Use current Auto Scaling group configuration.
- Instance warmup: 300 giây.
- Skip matching: Off.
- Target Group được theo dõi cho đến khi các instance thay thế chuyển sang `Healthy`.

{{% notice info %}}
Trong lần gặp lỗi Launch Template version rỗng, ASG đã được cập nhật sang version hợp lệ trước khi chạy lại refresh bằng current ASG configuration.
{{% /notice %}}
