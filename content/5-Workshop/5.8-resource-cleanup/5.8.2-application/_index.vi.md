---
title : "Dọn dẹp ứng dụng"
date : 2024-01-01
weight : 2
chapter : false
pre : " <b> 5.8.2 </b> "
---

#### 4. Hạ tải và xóa Auto Scaling Group

Auto Scaling Group phải được xử lý trước Launch Template. Việc giảm capacity về `0` giúp các EC2 instance được kết thúc có kiểm soát trước khi xóa cấu hình mở rộng.

**Quy trình thực hiện:**

1. **Mở cấu hình capacity**

   Trên giao diện, thực hiện lần lượt: **(1)** chọn `sportbooking-asg` và **(2)** trong phần **Capacity overview**, chọn **Edit**.

   ![Mở cấu hình capacity của Auto Scaling Group](/images/5-Workshop/5.8-resource-cleanup/cleanup-asg-01.png)

   <p class="image-caption"><em>Hình 1: Mở cấu hình capacity của Auto Scaling Group</em></p>
   ASG đang duy trì hai EC2 instance healthy. Nếu không giảm capacity, ASG tiếp tục tạo instance thay thế khi một instance bị kết thúc.


2. **Đưa capacity về 0**

   Trên giao diện, thực hiện lần lượt: **(1)** đặt **Desired capacity** bằng `0`; **(2)** đặt **Min desired capacity** bằng `0`; và **(3)** chọn **Update**.

   ![Đưa capacity của Auto Scaling Group về 0](/images/5-Workshop/5.8-resource-cleanup/cleanup-asg-02.png)

   <p class="image-caption"><em>Hình 2: Đưa capacity của Auto Scaling Group về 0</em></p>
   Desired và minimum capacity cùng bằng `0` cho phép ASG kết thúc toàn bộ EC2 instance mà không tạo instance mới. Maximum capacity có thể giữ nguyên vì ASG sẽ được xóa ở bước sau.


3. **Kiểm tra ASG không còn instance**

   Làm mới trang cho đến khi cột **Instances** bằng `0`. Sau đó, kiểm tra **Desired capacity** và giới hạn tối thiểu đều bằng `0`.

   ![Kiểm tra Auto Scaling Group đã hạ về 0](/images/5-Workshop/5.8-resource-cleanup/cleanup-asg-03.png)

   <p class="image-caption"><em>Hình 3: Kiểm tra Auto Scaling Group đã hạ về 0</em></p>
   Chỉ xóa ASG sau khi các EC2 instance đã kết thúc để giảm nguy cơ còn ENI hoặc EBS volume đang được sử dụng.


4. **Bắt đầu xóa ASG**

   Trên giao diện, thực hiện lần lượt: **(1)** mở menu **Actions** và **(2)** chọn **Delete**.

   ![Bắt đầu xóa Auto Scaling Group](/images/5-Workshop/5.8-resource-cleanup/cleanup-asg-04.png)

   <p class="image-caption"><em>Hình 4: Bắt đầu xóa Auto Scaling Group</em></p>
   Khi không còn instance, ASG có thể được xóa mà không ảnh hưởng đến dữ liệu đang chạy trên EC2.


5. **Xác nhận xóa ASG**

   Trên giao diện, thực hiện lần lượt: **(1)** nhập `delete` vào ô xác nhận và **(2)** chọn **Delete**.

   ![Xác nhận xóa Auto Scaling Group](/images/5-Workshop/5.8-resource-cleanup/cleanup-asg-05.png)

   <p class="image-caption"><em>Hình 5: Xác nhận xóa Auto Scaling Group</em></p>
   Sau khi ASG bị xóa, Launch Template không còn được cấu hình mở rộng tự động tham chiếu.


{{% notice tip %}}
Sau khi xóa ASG, kiểm tra thêm **EC2 > Instances** và **Elastic Block Store > Volumes**. Chỉ giữ volume hoặc snapshot đã được xác định là bản sao lưu cần thiết.
{{% /notice %}}

#### 5. Xóa Application Load Balancer

ALB phải được xóa trước Target Group vì listener của ALB đang tham chiếu Target Group.

**Quy trình thực hiện:**

1. **Chọn và xóa ALB**

   Trên giao diện, thực hiện lần lượt: **(1)** truy cập **EC2 > Load Balancers** và chọn `sportbooking-alb`; **(2)** mở menu **Actions**; và **(3)** chọn **Delete load balancer**, sau đó xác nhận trong hộp thoại được hiển thị.

   ![Chọn Application Load Balancer cần xóa](/images/5-Workshop/5.8-resource-cleanup/cleanup-alb-01.png)

   <p class="image-caption"><em>Hình 6: Chọn Application Load Balancer cần xóa</em></p>
   Xóa ALB đồng thời loại bỏ các listener HTTP/HTTPS và giải phóng liên kết sử dụng ACM certificate. Cần chờ ALB biến mất khỏi danh sách trước khi xóa Target Group.


#### 6. Xóa Target Group

**Quy trình thực hiện:**

1. **Chọn Target Group**

   Trên giao diện, thực hiện lần lượt: **(1)** truy cập **EC2 > Target Groups** và chọn `sportbooking-tg`; **(2)** mở menu **Actions**; và **(3)** chọn **Delete**.

   ![Chọn Target Group cần xóa](/images/5-Workshop/5.8-resource-cleanup/cleanup-target-group-01.png)

   <p class="image-caption"><em>Hình 7: Chọn Target Group cần xóa</em></p>
   Target Group chỉ được xóa sau khi ALB và listener không còn tham chiếu. Nếu AWS báo resource đang được sử dụng, cần kiểm tra lại ALB hoặc listener rule còn tồn tại.


2. **Xác nhận xóa Target Group**

   Kiểm tra tên `sportbooking-tg`. Sau đó, chọn **Delete**.

   ![Xác nhận xóa Target Group](/images/5-Workshop/5.8-resource-cleanup/cleanup-target-group-02.png)

   <p class="image-caption"><em>Hình 8: Xác nhận xóa Target Group</em></p>
   Các target EC2 đã được kết thúc khi ASG giảm về `0`, do đó Target Group không còn backend cần duy trì.


#### 7. Xóa Launch Template

**Quy trình thực hiện:**

1. **Chọn Launch Template**

   Trên giao diện, thực hiện lần lượt: **(1)** truy cập **EC2 > Launch Templates** và chọn `sportbooking-lt`; **(2)** mở menu **Actions**; và **(3)** chọn **Delete template**.

   ![Chọn Launch Template cần xóa](/images/5-Workshop/5.8-resource-cleanup/cleanup-launch-template-01.png)

   <p class="image-caption"><em>Hình 9: Chọn Launch Template cần xóa</em></p>
   Launch Template được xóa sau ASG để không còn cấu hình nào tham chiếu đến template hoặc các version của template.


2. **Xác nhận xóa Launch Template**

   Trên giao diện, thực hiện lần lượt: **(1)** nhập `Delete` đúng chữ hoa, chữ thường theo yêu cầu trên màn hình và **(2)** chọn **Delete**.

   ![Xác nhận xóa Launch Template](/images/5-Workshop/5.8-resource-cleanup/cleanup-launch-template-02.png)

   <p class="image-caption"><em>Hình 10: Xác nhận xóa Launch Template</em></p>
   Xóa template sẽ xóa toàn bộ version của `sportbooking-lt`; thao tác này không thể hoàn tác.


#### 8. Gỡ policy và xóa IAM Role

IAM Role chỉ được xóa sau khi các EC2 instance đã kết thúc và không còn instance profile đang được sử dụng.

**Quy trình thực hiện:**

1. **Chọn các policy đang gắn với role**

   Trên giao diện, thực hiện lần lượt: **(1)** mở role `sportbooking-ec2-role` và chọn toàn bộ policy đang được attach và **(2)** chọn **Remove**.

   ![Chọn các policy cần gỡ khỏi IAM Role](/images/5-Workshop/5.8-resource-cleanup/cleanup-iam-role-01.png)

   <p class="image-caption"><em>Hình 11: Chọn các policy cần gỡ khỏi IAM Role</em></p>
   Role trong hình đang gắn các AWS managed policy và customer managed policy `sportbooking-ec2-policy`. Cần gỡ các liên kết này trước khi xóa role.


2. **Xác nhận gỡ policy**

   Kiểm tra số lượng policy đã chọn. Sau đó, chọn **Remove**.

   ![Xác nhận gỡ policy khỏi IAM Role](/images/5-Workshop/5.8-resource-cleanup/cleanup-iam-role-02.png)

   <p class="image-caption"><em>Hình 12: Xác nhận gỡ policy khỏi IAM Role</em></p>
   AWS managed policy chỉ được detach khỏi role, không bị xóa khỏi AWS account.


3. **Chọn role cần xóa**

   Trên giao diện, thực hiện lần lượt: **(1)** quay lại **IAM > Roles** và chọn `sportbooking-ec2-role` và **(2)** chọn **Delete**.

   ![Chọn IAM Role cần xóa](/images/5-Workshop/5.8-resource-cleanup/cleanup-iam-role-03.png)

   <p class="image-caption"><em>Hình 13: Chọn IAM Role cần xóa</em></p>
   Chỉ role của dự án được chọn. Không xóa service-linked role hoặc role dùng chung với tài nguyên khác.


4. **Xác nhận xóa role**

   Nhập chính xác `sportbooking-ec2-role`. Sau đó, chọn **Delete**.

   ![Xác nhận xóa IAM Role](/images/5-Workshop/5.8-resource-cleanup/cleanup-iam-role-04.png)

   <p class="image-caption"><em>Hình 14: Xác nhận xóa IAM Role</em></p>
   Xóa role loại bỏ quyền mà EC2 đã sử dụng để truy cập S3, Secrets Manager, Systems Manager và CloudWatch.


{{% notice info %}}
Customer managed policy `sportbooking-ec2-policy` vẫn tồn tại sau khi detach. Nếu policy chỉ phục vụ dự án, vào **IAM > Policies**, xác nhận không còn attached entity rồi xóa policy và các version không mặc định của policy.
{{% /notice %}}
