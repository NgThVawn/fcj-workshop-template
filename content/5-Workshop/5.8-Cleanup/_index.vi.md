---
title : "Dọn dẹp tài nguyên"
date : 2024-01-01
weight : 8
chapter : false
pre : " <b> 5.8. </b> "
---

#### Phạm vi dọn dẹp

Sau khi hoàn tất triển khai, kiểm thử và lưu đầy đủ minh chứng, thu hồi tài nguyên của SportBooking theo đúng quan hệ phụ thuộc. Artifact bucket và lớp compute/network được xóa; upload bucket, final RDS snapshot và retained automated backup được giữ lại có chủ đích để bảo toàn dữ liệu cần phục hồi.

{{% notice warning %}}
Các thao tác xóa Auto Scaling Group, RDS, S3, VPC và IAM Role có thể làm mất dữ liệu hoặc gián đoạn hoàn toàn website. Chỉ thực hiện sau khi đã sao lưu dữ liệu cần thiết, hoàn tất báo cáo và xác nhận không còn sử dụng môi trường.
{{% /notice %}}

#### Thứ tự dọn dẹp

| Giai đoạn | Tài nguyên | Quan hệ phụ thuộc |
|---|---|---|
| 1 | Route 53, CloudWatch và SNS | Ngừng điều hướng người dùng; xóa alarm trước SNS topic. |
| 2 | ASG, ALB, Target Group và Launch Template | ASG được xóa trước Launch Template; ALB được xóa trước Target Group. |
| 3 | IAM, RDS, S3, Secrets Manager và ACM | EC2 đã dừng; ACM không còn được ALB sử dụng; CloudFront distribution chưa từng được tạo. |
| 4 | NAT Gateway, Elastic IP và VPC Endpoint | NAT Gateway đã chuyển sang `Deleted` trước khi Elastic IP được giải phóng. |
| 5 | Security Group, DB Subnet Group và VPC | ALB, EC2, RDS, endpoint và các tham chiếu Security Group đã được xóa. |
| 6 | Kiểm tra phần còn lại | Không còn snapshot, backup, log group, hosted zone hoặc policy ngoài kế hoạch lưu giữ. |

{{% notice info %}}
Trong toàn bộ quy trình, cần kiểm tra đúng AWS account và Region `us-east-1` trước mỗi thao tác. Chỉ chọn tài nguyên mang tên dự án SportBooking; không xóa tài nguyên mặc định hoặc tài nguyên dùng chung.
{{% /notice %}}

#### Chuẩn bị trước khi xóa

1. Xác định final RDS snapshot, retained automated backup và dữ liệu trong upload bucket cần giữ lại.
2. Ngừng Alias record `sport.younglilliu.id.vn` trỏ đến ALB trước khi xóa lớp cân bằng tải.
3. Xác nhận CloudFront distribution chưa được tạo, nên không có phụ thuộc CloudFront cần tháo gỡ.
4. Giữ public hosted zone vì domain vẫn tiếp tục được quản lý; chỉ các record không còn sử dụng được loại bỏ.
5. Ghi nhận rõ các tài nguyên lưu trữ được giữ lại để phân biệt với tài nguyên bị bỏ sót.

### Dọn dẹp tài nguyên giám sát

#### 1. Xóa CloudWatch alarm

CloudWatch alarm được xóa trước SNS để không còn action tham chiếu đến topic cảnh báo.

**Quy trình thực hiện:**

1. **Chọn alarm cần xóa**

   Trên giao diện, thực hiện lần lượt: **(1)** chọn alarm `SB-ALB-UnhealthyTargets`; **(2)** mở menu **Actions**; và **(3)** chọn **Delete**.

   ![Chọn CloudWatch alarm cần xóa](/images/5-Workshop/5.8-Cleanup/cleanup-cloudwatch-alarm-01.png)

   <p class="image-caption"><em>Hình 1: Chọn CloudWatch alarm cần xóa</em></p>
   Alarm này theo dõi Target Group của SportBooking và gửi thông báo đến SNS. Xóa alarm trước giúp loại bỏ liên kết giám sát trước khi các tài nguyên nguồn và kênh thông báo bị xóa.


2. **Xác nhận xóa alarm**

   Kiểm tra đúng tên `SB-ALB-UnhealthyTargets`. Sau đó, chọn **Delete** để xác nhận.

   ![Xác nhận xóa CloudWatch alarm](/images/5-Workshop/5.8-Cleanup/cleanup-cloudwatch-alarm-02.png)

   <p class="image-caption"><em>Hình 2: Xác nhận xóa CloudWatch alarm</em></p>
   Thao tác này chỉ xóa cấu hình cảnh báo, không xóa metric lịch sử của dịch vụ nguồn.


#### 2. Xóa SNS subscription

Subscription được xóa trước topic để ngắt endpoint email khỏi kênh cảnh báo một cách rõ ràng.

**Quy trình thực hiện:**

1. **Chọn subscription**

   Trên giao diện, thực hiện lần lượt: **(1)** truy cập **SNS > Subscriptions** và chọn subscription đã xác nhận của dự án và **(2)** chọn **Delete**.

   ![Chọn SNS subscription cần xóa](/images/5-Workshop/5.8-Cleanup/cleanup-sns-subscription-01.png)

   <p class="image-caption"><em>Hình 3: Chọn SNS subscription cần xóa</em></p>
   Subscription liên kết địa chỉ email nhận cảnh báo với topic `sportbooking-prod-alerts`.


2. **Xác nhận xóa subscription**

   Kiểm tra endpoint và ID subscription. Sau đó, chọn **Delete**.

   ![Xác nhận xóa SNS subscription](/images/5-Workshop/5.8-Cleanup/cleanup-sns-subscription-02.png)

   <p class="image-caption"><em>Hình 4: Xác nhận xóa SNS subscription</em></p>
   Sau bước này, endpoint email không còn nhận thông báo từ topic.


3. **Kiểm tra kết quả**

   Làm mới danh sách subscription. Sau đó, xác nhận thông báo **Subscription deleted successfully** và danh sách không còn subscription của dự án.

   ![Kiểm tra SNS subscription đã được xóa](/images/5-Workshop/5.8-Cleanup/cleanup-sns-subscription-03.png)

   <p class="image-caption"><em>Hình 5: Kiểm tra SNS subscription đã được xóa</em></p>
   Việc kiểm tra kết quả giúp tránh để lại endpoint không còn mục đích sử dụng.


#### 3. Xóa SNS topic

**Quy trình thực hiện:**

1. **Chọn topic cảnh báo**

   Trên giao diện, thực hiện lần lượt: **(1)** truy cập **SNS > Topics** và chọn `sportbooking-prod-alerts` và **(2)** chọn **Delete**.

   ![Chọn SNS topic cần xóa](/images/5-Workshop/5.8-Cleanup/cleanup-sns-topic-01.png)

   <p class="image-caption"><em>Hình 6: Chọn SNS topic cần xóa</em></p>
   CloudWatch alarm và subscription đã được xóa nên topic không còn thành phần phụ thuộc trong phạm vi dự án.


2. **Xác nhận xóa topic**

   Trên giao diện, thực hiện lần lượt: **(1)** nhập `delete me` theo yêu cầu xác nhận và **(2)** chọn **Delete**.

   ![Xác nhận xóa SNS topic](/images/5-Workshop/5.8-Cleanup/cleanup-sns-topic-02.png)

   <p class="image-caption"><em>Hình 7: Xác nhận xóa SNS topic</em></p>
   Xóa topic là thao tác không thể hoàn tác. Chuỗi xác nhận giúp hạn chế xóa nhầm kênh thông báo.


3. **Kiểm tra topic đã được xóa**

   Xác nhận thông báo **Topic deleted successfully**. Sau đó, kiểm tra danh sách không còn topic của dự án.

   ![Kiểm tra SNS topic đã được xóa](/images/5-Workshop/5.8-Cleanup/cleanup-sns-topic-03.png)

   <p class="image-caption"><em>Hình 8: Kiểm tra SNS topic đã được xóa</em></p>
   Kết quả này hoàn tất phần dọn dẹp kênh cảnh báo SNS.


### Dọn dẹp lớp ứng dụng và cân bằng tải

#### 4. Hạ tải và xóa Auto Scaling Group

Auto Scaling Group phải được xử lý trước Launch Template. Việc giảm capacity về `0` giúp các EC2 instance được kết thúc có kiểm soát trước khi xóa cấu hình mở rộng.

**Quy trình thực hiện:**

1. **Mở cấu hình capacity**

   Trên giao diện, thực hiện lần lượt: **(1)** chọn `sportbooking-asg` và **(2)** trong phần **Capacity overview**, chọn **Edit**.

   ![Mở cấu hình capacity của Auto Scaling Group](/images/5-Workshop/5.8-Cleanup/cleanup-asg-01.png)

   <p class="image-caption"><em>Hình 9: Mở cấu hình capacity của Auto Scaling Group</em></p>
   ASG đang duy trì hai EC2 instance healthy. Nếu không giảm capacity, ASG tiếp tục tạo instance thay thế khi một instance bị kết thúc.


2. **Đưa capacity về 0**

   Trên giao diện, thực hiện lần lượt: **(1)** đặt **Desired capacity** bằng `0`; **(2)** đặt **Min desired capacity** bằng `0`; và **(3)** chọn **Update**.

   ![Đưa capacity của Auto Scaling Group về 0](/images/5-Workshop/5.8-Cleanup/cleanup-asg-02.png)

   <p class="image-caption"><em>Hình 10: Đưa capacity của Auto Scaling Group về 0</em></p>
   Desired và minimum capacity cùng bằng `0` cho phép ASG kết thúc toàn bộ EC2 instance mà không tạo instance mới. Maximum capacity có thể giữ nguyên vì ASG sẽ được xóa ở bước sau.


3. **Kiểm tra ASG không còn instance**

   Làm mới trang cho đến khi cột **Instances** bằng `0`. Sau đó, kiểm tra **Desired capacity** và giới hạn tối thiểu đều bằng `0`.

   ![Kiểm tra Auto Scaling Group đã hạ về 0](/images/5-Workshop/5.8-Cleanup/cleanup-asg-03.png)

   <p class="image-caption"><em>Hình 11: Kiểm tra Auto Scaling Group đã hạ về 0</em></p>
   Chỉ xóa ASG sau khi các EC2 instance đã kết thúc để giảm nguy cơ còn ENI hoặc EBS volume đang được sử dụng.


4. **Bắt đầu xóa ASG**

   Trên giao diện, thực hiện lần lượt: **(1)** mở menu **Actions** và **(2)** chọn **Delete**.

   ![Bắt đầu xóa Auto Scaling Group](/images/5-Workshop/5.8-Cleanup/cleanup-asg-04.png)

   <p class="image-caption"><em>Hình 12: Bắt đầu xóa Auto Scaling Group</em></p>
   Khi không còn instance, ASG có thể được xóa mà không ảnh hưởng đến dữ liệu đang chạy trên EC2.


5. **Xác nhận xóa ASG**

   Trên giao diện, thực hiện lần lượt: **(1)** nhập `delete` vào ô xác nhận và **(2)** chọn **Delete**.

   ![Xác nhận xóa Auto Scaling Group](/images/5-Workshop/5.8-Cleanup/cleanup-asg-05.png)

   <p class="image-caption"><em>Hình 13: Xác nhận xóa Auto Scaling Group</em></p>
   Sau khi ASG bị xóa, Launch Template không còn được cấu hình mở rộng tự động tham chiếu.


{{% notice tip %}}
Sau khi xóa ASG, kiểm tra thêm **EC2 > Instances** và **Elastic Block Store > Volumes**. Chỉ giữ volume hoặc snapshot đã được xác định là bản sao lưu cần thiết.
{{% /notice %}}

#### 5. Xóa Application Load Balancer

ALB phải được xóa trước Target Group vì listener của ALB đang tham chiếu Target Group.

**Quy trình thực hiện:**

1. **Chọn và xóa ALB**

   Trên giao diện, thực hiện lần lượt: **(1)** truy cập **EC2 > Load Balancers** và chọn `sportbooking-alb`; **(2)** mở menu **Actions**; và **(3)** chọn **Delete load balancer**, sau đó xác nhận trong hộp thoại được hiển thị.

   ![Chọn Application Load Balancer cần xóa](/images/5-Workshop/5.8-Cleanup/cleanup-alb-01.png)

   <p class="image-caption"><em>Hình 14: Chọn Application Load Balancer cần xóa</em></p>
   Xóa ALB đồng thời loại bỏ các listener HTTP/HTTPS và giải phóng liên kết sử dụng ACM certificate. Cần chờ ALB biến mất khỏi danh sách trước khi xóa Target Group.


#### 6. Xóa Target Group

**Quy trình thực hiện:**

1. **Chọn Target Group**

   Trên giao diện, thực hiện lần lượt: **(1)** truy cập **EC2 > Target Groups** và chọn `sportbooking-tg`; **(2)** mở menu **Actions**; và **(3)** chọn **Delete**.

   ![Chọn Target Group cần xóa](/images/5-Workshop/5.8-Cleanup/cleanup-target-group-01.png)

   <p class="image-caption"><em>Hình 15: Chọn Target Group cần xóa</em></p>
   Target Group chỉ được xóa sau khi ALB và listener không còn tham chiếu. Nếu AWS báo resource đang được sử dụng, cần kiểm tra lại ALB hoặc listener rule còn tồn tại.


2. **Xác nhận xóa Target Group**

   Kiểm tra tên `sportbooking-tg`. Sau đó, chọn **Delete**.

   ![Xác nhận xóa Target Group](/images/5-Workshop/5.8-Cleanup/cleanup-target-group-02.png)

   <p class="image-caption"><em>Hình 16: Xác nhận xóa Target Group</em></p>
   Các target EC2 đã được kết thúc khi ASG giảm về `0`, do đó Target Group không còn backend cần duy trì.


#### 7. Xóa Launch Template

**Quy trình thực hiện:**

1. **Chọn Launch Template**

   Trên giao diện, thực hiện lần lượt: **(1)** truy cập **EC2 > Launch Templates** và chọn `sportbooking-lt`; **(2)** mở menu **Actions**; và **(3)** chọn **Delete template**.

   ![Chọn Launch Template cần xóa](/images/5-Workshop/5.8-Cleanup/cleanup-launch-template-01.png)

   <p class="image-caption"><em>Hình 17: Chọn Launch Template cần xóa</em></p>
   Launch Template được xóa sau ASG để không còn cấu hình nào tham chiếu đến template hoặc các version của template.


2. **Xác nhận xóa Launch Template**

   Trên giao diện, thực hiện lần lượt: **(1)** nhập `Delete` đúng chữ hoa, chữ thường theo yêu cầu trên màn hình và **(2)** chọn **Delete**.

   ![Xác nhận xóa Launch Template](/images/5-Workshop/5.8-Cleanup/cleanup-launch-template-02.png)

   <p class="image-caption"><em>Hình 18: Xác nhận xóa Launch Template</em></p>
   Xóa template sẽ xóa toàn bộ version của `sportbooking-lt`; thao tác này không thể hoàn tác.


#### 8. Gỡ policy và xóa IAM Role

IAM Role chỉ được xóa sau khi các EC2 instance đã kết thúc và không còn instance profile đang được sử dụng.

**Quy trình thực hiện:**

1. **Chọn các policy đang gắn với role**

   Trên giao diện, thực hiện lần lượt: **(1)** mở role `sportbooking-ec2-role` và chọn toàn bộ policy đang được attach và **(2)** chọn **Remove**.

   ![Chọn các policy cần gỡ khỏi IAM Role](/images/5-Workshop/5.8-Cleanup/cleanup-iam-role-01.png)

   <p class="image-caption"><em>Hình 19: Chọn các policy cần gỡ khỏi IAM Role</em></p>
   Role trong hình đang gắn các AWS managed policy và customer managed policy `sportbooking-ec2-policy`. Cần gỡ các liên kết này trước khi xóa role.


2. **Xác nhận gỡ policy**

   Kiểm tra số lượng policy đã chọn. Sau đó, chọn **Remove**.

   ![Xác nhận gỡ policy khỏi IAM Role](/images/5-Workshop/5.8-Cleanup/cleanup-iam-role-02.png)

   <p class="image-caption"><em>Hình 20: Xác nhận gỡ policy khỏi IAM Role</em></p>
   AWS managed policy chỉ được detach khỏi role, không bị xóa khỏi AWS account.


3. **Chọn role cần xóa**

   Trên giao diện, thực hiện lần lượt: **(1)** quay lại **IAM > Roles** và chọn `sportbooking-ec2-role` và **(2)** chọn **Delete**.

   ![Chọn IAM Role cần xóa](/images/5-Workshop/5.8-Cleanup/cleanup-iam-role-03.png)

   <p class="image-caption"><em>Hình 21: Chọn IAM Role cần xóa</em></p>
   Chỉ role của dự án được chọn. Không xóa service-linked role hoặc role dùng chung với tài nguyên khác.


4. **Xác nhận xóa role**

   Nhập chính xác `sportbooking-ec2-role`. Sau đó, chọn **Delete**.

   ![Xác nhận xóa IAM Role](/images/5-Workshop/5.8-Cleanup/cleanup-iam-role-04.png)

   <p class="image-caption"><em>Hình 22: Xác nhận xóa IAM Role</em></p>
   Xóa role loại bỏ quyền mà EC2 đã sử dụng để truy cập S3, Secrets Manager, Systems Manager và CloudWatch.


{{% notice info %}}
Customer managed policy `sportbooking-ec2-policy` vẫn tồn tại sau khi detach. Nếu policy chỉ phục vụ dự án, vào **IAM > Policies**, xác nhận không còn attached entity rồi xóa policy và các version không mặc định của policy.
{{% /notice %}}

### Dọn dẹp dữ liệu và cấu hình ứng dụng

#### 9. Xóa RDS MySQL

**Quy trình thực hiện:**

1. **Bắt đầu xóa DB instance**

   Trên giao diện, thực hiện lần lượt: **(1)** truy cập **RDS > Databases** và chọn `sportbooking-mysql`; **(2)** mở menu **Actions**; và **(3)** chọn **Delete**.

   ![Chọn RDS MySQL cần xóa](/images/5-Workshop/5.8-Cleanup/cleanup-rds-01.png)

   <p class="image-caption"><em>Hình 23: Chọn RDS MySQL cần xóa</em></p>
   EC2 đã được kết thúc nên không còn kết nối ứng dụng đang sử dụng database.


2. **Cấu hình phương án lưu dữ liệu và xác nhận**

   Chọn **Create final snapshot** và đặt tên `sportbooking-mysql-snapshot`. Sau đó, chọn **Retain automated backups** để duy trì khả năng phục hồi. Cuối cùng, **(1)** nhập `delete me` và **(2)** chọn **Delete**.

   ![Cấu hình snapshot và xác nhận xóa RDS](/images/5-Workshop/5.8-Cleanup/cleanup-rds-02.png)

   <p class="image-caption"><em>Hình 24: Cấu hình snapshot và xác nhận xóa RDS</em></p>
   Final snapshot `sportbooking-mysql-snapshot` và automated backup đã được giữ lại theo kế hoạch lưu trữ sau khi DB instance bị xóa.


3. **Kiểm tra DB instance đã được xóa**

   Chờ quá trình xóa hoàn tất. Sau đó, xác nhận thông báo **Successfully deleted DB instance sportbooking-mysql**.

   ![Kiểm tra RDS MySQL đã được xóa](/images/5-Workshop/5.8-Cleanup/cleanup-rds-03.png)

   <p class="image-caption"><em>Hình 25: Kiểm tra RDS MySQL đã được xóa</em></p>
   DB instance không còn phát sinh chi phí compute, nhưng snapshot và retained backup đã chọn vẫn là tài nguyên lưu trữ cần được quản lý.


{{% notice warning %}}
Final snapshot và retained automated backup vẫn có thể phát sinh chi phí lưu trữ sau khi DB instance bị xóa. Hai tài nguyên này phải được theo dõi theo thời hạn lưu đã thống nhất.
{{% /notice %}}

#### 10. Làm rỗng và xóa S3 bucket

Artifact bucket đã được làm rỗng trước khi xóa. Upload bucket được giữ lại trong lần dọn dẹp này để bảo toàn ảnh do người dùng tải lên.

**Quy trình thực hiện:**

1. **Chọn bucket và bắt đầu làm rỗng**

   Trên giao diện, thực hiện lần lượt: **(1)** chọn bucket `sportbooking-artifacts-3stars-us-east-1` và **(2)** chọn **Empty**.

   ![Chọn S3 artifact bucket cần làm rỗng](/images/5-Workshop/5.8-Cleanup/cleanup-s3-01.png)

   <p class="image-caption"><em>Hình 26: Chọn S3 artifact bucket cần làm rỗng</em></p>
   S3 không cho phép xóa bucket khi vẫn còn object, object version hoặc delete marker.


2. **Xác nhận làm rỗng bucket**

   Trên giao diện, thực hiện lần lượt: **(1)** nhập `permanently delete` và **(2)** chọn **Empty**.

   ![Xác nhận làm rỗng S3 bucket](/images/5-Workshop/5.8-Cleanup/cleanup-s3-02.png)

   <p class="image-caption"><em>Hình 27: Xác nhận làm rỗng S3 bucket</em></p>
   Thao tác làm rỗng xóa dữ liệu trong bucket và không thể hoàn tác nếu không có bản sao lưu ở nơi khác.


3. **Kiểm tra bucket đã rỗng**

   Xác nhận trạng thái **Successfully emptied bucket**. Sau đó, kiểm tra số object xóa thành công và số object xóa thất bại.

   ![Kiểm tra S3 bucket đã được làm rỗng](/images/5-Workshop/5.8-Cleanup/cleanup-s3-03.png)

   <p class="image-caption"><em>Hình 28: Kiểm tra S3 bucket đã được làm rỗng</em></p>
   Trong kết quả minh họa, hai object với tổng dung lượng 82,5 MB đã được xóa và không có object thất bại.


4. **Chọn xóa bucket**

   Trên giao diện, thực hiện lần lượt: **(1)** quay lại danh sách và chọn artifact bucket vừa làm rỗng và **(2)** chọn **Delete**.

   ![Chọn xóa S3 artifact bucket](/images/5-Workshop/5.8-Cleanup/cleanup-s3-04.png)

   <p class="image-caption"><em>Hình 29: Chọn xóa S3 artifact bucket</em></p>
   Chỉ xóa bucket của dự án. Bucket `cf-templates-...` có thể được CloudFormation hoặc môi trường khác sử dụng nên không được xóa khi chưa xác minh.


5. **Xác nhận tên bucket**

   Trên giao diện, thực hiện lần lượt: **(1)** nhập chính xác tên bucket và **(2)** chọn **Delete bucket**.

   ![Xác nhận xóa S3 artifact bucket](/images/5-Workshop/5.8-Cleanup/cleanup-s3-05.png)

   <p class="image-caption"><em>Hình 30: Xác nhận xóa S3 artifact bucket</em></p>
   Tên S3 bucket là duy nhất toàn cục; sau khi xóa, không có bảo đảm tên này vẫn còn khả dụng để tạo lại.


6. **Kiểm tra bucket đã được xóa**

   Xác nhận thông báo **Successfully deleted bucket**. Sau đó, kiểm tra artifact bucket không còn trong danh sách.

   ![Kiểm tra S3 artifact bucket đã được xóa](/images/5-Workshop/5.8-Cleanup/cleanup-s3-06.png)

   <p class="image-caption"><em>Hình 31: Kiểm tra S3 artifact bucket đã được xóa</em></p>
   Bucket `sportbooking-uploads-3stars-us-east-1` vẫn xuất hiện trong danh sách sau khi artifact bucket bị xóa; đây là tài nguyên lưu giữ có chủ đích, không phải tài nguyên bị bỏ sót.


#### 11. Lên lịch xóa secret

**Quy trình thực hiện:**

1. **Chọn xóa secret**

   Mở secret `sportbooking/prod/app-env`. Tiếp theo, **(1)** mở menu **Actions** và **(2)** chọn **Delete secret**.

   ![Chọn secret cần xóa](/images/5-Workshop/5.8-Cleanup/cleanup-secret-01.png)

   <p class="image-caption"><em>Hình 32: Chọn secret cần xóa</em></p>
   EC2 và IAM Role đã được xóa nên secret không còn workload nào đọc trong phạm vi dự án.


2. **Đặt thời gian chờ và lên lịch xóa**

   Trên giao diện, thực hiện lần lượt: **(1)** đặt **Waiting period** là `7` ngày hoặc thời gian phù hợp với chính sách lưu giữ và **(2)** chọn **Schedule deletion**.

   ![Lên lịch xóa Secrets Manager secret](/images/5-Workshop/5.8-Cleanup/cleanup-secret-02.png)

   <p class="image-caption"><em>Hình 33: Lên lịch xóa Secrets Manager secret</em></p>
   Secrets Manager không xóa ngay mà đưa secret vào thời gian chờ khôi phục. Trong thời gian này, secret bị vô hiệu hóa và có thể được restore nếu phát hiện xóa nhầm.


{{% notice tip %}}
Không đưa giá trị secret vào ảnh hoặc nội dung báo cáo. Nếu cần triển khai lại trong thời gian chờ, khôi phục secret trước khi cập nhật thay vì tạo trùng tên.
{{% /notice %}}

#### 12. Xóa ACM certificate

ACM certificate chỉ được xóa khi cột **In use** hiển thị **No**. Nếu còn được ALB hoặc CloudFront sử dụng, phải gỡ liên kết đó trước.

**Quy trình thực hiện:**

1. **Chọn certificate**

   Trên giao diện, thực hiện lần lượt: **(1)** chọn certificate của `sport.younglilliu.id.vn` và kiểm tra **In use = No** và **(2)** mở **More actions** và chọn **Delete**.

   ![Chọn ACM certificate cần xóa](/images/5-Workshop/5.8-Cleanup/cleanup-acm-01.png)

   <p class="image-caption"><em>Hình 34: Chọn ACM certificate cần xóa</em></p>
   ALB đã được xóa nên HTTPS listener không còn giữ certificate. CloudFront distribution chưa được tạo nên certificate không có thêm phụ thuộc từ CloudFront.


2. **Xác nhận xóa certificate**

   Trên giao diện, thực hiện lần lượt: **(1)** nhập `delete` và **(2)** chọn **Delete**.

   ![Xác nhận xóa ACM certificate](/images/5-Workshop/5.8-Cleanup/cleanup-acm-02.png)

   <p class="image-caption"><em>Hình 35: Xác nhận xóa ACM certificate</em></p>
   Sau khi xóa, certificate không thể tiếp tục phục vụ kết nối HTTPS và không thể khôi phục.


3. **Kiểm tra certificate đã được xóa**

   Xác nhận thông báo **Successfully deleted certificate**. Sau đó, kiểm tra danh sách không còn certificate của dự án.

   ![Kiểm tra ACM certificate đã được xóa](/images/5-Workshop/5.8-Cleanup/cleanup-acm-03.png)

   <p class="image-caption"><em>Hình 36: Kiểm tra ACM certificate đã được xóa</em></p>
   Sau bước này có thể xóa CNAME validation của certificate trong Route 53 nếu record không còn phục vụ certificate khác.


### Dọn dẹp lớp mạng

#### 13. Xóa NAT Gateway

Hai NAT Gateway được xóa trước Elastic IP và VPC. Cần chờ từng NAT Gateway chuyển sang `Deleted` để Elastic IP không còn association.

**Quy trình thực hiện:**

1. **Chọn NAT Gateway tại Availability Zone thứ nhất**

   Trên giao diện, thực hiện lần lượt: **(1)** chọn `sportbooking-nat-a`; **(2)** mở menu **Actions**; và **(3)** chọn **Delete NAT gateway**.

   ![Chọn NAT Gateway thứ nhất cần xóa](/images/5-Workshop/5.8-Cleanup/cleanup-nat-gateway-01.png)

   <p class="image-caption"><em>Hình 37: Chọn NAT Gateway thứ nhất cần xóa</em></p>
   EC2 private đã được kết thúc nên không còn nhu cầu truy cập Internet thông qua NAT Gateway.


2. **Xác nhận xóa NAT Gateway thứ nhất**

   Trên giao diện, thực hiện lần lượt: **(1)** nhập `delete` và **(2)** chọn **Delete**.

   ![Xác nhận xóa NAT Gateway thứ nhất](/images/5-Workshop/5.8-Cleanup/cleanup-nat-gateway-02.png)

   <p class="image-caption"><em>Hình 38: Xác nhận xóa NAT Gateway thứ nhất</em></p>
   NAT Gateway đã chuyển qua trạng thái `Deleting` trong thời gian AWS xử lý yêu cầu.


3. **Chọn NAT Gateway tại Availability Zone thứ hai**

   Kiểm tra thông báo xóa NAT Gateway thứ nhất. Sau đó, **(1)** chọn `sportbooking-nat-b`; **(2)** mở menu **Actions**; và **(3)** chọn **Delete NAT gateway**.

   ![Chọn NAT Gateway thứ hai cần xóa](/images/5-Workshop/5.8-Cleanup/cleanup-nat-gateway-03.png)

   <p class="image-caption"><em>Hình 39: Chọn NAT Gateway thứ hai cần xóa</em></p>
   Kiến trúc sử dụng NAT Gateway theo hai Availability Zone, do đó cần xử lý cả hai tài nguyên.


4. **Xác nhận xóa NAT Gateway thứ hai**

   Trên giao diện, thực hiện lần lượt: **(1)** nhập `delete` và **(2)** chọn **Delete**.

   ![Xác nhận xóa NAT Gateway thứ hai](/images/5-Workshop/5.8-Cleanup/cleanup-nat-gateway-04.png)

   <p class="image-caption"><em>Hình 40: Xác nhận xóa NAT Gateway thứ hai</em></p>
   Cả hai NAT Gateway phải được xóa để VPC không còn tài nguyên mạng có tính phí theo giờ.


5. **Kiểm tra trạng thái hai NAT Gateway**

   Làm mới danh sách cho đến khi `sportbooking-nat-a` và `sportbooking-nat-b` đều ở trạng thái **Deleted**.

   ![Kiểm tra hai NAT Gateway đã được xóa](/images/5-Workshop/5.8-Cleanup/cleanup-nat-gateway-05.png)

   <p class="image-caption"><em>Hình 41: Kiểm tra hai NAT Gateway đã được xóa</em></p>
   Chỉ giải phóng Elastic IP sau khi NAT Gateway đã xóa xong; nếu thực hiện quá sớm, địa chỉ vẫn đang được association và không thể release.


{{% notice info %}}
Làm mới trạng thái định kỳ và chỉ chuyển sang giải phóng Elastic IP sau khi cả hai NAT Gateway hiển thị `Deleted`.
{{% /notice %}}

#### 14. Giải phóng Elastic IP

**Quy trình thực hiện:**

1. **Chọn các Elastic IP của NAT Gateway**

   Trên giao diện, thực hiện lần lượt: **(1)** chọn `sportbooking-eip-us-east-1a` và `sportbooking-eip-us-east-1b`; **(2)** mở menu **Actions**; và **(3)** chọn **Release Elastic IP addresses**.

   ![Chọn Elastic IP cần giải phóng](/images/5-Workshop/5.8-Cleanup/cleanup-elastic-ip-01.png)

   <p class="image-caption"><em>Hình 42: Chọn Elastic IP cần giải phóng</em></p>
   Sau khi NAT Gateway bị xóa, hai Elastic IP không còn association và cần được trả lại AWS.


2. **Xác nhận giải phóng Elastic IP**

   Kiểm tra đúng hai địa chỉ IP và Allocation ID. Sau đó, chọn **Release**.

   ![Xác nhận giải phóng Elastic IP](/images/5-Workshop/5.8-Cleanup/cleanup-elastic-ip-02.png)

   <p class="image-caption"><em>Hình 43: Xác nhận giải phóng Elastic IP</em></p>
   Elastic IP đã release không còn thuộc AWS account và có thể được cấp cho tài khoản khác.


3. **Kiểm tra danh sách Elastic IP**

   Xác nhận thông báo **Elastic IP addresses released**. Sau đó, kiểm tra không còn Elastic IP của dự án trong Region.

   ![Kiểm tra Elastic IP đã được giải phóng](/images/5-Workshop/5.8-Cleanup/cleanup-elastic-ip-03.png)

   <p class="image-caption"><em>Hình 44: Kiểm tra Elastic IP đã được giải phóng</em></p>
   Kết quả này xác nhận lớp NAT không còn địa chỉ IPv4 public được cấp phát riêng.


#### 15. Xóa S3 Gateway VPC Endpoint

**Quy trình thực hiện:**

1. **Chọn VPC Endpoint**

   Trên giao diện, thực hiện lần lượt: **(1)** chọn `sportbooking-vpce-s3`; **(2)** mở menu **Actions**; và **(3)** chọn **Delete VPC endpoints**.

   ![Chọn S3 Gateway VPC Endpoint cần xóa](/images/5-Workshop/5.8-Cleanup/cleanup-vpc-endpoint-01.png)

   <p class="image-caption"><em>Hình 45: Chọn S3 Gateway VPC Endpoint cần xóa</em></p>
   EC2 và S3 bucket đã được xử lý nên endpoint không còn lưu lượng cần phục vụ. Xóa endpoint cũng loại bỏ liên kết endpoint khỏi route table.


2. **Xác nhận xóa endpoint**

   Trên giao diện, thực hiện lần lượt: **(1)** nhập `delete` và **(2)** chọn **Delete**.

   ![Xác nhận xóa S3 Gateway VPC Endpoint](/images/5-Workshop/5.8-Cleanup/cleanup-vpc-endpoint-02.png)

   <p class="image-caption"><em>Hình 46: Xác nhận xóa S3 Gateway VPC Endpoint</em></p>
   Endpoint bị xóa vĩnh viễn và không thể khôi phục từ cấu hình hiện tại.


3. **Kiểm tra endpoint đã được xóa**

   Xác nhận thông báo **Successfully deleted endpoints**. Sau đó, kiểm tra danh sách không còn `sportbooking-vpce-s3`.

   ![Kiểm tra S3 Gateway VPC Endpoint đã được xóa](/images/5-Workshop/5.8-Cleanup/cleanup-vpc-endpoint-03.png)

   <p class="image-caption"><em>Hình 47: Kiểm tra S3 Gateway VPC Endpoint đã được xóa</em></p>
   VPC không còn endpoint tùy chỉnh có thể cản trở thao tác xóa cuối cùng.


#### 16. Gỡ rule tham chiếu và xóa Security Group

Security Group không thể xóa khi còn gắn với ENI hoặc bị Security Group khác tham chiếu. ALB, EC2 và RDS đã được xóa trước khi thực hiện phần này.

**Quy trình thực hiện:**

1. **Mở outbound rules của ALB Security Group**

   Mở `sportbooking-alb` Security Group. Sau đó, chọn tab **Outbound rules** và chọn **Edit outbound rules**.

   ![Mở outbound rules của ALB Security Group](/images/5-Workshop/5.8-Cleanup/cleanup-security-group-01.png)

   <p class="image-caption"><em>Hình 48: Mở outbound rules của ALB Security Group</em></p>
   Outbound rule port `8080` đang tham chiếu EC2 App Security Group. Tham chiếu này cần được gỡ trước khi xóa đồng thời các Security Group của dự án.


2. **Xóa outbound rule tham chiếu**

   Trên giao diện, thực hiện lần lượt: **(1)** chọn **Delete** tại rule port `8080` trỏ đến App Security Group và **(2)** chọn **Save rules**.

   ![Xóa outbound rule tham chiếu App Security Group](/images/5-Workshop/5.8-Cleanup/cleanup-security-group-02.png)

   <p class="image-caption"><em>Hình 49: Xóa outbound rule tham chiếu App Security Group</em></p>
   Việc xóa rule phá vỡ liên kết từ ALB Security Group đến App Security Group.


3. **Mở inbound rules của ALB Security Group**

   Chọn tab **Inbound rules**. Sau đó, chọn **Edit inbound rules**.

   ![Mở inbound rules của ALB Security Group](/images/5-Workshop/5.8-Cleanup/cleanup-security-group-03.png)

   <p class="image-caption"><em>Hình 50: Mở inbound rules của ALB Security Group</em></p>
   Hai rule HTTP và HTTPS public được loại bỏ để đóng hoàn toàn đường truy cập vào Security Group trước khi xóa.


4. **Xóa các inbound rule còn lại**

   Trên giao diện, thực hiện lần lượt: **(1)** xóa rule HTTPS port `443`; **(2)** xóa rule HTTP port `80`; và **(3)** chọn **Save rules**.

   ![Xóa inbound rules của ALB Security Group](/images/5-Workshop/5.8-Cleanup/cleanup-security-group-04.png)

   <p class="image-caption"><em>Hình 51: Xóa inbound rules của ALB Security Group</em></p>
   Các rule dùng CIDR không tạo phụ thuộc giữa Security Group, nhưng việc xóa chúng giúp xác nhận Security Group không còn cho phép lưu lượng trong thời gian chờ dọn dẹp.


5. **Chọn các Security Group của dự án**

   Kiểm tra và gỡ tương tự mọi rule tham chiếu giữa `sportbooking-app`, `sportbooking-rds` và `sportbooking-alb` nếu còn tồn tại. Sau đó, **(1)** chọn đúng ba Security Group của dự án và **(2)** mở **Actions**, chọn **Delete security groups**.

   ![Chọn các Security Group của SportBooking](/images/5-Workshop/5.8-Cleanup/cleanup-security-group-05.png)

   <p class="image-caption"><em>Hình 52: Chọn các Security Group của SportBooking</em></p>
   Không chọn Security Group `default` hoặc Security Group thuộc VPC khác. Nếu AWS báo dependency, kiểm tra ENI và rule tham chiếu còn lại trước khi thử lại.


6. **Xác nhận xóa Security Group**

   Kiểm tra danh sách gồm `sportbooking-rds`, `sportbooking-app` và `sportbooking-alb`. Tiếp theo, **(1)** nhập `delete` và **(2)** chọn **Delete**.

   ![Xác nhận xóa các Security Group](/images/5-Workshop/5.8-Cleanup/cleanup-security-group-06.png)

   <p class="image-caption"><em>Hình 53: Xác nhận xóa các Security Group</em></p>
   Xóa đồng thời các Security Group sau khi gỡ tham chiếu giúp tránh để lại cấu hình mạng không còn tài nguyên sử dụng.


7. **Kiểm tra kết quả**

   Xác nhận thông báo **Successfully deleted 3 security groups**. Sau đó, kiểm tra chỉ còn các Security Group mặc định hoặc thuộc môi trường khác.

   ![Kiểm tra các Security Group đã được xóa](/images/5-Workshop/5.8-Cleanup/cleanup-security-group-07.png)

   <p class="image-caption"><em>Hình 54: Kiểm tra các Security Group đã được xóa</em></p>
   Thành công ở bước này cho thấy các ENI và liên kết Security Group của ALB, EC2 và RDS đã được giải phóng.


#### 17. Xóa DB Subnet Group

DB Subnet Group được xóa sau RDS vì DB instance đang sử dụng subnet group sẽ ngăn thao tác xóa.

**Quy trình thực hiện:**

1. **Chọn DB Subnet Group**

   Trên giao diện, thực hiện lần lượt: **(1)** truy cập **RDS > Subnet groups** và chọn `sportbooking-db-subnet-group` và **(2)** chọn **Delete**.

   ![Chọn DB Subnet Group cần xóa](/images/5-Workshop/5.8-Cleanup/cleanup-db-subnet-group-01.png)

   <p class="image-caption"><em>Hình 55: Chọn DB Subnet Group cần xóa</em></p>
   RDS đã được xóa hoàn tất nên subnet group không còn database nào tham chiếu.


2. **Xác nhận xóa DB Subnet Group**

   Kiểm tra đúng tên `sportbooking-db-subnet-group`. Sau đó, chọn **Delete**.

   ![Xác nhận xóa DB Subnet Group](/images/5-Workshop/5.8-Cleanup/cleanup-db-subnet-group-02.png)

   <p class="image-caption"><em>Hình 56: Xác nhận xóa DB Subnet Group</em></p>
   Thao tác này chỉ xóa cấu hình DB Subnet Group của RDS, không xóa subnet VPC trực tiếp.


3. **Kiểm tra DB Subnet Group đã được xóa**

   Xác nhận thông báo xóa thành công. Sau đó, kiểm tra danh sách chỉ còn subnet group mặc định hoặc của môi trường khác.

   ![Kiểm tra DB Subnet Group đã được xóa](/images/5-Workshop/5.8-Cleanup/cleanup-db-subnet-group-03.png)

   <p class="image-caption"><em>Hình 57: Kiểm tra DB Subnet Group đã được xóa</em></p>
   VPC không còn cấu hình RDS tùy chỉnh phụ thuộc vào các private subnet.


#### 18. Xóa VPC

VPC được xóa cuối cùng sau khi các tài nguyên compute, database, endpoint, NAT Gateway và Security Group đã được xử lý.

**Quy trình thực hiện:**

1. **Chọn VPC của dự án**

   Trên giao diện, thực hiện lần lượt: **(1)** truy cập **VPC > Your VPCs** và chọn `sportbooking-vpc`; **(2)** mở menu **Actions**; và **(3)** chọn **Delete VPC**.

   ![Chọn VPC của SportBooking cần xóa](/images/5-Workshop/5.8-Cleanup/cleanup-vpc-01.png)

   <p class="image-caption"><em>Hình 58: Chọn VPC của SportBooking cần xóa</em></p>
   Tên, VPC ID và CIDR đã được đối chiếu để không chọn nhầm default VPC hoặc VPC của môi trường khác.


2. **Kiểm tra tài nguyên phụ thuộc và xác nhận**

   Kiểm tra danh sách các tài nguyên sẽ bị xóa cùng VPC, gồm Internet Gateway, subnet và route table còn lại. Tiếp theo, **(1)** nhập `delete` và **(2)** chọn **Delete**.

   ![Kiểm tra tài nguyên phụ thuộc và xác nhận xóa VPC](/images/5-Workshop/5.8-Cleanup/cleanup-vpc-02.png)

   <p class="image-caption"><em>Hình 59: Kiểm tra tài nguyên phụ thuộc và xác nhận xóa VPC</em></p>
   Hộp thoại cho biết `sportbooking-vpc` và 12 tài nguyên mạng liên quan nằm trong yêu cầu xóa; toàn bộ danh sách đã được xác nhận thuộc dự án.


3. **Kiểm tra VPC đã được xóa**

   Xác nhận thông báo **successfully deleted sportbooking-vpc and 12 other resources**. Sau đó, kiểm tra danh sách chỉ còn default VPC hoặc các VPC khác cần giữ.

   ![Kiểm tra VPC của SportBooking đã được xóa](/images/5-Workshop/5.8-Cleanup/cleanup-vpc-03.png)

   <p class="image-caption"><em>Hình 60: Kiểm tra VPC của SportBooking đã được xóa</em></p>
   Đây là bước hoàn tất dọn dẹp hạ tầng mạng của SportBooking.


{{% notice warning %}}
Nếu VPC không thể xóa, kiểm tra các ENI còn tồn tại trong **EC2 > Network Interfaces**, VPC Endpoint, NAT Gateway, Load Balancer, RDS, Security Group tham chiếu chéo và tài nguyên do dịch vụ khác quản lý. Không xóa default VPC để xử lý lỗi phụ thuộc.
{{% /notice %}}

### Kiểm tra sau khi dọn dẹp

| Dịch vụ | Trạng thái đã ghi nhận |
|---|---|
| EC2 / Auto Scaling | Không còn `sportbooking-asg`, EC2 instance, Launch Template hoặc EBS volume ngoài kế hoạch lưu giữ. |
| Elastic Load Balancing | Không còn `sportbooking-alb` và `sportbooking-tg`. |
| RDS | Không còn DB instance và DB Subnet Group; snapshot/backup giữ lại đã được ghi nhận. |
| S3 | Artifact bucket đã xóa; `sportbooking-uploads-3stars-us-east-1` được giữ có chủ đích. |
| Secrets Manager | Secret ở trạng thái scheduled for deletion với ngày xóa xác định. |
| CloudWatch / SNS | Không còn alarm, subscription và topic của SportBooking; kiểm tra thêm log group nếu đã cấu hình CloudWatch Agent. |
| VPC | Không còn NAT Gateway, Elastic IP, VPC Endpoint, Security Group tùy chỉnh và `sportbooking-vpc`. |
| IAM | Không còn `sportbooking-ec2-role`; customer managed policy và instance profile của dự án đã được xử lý. |
| ACM / Route 53 | Certificate đã xóa; Alias record và CNAME validation không còn nếu không sử dụng; hosted zone chỉ được giữ khi domain vẫn hoạt động. |
| CloudFront | Không có distribution cần xóa vì bước tạo CloudFront đã dừng tại lỗi xác minh tài khoản. |

{{% notice info %}}
Việc dọn dẹp chỉ được xem là hoàn tất khi danh sách tài nguyên thực tế khớp với kế hoạch lưu giữ. Các snapshot, backup, S3 object, CloudWatch log group, Route 53 hosted zone và customer managed policy còn tồn tại phải có lý do và thời hạn lưu cụ thể.
{{% /notice %}}

Sau bước này, quy trình triển khai SportBooking đã hoàn tất từ xây dựng kiến trúc, triển khai dịch vụ, kiểm thử, vận hành đến thu hồi tài nguyên AWS.


