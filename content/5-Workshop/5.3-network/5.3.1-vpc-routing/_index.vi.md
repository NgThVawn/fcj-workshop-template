---
title : "VPC và định tuyến"
date : 2024-01-01
weight : 1
chapter : false
pre : " <b> 5.3.1 </b> "
aliases:
  - /vi/5-workshop/5.3-s3-vpc/5.3.1-create-gwe/
---

#### Cấu hình VPC và subnet

Tạo một VPC riêng cho SportBooking và chia subnet thành ba lớp:

- Public subnet: đặt Application Load Balancer và NAT Gateway.
- Private app subnet: đặt EC2 app instances do Auto Scaling Group quản lý.
- Private DB subnet: đặt RDS MySQL thông qua DB subnet group.

| Thành phần | Cấu hình |
|---|---|
| VPC | `sportbooking-vpc`, CIDR `10.20.0.0/16`. |
| Public subnet | 2 subnet ở 2 AZ, có route ra Internet Gateway. |
| Private app subnet | 2 subnet ở 2 AZ, có outbound qua NAT Gateway. |
| Private DB subnet | 2 subnet ở 2 AZ, chỉ dùng route local trong VPC. |

#### Internet Gateway, NAT Gateway và route table

| Route/NAT | Cấu hình | Vai trò |
|---|---|---|
| Public route table | `0.0.0.0/0 -> Internet Gateway` | Cho ALB public nhận request từ Internet. |
| Private app route table | `0.0.0.0/0 -> NAT Gateway` | Cho EC2 private tải package, pull S3 và gọi Secrets Manager khi chưa dùng endpoint riêng. |
| Private DB route table | Chỉ route local trong VPC | RDS không cần Internet inbound/outbound trong bài triển khai cơ bản. |
| NAT Gateway | 2 NAT Gateway đặt tại 2 public subnet và gắn Elastic IP | Mỗi private app subnet sử dụng NAT Gateway cùng AZ để có đường outbound độc lập. |

#### Đối chiếu cấu hình mạng

1. Mở VPC console và đối chiếu Resource map.
2. Xác nhận public subnet liên kết với route table có default route tới Internet Gateway.
3. Xác nhận mỗi private app subnet liên kết với route table có default route tới NAT Gateway.
4. Xác nhận private DB subnet không public và chỉ sử dụng route nội bộ cần thiết.
5. Kiểm tra hai NAT Gateway ở trạng thái `Available`.

{{% notice info %}}
Sau khi workflow hoàn tất, đối chiếu Resource map với thiết kế gồm CIDR `10.20.0.0/16`, hai Availability Zone, hai public subnet, bốn private subnet, các route mặc định và hai NAT Gateway ở trạng thái `Available`.
{{% /notice %}}

#### Quy trình thực hiện

Quy trình bắt đầu từ VPC, sau đó tạo đồng bộ subnet, Internet Gateway, NAT Gateway và route table cho môi trường SportBooking.

**Quy trình thực hiện:**

1. **Mở trang tạo VPC**

   Từ AWS Management Console, truy cập **VPC > Your VPCs**, kiểm tra Region `us-east-1` rồi chọn **Create VPC** để bắt đầu khởi tạo lớp mạng riêng cho dự án.

   ![Mở màn hình tạo VPC](/images/5-Workshop/5.3-network/vpc-01.png)

   <p class="image-caption"><em>Hình 1: Mở màn hình tạo VPC</em></p>
   VPC là lớp mạng riêng của dự án. Tạo VPC riêng giúp tách hạ tầng SportBooking khỏi default VPC và dễ kiểm soát subnet, route table, NAT Gateway.


2. **Chọn chế độ VPC and more**

   Chọn **VPC and more** để AWS tạo đồng bộ VPC, subnet, route table và gateway. Sau đó, nhập tên VPC `sportbooking-vpc`. Tiếp theo, đặt CIDR IPv4 riêng là `10.20.0.0/16`.

   ![Chọn kiểu tạo VPC and more](/images/5-Workshop/5.3-network/vpc-02.png)

   <p class="image-caption"><em>Hình 2: Chọn kiểu tạo VPC and more</em></p>
   Tùy chọn này phù hợp với quy trình triển khai vì tạo đủ network foundation trong một luồng, hạn chế sai sót khi tự tạo từng subnet và route table rời rạc.


3. **Cấu hình số AZ, subnet public/private và NAT**

   Chọn **2 Availability Zones**. Sau đó, tạo public subnet cho ALB/NAT Gateway và private subnet cho EC2/RDS. Tiếp theo, chọn **2 NAT Gateway**, mỗi Availability Zone một tài nguyên.

   ![Cấu hình số AZ, subnet public/private và NAT](/images/5-Workshop/5.3-network/vpc-03.png)

   <p class="image-caption"><em>Hình 3: Cấu hình số AZ, subnet public/private và NAT</em></p>
   Hai AZ giúp ALB và Auto Scaling Group có khả năng chịu lỗi cơ bản. NAT Gateway cho EC2 private outbound Internet mà không mở inbound từ Internet.


4. **Kiểm tra lại cấu hình trước khi tạo**

   Rà soát lại tên VPC, CIDR, số AZ, số subnet và NAT Gateway. Sau đó, cuộn xuống cuối trang. Tiếp theo, chọn **Create VPC**.

   ![Kiểm tra lại cấu hình trước khi tạo](/images/5-Workshop/5.3-network/vpc-04.png)

   <p class="image-caption"><em>Hình 4: Kiểm tra lại cấu hình trước khi tạo</em></p>
   Cấu hình mạng được đối chiếu trước khi tạo ALB, ASG và RDS để tránh sai Availability Zone hoặc subnet.


5. **Theo dõi tiến trình Create VPC workflow**

   Theo dõi danh sách tài nguyên đang được tạo. Sau đó, chờ các mục chuyển sang trạng thái hoàn tất.

   ![Theo dõi tiến trình Create VPC workflow](/images/5-Workshop/5.3-network/vpc-05.png)

   <p class="image-caption"><em>Hình 5: Theo dõi tiến trình Create VPC workflow</em></p>
   Workflow cho biết tài nguyên nào đã tạo thành công và tài nguyên nào lỗi, đặc biệt là route table, NAT Gateway và subnet associations.


6. **Hoàn tất workflow**

   Kiểm tra danh sách resource đã tạo. Sau đó, chọn **View VPC** hoặc quay lại danh sách VPC.

   ![Hoàn tất workflow](/images/5-Workshop/5.3-network/vpc-06.png)

   <p class="image-caption"><em>Hình 6: Hoàn tất workflow</em></p>
   Sau khi workflow thành công, cần mở VPC để kiểm tra Resource map và dùng VPC này cho RDS, ALB, Launch Template, ASG.


7. **Kiểm tra Resource map của VPC**

   Mở VPC vừa tạo. Sau đó, xem **Resource map**. Tiếp theo, xác nhận public subnet, private subnet, route table, Internet Gateway và NAT Gateway đúng như thiết kế.

   ![Kiểm tra Resource map của VPC](/images/5-Workshop/5.3-network/vpc-07.png)

   <p class="image-caption"><em>Hình 7: Kiểm tra Resource map của VPC</em></p>
   Resource map là bằng chứng trực quan nhất để giải thích kiến trúc mạng trong báo cáo: ALB ở public subnet, EC2/RDS ở private subnet.




