---
title : "Dọn dẹp dữ liệu"
date : 2024-01-01
weight : 3
chapter : false
pre : " <b> 5.8.3 </b> "
---

#### 9. Xóa RDS MySQL

**Quy trình thực hiện:**

1. **Bắt đầu xóa DB instance**

   Trên giao diện, thực hiện lần lượt: **(1)** truy cập **RDS > Databases** và chọn `sportbooking-mysql`; **(2)** mở menu **Actions**; và **(3)** chọn **Delete**.

   ![Chọn RDS MySQL cần xóa](/images/5-Workshop/5.8-resource-cleanup/cleanup-rds-01.png)

   <p class="image-caption"><em>Hình 1: Chọn RDS MySQL cần xóa</em></p>
   EC2 đã được kết thúc nên không còn kết nối ứng dụng đang sử dụng database.


2. **Cấu hình phương án lưu dữ liệu và xác nhận**

   Chọn **Create final snapshot** và đặt tên `sportbooking-mysql-snapshot`. Sau đó, chọn **Retain automated backups** để duy trì khả năng phục hồi. Cuối cùng, **(1)** nhập `delete me` và **(2)** chọn **Delete**.

   ![Cấu hình snapshot và xác nhận xóa RDS](/images/5-Workshop/5.8-resource-cleanup/cleanup-rds-02.png)

   <p class="image-caption"><em>Hình 2: Cấu hình snapshot và xác nhận xóa RDS</em></p>
   Final snapshot `sportbooking-mysql-snapshot` và automated backup đã được giữ lại theo kế hoạch lưu trữ sau khi DB instance bị xóa.


3. **Kiểm tra DB instance đã được xóa**

   Chờ quá trình xóa hoàn tất. Sau đó, xác nhận thông báo **Successfully deleted DB instance sportbooking-mysql**.

   ![Kiểm tra RDS MySQL đã được xóa](/images/5-Workshop/5.8-resource-cleanup/cleanup-rds-03.png)

   <p class="image-caption"><em>Hình 3: Kiểm tra RDS MySQL đã được xóa</em></p>
   DB instance không còn phát sinh chi phí compute, nhưng snapshot và retained backup đã chọn vẫn là tài nguyên lưu trữ cần được quản lý.


{{% notice warning %}}
Final snapshot và retained automated backup vẫn có thể phát sinh chi phí lưu trữ sau khi DB instance bị xóa. Hai tài nguyên này phải được theo dõi theo thời hạn lưu đã thống nhất.
{{% /notice %}}

#### 10. Làm rỗng và xóa S3 bucket

Artifact bucket đã được làm rỗng trước khi xóa. Upload bucket được giữ lại trong lần dọn dẹp này để bảo toàn ảnh do người dùng tải lên.

**Quy trình thực hiện:**

1. **Chọn bucket và bắt đầu làm rỗng**

   Trên giao diện, thực hiện lần lượt: **(1)** chọn bucket `sportbooking-artifacts-3stars-us-east-1` và **(2)** chọn **Empty**.

   ![Chọn S3 artifact bucket cần làm rỗng](/images/5-Workshop/5.8-resource-cleanup/cleanup-s3-01.png)

   <p class="image-caption"><em>Hình 4: Chọn S3 artifact bucket cần làm rỗng</em></p>
   S3 không cho phép xóa bucket khi vẫn còn object, object version hoặc delete marker.


2. **Xác nhận làm rỗng bucket**

   Trên giao diện, thực hiện lần lượt: **(1)** nhập `permanently delete` và **(2)** chọn **Empty**.

   ![Xác nhận làm rỗng S3 bucket](/images/5-Workshop/5.8-resource-cleanup/cleanup-s3-02.png)

   <p class="image-caption"><em>Hình 5: Xác nhận làm rỗng S3 bucket</em></p>
   Thao tác làm rỗng xóa dữ liệu trong bucket và không thể hoàn tác nếu không có bản sao lưu ở nơi khác.


3. **Kiểm tra bucket đã rỗng**

   Xác nhận trạng thái **Successfully emptied bucket**. Sau đó, kiểm tra số object xóa thành công và số object xóa thất bại.

   ![Kiểm tra S3 bucket đã được làm rỗng](/images/5-Workshop/5.8-resource-cleanup/cleanup-s3-03.png)

   <p class="image-caption"><em>Hình 6: Kiểm tra S3 bucket đã được làm rỗng</em></p>
   Trong kết quả minh họa, hai object với tổng dung lượng 82,5 MB đã được xóa và không có object thất bại.


4. **Chọn xóa bucket**

   Trên giao diện, thực hiện lần lượt: **(1)** quay lại danh sách và chọn artifact bucket vừa làm rỗng và **(2)** chọn **Delete**.

   ![Chọn xóa S3 artifact bucket](/images/5-Workshop/5.8-resource-cleanup/cleanup-s3-04.png)

   <p class="image-caption"><em>Hình 7: Chọn xóa S3 artifact bucket</em></p>
   Chỉ xóa bucket của dự án. Bucket `cf-templates-...` có thể được CloudFormation hoặc môi trường khác sử dụng nên không được xóa khi chưa xác minh.


5. **Xác nhận tên bucket**

   Trên giao diện, thực hiện lần lượt: **(1)** nhập chính xác tên bucket và **(2)** chọn **Delete bucket**.

   ![Xác nhận xóa S3 artifact bucket](/images/5-Workshop/5.8-resource-cleanup/cleanup-s3-05.png)

   <p class="image-caption"><em>Hình 8: Xác nhận xóa S3 artifact bucket</em></p>
   Tên S3 bucket là duy nhất toàn cục; sau khi xóa, không có bảo đảm tên này vẫn còn khả dụng để tạo lại.


6. **Kiểm tra bucket đã được xóa**

   Xác nhận thông báo **Successfully deleted bucket**. Sau đó, kiểm tra artifact bucket không còn trong danh sách.

   ![Kiểm tra S3 artifact bucket đã được xóa](/images/5-Workshop/5.8-resource-cleanup/cleanup-s3-06.png)

   <p class="image-caption"><em>Hình 9: Kiểm tra S3 artifact bucket đã được xóa</em></p>
   Bucket `sportbooking-uploads-3stars-us-east-1` vẫn xuất hiện trong danh sách sau khi artifact bucket bị xóa; đây là tài nguyên lưu giữ có chủ đích, không phải tài nguyên bị bỏ sót.


#### 11. Lên lịch xóa secret

**Quy trình thực hiện:**

1. **Chọn xóa secret**

   Mở secret `sportbooking/prod/app-env`. Tiếp theo, **(1)** mở menu **Actions** và **(2)** chọn **Delete secret**.

   ![Chọn secret cần xóa](/images/5-Workshop/5.8-resource-cleanup/cleanup-secret-01.png)

   <p class="image-caption"><em>Hình 10: Chọn secret cần xóa</em></p>
   EC2 và IAM Role đã được xóa nên secret không còn workload nào đọc trong phạm vi dự án.


2. **Đặt thời gian chờ và lên lịch xóa**

   Trên giao diện, thực hiện lần lượt: **(1)** đặt **Waiting period** là `7` ngày hoặc thời gian phù hợp với chính sách lưu giữ và **(2)** chọn **Schedule deletion**.

   ![Lên lịch xóa Secrets Manager secret](/images/5-Workshop/5.8-resource-cleanup/cleanup-secret-02.png)

   <p class="image-caption"><em>Hình 11: Lên lịch xóa Secrets Manager secret</em></p>
   Secrets Manager không xóa ngay mà đưa secret vào thời gian chờ khôi phục. Trong thời gian này, secret bị vô hiệu hóa và có thể được restore nếu phát hiện xóa nhầm.


{{% notice tip %}}
Không đưa giá trị secret vào ảnh hoặc nội dung báo cáo. Nếu cần triển khai lại trong thời gian chờ, khôi phục secret trước khi cập nhật thay vì tạo trùng tên.
{{% /notice %}}

#### 12. Xóa ACM certificate

ACM certificate chỉ được xóa khi cột **In use** hiển thị **No**. Nếu còn được ALB hoặc CloudFront sử dụng, phải gỡ liên kết đó trước.

**Quy trình thực hiện:**

1. **Chọn certificate**

   Trên giao diện, thực hiện lần lượt: **(1)** chọn certificate của `sport.younglilliu.id.vn` và kiểm tra **In use = No** và **(2)** mở **More actions** và chọn **Delete**.

   ![Chọn ACM certificate cần xóa](/images/5-Workshop/5.8-resource-cleanup/cleanup-acm-01.png)

   <p class="image-caption"><em>Hình 12: Chọn ACM certificate cần xóa</em></p>
   ALB đã được xóa nên HTTPS listener không còn giữ certificate. CloudFront distribution chưa được tạo nên certificate không có thêm phụ thuộc từ CloudFront.


2. **Xác nhận xóa certificate**

   Trên giao diện, thực hiện lần lượt: **(1)** nhập `delete` và **(2)** chọn **Delete**.

   ![Xác nhận xóa ACM certificate](/images/5-Workshop/5.8-resource-cleanup/cleanup-acm-02.png)

   <p class="image-caption"><em>Hình 13: Xác nhận xóa ACM certificate</em></p>
   Sau khi xóa, certificate không thể tiếp tục phục vụ kết nối HTTPS và không thể khôi phục.


3. **Kiểm tra certificate đã được xóa**

   Xác nhận thông báo **Successfully deleted certificate**. Sau đó, kiểm tra danh sách không còn certificate của dự án.

   ![Kiểm tra ACM certificate đã được xóa](/images/5-Workshop/5.8-resource-cleanup/cleanup-acm-03.png)

   <p class="image-caption"><em>Hình 14: Kiểm tra ACM certificate đã được xóa</em></p>
   Sau bước này có thể xóa CNAME validation của certificate trong Route 53 nếu record không còn phục vụ certificate khác.
