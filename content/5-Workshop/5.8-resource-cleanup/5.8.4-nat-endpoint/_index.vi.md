---
title : "NAT và Endpoint"
date : 2024-01-01
weight : 4
chapter : false
pre : " <b> 5.8.4 </b> "
---

#### 13. Xóa NAT Gateway

Hai NAT Gateway được xóa trước Elastic IP và VPC. Cần chờ từng NAT Gateway chuyển sang `Deleted` để Elastic IP không còn association.

**Quy trình thực hiện:**

1. **Chọn NAT Gateway tại Availability Zone thứ nhất**

   Trên giao diện, thực hiện lần lượt: **(1)** chọn `sportbooking-nat-a`; **(2)** mở menu **Actions**; và **(3)** chọn **Delete NAT gateway**.

   ![Chọn NAT Gateway thứ nhất cần xóa](/images/5-Workshop/5.8-resource-cleanup/cleanup-nat-gateway-01.png)

   <p class="image-caption"><em>Hình 1: Chọn NAT Gateway thứ nhất cần xóa</em></p>
   EC2 private đã được kết thúc nên không còn nhu cầu truy cập Internet thông qua NAT Gateway.


2. **Xác nhận xóa NAT Gateway thứ nhất**

   Trên giao diện, thực hiện lần lượt: **(1)** nhập `delete` và **(2)** chọn **Delete**.

   ![Xác nhận xóa NAT Gateway thứ nhất](/images/5-Workshop/5.8-resource-cleanup/cleanup-nat-gateway-02.png)

   <p class="image-caption"><em>Hình 2: Xác nhận xóa NAT Gateway thứ nhất</em></p>
   NAT Gateway đã chuyển qua trạng thái `Deleting` trong thời gian AWS xử lý yêu cầu.


3. **Chọn NAT Gateway tại Availability Zone thứ hai**

   Kiểm tra thông báo xóa NAT Gateway thứ nhất. Sau đó, **(1)** chọn `sportbooking-nat-b`; **(2)** mở menu **Actions**; và **(3)** chọn **Delete NAT gateway**.

   ![Chọn NAT Gateway thứ hai cần xóa](/images/5-Workshop/5.8-resource-cleanup/cleanup-nat-gateway-03.png)

   <p class="image-caption"><em>Hình 3: Chọn NAT Gateway thứ hai cần xóa</em></p>
   Kiến trúc sử dụng NAT Gateway theo hai Availability Zone, do đó cần xử lý cả hai tài nguyên.


4. **Xác nhận xóa NAT Gateway thứ hai**

   Trên giao diện, thực hiện lần lượt: **(1)** nhập `delete` và **(2)** chọn **Delete**.

   ![Xác nhận xóa NAT Gateway thứ hai](/images/5-Workshop/5.8-resource-cleanup/cleanup-nat-gateway-04.png)

   <p class="image-caption"><em>Hình 4: Xác nhận xóa NAT Gateway thứ hai</em></p>
   Cả hai NAT Gateway phải được xóa để VPC không còn tài nguyên mạng có tính phí theo giờ.


5. **Kiểm tra trạng thái hai NAT Gateway**

   Làm mới danh sách cho đến khi `sportbooking-nat-a` và `sportbooking-nat-b` đều ở trạng thái **Deleted**.

   ![Kiểm tra hai NAT Gateway đã được xóa](/images/5-Workshop/5.8-resource-cleanup/cleanup-nat-gateway-05.png)

   <p class="image-caption"><em>Hình 5: Kiểm tra hai NAT Gateway đã được xóa</em></p>
   Chỉ giải phóng Elastic IP sau khi NAT Gateway đã xóa xong; nếu thực hiện quá sớm, địa chỉ vẫn đang được association và không thể release.


{{% notice info %}}
Làm mới trạng thái định kỳ và chỉ chuyển sang giải phóng Elastic IP sau khi cả hai NAT Gateway hiển thị `Deleted`.
{{% /notice %}}

#### 14. Giải phóng Elastic IP

**Quy trình thực hiện:**

1. **Chọn các Elastic IP của NAT Gateway**

   Trên giao diện, thực hiện lần lượt: **(1)** chọn `sportbooking-eip-us-east-1a` và `sportbooking-eip-us-east-1b`; **(2)** mở menu **Actions**; và **(3)** chọn **Release Elastic IP addresses**.

   ![Chọn Elastic IP cần giải phóng](/images/5-Workshop/5.8-resource-cleanup/cleanup-elastic-ip-01.png)

   <p class="image-caption"><em>Hình 6: Chọn Elastic IP cần giải phóng</em></p>
   Sau khi NAT Gateway bị xóa, hai Elastic IP không còn association và cần được trả lại AWS.


2. **Xác nhận giải phóng Elastic IP**

   Kiểm tra đúng hai địa chỉ IP và Allocation ID. Sau đó, chọn **Release**.

   ![Xác nhận giải phóng Elastic IP](/images/5-Workshop/5.8-resource-cleanup/cleanup-elastic-ip-02.png)

   <p class="image-caption"><em>Hình 7: Xác nhận giải phóng Elastic IP</em></p>
   Elastic IP đã release không còn thuộc AWS account và có thể được cấp cho tài khoản khác.


3. **Kiểm tra danh sách Elastic IP**

   Xác nhận thông báo **Elastic IP addresses released**. Sau đó, kiểm tra không còn Elastic IP của dự án trong Region.

   ![Kiểm tra Elastic IP đã được giải phóng](/images/5-Workshop/5.8-resource-cleanup/cleanup-elastic-ip-03.png)

   <p class="image-caption"><em>Hình 8: Kiểm tra Elastic IP đã được giải phóng</em></p>
   Kết quả này xác nhận lớp NAT không còn địa chỉ IPv4 public được cấp phát riêng.


#### 15. Xóa S3 Gateway VPC Endpoint

**Quy trình thực hiện:**

1. **Chọn VPC Endpoint**

   Trên giao diện, thực hiện lần lượt: **(1)** chọn `sportbooking-vpce-s3`; **(2)** mở menu **Actions**; và **(3)** chọn **Delete VPC endpoints**.

   ![Chọn S3 Gateway VPC Endpoint cần xóa](/images/5-Workshop/5.8-resource-cleanup/cleanup-vpc-endpoint-01.png)

   <p class="image-caption"><em>Hình 9: Chọn S3 Gateway VPC Endpoint cần xóa</em></p>
   EC2 và S3 bucket đã được xử lý nên endpoint không còn lưu lượng cần phục vụ. Xóa endpoint cũng loại bỏ liên kết endpoint khỏi route table.


2. **Xác nhận xóa endpoint**

   Trên giao diện, thực hiện lần lượt: **(1)** nhập `delete` và **(2)** chọn **Delete**.

   ![Xác nhận xóa S3 Gateway VPC Endpoint](/images/5-Workshop/5.8-resource-cleanup/cleanup-vpc-endpoint-02.png)

   <p class="image-caption"><em>Hình 10: Xác nhận xóa S3 Gateway VPC Endpoint</em></p>
   Endpoint bị xóa vĩnh viễn và không thể khôi phục từ cấu hình hiện tại.


3. **Kiểm tra endpoint đã được xóa**

   Xác nhận thông báo **Successfully deleted endpoints**. Sau đó, kiểm tra danh sách không còn `sportbooking-vpce-s3`.

   ![Kiểm tra S3 Gateway VPC Endpoint đã được xóa](/images/5-Workshop/5.8-resource-cleanup/cleanup-vpc-endpoint-03.png)

   <p class="image-caption"><em>Hình 11: Kiểm tra S3 Gateway VPC Endpoint đã được xóa</em></p>
   VPC không còn endpoint tùy chỉnh có thể cản trở thao tác xóa cuối cùng.
