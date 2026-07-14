---
title : "Launch Template"
date : 2024-01-01
weight : 2
chapter : false
pre : " <b> 5.5.2 </b> "
---

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

   ![Tạo AMI từ EC2 seed](/images/5-Workshop/5.5-compute/launch-template-01.png)

   <p class="image-caption"><em>Hình 1: Tạo AMI từ EC2 seed</em></p>
   AMI đóng gói trạng thái máy seed để Launch Template có thể tạo instance mới nhất quán.


2. **Đặt tên AMI**

   Nhập image name theo dự án. Sau đó, thêm description nhận diện AMI của SportBooking.

   ![Đặt tên AMI](/images/5-Workshop/5.5-compute/launch-template-02.png)

   <p class="image-caption"><em>Hình 2: Đặt tên AMI</em></p>
   Tên AMI rõ ràng giúp biết image nào dùng cho SportBooking và phiên bản triển khai nào.


3. **Tạo image**

   Kiểm tra volume và tag. Sau đó, chọn **Create image**.

   ![Tạo image](/images/5-Workshop/5.5-compute/launch-template-03.png)

   <p class="image-caption"><em>Hình 3: Tạo image</em></p>
   Image này là base cho instance trong ASG. Nếu chưa tạo AMI hoặc AMI pending, Launch Template không thể dùng ổn định.


4. **Xác nhận AMI tạo thành công**

   Quay lại EC2/AMI hoặc instance. Sau đó, chờ thông báo tạo image thành công.

   ![Xác nhận AMI tạo thành công](/images/5-Workshop/5.5-compute/launch-template-04.png)

   <p class="image-caption"><em>Hình 4: Xác nhận AMI tạo thành công</em></p>
   Chỉ dùng AMI khi trạng thái available để tránh ASG launch instance lỗi.


5. **Bắt đầu tạo Launch Template**

   Truy cập **EC2 > Launch Templates**. Sau đó, chọn **Create launch template**.

   ![Bắt đầu tạo Launch Template](/images/5-Workshop/5.5-compute/launch-template-05.png)

   <p class="image-caption"><em>Hình 5: Bắt đầu tạo Launch Template</em></p>
   Launch Template chuẩn hóa AMI, instance type, IAM role, security group và User Data cho mọi EC2 trong ASG.


6. **Nhập tên Launch Template**

   Nhập launch template name `sportbooking-lt` và version description `v1`. Sau đó, bật tùy chọn hướng dẫn cấu hình cho EC2 Auto Scaling.

   ![Nhập tên Launch Template](/images/5-Workshop/5.5-compute/launch-template-06.png)

   <p class="image-caption"><em>Hình 6: Nhập tên Launch Template</em></p>
   ASG đã tham chiếu Launch Template này bằng tên `sportbooking-lt` và version mặc định `1`.


7. **Chọn AMI cho Launch Template**

   Mở mục AMI. Sau đó, chọn AMI vừa tạo từ seed instance.

   ![Chọn AMI cho Launch Template](/images/5-Workshop/5.5-compute/launch-template-07.png)

   <p class="image-caption"><em>Hình 7: Chọn AMI cho Launch Template</em></p>
   AMI đảm bảo instance mới có base runtime giống môi trường đã kiểm thử.


8. **Chọn instance type**

   Chọn instance type `t3.small`, gồm 2 vCPU và 2 GiB RAM.

   ![Chọn instance type](/images/5-Workshop/5.5-compute/launch-template-08.png)

   <p class="image-caption"><em>Hình 8: Chọn instance type</em></p>
   Compute chạy liên tục trong ASG. Chọn instance nhỏ giúp tối ưu chi phí nhưng vẫn đủ chạy Spring Boot.


9. **Chọn key pair và network security**

   Chọn **Don't include in launch template** cho key pair. Sau đó, chọn Security Group `sportbooking-app`.

   ![Chọn key pair và network security](/images/5-Workshop/5.5-compute/launch-template-09.png)

   <p class="image-caption"><em>Hình 9: Chọn key pair và network security</em></p>
   `sportbooking-app` chỉ cho phép port `8080` từ `sportbooking-alb`; SSH public không được mở.


10. **Gắn IAM instance profile**

   Mở **Advanced details**. Sau đó, chọn IAM instance profile `sportbooking-ec2-role`.

   ![Gắn IAM instance profile](/images/5-Workshop/5.5-compute/launch-template-10.png)

   <p class="image-caption"><em>Hình 10: Gắn IAM instance profile</em></p>
   Role giúp EC2 đọc S3 artifact, đọc Secrets Manager và đăng ký Session Manager khi khởi động.


11. **Nhập User Data bootstrap**

   Dán script User Data cài runtime, lấy secret, tải jar từ S3 và tạo systemd service. Sau đó, kiểm tra S3 URI, secret id và region.

   ![Nhập User Data bootstrap](/images/5-Workshop/5.5-compute/launch-template-11.png)

   <p class="image-caption"><em>Hình 11: Nhập User Data bootstrap</em></p>
   User Data biến Launch Template thành quy trình deploy tự động. Instance mới tự kéo cấu hình và artifact mà không cần thao tác tay.


12. **Tạo Launch Template**

   Rà soát cấu hình. Sau đó, chọn **Create launch template**.

   ![Tạo Launch Template](/images/5-Workshop/5.5-compute/launch-template-12.png)

   <p class="image-caption"><em>Hình 12: Tạo Launch Template</em></p>
   Template là đầu vào của ASG và Instance Refresh. Mỗi lần đổi User Data/AMI có thể tạo version mới.
