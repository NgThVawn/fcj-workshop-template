---
title : "Tạo Secrets Manager cho cấu hình ứng dụng"
date : 2024-01-01
weight : 3
chapter : false
pre : " <b> 5.4.3 </b> "
---

#### Phạm vi triển khai

Tạo secret `sportbooking/prod/app-env` sau khi RDS chuyển sang trạng thái `Available`. Endpoint và credentials của RDS được lưu cùng các biến môi trường của ứng dụng; EC2 đã đọc secret này trong quá trình bootstrap ở phần 5.5 để tạo `/opt/sportbooking/app.env`.

#### Các biến cấu hình

| Biến | Giá trị đã sử dụng | Mục đích |
|---|---|---|
| `DB_URL` | `jdbc:mysql://sportbooking-mysql.cudgu8qk8z19.us-east-1.rds.amazonaws.com:3306/sport_booking?useSSL=true&serverTimezone=Asia/Ho_Chi_Minh` | Kết nối RDS MySQL. |
| `DB_USERNAME` | `sportbooking_ad` | Tài khoản database. |
| `DB_PASSWORD` | Giá trị đã được che trong ảnh | Mật khẩu database. |
| `SPRING_JPA_HIBERNATE_DDL_AUTO` | `update` | Cho phép cập nhật schema trong giai đoạn triển khai thử nghiệm. |
| `APP_BASE_URL` | `https://sport.younglilliu.id.vn` | Domain chính của app. |
| `APP_UPLOADS_BASE_URL` | `https://sport.younglilliu.id.vn/uploads` | Base URL cho ảnh upload. |
| `SERVER_FORWARD_HEADERS_STRATEGY` | `framework` | Giúp Spring nhận đúng `X-Forwarded-*` sau ALB HTTPS. |
| `APP_STORAGE_TYPE` | `s3` | Bật cơ chế lưu ảnh trên S3. |
| `APP_STORAGE_S3_BUCKET` | `sportbooking-uploads-3stars-us-east-1` | Bucket ảnh upload. |
| `APP_STORAGE_S3_KEY_PREFIX` | `uploads` | Prefix object ảnh. |
| `AWS_REGION`, `AWS_DEFAULT_REGION`, `AWS_S3_REGION` | `us-east-1` | Đồng bộ region cho AWS SDK và CLI. |

{{% notice warning %}}
Không hiển thị `DB_PASSWORD`, OAuth client secret, token hoặc secret value trong ảnh báo cáo. Chỉ cần chứng minh secret có đủ key cấu hình và tên secret đúng.
{{% /notice %}}

#### Kiểm tra quyền đọc secret sau khi triển khai EC2

Sau khi EC2 app được tạo ở phần 5.5, sử dụng Session Manager để kiểm tra quyền đọc secret:

```bash
sudo -i
aws secretsmanager get-secret-value \
  --secret-id sportbooking/prod/app-env \
  --region us-east-1 \
  --query SecretString \
  --output text | jq -r 'to_entries[] | "\(.key)=\(.value)"' > /tmp/app.env

grep -n -E "APP_BASE_URL|APP_STORAGE_TYPE|APP_STORAGE_S3_BUCKET|AWS_REGION" /tmp/app.env
```

{{% notice tip %}}
Kết quả minh chứng chỉ hiển thị các key không nhạy cảm bằng `grep`; toàn bộ file môi trường và các giá trị bí mật không được in ra màn hình.
{{% /notice %}}

#### Quy trình thực hiện

Quy trình lưu cấu hình sử dụng secret dạng key/value, đặt tên theo môi trường và kiểm tra sự tồn tại của secret trước khi cấp quyền cho EC2.

**Quy trình thực hiện:**

1. **Bắt đầu tạo secret**

   Truy cập **Secrets Manager > Secrets**. Sau đó, chọn **Store a new secret**.

   ![Bắt đầu tạo secret](/images/5-Workshop/5.4-S3-onprem/secrets-manager-01.png)

   <p class="image-caption"><em>Hình 1: Bắt đầu tạo secret</em></p>
   Secret tách cấu hình nhạy cảm khỏi source code và file jar, đặc biệt là DB password, OAuth secret và cấu hình storage.


2. **Chọn loại secret và nhập key/value**

   Chọn **Other type of secret**. Sau đó, nhập các cặp key/value như `DB_URL`, `DB_USERNAME`, `APP_BASE_URL`, `APP_STORAGE_TYPE`.

   ![Chọn loại secret và nhập key/value](/images/5-Workshop/5.4-S3-onprem/secrets-manager-02.png)

   <p class="image-caption"><em>Hình 2: Chọn loại secret và nhập key/value</em></p>
   Ứng dụng Spring Boot dùng nhiều biến môi trường, vì vậy dạng key/value dễ chuyển thành file `/opt/sportbooking/app.env` khi EC2 khởi động.


3. **Hoàn thiện các biến môi trường**

   Bổ sung các key còn lại: S3 bucket, prefix uploads, AWS Region và cấu hình OAuth. Sau đó, chọn **Next**.

   ![Hoàn thiện các biến môi trường](/images/5-Workshop/5.4-S3-onprem/secrets-manager-03.png)

   <p class="image-caption"><em>Hình 3: Hoàn thiện các biến môi trường</em></p>
   Thiếu một biến như `AWS_S3_REGION` hoặc `APP_STORAGE_S3_BUCKET` có thể làm upload ảnh lỗi dù EC2 và RDS đã chạy.


4. **Đặt tên secret**

   Nhập secret name `sportbooking/prod/app-env`. Sau đó, thêm mô tả xác định secret dùng cho môi trường triển khai của SportBooking. Tiếp theo, chọn **Next**.

   ![Đặt tên secret](/images/5-Workshop/5.4-S3-onprem/secrets-manager-04.png)

   <p class="image-caption"><em>Hình 4: Đặt tên secret</em></p>
   Tên secret được giữ cố định để khớp với tham chiếu trong IAM policy và User Data.


5. **Cấu hình rotation**

   Rotation tự động chưa được bật trong phạm vi triển khai này. Chọn **Next**.

   ![Cấu hình rotation](/images/5-Workshop/5.4-S3-onprem/secrets-manager-05.png)

   <p class="image-caption"><em>Hình 5: Cấu hình rotation</em></p>
   Việc bật rotation được để lại cho giai đoạn hoàn thiện vận hành, khi ứng dụng và tài khoản database đã hỗ trợ thay đổi mật khẩu đồng bộ.


6. **Rà soát và tạo secret**

   Kiểm tra lại key/value không nhạy cảm. Không để lộ password/token khi chụp báo cáo. Sau đó, chọn **Store**.

   ![Rà soát và tạo secret](/images/5-Workshop/5.4-S3-onprem/secrets-manager-06.png)

   <p class="image-caption"><em>Hình 6: Rà soát và tạo secret</em></p>
   Tên key và Region đã được rà soát trước khi lưu; mọi giá trị nhạy cảm được che trong ảnh báo cáo.


7. **Xác nhận secret đã tồn tại**

   Quay lại danh sách Secrets. Sau đó, kiểm tra secret `sportbooking/prod/app-env` xuất hiện.

   ![Xác nhận secret đã tồn tại](/images/5-Workshop/5.4-S3-onprem/secrets-manager-07.png)

   <p class="image-caption"><em>Hình 7: Xác nhận secret đã tồn tại</em></p>
   Đây là bằng chứng để gắn với bước IAM Role: EC2 Role cần quyền `secretsmanager:GetSecretValue` trên secret này.




