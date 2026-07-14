---
title : "Cảnh báo SNS"
date : 2024-01-01
weight : 2
chapter : false
pre : " <b> 5.7.2 </b> "
---

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

   ![Bắt đầu tạo SNS topic](/images/5-Workshop/5.7-operations/sns-01.png)

   <p class="image-caption"><em>Hình 1: Bắt đầu tạo SNS topic</em></p>
   SNS topic là kênh nhận thông báo từ CloudWatch alarm.


2. **Chọn topic type và nhập tên**

   Chọn type **Standard**. Sau đó, nhập tên topic `sportbooking-prod-alerts`.

   ![Chọn topic type và nhập tên](/images/5-Workshop/5.7-operations/sns-02.png)

   <p class="image-caption"><em>Hình 2: Chọn topic type và nhập tên</em></p>
   Standard topic phù hợp gửi cảnh báo email đơn giản, không yêu cầu FIFO ordering.


3. **Tạo topic**

   Giữ cấu hình delivery và tag ở giá trị mặc định. Sau đó, chọn **Create topic**.

   ![Tạo topic](/images/5-Workshop/5.7-operations/sns-03.png)

   <p class="image-caption"><em>Hình 3: Tạo topic</em></p>
   Topic Standard được sử dụng cho kênh email của CloudWatch alarm; các cấu hình delivery nâng cao không được bật.


4. **Mở topic vừa tạo**

   Kiểm tra ARN và thông tin topic.

   ![Mở topic vừa tạo](/images/5-Workshop/5.7-operations/sns-04.png)

   <p class="image-caption"><em>Hình 4: Mở topic vừa tạo</em></p>
   ARN của topic này đã được chọn làm action gửi thông báo trong CloudWatch alarm.


5. **Bắt đầu tạo subscription**

   Trong topic, chọn **Create subscription**.

   ![Bắt đầu tạo subscription](/images/5-Workshop/5.7-operations/sns-05.png)

   <p class="image-caption"><em>Hình 5: Bắt đầu tạo subscription</em></p>
   Nếu không có subscription, alarm publish vào topic nhưng không có người nhận.


6. **Nhập protocol và endpoint**

   Chọn protocol **Email**. Sau đó, nhập email nhận cảnh báo.

   ![Nhập protocol và endpoint](/images/5-Workshop/5.7-operations/sns-06.png)

   <p class="image-caption"><em>Hình 6: Nhập protocol và endpoint</em></p>
   Email là cách đơn giản nhất để nhận thông báo trong bài triển khai thử nghiệm.


7. **Tạo subscription**

   Kiểm tra email đã nhập. Sau đó, chọn **Create subscription**.

   ![Tạo subscription](/images/5-Workshop/5.7-operations/sns-07.png)

   <p class="image-caption"><em>Hình 7: Tạo subscription</em></p>
   AWS đã gửi email xác nhận; subscription ở trạng thái pending cho đến khi liên kết xác nhận được mở.


8. **Mở email xác nhận**

   Truy cập mailbox. Sau đó, mở email **AWS Notification - Subscription Confirmation**.

   ![Mở email xác nhận](/images/5-Workshop/5.7-operations/sns-08.png)

   <p class="image-caption"><em>Hình 8: Mở email xác nhận</em></p>
   Đây là bước bảo mật để AWS xác nhận email thật sự đồng ý nhận cảnh báo.


9. **Xác nhận subscription**

   Chọn link **Confirm subscription** trong email.

   ![Confirm subscription](/images/5-Workshop/5.7-operations/sns-09.png)

   <p class="image-caption"><em>Hình 9: Confirm subscription</em></p>
   Chỉ sau khi xác nhận, endpoint mới nhận được thông báo từ SNS topic.


10. **Kiểm tra subscription confirmed**

   Quay lại SNS subscription. Sau đó, kiểm tra status là **Confirmed**.

   ![Kiểm tra subscription confirmed](/images/5-Workshop/5.7-operations/sns-10.png)

   <p class="image-caption"><em>Hình 10: Kiểm tra subscription confirmed</em></p>
   Subscription đã ở trạng thái `Confirmed` trước khi topic được gắn vào CloudWatch alarm.
