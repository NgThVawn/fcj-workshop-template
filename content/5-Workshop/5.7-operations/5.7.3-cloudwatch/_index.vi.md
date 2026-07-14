---
title : "Giám sát CloudWatch"
date : 2024-01-01
weight : 3
chapter : false
pre : " <b> 5.7.3 </b> "
---

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

   ![Bắt đầu tạo CloudWatch alarm](/images/5-Workshop/5.7-operations/cloudwatch-01.png)

   <p class="image-caption"><em>Hình 1: Bắt đầu tạo CloudWatch alarm</em></p>
   Alarm giúp phát hiện sớm khi EC2/ALB/RDS có lỗi hoặc metric vượt ngưỡng.


2. **Chọn metric**

   Chọn **Select metric**. Sau đó, chọn namespace **ApplicationELB**.

   ![Chọn metric](/images/5-Workshop/5.7-operations/cloudwatch-02.png)

   <p class="image-caption"><em>Hình 2: Chọn metric</em></p>
   Alarm trong báo cáo tập trung vào số target không đạt health check phía sau ALB.


3. **Chọn namespace ApplicationELB**

   Mở danh mục metric của Application Load Balancer. Sau đó, chọn loại metric liên quan target group hoặc load balancer.

   ![Chọn namespace ApplicationELB](/images/5-Workshop/5.7-operations/cloudwatch-03.png)

   <p class="image-caption"><em>Hình 3: Chọn namespace ApplicationELB</em></p>
   ALB metric cho biết backend có healthy không và người dùng có gặp lỗi 5xx hay không.


4. **Chọn metric cụ thể**

   Chọn metric `UnHealthyHostCount` của `sportbooking-alb` và `sportbooking-tg`. Sau đó, chọn **Select metric**.

   ![Chọn metric cụ thể](/images/5-Workshop/5.7-operations/cloudwatch-04.png)

   <p class="image-caption"><em>Hình 4: Chọn metric cụ thể</em></p>
   Metric này tăng khi một hoặc nhiều EC2 app phía sau ALB không vượt qua health check.


5. **Đặt điều kiện alarm**

   Chọn statistic **Maximum**. Sau đó, đặt period là **1 minute**.

   ![Đặt điều kiện alarm](/images/5-Workshop/5.7-operations/cloudwatch-05.png)

   <p class="image-caption"><em>Hình 5: Đặt điều kiện alarm</em></p>
   Statistic và period này ghi nhận giá trị không healthy lớn nhất trong từng khoảng một phút.


6. **Cấu hình trạng thái và datapoints**

   Chọn điều kiện `UnHealthyHostCount >= 1`. Sau đó, đặt **Datapoints to alarm** là `2 out of 2`. Tiếp theo, giữ **Treat missing data as missing**. Đồng thời, chọn **Next**.

   ![Cấu hình trạng thái và datapoints](/images/5-Workshop/5.7-operations/cloudwatch-06.png)

   <p class="image-caption"><em>Hình 6: Cấu hình trạng thái và datapoints</em></p>
   Datapoints giúp tránh cảnh báo giả do một điểm dữ liệu bất thường ngắn hạn.


7. **Chọn hành động gửi notification**

   Chọn trạng thái alarm cần gửi thông báo. Sau đó, chọn SNS topic đã tạo.

   ![Chọn hành động gửi notification](/images/5-Workshop/5.7-operations/cloudwatch-07.png)

   <p class="image-caption"><em>Hình 7: Chọn hành động gửi notification</em></p>
   Alarm chỉ hữu ích khi có kênh thông báo. SNS giúp gửi email khi hệ thống gặp vấn đề.


8. **Bỏ qua hành động phụ nếu không cần**

   EC2 action, Auto Scaling action và Systems Manager action không được cấu hình cho alarm này. Chọn **Next**.

   ![Bỏ qua hành động phụ nếu không cần](/images/5-Workshop/5.7-operations/cloudwatch-08.png)

   <p class="image-caption"><em>Hình 8: Bỏ qua hành động phụ nếu không cần</em></p>
   Alarm chỉ gửi notification qua SNS và không tự động thay đổi tài nguyên.


9. **Đặt tên alarm**

   Nhập alarm name `SB-ALB-UnhealthyTargets`. Description và tag được để trống trong lần tạo này. Sau đó, chọn **Next**.

   ![Đặt tên alarm](/images/5-Workshop/5.7-operations/cloudwatch-09.png)

   <p class="image-caption"><em>Hình 9: Đặt tên alarm</em></p>
   Tên alarm cần nói rõ tài nguyên và điều kiện để khi nhận email biết ngay vấn đề.


10. **Rà soát và tạo alarm**

   Kiểm tra metric, condition, action và name. Sau đó, chọn **Create alarm**.

   ![Rà soát và tạo alarm](/images/5-Workshop/5.7-operations/cloudwatch-10.png)

   <p class="image-caption"><em>Hình 10: Rà soát và tạo alarm</em></p>
   Metric, điều kiện, SNS action và tên alarm đã được rà soát trước khi tạo.


11. **Kiểm tra alarm trong danh sách**

   Quay lại danh sách Alarms. Sau đó, kiểm tra alarm `SB-ALB-UnhealthyTargets` xuất hiện với action enabled và trạng thái ban đầu `Insufficient data`.

   ![Kiểm tra alarm trong danh sách](/images/5-Workshop/5.7-operations/cloudwatch-11.png)

   <p class="image-caption"><em>Hình 11: Kiểm tra alarm trong danh sách</em></p>
   Trạng thái OK/Alarm/Insufficient data cho biết CloudWatch đã nhận metric và đánh giá điều kiện.
