---
title : "EC2, ALB và Auto Scaling Group"
date : 2024-01-01
weight : 5
chapter : false
pre : " <b> 5.5. </b> "
---

#### Phạm vi triển khai

Sau khi S3 chứa artifact và các thành phần IAM Role, RDS, Secrets Manager sẵn sàng, lớp compute mới được triển khai. Trình tự gồm: tạo EC2 seed để kiểm tra runtime và quyền truy cập, tạo AMI/Launch Template, tạo Target Group và Application Load Balancer, sau đó tạo Auto Scaling Group trong private app subnet.

{{% notice info %}}
Auto Scaling Group được tạo sau Target Group và ALB vì ASG cần gắn Target Group ngay trong quá trình cấu hình. Thứ tự này giúp các EC2 instance mới tự động được đăng ký và health check khi khởi tạo.
{{% /notice %}}

#### Cấu hình EC2 seed

EC2 seed được tạo sau khi object `releases/SportBooingSystem-0.0.1-SNAPSHOT.jar` đã có trong artifact bucket. Instance này được dùng để kiểm tra Amazon Linux, Java runtime, Session Manager, IAM Role và khả năng truy cập các dịch vụ nền trước khi chuẩn hóa thành Launch Template.

#### Báo cáo các bước tạo EC2 seed

EC2 seed chỉ được khởi tạo sau khi S3, IAM, RDS và Secrets Manager sẵn sàng, nhờ đó quá trình kiểm thử có đủ artifact, quyền truy cập và cấu hình ứng dụng.

**Quy trình thực hiện:**

1. **Bắt đầu tạo EC2 seed instance**

   Truy cập **EC2 > Instances**. Sau đó, chọn **Launch instances**.

   ![Bắt đầu tạo EC2 seed instance](/images/5-Workshop/5.5-Policy/ec2-seed-01.png)

   <p class="image-caption"><em>Hình 1: Bắt đầu tạo EC2 seed instance</em></p>
   EC2 seed được dùng để xác minh runtime, IAM, mạng và hoạt động của ứng dụng trước khi tạo AMI cho Launch Template.


2. **Đặt tên instance và chọn AMI**

   Nhập tên instance theo dự án. Sau đó, chọn Amazon Linux 2023.

   ![Đặt tên instance và chọn AMI](/images/5-Workshop/5.5-Policy/ec2-seed-02.png)

   <p class="image-caption"><em>Hình 2: Đặt tên instance và chọn AMI</em></p>
   Tên instance giúp phân biệt máy seed với EC2 do ASG tạo. AMI phải tương thích với Java/runtime của ứng dụng.


3. **Chọn AMI và instance type**

   Xác nhận AMI đã chọn. Sau đó, chọn instance type `t3.small`.

   ![Chọn AMI và instance type](/images/5-Workshop/5.5-Policy/ec2-seed-03.png)

   <p class="image-caption"><em>Hình 3: Chọn AMI và instance type</em></p>
   Seed instance sử dụng `t3.small`, phù hợp với phạm vi kiểm tra runtime và ứng dụng của dự án.


4. **Chọn key pair và network**

   Chọn **Proceed without a key pair** vì instance được quản trị bằng Session Manager. Sau đó, chọn `sportbooking-vpc` và private subnet `sportbooking-app-a`. Tiếp theo, tắt **Auto-assign public IP**.

   ![Chọn key pair và network](/images/5-Workshop/5.5-Policy/ec2-seed-04.png)

   <p class="image-caption"><em>Hình 4: Chọn key pair và network</em></p>
   Instance không có public IP và không dùng key pair; toàn bộ phiên quản trị được thực hiện qua Session Manager.


5. **Gắn security group và storage**

   Chọn Security Group `sportbooking-app`. Sau đó, đặt root volume `gp3` dung lượng `20 GiB`.

   ![Gắn security group và storage](/images/5-Workshop/5.5-Policy/ec2-seed-05.png)

   <p class="image-caption"><em>Hình 5: Gắn security group và storage</em></p>
   Security Group chỉ nhận traffic ứng dụng từ ALB; root volume lưu runtime, file cấu hình và artifact tải từ S3.


6. **Gắn IAM instance profile**

   Mở **Advanced details**. Sau đó, chọn IAM role `sportbooking-ec2-role`. Tiếp theo, chọn **Launch instance**.

   ![Gắn IAM instance profile](/images/5-Workshop/5.5-Policy/ec2-seed-06.png)

   <p class="image-caption"><em>Hình 6: Gắn IAM instance profile</em></p>
   Role cho phép instance đọc artifact S3, đọc secret, thao tác với bucket upload và dùng Session Manager mà không lưu access key.


7. **Kiểm tra instance sau khi launch**

   Mở instance vừa tạo. Sau đó, chờ trạng thái running và status check ổn định.

   ![Kiểm tra instance sau khi launch](/images/5-Workshop/5.5-Policy/ec2-seed-07.png)

   <p class="image-caption"><em>Hình 7: Kiểm tra instance sau khi launch</em></p>
   Phiên kết nối được mở sau khi instance ở trạng thái `Running` và hoàn tất status checks.


8. **Chọn Connect**

   Chọn instance. Sau đó, chọn **Connect**.

   ![Chọn Connect](/images/5-Workshop/5.5-Policy/ec2-seed-08.png)

   <p class="image-caption"><em>Hình 8: Chọn Connect</em></p>
   Phiên kết nối được dùng để kiểm tra quyền IAM, nạp secret, khởi động ứng dụng và kiểm tra HTTP nội bộ.


9. **Kết nối bằng Session Manager**

   Chọn tab **Session Manager**. Sau đó, chọn **Connect**.

   ![Kết nối bằng Session Manager](/images/5-Workshop/5.5-Policy/ec2-seed-09.png)

   <p class="image-caption"><em>Hình 9: Kết nối bằng Session Manager</em></p>
   Session Manager phù hợp private EC2 vì không cần mở SSH inbound hoặc quản lý bastion host.


10. **Kiểm tra terminal EC2**

   Đọc secret `sportbooking/prod/app-env` và tạo `/opt/sportbooking/app.env`. Sau đó, khởi động lại service `sportbooking` và kiểm tra `curl -I http://127.0.0.1:8080/`. Tiếp theo, ghi nhận phản hồi `HTTP/1.1 200` sau khi service khởi động thành công.

   ![Kiểm tra terminal EC2](/images/5-Workshop/5.5-Policy/ec2-seed-10.png)

   <p class="image-caption"><em>Hình 10: Kiểm tra terminal EC2</em></p>
   Terminal giúp xác minh EC2 có quyền IAM, network và runtime đúng trước khi tự động hóa bằng Launch Template.



#### Launch Template

Launch Template định nghĩa cấu hình chuẩn cho EC2 app: AMI, instance type, IAM instance profile, Security Group và User Data. Trong kiến trúc này, EC2 nằm trong private app subnet, không gắn public IP và chỉ nhận traffic từ ALB.

| Mục | Cấu hình |
|---|---|
| Launch Template | `sportbooking-lt`, version description `v1`. |
| AMI | AMI đã kiểm tra từ EC2 seed hoặc Amazon Linux 2023 đã chuẩn hóa runtime. |
| Instance type | `t3.small`. |
| IAM instance profile | `sportbooking-ec2-role`. |
| Security Group | `sportbooking-app`, chỉ nhận port `8080` từ `sportbooking-alb`. |
| Subnet khi tạo ASG | Private app subnets. |
| Public IP | Disable. |
| User Data | Cài runtime, lấy secret, tải jar từ S3, tạo systemd service. |

User Data sử dụng trong Launch Template:

```bash
#!/bin/bash
set -euxo pipefail

APP_DIR="/opt/sportbooking"
APP_ARTIFACT_S3_URI="s3://sportbooking-artifacts-3stars-us-east-1/releases/SportBooingSystem-0.0.1-SNAPSHOT.jar"
APP_ENV_SECRET_ID="sportbooking/prod/app-env"
AWS_DEFAULT_REGION="us-east-1"

mkdir -p "$APP_DIR"
dnf install -y java-21-amazon-corretto-headless jq awscli || true

aws secretsmanager get-secret-value \
  --secret-id "$APP_ENV_SECRET_ID" \
  --region "$AWS_DEFAULT_REGION" \
  --query SecretString \
  --output text | jq -r 'to_entries[] | "\(.key)=\(.value)"' > "$APP_DIR/app.env"

chmod 600 "$APP_DIR/app.env"
aws s3 cp "$APP_ARTIFACT_S3_URI" "$APP_DIR/app.jar" --region "$AWS_DEFAULT_REGION"

cat > /etc/systemd/system/sportbooking.service <<'EOF'
[Unit]
Description=SportBooking Spring Boot Application
After=network-online.target
Wants=network-online.target

[Service]
WorkingDirectory=/opt/sportbooking
EnvironmentFile=/opt/sportbooking/app.env
Environment=AWS_REGION=us-east-1
Environment=AWS_DEFAULT_REGION=us-east-1
Environment=AWS_S3_REGION=us-east-1
ExecStart=/usr/bin/java -Daws.region=us-east-1 -Daws.s3.region=us-east-1 -Dcloud.aws.region.static=us-east-1 -jar /opt/sportbooking/app.jar
Restart=always
RestartSec=10
User=root

[Install]
WantedBy=multi-user.target
EOF

systemctl daemon-reload
systemctl enable sportbooking
systemctl restart sportbooking
```

{{% notice warning %}}
Kiểm tra kỹ `APP_ARTIFACT_S3_URI`, `APP_ENV_SECRET_ID` và `AWS_DEFAULT_REGION` trong User Data. Sai một trong ba giá trị này sẽ làm EC2 khởi động nhưng service ứng dụng không chạy được.
{{% /notice %}}

#### Quy trình thực hiện: Launch Template

Từ EC2 seed đã kiểm thử, tạo AMI rồi xây dựng Launch Template với IAM Role, Security Group và User Data dùng chung cho các instance trong Auto Scaling Group.

**Quy trình thực hiện:**

1. **Tạo AMI từ EC2 seed**

   Mở EC2 seed instance đã chuẩn bị. Sau đó, chọn **Actions > Image and templates > Create image**.

   ![Tạo AMI từ EC2 seed](/images/5-Workshop/5.5-Policy/launch-template-01.png)

   <p class="image-caption"><em>Hình 1: Tạo AMI từ EC2 seed</em></p>
   AMI đóng gói trạng thái máy seed để Launch Template có thể tạo instance mới nhất quán.


2. **Đặt tên AMI**

   Nhập image name theo dự án. Sau đó, thêm description nhận diện AMI của SportBooking.

   ![Đặt tên AMI](/images/5-Workshop/5.5-Policy/launch-template-02.png)

   <p class="image-caption"><em>Hình 2: Đặt tên AMI</em></p>
   Tên AMI rõ ràng giúp biết image nào dùng cho SportBooking và phiên bản triển khai nào.


3. **Tạo image**

   Kiểm tra volume và tag. Sau đó, chọn **Create image**.

   ![Tạo image](/images/5-Workshop/5.5-Policy/launch-template-03.png)

   <p class="image-caption"><em>Hình 3: Tạo image</em></p>
   Image này là base cho instance trong ASG. Nếu chưa tạo AMI hoặc AMI pending, Launch Template không thể dùng ổn định.


4. **Xác nhận AMI tạo thành công**

   Quay lại EC2/AMI hoặc instance. Sau đó, chờ thông báo tạo image thành công.

   ![Xác nhận AMI tạo thành công](/images/5-Workshop/5.5-Policy/launch-template-04.png)

   <p class="image-caption"><em>Hình 4: Xác nhận AMI tạo thành công</em></p>
   Chỉ dùng AMI khi trạng thái available để tránh ASG launch instance lỗi.


5. **Bắt đầu tạo Launch Template**

   Truy cập **EC2 > Launch Templates**. Sau đó, chọn **Create launch template**.

   ![Bắt đầu tạo Launch Template](/images/5-Workshop/5.5-Policy/launch-template-05.png)

   <p class="image-caption"><em>Hình 5: Bắt đầu tạo Launch Template</em></p>
   Launch Template chuẩn hóa AMI, instance type, IAM role, security group và User Data cho mọi EC2 trong ASG.


6. **Nhập tên Launch Template**

   Nhập launch template name `sportbooking-lt` và version description `v1`. Sau đó, bật tùy chọn hướng dẫn cấu hình cho EC2 Auto Scaling.

   ![Nhập tên Launch Template](/images/5-Workshop/5.5-Policy/launch-template-06.png)

   <p class="image-caption"><em>Hình 6: Nhập tên Launch Template</em></p>
   ASG đã tham chiếu Launch Template này bằng tên `sportbooking-lt` và version mặc định `1`.


7. **Chọn AMI cho Launch Template**

   Mở mục AMI. Sau đó, chọn AMI vừa tạo từ seed instance.

   ![Chọn AMI cho Launch Template](/images/5-Workshop/5.5-Policy/launch-template-07.png)

   <p class="image-caption"><em>Hình 7: Chọn AMI cho Launch Template</em></p>
   AMI đảm bảo instance mới có base runtime giống môi trường đã kiểm thử.


8. **Chọn instance type**

   Chọn instance type `t3.small`, gồm 2 vCPU và 2 GiB RAM.

   ![Chọn instance type](/images/5-Workshop/5.5-Policy/launch-template-08.png)

   <p class="image-caption"><em>Hình 8: Chọn instance type</em></p>
   Compute chạy liên tục trong ASG. Chọn instance nhỏ giúp tối ưu chi phí nhưng vẫn đủ chạy Spring Boot.


9. **Chọn key pair và network security**

   Chọn **Don't include in launch template** cho key pair. Sau đó, chọn Security Group `sportbooking-app`.

   ![Chọn key pair và network security](/images/5-Workshop/5.5-Policy/launch-template-09.png)

   <p class="image-caption"><em>Hình 9: Chọn key pair và network security</em></p>
   `sportbooking-app` chỉ cho phép port `8080` từ `sportbooking-alb`; SSH public không được mở.


10. **Gắn IAM instance profile**

   Mở **Advanced details**. Sau đó, chọn IAM instance profile `sportbooking-ec2-role`.

   ![Gắn IAM instance profile](/images/5-Workshop/5.5-Policy/launch-template-10.png)

   <p class="image-caption"><em>Hình 10: Gắn IAM instance profile</em></p>
   Role giúp EC2 đọc S3 artifact, đọc Secrets Manager và đăng ký Session Manager khi khởi động.


11. **Nhập User Data bootstrap**

   Dán script User Data cài runtime, lấy secret, tải jar từ S3 và tạo systemd service. Sau đó, kiểm tra S3 URI, secret id và region.

   ![Nhập User Data bootstrap](/images/5-Workshop/5.5-Policy/launch-template-11.png)

   <p class="image-caption"><em>Hình 11: Nhập User Data bootstrap</em></p>
   User Data biến Launch Template thành quy trình deploy tự động. Instance mới tự kéo cấu hình và artifact mà không cần thao tác tay.


12. **Tạo Launch Template**

   Rà soát cấu hình. Sau đó, chọn **Create launch template**.

   ![Tạo Launch Template](/images/5-Workshop/5.5-Policy/launch-template-12.png)

   <p class="image-caption"><em>Hình 12: Tạo Launch Template</em></p>
   Template là đầu vào của ASG và Instance Refresh. Mỗi lần đổi User Data/AMI có thể tạo version mới.



#### Target Group và Application Load Balancer

Target Group kiểm tra health của EC2 app trên port `8080`. Application Load Balancer là điểm truy cập public, nhận request HTTP/HTTPS và forward vào Target Group.

| Thành phần | Cấu hình |
|---|---|
| Target type | Instances |
| Target Group protocol/port | HTTP : `8080` |
| Health check path | `/actuator/health` |
| ALB scheme | Internet-facing |
| ALB subnets | 2 public subnets ở 2 AZ |
| ALB Security Group | Mở inbound 80/443 từ Internet |
| Listener ban đầu | HTTP:80 forward `sportbooking-tg` |

Kết quả đã ghi nhận: Target Group có health check đúng path, ALB ở trạng thái active và có DNS name để kiểm thử trước khi trỏ domain.

#### Quy trình thực hiện: Target Group và ALB

Target Group được tạo và kiểm tra health check trước khi ALB public chuyển lưu lượng đến ứng dụng; website sau đó được kiểm thử qua DNS của ALB.

**Quy trình thực hiện:**

1. **Bắt đầu tạo Target Group**

   Truy cập **EC2 > Target Groups**. Sau đó, chọn **Create target group**.

   ![Bắt đầu tạo Target Group](/images/5-Workshop/5.5-Policy/alb-01.png)

   <p class="image-caption"><em>Hình 1: Bắt đầu tạo Target Group</em></p>
   Target Group là nơi ALB gửi request tới EC2 app và kiểm tra health.


2. **Chọn target type và protocol**

   Chọn target type **Instances**. Sau đó, nhập tên target group. Tiếp theo, chọn protocol HTTP và port `8080`.

   ![Chọn target type và protocol](/images/5-Workshop/5.5-Policy/alb-02.png)

   <p class="image-caption"><em>Hình 2: Chọn target type và protocol</em></p>
   Spring Boot app chạy ở port 8080 trên EC2. ALB nhận public 80/443 nhưng forward nội bộ tới 8080.


3. **Chọn VPC và protocol version**

   Chọn VPC SportBooking. Sau đó, giữ protocol version HTTP1 nếu app không yêu cầu khác.

   ![Chọn VPC và protocol version](/images/5-Workshop/5.5-Policy/alb-03.png)

   <p class="image-caption"><em>Hình 3: Chọn VPC và protocol version</em></p>
   Target Group phải cùng VPC với EC2 app để đăng ký target được.


4. **Cấu hình health check**

   Nhập health check path `/actuator/health`. Sau đó, giữ protocol HTTP.

   ![Cấu hình health check](/images/5-Workshop/5.5-Policy/alb-04.png)

   <p class="image-caption"><em>Hình 4: Cấu hình health check</em></p>
   Health check giúp ALB chỉ gửi traffic tới instance app đang hoạt động. `/actuator/health` phản ánh trạng thái Spring Boot tốt hơn `/`.


5. **Chuyển sang bước đăng ký target**

   Kiểm tra target selection và attribute. Sau đó, chọn **Next**.

   ![Hoàn tất cấu hình Target Group](/images/5-Workshop/5.5-Policy/alb-05.png)

   <p class="image-caption"><em>Hình 5: Hoàn tất cấu hình Target Group và chuyển sang đăng ký target</em></p>
   Sau khi hoàn tất protocol và health check, quy trình chuyển sang đăng ký EC2 seed để kiểm thử ALB trước khi tạo ASG.


6. **Chọn EC2 seed làm target ban đầu**

   Chọn instance `sportbooking-seed`. Sau đó, nhập port `8080`. Tiếp theo, chọn **Include as pending below**.

   ![Chọn instance target nếu đăng ký thủ công](/images/5-Workshop/5.5-Policy/alb-06.png)

   <p class="image-caption"><em>Hình 6: Chọn sportbooking-seed làm target trên port 8080</em></p>
   EC2 seed được đăng ký để kiểm tra Target Group và ALB trước; sau đó ASG tiếp quản việc đăng ký các instance do Auto Scaling Group tạo ra.


7. **Rà soát target**

   Kiểm tra `sportbooking-seed`, port `8080` và trạng thái `Running` trong danh sách pending. Sau đó, chọn **Create target group**.

   ![Rà soát target](/images/5-Workshop/5.5-Policy/alb-07.png)

   <p class="image-caption"><em>Hình 7: Rà soát target</em></p>
   Sai port target là nguyên nhân phổ biến khiến health check fail.


8. **Xác nhận tạo Target Group**

   Rà soát health check `/actuator/health` và target `sportbooking-seed`. Sau đó, chọn **Create target group**.

   ![Xác nhận tạo Target Group](/images/5-Workshop/5.5-Policy/alb-08.png)

   <p class="image-caption"><em>Hình 8: Xác nhận tạo Target Group</em></p>
   Đây là bước cuối để Target Group sẵn sàng gắn vào ALB hoặc ASG.


9. **Xác nhận đăng ký target pending**

   AWS Console hiển thị hộp thoại xác nhận pending target; chọn **Continue**.

   ![Xác nhận đăng ký target pending](/images/5-Workshop/5.5-Policy/alb-09.png)

   <p class="image-caption"><em>Hình 9: Xác nhận đăng ký target pending</em></p>
   AWS cảnh báo vì target đang pending health check. Sau vài phút và app chạy ổn, trạng thái mới chuyển healthy.


10. **Kiểm tra Target Group**

   Mở Target Group vừa tạo. Sau đó, kiểm tra tab Targets và health check.

   ![Kiểm tra Target Group](/images/5-Workshop/5.5-Policy/alb-10.png)

   <p class="image-caption"><em>Hình 10: Kiểm tra Target Group</em></p>
   Trạng thái Healthy là điều kiện quan trọng trước khi đưa domain về ALB.


11. **Bắt đầu tạo Load Balancer**

   Truy cập **EC2 > Load Balancers**. Sau đó, chọn **Create load balancer**.

   ![Bắt đầu tạo Load Balancer](/images/5-Workshop/5.5-Policy/alb-11.png)

   <p class="image-caption"><em>Hình 11: Bắt đầu tạo Load Balancer</em></p>
   Load Balancer là entry point public cho website, thay vì cho người dùng truy cập trực tiếp EC2 private.


12. **Chọn Application Load Balancer**

   Chọn **Application Load Balancer**. Sau đó, chọn **Create**.

   ![Chọn Application Load Balancer](/images/5-Workshop/5.5-Policy/alb-12.png)

   <p class="image-caption"><em>Hình 12: Chọn Application Load Balancer</em></p>
   ALB phù hợp web HTTP/HTTPS, hỗ trợ listener, rule, target group và TLS termination.


13. **Nhập cấu hình cơ bản ALB**

   Nhập tên `sportbooking-alb`. Sau đó, chọn scheme **Internet-facing**. Tiếp theo, chọn IP address type IPv4.

   ![Nhập cấu hình cơ bản ALB](/images/5-Workshop/5.5-Policy/alb-13.png)

   <p class="image-caption"><em>Hình 13: Nhập cấu hình cơ bản ALB</em></p>
   Internet-facing ALB nhận traffic từ người dùng. EC2 app vẫn private phía sau.


14. **Chọn VPC và public subnets**

   Chọn VPC SportBooking. Sau đó, chọn 2 public subnet ở 2 AZ.

   ![Chọn VPC và public subnets](/images/5-Workshop/5.5-Policy/alb-14.png)

   <p class="image-caption"><em>Hình 14: Chọn VPC và public subnets</em></p>
   ALB cần public subnet để nhận request Internet và đa AZ để tăng availability.


15. **Chọn Security Group và listener**

   Chọn ALB Security Group mở 80/443. Sau đó, tạo listener HTTP:80 ban đầu. Tiếp theo, cấu hình forward đến Target Group `sportbooking-tg`.

   ![Chọn Security Group và listener](/images/5-Workshop/5.5-Policy/alb-15.png)

   <p class="image-caption"><em>Hình 15: Chọn Security Group và listener</em></p>
   ALB SG là điểm public duy nhất. Listener forward request vào app qua Target Group.


16. **Rà soát và tạo ALB**

   Kiểm tra summary: VPC, subnets, SG, listener, target group. Sau đó, chọn **Create load balancer**.

   ![Rà soát và tạo ALB](/images/5-Workshop/5.5-Policy/alb-16.png)

   <p class="image-caption"><em>Hình 16: Rà soát và tạo ALB</em></p>
   VPC, hai public subnet, Security Group, listener và Target Group đã được rà soát trước khi tạo ALB.


17. **Xác nhận ALB tạo thành công**

   Mở ALB vừa tạo. Sau đó, ghi nhận DNS name `sportbooking-alb-1198360694.us-east-1.elb.amazonaws.com`.

   ![Xác nhận ALB tạo thành công](/images/5-Workshop/5.5-Policy/alb-17.png)

   <p class="image-caption"><em>Hình 17: Xác nhận ALB tạo thành công</em></p>
   DNS name của ALB dùng để test trước khi cấu hình Route 53 Alias.


18. **Kiểm tra listener và security**

   Mở tab listener/security. Sau đó, kiểm tra HTTP listener forward đúng Target Group.

   ![Kiểm tra listener và security](/images/5-Workshop/5.5-Policy/alb-18.png)

   <p class="image-caption"><em>Hình 18: Kiểm tra listener và security</em></p>
   Nếu listener sai action, browser có thể truy cập ALB nhưng không tới được app.


19. **Kiểm thử website qua ALB**

   Mở ALB DNS hoặc domain đã trỏ. Sau đó, kiểm tra trang chủ SportBooking hiển thị.

   ![Kiểm thử website qua ALB](/images/5-Workshop/5.5-Policy/alb-19.png)

   <p class="image-caption"><em>Hình 19: Kiểm thử website qua ALB</em></p>
   Đây là bằng chứng end-to-end: user -> ALB -> Target Group -> EC2 app -> RDS/S3.



#### Auto Scaling Group

Auto Scaling Group dùng Launch Template để tạo EC2 app trong private app subnets và gắn các instance vào Target Group đã tạo ở bước trước.

| Mục | Cấu hình |
|---|---|
| Launch Template | `sportbooking-lt`, version `Default (1)`. |
| Subnet | 2 private app subnets ở 2 AZ. |
| Desired capacity | `2`. |
| Min/Max | Min `2`, Max `4`. |
| Target Group | `sportbooking-tg`. |
| Health check | EC2 và Elastic Load Balancing; grace period `300` giây. |
| Scaling policy | Target tracking theo Average CPU utilization, target `60`, warmup `300` giây. |

{{% notice info %}}
Desired capacity và minimum capacity đều được đặt bằng `2` để ASG duy trì hai instance trên hai Availability Zone. Maximum capacity `4` tạo khoảng mở rộng cho target tracking policy khi CPU trung bình vượt mục tiêu.
{{% /notice %}}

#### Quy trình thực hiện: Auto Scaling Group

Auto Scaling Group sử dụng Launch Template đã hoàn thiện, triển khai instance trên các private app subnet và tự đăng ký chúng vào Target Group để kiểm tra trạng thái healthy.

**Quy trình thực hiện:**

1. **Bắt đầu tạo Auto Scaling Group**

   Truy cập **EC2 > Auto Scaling Groups**. Sau đó, chọn **Create Auto Scaling group**.

   ![Bắt đầu tạo Auto Scaling Group](/images/5-Workshop/5.5-Policy/auto-scaling-group-01.png)

   <p class="image-caption"><em>Hình 1: Bắt đầu tạo Auto Scaling Group</em></p>
   ASG duy trì số lượng EC2 app, tự thay instance lỗi và hỗ trợ rollout bằng Instance Refresh.


2. **Chọn Launch Template**

   Nhập tên ASG `sportbooking-asg`. Sau đó, chọn Launch Template `sportbooking-lt`. Tiếp theo, chọn version `Default (1)`.

   ![Chọn Launch Template](/images/5-Workshop/5.5-Policy/auto-scaling-group-02.png)

   <p class="image-caption"><em>Hình 2: Chọn Launch Template</em></p>
   ASG tạo instance dựa trên Launch Template. Chọn sai version có thể khiến EC2 dùng User Data/AMI cũ.


3. **Xác nhận template version**

   Kiểm tra AMI, instance type, key pair, security group trong summary. Sau đó, chọn **Next**.

   ![Xác nhận template version](/images/5-Workshop/5.5-Policy/auto-scaling-group-03.png)

   <p class="image-caption"><em>Hình 3: Xác nhận template version</em></p>
   Đây là điểm kiểm tra trước khi ASG tạo instance thật trong subnet private.


4. **Chọn VPC và private app subnets**

   Chọn VPC SportBooking. Sau đó, chọn các private app subnet ở nhiều AZ.

   ![Chọn VPC và private app subnets](/images/5-Workshop/5.5-Policy/auto-scaling-group-04.png)

   <p class="image-caption"><em>Hình 4: Chọn VPC và private app subnets</em></p>
   EC2 app không cần public IP. Đặt trong private subnet giúp chỉ ALB gọi được app.


5. **Cấu hình capacity nâng cao**

   Chọn chiến lược phân bố **Balanced best effort**. Sau đó, giữ Capacity Reservation preference ở giá trị **Default**. Tiếp theo, chọn **Next**.

   ![Cấu hình capacity nâng cao](/images/5-Workshop/5.5-Policy/auto-scaling-group-05.png)

   <p class="image-caption"><em>Hình 5: Cấu hình capacity nâng cao</em></p>
   Quy trình triển khai ưu tiên cấu hình đơn giản và ổn định. Trong môi trường production, có thể sử dụng mixed instances hoặc Spot Instance để tối ưu chi phí.


6. **Gắn load balancing**

   Chọn tích hợp với load balancer. Sau đó, gắn ASG vào Target Group hiện có `sportbooking-tg`.

   ![Gắn load balancing](/images/5-Workshop/5.5-Policy/auto-scaling-group-06.png)

   <p class="image-caption"><em>Hình 6: Gắn load balancing</em></p>
   ASG cần đăng ký EC2 vào Target Group để ALB forward traffic và kiểm tra health.


7. **Cấu hình health checks**

   Bật Elastic Load Balancing health checks. Sau đó, đặt health check grace period là `300` giây. Tiếp theo, chọn **Next**.

   ![Cấu hình health checks](/images/5-Workshop/5.5-Policy/auto-scaling-group-07.png)

   <p class="image-caption"><em>Hình 7: Cấu hình health checks</em></p>
   Nếu grace period quá ngắn, ASG có thể thay instance khi app chưa kịp khởi động.


8. **Cấu hình group size và scaling**

   Đặt desired capacity là `2`. Sau đó, đặt minimum capacity `2` và maximum capacity `4`.

   ![Cấu hình group size và scaling](/images/5-Workshop/5.5-Policy/auto-scaling-group-08.png)

   <p class="image-caption"><em>Hình 8: Cấu hình group size và scaling</em></p>
   Hai instance được duy trì làm mức nền, đồng thời ASG có thể mở rộng tối đa bốn instance.


9. **Cấu hình target tracking policy**

   Chọn **Target tracking scaling policy**. Sau đó, chọn metric **Average CPU utilization**, target value `60` và instance warmup `300` giây.

   ![Cấu hình target tracking policy](/images/5-Workshop/5.5-Policy/auto-scaling-group-09.png)

   <p class="image-caption"><em>Hình 9: Cấu hình target tracking theo CPU trung bình 60%</em></p>
   Chính sách này điều chỉnh desired capacity trong giới hạn từ hai đến bốn instance dựa trên CPU trung bình của Auto Scaling Group.


10. **Hoàn tất các tùy chọn của Auto Scaling Group**

   Giữ deletion protection ở giá trị **None (default)** và không sử dụng placement group. Sau đó, rà soát các tùy chọn monitoring, warmup và instance maintenance. Tiếp theo, chọn **Next**.

   ![Hoàn tất các tùy chọn của Auto Scaling Group](/images/5-Workshop/5.5-Policy/auto-scaling-group-10.png)

   <p class="image-caption"><em>Hình 10: Rà soát tùy chọn bảo trì và chuyển sang bước tiếp theo</em></p>
   Các tùy chọn hoàn tất trước khi chuyển sang cấu hình notification và tag.


11. **Bỏ qua notification khi tạo ASG**

   Notification chưa được thêm tại bước tạo ASG. Chọn **Next**.

   ![Bỏ qua hoặc thêm notification](/images/5-Workshop/5.5-Policy/auto-scaling-group-11.png)

   <p class="image-caption"><em>Hình 11: Bỏ qua hoặc thêm notification</em></p>
   Kênh SNS được tạo riêng ở phần 5.7 và dùng làm action cho CloudWatch alarm.


12. **Thêm tag cho ASG**

   Thêm tag như `Project=SportBooking`. Sau đó, chọn **Next**.

   ![Thêm tag cho ASG](/images/5-Workshop/5.5-Policy/auto-scaling-group-12.png)

   <p class="image-caption"><em>Hình 12: Thêm tag cho ASG</em></p>
   Tag giúp lọc tài nguyên, quản lý chi phí và nhận biết instance thuộc dự án.


13. **Rà soát và tạo ASG**

   Rà soát lại Launch Template, subnets, target group, capacity và tag. Sau đó, chọn **Create Auto Scaling group**.

   ![Rà soát và tạo ASG](/images/5-Workshop/5.5-Policy/auto-scaling-group-13.png)

   <p class="image-caption"><em>Hình 13: Rà soát và tạo ASG</em></p>
   Launch Template, private subnet, Target Group, capacity và tag đã được rà soát trước khi tạo ASG.


14. **Kiểm tra ASG sau khi tạo**

   Mở ASG vừa tạo. Sau đó, kiểm tra instance, desired capacity, launch template và health.

   ![Kiểm tra ASG sau khi tạo](/images/5-Workshop/5.5-Policy/auto-scaling-group-14.png)

   <p class="image-caption"><em>Hình 14: Kiểm tra ASG sau khi tạo</em></p>
   Đây là bằng chứng compute layer đã chạy và sẵn sàng nhận traffic qua Target Group/ALB.



#### Quy trình Instance Refresh

Trong lần cập nhật phiên bản ứng dụng, build lại file jar và upload đè object release trên S3:

```powershell
.\mvnw.cmd clean package -DskipTests

aws s3 cp target\SportBooingSystem-0.0.1-SNAPSHOT.jar `
  s3://sportbooking-artifacts-3stars-us-east-1/releases/SportBooingSystem-0.0.1-SNAPSHOT.jar `
  --region us-east-1
```

Sau đó, mở **EC2 > Auto Scaling Groups > sportbooking-asg > Instance refresh > Start** và sử dụng:

- Use current Auto Scaling group configuration.
- Instance warmup: 300 giây.
- Skip matching: Off.
- Target Group được theo dõi cho đến khi các instance thay thế chuyển sang `Healthy`.

{{% notice info %}}
Trong lần gặp lỗi Launch Template version rỗng, ASG đã được cập nhật sang version hợp lệ trước khi chạy lại refresh bằng current ASG configuration.
{{% /notice %}}


