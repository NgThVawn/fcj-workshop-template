---
title : "Amazon S3"
date : 2024-01-01
weight : 1
chapter : false
pre : " <b> 5.4.1 </b> "
aliases:
  - /vi/5-workshop/5.4-s3-onprem/5.4.1-prepare/
---

#### Cấu hình triển khai

Hai S3 bucket được tạo trước RDS, Secrets Manager, IAM Role và EC2 để artifact triển khai có sẵn trước khi User Data được chạy.

| Mục | Artifact bucket | Upload bucket |
|---|---|---|
| Tên bucket | `sportbooking-artifacts-3stars-us-east-1` | `sportbooking-uploads-3stars-us-east-1` |
| Prefix | `releases/` | `uploads/` |
| Dữ liệu | File jar phát hành | Ảnh sân bóng do người dùng tải lên |
| Public access | Block Public Access: On | Block Public Access: On |
| Quyền EC2 | `s3:GetObject` | `s3:GetObject`, `s3:PutObject`, `s3:DeleteObject` |

{{% notice warning %}}
Block Public Access được giữ nguyên cho cả hai bucket. Ảnh được ứng dụng Spring Boot đọc từ S3 và trả về qua route `/uploads/**`, không public trực tiếp toàn bộ bucket.
{{% /notice %}}

#### Quy trình thực hiện

1. **Mở chức năng tạo S3 bucket**

   Truy cập **S3 > Buckets**. Sau đó, chọn **Create bucket**.

   ![Mở chức năng tạo S3 bucket](/images/5-Workshop/5.4-data-access/s3-01.png)

   <p class="image-caption"><em>Hình 1: Mở chức năng tạo S3 bucket</em></p>

   S3 được lựa chọn để tách artifact và dữ liệu upload khỏi vòng đời của EC2. Khi Auto Scaling Group thay instance, dữ liệu vẫn được giữ độc lập.

2. **Nhập tên và Region cho artifact bucket**

   Nhập `sportbooking-artifacts-3stars-us-east-1`. Sau đó, chọn Region `us-east-1`. Tiếp theo, giữ **Object Ownership** ở chế độ không sử dụng ACL public.

   ![Nhập tên và Region cho artifact bucket](/images/5-Workshop/5.4-data-access/s3-02.png)

   <p class="image-caption"><em>Hình 2: Nhập tên và Region cho artifact bucket</em></p>

   Tên bucket được dùng thống nhất trong IAM policy, User Data và các lệnh vận hành ở những phần sau.

3. **Giữ Block Public Access và tạo bucket**

   Bật **Block all public access**. Sau đó, giữ mã hóa mặc định của S3. Tiếp theo, chọn **Create bucket**.

   ![Giữ Block Public Access và tạo artifact bucket](/images/5-Workshop/5.4-data-access/s3-03.png)

   <p class="image-caption"><em>Hình 3: Giữ Block Public Access và tạo artifact bucket</em></p>

   Artifact jar không cần được truy cập công khai vì EC2 tải object bằng IAM Role.

4. **Kiểm tra artifact bucket sau khi tạo**

   Mở bucket vừa tạo và kiểm tra tab **Objects**. Sau đó, xác nhận thông báo tạo bucket thành công.

   ![Kiểm tra artifact bucket sau khi tạo](/images/5-Workshop/5.4-data-access/s3-04.png)

   <p class="image-caption"><em>Hình 4: Kiểm tra artifact bucket sau khi tạo</em></p>

   Sau khi artifact bucket sẵn sàng, lặp lại cùng quy trình để tạo `sportbooking-uploads-3stars-us-east-1` và tiếp tục giữ Block Public Access.

5. **Đóng gói tệp JAR phát hành**

   Trong VS Code, mở mã nguồn `SportBookingSystem` và chạy `.\mvnw.cmd -Pprod clean package`. Maven hoàn tất với trạng thái **BUILD SUCCESS** và tạo `target\SportBooingSystem-0.0.1-SNAPSHOT.jar` để đưa lên S3.

   ![Đóng gói tệp JAR phát hành của SportBooking](/images/5-Workshop/5.4-data-access/s3-artifact-upload-01.png)

   <p class="image-caption"><em>Hình 5: Đóng gói tệp JAR phát hành của SportBooking</em></p>

   Đây là bản build chính thức được sử dụng cho EC2 seed, Launch Template và Auto Scaling Group.

6. **Mở artifact bucket**

   Truy cập danh sách S3 bucket. Sau đó, chọn `sportbooking-artifacts-3stars-us-east-1`.

   ![Mở artifact bucket đã tạo](/images/5-Workshop/5.4-data-access/s3-artifact-upload-02.png)

   <p class="image-caption"><em>Hình 6: Mở artifact bucket đã tạo</em></p>

   Danh sách S3 xác nhận artifact bucket và upload bucket đều tồn tại trước khi EC2 được khởi tạo.

7. **Tạo prefix releases**

   Mở artifact bucket. Sau đó, tạo và mở prefix `releases/`.

   ![Tạo prefix releases trong artifact bucket](/images/5-Workshop/5.4-data-access/s3-artifact-upload-03.png)

   <p class="image-caption"><em>Hình 7: Tạo prefix releases trong artifact bucket</em></p>

   Prefix này tách file phát hành khỏi những object khác và khớp với S3 URI trong User Data.

8. **Bắt đầu upload artifact**

   Chọn **Upload** trong prefix `releases/`.

   ![Bắt đầu upload artifact jar](/images/5-Workshop/5.4-data-access/s3-artifact-upload-04.png)

   <p class="image-caption"><em>Hình 8: Bắt đầu upload artifact jar</em></p>

9. **Chọn file jar từ máy cục bộ**

   Chọn **Add files**. Sau đó, chọn đúng file jar vừa build trong thư mục `target`.

   ![Chọn file jar để upload](/images/5-Workshop/5.4-data-access/s3-artifact-upload-05.png)

   <p class="image-caption"><em>Hình 9: Chọn file jar để upload</em></p>

10. **Xác nhận đích upload**

   Kiểm tra destination là `sportbooking-artifacts-3stars-us-east-1/releases/`. Sau đó, chọn **Upload**.

   ![Xác nhận đích upload artifact](/images/5-Workshop/5.4-data-access/s3-artifact-upload-06.png)

   <p class="image-caption"><em>Hình 10: Xác nhận đích upload artifact</em></p>

   Đường dẫn này được sử dụng nguyên vẹn trong biến `APP_ARTIFACT_S3_URI` của Launch Template.

11. **Kiểm tra artifact upload thành công**

   Kiểm tra trạng thái upload hoàn tất. Sau đó, xác nhận object jar xuất hiện trong prefix `releases/`.

   ![Kiểm tra artifact upload thành công](/images/5-Workshop/5.4-data-access/s3-artifact-upload-07.png)

   <p class="image-caption"><em>Hình 11: Kiểm tra artifact upload thành công</em></p>

#### Cấu hình S3 Gateway Endpoint

Sau khi VPC và S3 bucket tồn tại, tạo Gateway Endpoint để EC2 trong private app subnet truy cập S3 qua mạng AWS:

1. Mở **VPC > Endpoints** và chọn **Create endpoint**.
2. Đặt tên `sportbooking-vpce-s3` và chọn dịch vụ `com.amazonaws.us-east-1.s3` có loại **Gateway**.
3. Chọn `sportbooking-vpc` và gắn endpoint với route table của hai private app subnet.
4. Sử dụng endpoint policy cho phép các thao tác cần thiết; quyền truy cập object vẫn được giới hạn bằng IAM Role và bucket policy.
5. Kiểm tra endpoint ở trạng thái **Available** trước khi tạo EC2.

{{% notice info %}}
S3 Gateway Endpoint chỉ cung cấp đường truyền riêng đến S3, không thay thế IAM. EC2 vẫn phải có `s3:GetObject` đối với artifact bucket và quyền đọc/ghi phù hợp đối với upload bucket.
{{% /notice %}}

#### Lệnh kiểm tra

```powershell
aws s3 ls s3://sportbooking-artifacts-3stars-us-east-1/releases/ --region us-east-1
aws s3 ls s3://sportbooking-uploads-3stars-us-east-1/uploads/ --region us-east-1
```

Kết quả kiểm tra xác nhận artifact jar đã có trên S3 trước khi tạo các dịch vụ nền còn lại và lớp EC2.



