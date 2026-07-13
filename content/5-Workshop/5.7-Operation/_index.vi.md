---
title : "Vận hành, giám sát và xử lý lỗi"
date : 2024-01-01
weight : 7
chapter : false
pre : " <b> 5.7. </b> "
---

#### Phạm vi triển khai

Phần này trình bày quy trình vận hành sau khi website đã truy cập được qua HTTPS: cập nhật artifact, cập nhật secret, kiểm tra database, cấu hình OAuth, tạo kênh cảnh báo bằng SNS, tạo CloudWatch alarm và ghi nhận các lỗi thường gặp.

#### Quy trình cập nhật artifact

| Bước | Việc làm | Lệnh/màn hình |
|---|---|---|
| 1 | Đóng gói JAR tại máy cục bộ | `.\mvnw.cmd clean package -DskipTests` |
| 2 | Ghi đè artifact trên S3 | `aws s3 cp target\SportBooingSystem-0.0.1-SNAPSHOT.jar s3://sportbooking-artifacts-3stars-us-east-1/releases/SportBooingSystem-0.0.1-SNAPSHOT.jar --region us-east-1` |
| 3 | Khởi chạy Instance Refresh | ASG > Instance refresh > Start > Use current Auto Scaling group configuration |
| 4 | Chờ target chuyển sang Healthy | Target Group > Targets > Healthy |
| 5 | Kiểm thử lại domain | `curl.exe -I https://sport.younglilliu.id.vn` và trình duyệt |

#### Quy trình cập nhật secret

| Bước | Việc làm | Chi tiết |
|---|---|---|
| 1 | Chỉnh sửa secret | Secrets Manager > `sportbooking/prod/app-env` > Edit. |
| 2 | Bảo vệ dữ liệu nhạy cảm | Không ghi `DB_PASSWORD` hoặc OAuth secrets vào ảnh. |
| 3 | Triển khai lại EC2 | Instance Refresh tạo instance mới và đọc phiên bản secret hiện tại. |
| 4 | Kiểm tra `app.env` | Chỉ dùng `grep` với các key không nhạy cảm trên EC2 mới. |
| 5 | Kiểm thử chức năng | Đăng ký, đăng nhập, OAuth và upload ảnh. |

{{% notice warning %}}
Thay đổi secret không tự động cập nhật vào EC2 đang chạy nếu ứng dụng chỉ đọc secret trong lúc boot. Sau khi sửa secret, cần restart service hoặc chạy Instance Refresh để instance mới đọc cấu hình mới.
{{% /notice %}}

#### Kiểm tra database và dữ liệu khởi tạo

Sau khi EC2 app hoạt động, kết nối RDS từ EC2 bằng Session Manager:

```bash
sudo -i
mariadb -h sportbooking-mysql.cudgu8qk8z19.us-east-1.rds.amazonaws.com -P 3306 -u sportbooking_ad -p
```

Schema được kiểm tra bằng:

```sql
USE sport_booking;
SHOW TABLES;
```

Khi xác nhận bảng `membership_levels` đã tồn tại nhưng chưa có dữ liệu, chạy câu lệnh idempotent sau:

```sql
INSERT INTO membership_levels
  (name, min_bookings, discount_rate, description, badge_color)
VALUES
  ('NONE', 0, 0.0000, 'Thanh vien thuong', '#888888'),
  ('SILVER', 10, 0.0500, 'Bac - giam 5%', '#C0C0C0'),
  ('GOLD', 30, 0.0700, 'Vang - giam 7%', '#FFD700'),
  ('DIAMOND', 100, 0.1000, 'Kim cuong - giam 10%', '#B9F2FF')
ON DUPLICATE KEY UPDATE
  min_bookings = VALUES(min_bookings),
  discount_rate = VALUES(discount_rate),
  description = VALUES(description),
  badge_color = VALUES(badge_color);
```

{{% notice info %}}
Giá trị `SPRING_JPA_HIBERNATE_DDL_AUTO=update` đã được sử dụng trong đợt triển khai này. Flyway hoặc Liquibase chưa nằm trong phạm vi thực hiện và được xác định là hạng mục hoàn thiện quản lý phiên bản schema.
{{% /notice %}}

#### Cấu hình OAuth đã cập nhật sau khi có HTTPS

Google OAuth:

| Mục | Giá trị cấu hình |
|---|---|
| Authorized JavaScript origins | `https://sport.younglilliu.id.vn` |
| Authorized redirect URIs | `https://sport.younglilliu.id.vn/login/oauth2/code/google` |
| Lỗi thường gặp | `redirect_uri_mismatch` khi redirect URI không trùng tuyệt đối. |
| Cách xử lý | Thêm đúng HTTPS callback, dùng đúng `younglilliu` và không dùng ALB DNS. |

Facebook OAuth:

| Mục | Giá trị cấu hình |
|---|---|
| App domain | `sport.younglilliu.id.vn` |
| Valid OAuth Redirect URI | `https://sport.younglilliu.id.vn/login/oauth2/code/facebook` |
| Lỗi thường gặp | Facebook yêu cầu secure connection khi dùng HTTP hoặc ALB DNS. |
| Cách xử lý | Sử dụng HTTPS domain qua ACM và ALB listener 443. |

#### Cảnh báo bằng SNS

SNS đã được tạo làm kênh nhận thông báo từ CloudWatch alarm. Topic và email subscription được xác nhận trước khi tạo alarm để action có endpoint nhận hợp lệ.

| Bước | Thông tin đã đối chiếu |
|---|---|
| 1 | Topic SNS được tạo đúng region `us-east-1`. |
| 2 | Subscription email ở trạng thái `Confirmed`. |
| 3 | CloudWatch alarm chọn đúng SNS topic trong action. |
| 4 | Khi alarm đổi trạng thái, email cảnh báo được gửi thành công. |

#### Quy trình thực hiện: SNS

Kênh cảnh báo được thiết lập bằng SNS topic, email subscription và bước xác nhận địa chỉ nhận thông báo.

**Quy trình thực hiện:**

1. **Bắt đầu tạo SNS topic**

   Truy cập **SNS > Topics**. Sau đó, chọn **Create topic**.

   ![Bắt đầu tạo SNS topic](/images/5-Workshop/5.7-Operation/sns-01.png)

   <p class="image-caption"><em>Hình 1: Bắt đầu tạo SNS topic</em></p>
   SNS topic là kênh nhận thông báo từ CloudWatch alarm.


2. **Chọn topic type và nhập tên**

   Chọn type **Standard**. Sau đó, nhập tên topic `sportbooking-prod-alerts`.

   ![Chọn topic type và nhập tên](/images/5-Workshop/5.7-Operation/sns-02.png)

   <p class="image-caption"><em>Hình 2: Chọn topic type và nhập tên</em></p>
   Standard topic phù hợp gửi cảnh báo email đơn giản, không yêu cầu FIFO ordering.


3. **Tạo topic**

   Giữ cấu hình delivery và tag ở giá trị mặc định. Sau đó, chọn **Create topic**.

   ![Tạo topic](/images/5-Workshop/5.7-Operation/sns-03.png)

   <p class="image-caption"><em>Hình 3: Tạo topic</em></p>
   Topic Standard được sử dụng cho kênh email của CloudWatch alarm; các cấu hình delivery nâng cao không được bật.


4. **Mở topic vừa tạo**

   Kiểm tra ARN và thông tin topic.

   ![Mở topic vừa tạo](/images/5-Workshop/5.7-Operation/sns-04.png)

   <p class="image-caption"><em>Hình 4: Mở topic vừa tạo</em></p>
   ARN của topic này đã được chọn làm action gửi thông báo trong CloudWatch alarm.


5. **Bắt đầu tạo subscription**

   Trong topic, chọn **Create subscription**.

   ![Bắt đầu tạo subscription](/images/5-Workshop/5.7-Operation/sns-05.png)

   <p class="image-caption"><em>Hình 5: Bắt đầu tạo subscription</em></p>
   Nếu không có subscription, alarm publish vào topic nhưng không có người nhận.


6. **Nhập protocol và endpoint**

   Chọn protocol **Email**. Sau đó, nhập email nhận cảnh báo.

   ![Nhập protocol và endpoint](/images/5-Workshop/5.7-Operation/sns-06.png)

   <p class="image-caption"><em>Hình 6: Nhập protocol và endpoint</em></p>
   Email là cách đơn giản nhất để nhận thông báo trong bài triển khai thử nghiệm.


7. **Tạo subscription**

   Kiểm tra email đã nhập. Sau đó, chọn **Create subscription**.

   ![Tạo subscription](/images/5-Workshop/5.7-Operation/sns-07.png)

   <p class="image-caption"><em>Hình 7: Tạo subscription</em></p>
   AWS đã gửi email xác nhận; subscription ở trạng thái pending cho đến khi liên kết xác nhận được mở.


8. **Mở email xác nhận**

   Truy cập mailbox. Sau đó, mở email **AWS Notification - Subscription Confirmation**.

   ![Mở email xác nhận](/images/5-Workshop/5.7-Operation/sns-08.png)

   <p class="image-caption"><em>Hình 8: Mở email xác nhận</em></p>
   Đây là bước bảo mật để AWS xác nhận email thật sự đồng ý nhận cảnh báo.


9. **Xác nhận subscription**

   Chọn link **Confirm subscription** trong email.

   ![Confirm subscription](/images/5-Workshop/5.7-Operation/sns-09.png)

   <p class="image-caption"><em>Hình 9: Confirm subscription</em></p>
   Chỉ sau khi xác nhận, endpoint mới nhận được thông báo từ SNS topic.


10. **Kiểm tra subscription confirmed**

   Quay lại SNS subscription. Sau đó, kiểm tra status là **Confirmed**.

   ![Kiểm tra subscription confirmed](/images/5-Workshop/5.7-Operation/sns-10.png)

   <p class="image-caption"><em>Hình 10: Kiểm tra subscription confirmed</em></p>
   Subscription đã ở trạng thái `Confirmed` trước khi topic được gắn vào CloudWatch alarm.



#### Giám sát bằng CloudWatch

CloudWatch được dùng để theo dõi metric vận hành của EC2, ALB/Target Group và RDS. Alarm `SB-ALB-UnhealthyTargets` được tạo sau SNS và theo dõi `UnHealthyHostCount` của `sportbooking-tg`.

| Theo dõi | Metric đã đối chiếu | Mục đích |
|---|---|---|
| EC2/ASG | CPUUtilization, StatusCheckFailed | Phát hiện instance lỗi hoặc quá tải. |
| ALB | HTTPCode_Target_5XX_Count, TargetResponseTime, HealthyHostCount | Theo dõi lỗi backend và health của target. |
| RDS | CPUUtilization, FreeStorageSpace, DatabaseConnections | Theo dõi database trong quá trình vận hành. |

#### Quy trình thực hiện: CloudWatch alarm

CloudWatch alarm sử dụng metric của Target Group, ngưỡng cảnh báo liên tục trong hai chu kỳ và action gửi thông báo qua SNS.

**Quy trình thực hiện:**

1. **Bắt đầu tạo CloudWatch alarm**

   Truy cập **CloudWatch > Alarms**. Sau đó, chọn **Create alarm**.

   ![Bắt đầu tạo CloudWatch alarm](/images/5-Workshop/5.7-Operation/cloudwatch-01.png)

   <p class="image-caption"><em>Hình 1: Bắt đầu tạo CloudWatch alarm</em></p>
   Alarm giúp phát hiện sớm khi EC2/ALB/RDS có lỗi hoặc metric vượt ngưỡng.


2. **Chọn metric**

   Chọn **Select metric**. Sau đó, chọn namespace **ApplicationELB**.

   ![Chọn metric](/images/5-Workshop/5.7-Operation/cloudwatch-02.png)

   <p class="image-caption"><em>Hình 2: Chọn metric</em></p>
   Alarm trong báo cáo tập trung vào số target không đạt health check phía sau ALB.


3. **Chọn namespace ApplicationELB**

   Mở danh mục metric của Application Load Balancer. Sau đó, chọn loại metric liên quan target group hoặc load balancer.

   ![Chọn namespace ApplicationELB](/images/5-Workshop/5.7-Operation/cloudwatch-03.png)

   <p class="image-caption"><em>Hình 3: Chọn namespace ApplicationELB</em></p>
   ALB metric cho biết backend có healthy không và người dùng có gặp lỗi 5xx hay không.


4. **Chọn metric cụ thể**

   Chọn metric `UnHealthyHostCount` của `sportbooking-alb` và `sportbooking-tg`. Sau đó, chọn **Select metric**.

   ![Chọn metric cụ thể](/images/5-Workshop/5.7-Operation/cloudwatch-04.png)

   <p class="image-caption"><em>Hình 4: Chọn metric cụ thể</em></p>
   Metric này tăng khi một hoặc nhiều EC2 app phía sau ALB không vượt qua health check.


5. **Đặt điều kiện alarm**

   Chọn statistic **Maximum**. Sau đó, đặt period là **1 minute**.

   ![Đặt điều kiện alarm](/images/5-Workshop/5.7-Operation/cloudwatch-05.png)

   <p class="image-caption"><em>Hình 5: Đặt điều kiện alarm</em></p>
   Statistic và period này ghi nhận giá trị không healthy lớn nhất trong từng khoảng một phút.


6. **Cấu hình trạng thái và datapoints**

   Chọn điều kiện `UnHealthyHostCount >= 1`. Sau đó, đặt **Datapoints to alarm** là `2 out of 2`. Tiếp theo, giữ **Treat missing data as missing**. Đồng thời, chọn **Next**.

   ![Cấu hình trạng thái và datapoints](/images/5-Workshop/5.7-Operation/cloudwatch-06.png)

   <p class="image-caption"><em>Hình 6: Cấu hình trạng thái và datapoints</em></p>
   Datapoints giúp tránh cảnh báo giả do một điểm dữ liệu bất thường ngắn hạn.


7. **Chọn hành động gửi notification**

   Chọn trạng thái alarm cần gửi thông báo. Sau đó, chọn SNS topic đã tạo.

   ![Chọn hành động gửi notification](/images/5-Workshop/5.7-Operation/cloudwatch-07.png)

   <p class="image-caption"><em>Hình 7: Chọn hành động gửi notification</em></p>
   Alarm chỉ hữu ích khi có kênh thông báo. SNS giúp gửi email khi hệ thống gặp vấn đề.


8. **Bỏ qua hành động phụ nếu không cần**

   EC2 action, Auto Scaling action và Systems Manager action không được cấu hình cho alarm này. Chọn **Next**.

   ![Bỏ qua hành động phụ nếu không cần](/images/5-Workshop/5.7-Operation/cloudwatch-08.png)

   <p class="image-caption"><em>Hình 8: Bỏ qua hành động phụ nếu không cần</em></p>
   Alarm chỉ gửi notification qua SNS và không tự động thay đổi tài nguyên.


9. **Đặt tên alarm**

   Nhập alarm name `SB-ALB-UnhealthyTargets`. Description và tag được để trống trong lần tạo này. Sau đó, chọn **Next**.

   ![Đặt tên alarm](/images/5-Workshop/5.7-Operation/cloudwatch-09.png)

   <p class="image-caption"><em>Hình 9: Đặt tên alarm</em></p>
   Tên alarm cần nói rõ tài nguyên và điều kiện để khi nhận email biết ngay vấn đề.


10. **Rà soát và tạo alarm**

   Kiểm tra metric, condition, action và name. Sau đó, chọn **Create alarm**.

   ![Rà soát và tạo alarm](/images/5-Workshop/5.7-Operation/cloudwatch-10.png)

   <p class="image-caption"><em>Hình 10: Rà soát và tạo alarm</em></p>
   Metric, điều kiện, SNS action và tên alarm đã được rà soát trước khi tạo.


11. **Kiểm tra alarm trong danh sách**

   Quay lại danh sách Alarms. Sau đó, kiểm tra alarm `SB-ALB-UnhealthyTargets` xuất hiện với action enabled và trạng thái ban đầu `Insufficient data`.

   ![Kiểm tra alarm trong danh sách](/images/5-Workshop/5.7-Operation/cloudwatch-11.png)

   <p class="image-caption"><em>Hình 11: Kiểm tra alarm trong danh sách</em></p>
   Trạng thái OK/Alarm/Insufficient data cho biết CloudWatch đã nhận metric và đánh giá điều kiện.



#### Các lỗi đã ghi nhận và xử lý

| Lỗi | Nguyên nhân đã xác định | Cách đã xử lý |
|---|---|---|
| Upload jar báo `NoSuchBucket` | Bucket S3 chưa tồn tại hoặc sai tên bucket. | Tạo artifact bucket trước và đối chiếu lại S3 URI. |
| `Invalid bucket name <artifact-bucket>` | Lệnh release vẫn dùng placeholder. | Thay placeholder bằng `sportbooking-artifacts-3stars-us-east-1`. |
| RDS missing table | Schema DB chưa có bảng mà Hibernate cần. | Kiểm tra `ddl-auto`, log ứng dụng và tạo dữ liệu khởi tạo cần thiết. |
| S3 Region hiển thị literal `${AWS_REGION}` | Biến Region chưa được truyền vào ứng dụng. | Bổ sung `AWS_REGION`, `AWS_DEFAULT_REGION`, `AWS_S3_REGION=us-east-1`. |
| Ảnh `/uploads` bị 404 | Controller S3 chưa bật, prefix/bucket sai hoặc IAM thiếu quyền. | Đối chiếu `APP_STORAGE_TYPE=s3`, bucket, prefix `uploads` và IAM policy. |
| Đăng ký lỗi membership levels | DB thiếu dữ liệu trong bảng `membership_levels`. | Chèn các mức `NONE`, `SILVER`, `GOLD`, `DIAMOND`. |
| ACM Pending validation | DNS validation CNAME chưa public hoặc sai chính tả domain. | Kiểm tra hosted zone, NS tại registrar và CNAME validation. |
| Instance Refresh version empty | Desired configuration không có Launch Template version. | Cập nhật ASG sang version hợp lệ và dùng current ASG configuration. |

#### Đối chiếu Well-Architected Framework

| Trụ cột | Áp dụng trong kiến trúc |
|---|---|
| Security | Private subnet cho EC2/RDS; SG least privilege; Secrets Manager; IAM Role thay access key; Session Manager không mở SSH; HTTPS bằng ACM. |
| Reliability | ALB + ASG + 2 AZ; Target Group health check; RDS backup/snapshot; Instance Refresh thay instance lỗi. |
| Operational Excellence | User Data tự động bootstrap; deploy bằng S3 artifact; kiểm tra bằng curl/systemctl; CloudWatch/SNS hỗ trợ vận hành. |
| Cost Optimization | ASG được đưa về desired `0` trước khi xóa; RDS và NAT Gateway được thu hồi sau kiểm thử; kích thước tài nguyên được ghi nhận để kiểm soát chi phí. |
| Performance Efficiency | ALB phân phối request; S3 tách file upload khỏi EC2; CloudFront được giữ ở mức kiến trúc mở rộng do distribution chưa được tạo. |

#### Các lệnh đã sử dụng khi kiểm tra vận hành

```powershell
ipconfig /flushdns
nslookup -type=NS younglilliu.id.vn
nslookup sport.younglilliu.id.vn
curl.exe -I http://sport.younglilliu.id.vn
curl.exe -I https://sport.younglilliu.id.vn
```

```bash
sudo -i
systemctl status sportbooking --no-pager
journalctl -u sportbooking -n 100 --no-pager
curl -I http://127.0.0.1:8080/
curl http://127.0.0.1:8080/actuator/health
```


