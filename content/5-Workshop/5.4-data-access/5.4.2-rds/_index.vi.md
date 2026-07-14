---
title : "Amazon RDS"
date : 2024-01-01
weight : 2
chapter : false
pre : " <b> 5.4.2 </b> "
aliases:
  - /vi/5-workshop/5.4-s3-onprem/5.4.2-rds/
---

#### Cấu hình RDS

Tạo DB subnet group gồm hai private DB subnet tại hai Availability Zone, sau đó triển khai RDS for MySQL theo mô hình Multi-AZ. Database không sử dụng public subnet và không bật truy cập công khai.

| Mục | Cấu hình |
|---|---|
| Subnet group | 2 private DB subnets ở 2 AZ. |
| Deployment | Multi-AZ DB instance deployment, gồm primary và standby. |
| DB instance class | `db.t3.small`. |
| Storage | General Purpose SSD `gp3`, dung lượng `20 GiB`. |
| Public access | No. |
| Security Group | `sportbooking-rds`, chỉ cho phép `3306` từ `sportbooking-app`. |
| Database name | `sport_booking`. |
| Master username | `sportbooking_ad`. |
| Endpoint | `sportbooking-mysql.cudgu8qk8z19.us-east-1.rds.amazonaws.com`. |
| Monitoring | Database Insights Standard; Enhanced Monitoring mỗi 60 giây. |
| Backup | Automated backup, retention 7 ngày. |

#### Lựa chọn cấu hình

1. Chọn engine MySQL và template Production.
2. Đặt DB identifier `sportbooking-mysql`.
3. Chọn VPC `sportbooking-vpc` và DB subnet group private.
4. Chọn Multi-AZ DB instance deployment.
5. Tắt Public access và gắn `sportbooking-rds`.
6. Tạo database ban đầu `sport_booking`.

Đối chiếu các thông tin sau trong tab **Connectivity & security**:

- Endpoint và port `3306` đã được ghi nhận để tạo `DB_URL` ở phần Secrets Manager.
- Thuộc tính **Publicly accessible** có giá trị `No`.
- VPC Security Group là `sportbooking-rds`, chỉ nhận kết nối từ `sportbooking-app`.

Việc kiểm tra kết nối từ EC2 chỉ được thực hiện sau khi lớp compute đã được triển khai ở phần 5.5. Kết quả truy vấn schema và dữ liệu được ghi nhận tại phần 5.7, do đó không phát sinh bước sử dụng EC2 trước khi EC2 tồn tại.

{{% notice warning %}}
RDS đã được cấu hình **Public access: No**. Rule MySQL port `3306` chỉ nhận nguồn từ Security Group `sportbooking-app`.
{{% /notice %}}

#### Quy trình thực hiện

Quy trình tạo RDS bắt đầu với DB subnet group, tiếp tục bằng database MySQL private và kết thúc bằng việc đối chiếu endpoint cùng Security Group.

**Quy trình thực hiện:**

1. **Mở chức năng tạo DB subnet group**

   Từ Amazon RDS, mở khu vực quản lý cơ sở dữ liệu, chuyển sang **Subnet groups** để chuẩn bị DB Subnet Group riêng cho lớp dữ liệu.

   ![Bắt đầu tạo DB subnet group](/images/5-Workshop/5.4-data-access/rds-01.png)

   <p class="image-caption"><em>Hình 1: Bắt đầu tạo DB subnet group</em></p>
   RDS cần DB subnet group để biết database được đặt vào các private DB subnet nào.


2. **Khởi tạo DB subnet group**

   Trong danh sách **Subnet groups**, chọn **Create DB subnet group**. Thành phần này phải được tạo trước database để RDS có thể phân bố hai instance trên các private DB subnet ở hai Availability Zone.

   ![Mở danh sách Subnet groups](/images/5-Workshop/5.4-data-access/rds-02.png)

   <p class="image-caption"><em>Hình 2: Mở danh sách Subnet groups</em></p>
   Tạo subnet group trước giúp khi tạo RDS chỉ cần chọn đúng tập hợp private DB subnet đã chuẩn bị.


3. **Nhập thông tin DB subnet group**

   Nhập tên subnet group `sportbooking-db-subnet-group`. Sau đó, nhập mô tả. Tiếp theo, chọn đúng VPC SportBooking.

   ![Nhập thông tin DB subnet group](/images/5-Workshop/5.4-data-access/rds-03.png)

   <p class="image-caption"><em>Hình 3: Nhập thông tin DB subnet group</em></p>
   Subnet group phải cùng VPC với EC2 app. Nếu chọn nhầm VPC, app không thể kết nối database bằng private network.


4. **Chọn AZ và private DB subnets**

   Chọn các Availability Zones dùng cho DB. Sau đó, chọn các private DB subnet. Tiếp theo, chọn **Create**.

   ![Chọn AZ và private DB subnets](/images/5-Workshop/5.4-data-access/rds-04.png)

   <p class="image-caption"><em>Hình 4: Chọn AZ và private DB subnets</em></p>
   Hai private DB subnet ở hai AZ tạo nền tảng mạng cho RDS Multi-AZ và tách database khỏi Internet.


5. **Bắt đầu tạo database MySQL**

   Chọn **Create database**. Sau đó, chọn engine **MySQL**. Tiếp theo, chọn template **Production**.

   ![Bắt đầu tạo database MySQL](/images/5-Workshop/5.4-data-access/rds-05.png)

   <p class="image-caption"><em>Hình 5: Bắt đầu tạo database MySQL</em></p>
   SportBooking dùng MySQL nên RDS MySQL là lựa chọn managed database phù hợp, giảm việc tự vận hành database trên EC2.


6. **Chọn template và availability**

   Chọn template **Production**. Sau đó, chọn **Multi-AZ DB instance deployment (2 instances)**.

   ![Chọn template và availability](/images/5-Workshop/5.4-data-access/rds-06.png)

   <p class="image-caption"><em>Hình 6: Chọn template và availability</em></p>
   Cấu hình này tạo một DB instance chính và một standby do RDS quản lý tại Availability Zone khác.


7. **Đặt tên database và credentials**

   Nhập DB identifier `sportbooking-mysql`. Sau đó, chọn phương thức **Self managed** cho credentials. Tiếp theo, nhập master username `sportbooking_ad`; mật khẩu được che trong ảnh.

   ![Đặt tên database và credentials](/images/5-Workshop/5.4-data-access/rds-07.png)

   <p class="image-caption"><em>Hình 7: Đặt tên database và credentials</em></p>
   DB identifier và credentials đã được dùng để tạo `DB_URL`, `DB_USERNAME`, `DB_PASSWORD` trong secret ứng dụng.


8. **Chọn instance class**

   Chọn lớp **Burstable classes**. Sau đó, chọn instance type `db.t3.small` gồm 2 vCPU và 2 GiB RAM.

   ![Chọn instance class](/images/5-Workshop/5.4-data-access/rds-08.png)

   <p class="image-caption"><em>Hình 8: Chọn instance class</em></p>
   Cấu hình `db.t3.small` đáp ứng phạm vi kiểm thử chức năng của dự án và được ghi nhận làm cơ sở theo dõi chi phí.


9. **Cấu hình storage**

   Chọn storage type **General Purpose SSD (gp3)**. Sau đó, đặt allocated storage là `20 GiB`.

   ![Cấu hình storage](/images/5-Workshop/5.4-data-access/rds-09.png)

   <p class="image-caption"><em>Hình 9: Cấu hình storage</em></p>
   Dung lượng `20 GiB` được sử dụng cho phạm vi dữ liệu của đợt triển khai này.


10. **Chọn network cho RDS**

   Chọn **Don’t connect to an EC2 compute resource** nếu tự cấu hình SG. Sau đó, chọn VPC SportBooking. Tiếp theo, chọn DB subnet group private đã tạo.

   ![Chọn network cho RDS](/images/5-Workshop/5.4-data-access/rds-10.png)

   <p class="image-caption"><em>Hình 10: Chọn network cho RDS</em></p>
   Tự chọn VPC/subnet group giúp kiểm soát rõ database nằm trong private DB subnet và không tự mở rule ngoài ý muốn.


11. **Tắt public access và chọn Security Group**

   Chọn **Public access: No**. Sau đó, chọn hoặc tạo RDS Security Group. Tiếp theo, xác nhận SG chỉ mở MySQL `3306` từ EC2 App SG.

   ![Tắt public access và chọn Security Group](/images/5-Workshop/5.4-data-access/rds-11.png)

   <p class="image-caption"><em>Hình 11: Tắt public access và chọn Security Group</em></p>
   Database không có đường truy cập công khai; chỉ EC2 mang Security Group `sportbooking-app` kết nối MySQL.


12. **Cấu hình monitoring**

   Chọn **Database Insights - Standard**. Sau đó, bật **Enhanced Monitoring** với độ chi tiết 60 giây.

   ![Cấu hình monitoring](/images/5-Workshop/5.4-data-access/rds-12.png)

   <p class="image-caption"><em>Hình 12: Cấu hình monitoring</em></p>
   Cấu hình monitoring cung cấp metric hệ điều hành và database phục vụ theo dõi CPU, kết nối và dung lượng.


13. **Đặt database name và port**

   Mở **Additional configuration**. Sau đó, nhập initial database name `sport_booking`. Tiếp theo, giữ port MySQL `3306`.

   ![Đặt database name và port](/images/5-Workshop/5.4-data-access/rds-13.png)

   <p class="image-caption"><em>Hình 13: Đặt database name và port</em></p>
   Ứng dụng cần database `sport_booking`. Nếu bỏ trống hoặc đặt sai tên, app có thể kết nối được server nhưng không thấy schema đúng.


14. **Cấu hình backup**

   Bật automated backup. Sau đó, đặt backup retention period là 7 ngày.

   ![Cấu hình backup](/images/5-Workshop/5.4-data-access/rds-14.png)

   <p class="image-caption"><em>Hình 14: Cấu hình backup</em></p>
   Cấu hình cho phép RDS duy trì các bản sao lưu tự động trong bảy ngày để phục hồi khi cần.


15. **Rà soát chi phí và tạo database**

   Kiểm tra estimated monthly cost. Sau đó, rà soát lại public access, VPC, SG, storage. Tiếp theo, chọn **Create database**.

   ![Rà soát chi phí và tạo database](/images/5-Workshop/5.4-data-access/rds-15.png)

   <p class="image-caption"><em>Hình 15: Rà soát chi phí và tạo database</em></p>
   Bước rà soát đã xác nhận kích thước instance, storage và trạng thái public access trước khi tạo database.


16. **Theo dõi trạng thái database**

   Quay lại danh sách Databases. Sau đó, chờ trạng thái chuyển sang **Available**.

   ![Theo dõi trạng thái database](/images/5-Workshop/5.4-data-access/rds-16.png)

   <p class="image-caption"><em>Hình 16: Theo dõi trạng thái database</em></p>
   Chỉ khi database Available, EC2 app mới có thể kết nối ổn định.


17. **Kiểm tra endpoint và network**

   Mở database vừa tạo. Sau đó, xem tab **Connectivity & security**. Tiếp theo, ghi nhận endpoint, port, VPC, subnet group và Security Group.

   ![Kiểm tra endpoint và network](/images/5-Workshop/5.4-data-access/rds-17.png)

   <p class="image-caption"><em>Hình 17: Kiểm tra endpoint và network</em></p>
   Endpoint và port cần đưa vào Secrets Manager. VPC/SG là bằng chứng database nằm private và chỉ app truy cập được.




