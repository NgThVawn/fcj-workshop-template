---
title : "Security Group và VPC"
date : 2024-01-01
weight : 5
chapter : false
pre : " <b> 5.8.5 </b> "
---

#### 16. Gỡ rule tham chiếu và xóa Security Group

Security Group không thể xóa khi còn gắn với ENI hoặc bị Security Group khác tham chiếu. ALB, EC2 và RDS đã được xóa trước khi thực hiện phần này.

**Quy trình thực hiện:**

1. **Mở outbound rules của ALB Security Group**

   Mở `sportbooking-alb` Security Group. Sau đó, chọn tab **Outbound rules** và chọn **Edit outbound rules**.

   ![Mở outbound rules của ALB Security Group](/images/5-Workshop/5.8-resource-cleanup/cleanup-security-group-01.png)

   <p class="image-caption"><em>Hình 1: Mở outbound rules của ALB Security Group</em></p>
   Outbound rule port `8080` đang tham chiếu EC2 App Security Group. Tham chiếu này cần được gỡ trước khi xóa đồng thời các Security Group của dự án.


2. **Xóa outbound rule tham chiếu**

   Trên giao diện, thực hiện lần lượt: **(1)** chọn **Delete** tại rule port `8080` trỏ đến App Security Group và **(2)** chọn **Save rules**.

   ![Xóa outbound rule tham chiếu App Security Group](/images/5-Workshop/5.8-resource-cleanup/cleanup-security-group-02.png)

   <p class="image-caption"><em>Hình 2: Xóa outbound rule tham chiếu App Security Group</em></p>
   Việc xóa rule phá vỡ liên kết từ ALB Security Group đến App Security Group.


3. **Mở inbound rules của ALB Security Group**

   Chọn tab **Inbound rules**. Sau đó, chọn **Edit inbound rules**.

   ![Mở inbound rules của ALB Security Group](/images/5-Workshop/5.8-resource-cleanup/cleanup-security-group-03.png)

   <p class="image-caption"><em>Hình 3: Mở inbound rules của ALB Security Group</em></p>
   Hai rule HTTP và HTTPS public được loại bỏ để đóng hoàn toàn đường truy cập vào Security Group trước khi xóa.


4. **Xóa các inbound rule còn lại**

   Trên giao diện, thực hiện lần lượt: **(1)** xóa rule HTTPS port `443`; **(2)** xóa rule HTTP port `80`; và **(3)** chọn **Save rules**.

   ![Xóa inbound rules của ALB Security Group](/images/5-Workshop/5.8-resource-cleanup/cleanup-security-group-04.png)

   <p class="image-caption"><em>Hình 4: Xóa inbound rules của ALB Security Group</em></p>
   Các rule dùng CIDR không tạo phụ thuộc giữa Security Group, nhưng việc xóa chúng giúp xác nhận Security Group không còn cho phép lưu lượng trong thời gian chờ dọn dẹp.


5. **Chọn các Security Group của dự án**

   Kiểm tra và gỡ tương tự mọi rule tham chiếu giữa `sportbooking-app`, `sportbooking-rds` và `sportbooking-alb` nếu còn tồn tại. Sau đó, **(1)** chọn đúng ba Security Group của dự án và **(2)** mở **Actions**, chọn **Delete security groups**.

   ![Chọn các Security Group của SportBooking](/images/5-Workshop/5.8-resource-cleanup/cleanup-security-group-05.png)

   <p class="image-caption"><em>Hình 5: Chọn các Security Group của SportBooking</em></p>
   Không chọn Security Group `default` hoặc Security Group thuộc VPC khác. Nếu AWS báo dependency, kiểm tra ENI và rule tham chiếu còn lại trước khi thử lại.


6. **Xác nhận xóa Security Group**

   Kiểm tra danh sách gồm `sportbooking-rds`, `sportbooking-app` và `sportbooking-alb`. Tiếp theo, **(1)** nhập `delete` và **(2)** chọn **Delete**.

   ![Xác nhận xóa các Security Group](/images/5-Workshop/5.8-resource-cleanup/cleanup-security-group-06.png)

   <p class="image-caption"><em>Hình 6: Xác nhận xóa các Security Group</em></p>
   Xóa đồng thời các Security Group sau khi gỡ tham chiếu giúp tránh để lại cấu hình mạng không còn tài nguyên sử dụng.


7. **Kiểm tra kết quả**

   Xác nhận thông báo **Successfully deleted 3 security groups**. Sau đó, kiểm tra chỉ còn các Security Group mặc định hoặc thuộc môi trường khác.

   ![Kiểm tra các Security Group đã được xóa](/images/5-Workshop/5.8-resource-cleanup/cleanup-security-group-07.png)

   <p class="image-caption"><em>Hình 7: Kiểm tra các Security Group đã được xóa</em></p>
   Thành công ở bước này cho thấy các ENI và liên kết Security Group của ALB, EC2 và RDS đã được giải phóng.


#### 17. Xóa DB Subnet Group

DB Subnet Group được xóa sau RDS vì DB instance đang sử dụng subnet group sẽ ngăn thao tác xóa.

**Quy trình thực hiện:**

1. **Chọn DB Subnet Group**

   Trên giao diện, thực hiện lần lượt: **(1)** truy cập **RDS > Subnet groups** và chọn `sportbooking-db-subnet-group` và **(2)** chọn **Delete**.

   ![Chọn DB Subnet Group cần xóa](/images/5-Workshop/5.8-resource-cleanup/cleanup-db-subnet-group-01.png)

   <p class="image-caption"><em>Hình 8: Chọn DB Subnet Group cần xóa</em></p>
   RDS đã được xóa hoàn tất nên subnet group không còn database nào tham chiếu.


2. **Xác nhận xóa DB Subnet Group**

   Kiểm tra đúng tên `sportbooking-db-subnet-group`. Sau đó, chọn **Delete**.

   ![Xác nhận xóa DB Subnet Group](/images/5-Workshop/5.8-resource-cleanup/cleanup-db-subnet-group-02.png)

   <p class="image-caption"><em>Hình 9: Xác nhận xóa DB Subnet Group</em></p>
   Thao tác này chỉ xóa cấu hình DB Subnet Group của RDS, không xóa subnet VPC trực tiếp.


3. **Kiểm tra DB Subnet Group đã được xóa**

   Xác nhận thông báo xóa thành công. Sau đó, kiểm tra danh sách chỉ còn subnet group mặc định hoặc của môi trường khác.

   ![Kiểm tra DB Subnet Group đã được xóa](/images/5-Workshop/5.8-resource-cleanup/cleanup-db-subnet-group-03.png)

   <p class="image-caption"><em>Hình 10: Kiểm tra DB Subnet Group đã được xóa</em></p>
   VPC không còn cấu hình RDS tùy chỉnh phụ thuộc vào các private subnet.


#### 18. Xóa VPC

VPC được xóa cuối cùng sau khi các tài nguyên compute, database, endpoint, NAT Gateway và Security Group đã được xử lý.

**Quy trình thực hiện:**

1. **Chọn VPC của dự án**

   Trên giao diện, thực hiện lần lượt: **(1)** truy cập **VPC > Your VPCs** và chọn `sportbooking-vpc`; **(2)** mở menu **Actions**; và **(3)** chọn **Delete VPC**.

   ![Chọn VPC của SportBooking cần xóa](/images/5-Workshop/5.8-resource-cleanup/cleanup-vpc-01.png)

   <p class="image-caption"><em>Hình 11: Chọn VPC của SportBooking cần xóa</em></p>
   Tên, VPC ID và CIDR đã được đối chiếu để không chọn nhầm default VPC hoặc VPC của môi trường khác.


2. **Kiểm tra tài nguyên phụ thuộc và xác nhận**

   Kiểm tra danh sách các tài nguyên sẽ bị xóa cùng VPC, gồm Internet Gateway, subnet và route table còn lại. Tiếp theo, **(1)** nhập `delete` và **(2)** chọn **Delete**.

   ![Kiểm tra tài nguyên phụ thuộc và xác nhận xóa VPC](/images/5-Workshop/5.8-resource-cleanup/cleanup-vpc-02.png)

   <p class="image-caption"><em>Hình 12: Kiểm tra tài nguyên phụ thuộc và xác nhận xóa VPC</em></p>
   Hộp thoại cho biết `sportbooking-vpc` và 12 tài nguyên mạng liên quan nằm trong yêu cầu xóa; toàn bộ danh sách đã được xác nhận thuộc dự án.


3. **Kiểm tra VPC đã được xóa**

   Xác nhận thông báo **successfully deleted sportbooking-vpc and 12 other resources**. Sau đó, kiểm tra danh sách chỉ còn default VPC hoặc các VPC khác cần giữ.

   ![Kiểm tra VPC của SportBooking đã được xóa](/images/5-Workshop/5.8-resource-cleanup/cleanup-vpc-03.png)

   <p class="image-caption"><em>Hình 13: Kiểm tra VPC của SportBooking đã được xóa</em></p>
   Đây là bước hoàn tất dọn dẹp hạ tầng mạng của SportBooking.


{{% notice warning %}}
Nếu VPC không thể xóa, kiểm tra các ENI còn tồn tại trong **EC2 > Network Interfaces**, VPC Endpoint, NAT Gateway, Load Balancer, RDS, Security Group tham chiếu chéo và tài nguyên do dịch vụ khác quản lý. Không xóa default VPC để xử lý lỗi phụ thuộc.
{{% /notice %}}
