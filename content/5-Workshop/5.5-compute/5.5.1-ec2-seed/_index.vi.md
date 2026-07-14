---
title : "EC2 Seed"
date : 2024-01-01
weight : 1
chapter : false
pre : " <b> 5.5.1 </b> "
---

#### Cấu hình EC2 seed

EC2 seed được tạo sau khi object `releases/SportBooingSystem-0.0.1-SNAPSHOT.jar` đã có trong artifact bucket. Instance này được dùng để kiểm tra Amazon Linux, Java runtime, Session Manager, IAM Role và khả năng truy cập các dịch vụ nền trước khi chuẩn hóa thành Launch Template.

#### Báo cáo các bước tạo EC2 seed

EC2 seed chỉ được khởi tạo sau khi S3, IAM, RDS và Secrets Manager sẵn sàng, nhờ đó quá trình kiểm thử có đủ artifact, quyền truy cập và cấu hình ứng dụng.

**Quy trình thực hiện:**

1. **Bắt đầu tạo EC2 seed instance**

   Truy cập **EC2 > Instances**. Sau đó, chọn **Launch instances**.

   ![Bắt đầu tạo EC2 seed instance](/images/5-Workshop/5.5-compute/ec2-seed-01.png)

   <p class="image-caption"><em>Hình 1: Bắt đầu tạo EC2 seed instance</em></p>
   EC2 seed được dùng để xác minh runtime, IAM, mạng và hoạt động của ứng dụng trước khi tạo AMI cho Launch Template.


2. **Đặt tên instance và chọn AMI**

   Nhập tên instance theo dự án. Sau đó, chọn Amazon Linux 2023.

   ![Đặt tên instance và chọn AMI](/images/5-Workshop/5.5-compute/ec2-seed-02.png)

   <p class="image-caption"><em>Hình 2: Đặt tên instance và chọn AMI</em></p>
   Tên instance giúp phân biệt máy seed với EC2 do ASG tạo. AMI phải tương thích với Java/runtime của ứng dụng.


3. **Chọn AMI và instance type**

   Xác nhận AMI đã chọn. Sau đó, chọn instance type `t3.small`.

   ![Chọn AMI và instance type](/images/5-Workshop/5.5-compute/ec2-seed-03.png)

   <p class="image-caption"><em>Hình 3: Chọn AMI và instance type</em></p>
   Seed instance sử dụng `t3.small`, phù hợp với phạm vi kiểm tra runtime và ứng dụng của dự án.


4. **Chọn key pair và network**

   Chọn **Proceed without a key pair** vì instance được quản trị bằng Session Manager. Sau đó, chọn `sportbooking-vpc` và private subnet `sportbooking-app-a`. Tiếp theo, tắt **Auto-assign public IP**.

   ![Chọn key pair và network](/images/5-Workshop/5.5-compute/ec2-seed-04.png)

   <p class="image-caption"><em>Hình 4: Chọn key pair và network</em></p>
   Instance không có public IP và không dùng key pair; toàn bộ phiên quản trị được thực hiện qua Session Manager.


5. **Gắn security group và storage**

   Chọn Security Group `sportbooking-app`. Sau đó, đặt root volume `gp3` dung lượng `20 GiB`.

   ![Gắn security group và storage](/images/5-Workshop/5.5-compute/ec2-seed-05.png)

   <p class="image-caption"><em>Hình 5: Gắn security group và storage</em></p>
   Security Group chỉ nhận traffic ứng dụng từ ALB; root volume lưu runtime, file cấu hình và artifact tải từ S3.


6. **Gắn IAM instance profile**

   Mở **Advanced details**. Sau đó, chọn IAM role `sportbooking-ec2-role`. Tiếp theo, chọn **Launch instance**.

   ![Gắn IAM instance profile](/images/5-Workshop/5.5-compute/ec2-seed-06.png)

   <p class="image-caption"><em>Hình 6: Gắn IAM instance profile</em></p>
   Role cho phép instance đọc artifact S3, đọc secret, thao tác với bucket upload và dùng Session Manager mà không lưu access key.


7. **Kiểm tra instance sau khi launch**

   Mở instance vừa tạo. Sau đó, chờ trạng thái running và status check ổn định.

   ![Kiểm tra instance sau khi launch](/images/5-Workshop/5.5-compute/ec2-seed-07.png)

   <p class="image-caption"><em>Hình 7: Kiểm tra instance sau khi launch</em></p>
   Phiên kết nối được mở sau khi instance ở trạng thái `Running` và hoàn tất status checks.


8. **Chọn Connect**

   Chọn instance. Sau đó, chọn **Connect**.

   ![Chọn Connect](/images/5-Workshop/5.5-compute/ec2-seed-08.png)

   <p class="image-caption"><em>Hình 8: Chọn Connect</em></p>
   Phiên kết nối được dùng để kiểm tra quyền IAM, nạp secret, khởi động ứng dụng và kiểm tra HTTP nội bộ.


9. **Kết nối bằng Session Manager**

   Chọn tab **Session Manager**. Sau đó, chọn **Connect**.

   ![Kết nối bằng Session Manager](/images/5-Workshop/5.5-compute/ec2-seed-09.png)

   <p class="image-caption"><em>Hình 9: Kết nối bằng Session Manager</em></p>
   Session Manager phù hợp private EC2 vì không cần mở SSH inbound hoặc quản lý bastion host.


10. **Kiểm tra terminal EC2**

   Đọc secret `sportbooking/prod/app-env` và tạo `/opt/sportbooking/app.env`. Sau đó, khởi động lại service `sportbooking` và kiểm tra `curl -I http://127.0.0.1:8080/`. Tiếp theo, ghi nhận phản hồi `HTTP/1.1 200` sau khi service khởi động thành công.

   ![Kiểm tra terminal EC2](/images/5-Workshop/5.5-compute/ec2-seed-10.png)

   <p class="image-caption"><em>Hình 10: Kiểm tra terminal EC2</em></p>
   Terminal giúp xác minh EC2 có quyền IAM, network và runtime đúng trước khi tự động hóa bằng Launch Template.
