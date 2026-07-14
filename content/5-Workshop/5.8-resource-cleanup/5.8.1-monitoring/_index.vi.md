---
title : "Dọn dẹp Monitoring"
date : 2024-01-01
weight : 1
chapter : false
pre : " <b> 5.8.1 </b> "
---

#### 1. Xóa CloudWatch alarm

CloudWatch alarm được xóa trước SNS để không còn action tham chiếu đến topic cảnh báo.

**Quy trình thực hiện:**

1. **Chọn alarm cần xóa**

   Trên giao diện, thực hiện lần lượt: **(1)** chọn alarm `SB-ALB-UnhealthyTargets`; **(2)** mở menu **Actions**; và **(3)** chọn **Delete**.

   ![Chọn CloudWatch alarm cần xóa](/images/5-Workshop/5.8-resource-cleanup/cleanup-cloudwatch-alarm-01.png)

   <p class="image-caption"><em>Hình 1: Chọn CloudWatch alarm cần xóa</em></p>
   Alarm này theo dõi Target Group của SportBooking và gửi thông báo đến SNS. Xóa alarm trước giúp loại bỏ liên kết giám sát trước khi các tài nguyên nguồn và kênh thông báo bị xóa.


2. **Xác nhận xóa alarm**

   Kiểm tra đúng tên `SB-ALB-UnhealthyTargets`. Sau đó, chọn **Delete** để xác nhận.

   ![Xác nhận xóa CloudWatch alarm](/images/5-Workshop/5.8-resource-cleanup/cleanup-cloudwatch-alarm-02.png)

   <p class="image-caption"><em>Hình 2: Xác nhận xóa CloudWatch alarm</em></p>
   Thao tác này chỉ xóa cấu hình cảnh báo, không xóa metric lịch sử của dịch vụ nguồn.


#### 2. Xóa SNS subscription

Subscription được xóa trước topic để ngắt endpoint email khỏi kênh cảnh báo một cách rõ ràng.

**Quy trình thực hiện:**

1. **Chọn subscription**

   Trên giao diện, thực hiện lần lượt: **(1)** truy cập **SNS > Subscriptions** và chọn subscription đã xác nhận của dự án và **(2)** chọn **Delete**.

   ![Chọn SNS subscription cần xóa](/images/5-Workshop/5.8-resource-cleanup/cleanup-sns-subscription-01.png)

   <p class="image-caption"><em>Hình 3: Chọn SNS subscription cần xóa</em></p>
   Subscription liên kết địa chỉ email nhận cảnh báo với topic `sportbooking-prod-alerts`.


2. **Xác nhận xóa subscription**

   Kiểm tra endpoint và ID subscription. Sau đó, chọn **Delete**.

   ![Xác nhận xóa SNS subscription](/images/5-Workshop/5.8-resource-cleanup/cleanup-sns-subscription-02.png)

   <p class="image-caption"><em>Hình 4: Xác nhận xóa SNS subscription</em></p>
   Sau bước này, endpoint email không còn nhận thông báo từ topic.


3. **Kiểm tra kết quả**

   Làm mới danh sách subscription. Sau đó, xác nhận thông báo **Subscription deleted successfully** và danh sách không còn subscription của dự án.

   ![Kiểm tra SNS subscription đã được xóa](/images/5-Workshop/5.8-resource-cleanup/cleanup-sns-subscription-03.png)

   <p class="image-caption"><em>Hình 5: Kiểm tra SNS subscription đã được xóa</em></p>
   Việc kiểm tra kết quả giúp tránh để lại endpoint không còn mục đích sử dụng.


#### 3. Xóa SNS topic

**Quy trình thực hiện:**

1. **Chọn topic cảnh báo**

   Trên giao diện, thực hiện lần lượt: **(1)** truy cập **SNS > Topics** và chọn `sportbooking-prod-alerts` và **(2)** chọn **Delete**.

   ![Chọn SNS topic cần xóa](/images/5-Workshop/5.8-resource-cleanup/cleanup-sns-topic-01.png)

   <p class="image-caption"><em>Hình 6: Chọn SNS topic cần xóa</em></p>
   CloudWatch alarm và subscription đã được xóa nên topic không còn thành phần phụ thuộc trong phạm vi dự án.


2. **Xác nhận xóa topic**

   Trên giao diện, thực hiện lần lượt: **(1)** nhập `delete me` theo yêu cầu xác nhận và **(2)** chọn **Delete**.

   ![Xác nhận xóa SNS topic](/images/5-Workshop/5.8-resource-cleanup/cleanup-sns-topic-02.png)

   <p class="image-caption"><em>Hình 7: Xác nhận xóa SNS topic</em></p>
   Xóa topic là thao tác không thể hoàn tác. Chuỗi xác nhận giúp hạn chế xóa nhầm kênh thông báo.


3. **Kiểm tra topic đã được xóa**

   Xác nhận thông báo **Topic deleted successfully**. Sau đó, kiểm tra danh sách không còn topic của dự án.

   ![Kiểm tra SNS topic đã được xóa](/images/5-Workshop/5.8-resource-cleanup/cleanup-sns-topic-03.png)

   <p class="image-caption"><em>Hình 8: Kiểm tra SNS topic đã được xóa</em></p>
   Kết quả này hoàn tất phần dọn dẹp kênh cảnh báo SNS.
