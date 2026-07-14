---
title : "IAM Role"
date : 2024-01-01
weight : 4
chapter : false
pre : " <b> 5.4.4 </b> "
aliases:
  - /vi/5-workshop/5.4-s3-onprem/5.4.4-iam-role/
---

#### Phạm vi triển khai

Tạo IAM Role để EC2 app có thể đọc artifact từ S3, đọc/ghi file upload, lấy cấu hình từ Secrets Manager và truy cập bằng Systems Manager Session Manager. Cách này loại bỏ nhu cầu lưu access key trực tiếp trên EC2.

#### Cấu hình role

| Mục | Giá trị cấu hình |
|---|---|
| Trusted entity | AWS service: EC2 |
| Role name | `sportbooking-ec2-role` |
| Managed policy | `AmazonSSMManagedInstanceCore` |
| Inline policy | Quyền đọc artifact S3, đọc/ghi upload S3 và đọc secret |
| Gắn vào EC2 | Thông qua IAM instance profile trong Launch Template |

#### Inline policy sử dụng cho ứng dụng

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "ReadArtifactJarFromS3",
      "Effect": "Allow",
      "Action": ["s3:GetObject"],
      "Resource": ["arn:aws:s3:::sportbooking-artifacts-3stars-us-east-1/releases/*"]
    },
    {
      "Sid": "AccessUploadFilesInS3",
      "Effect": "Allow",
      "Action": ["s3:GetObject", "s3:PutObject", "s3:DeleteObject"],
      "Resource": ["arn:aws:s3:::sportbooking-uploads-3stars-us-east-1/uploads/*"]
    },
    {
      "Sid": "ReadAppEnvSecret",
      "Effect": "Allow",
      "Action": ["secretsmanager:GetSecretValue"],
      "Resource": ["arn:aws:secretsmanager:us-east-1:<account-id>:secret:sportbooking/prod/app-env*"]
    }
  ]
}
```

{{% notice info %}}
IAM Role được tạo sau S3 bucket và secret. Vì vậy, policy tùy chỉnh đã được đối chiếu trực tiếp với ARN của artifact bucket, upload bucket và `sportbooking/prod/app-env` trước khi gắn vào role.
{{% /notice %}}

#### Quy trình thực hiện

Quy trình IAM bắt đầu bằng trust relationship dành cho EC2, sau đó gắn policy SSM và policy giới hạn quyền truy cập S3 cùng Secrets Manager.

**Quy trình thực hiện:**

1. **Bắt đầu tạo IAM Role**

   Truy cập **IAM > Roles**. Sau đó, chọn **Create role**.

   ![Bắt đầu tạo IAM Role](/images/5-Workshop/5.4-data-access/iam-role-01.png)

   <p class="image-caption"><em>Hình 1: Bắt đầu tạo IAM Role</em></p>
   EC2 cần quyền đọc S3, đọc Secrets Manager và dùng Session Manager mà không lưu access key trong máy.


2. **Chọn trusted entity là EC2**

   Chọn **AWS service**. Sau đó, chọn use case **EC2**. Tiếp theo, chọn **Next**.

   ![Chọn trusted entity là EC2](/images/5-Workshop/5.4-data-access/iam-role-02.png)

   <p class="image-caption"><em>Hình 2: Chọn trusted entity là EC2</em></p>
   Trusted entity quyết định dịch vụ nào được assume role. Chọn EC2 để instance nhận temporary credentials qua instance profile.


3. **Tìm policy cần gắn**

   Tìm policy SSM tại bước **Add permissions**. Sau đó, chọn **AmazonSSMManagedInstanceCore**.

   ![Tìm policy cần gắn](/images/5-Workshop/5.4-data-access/iam-role-03.png)

   <p class="image-caption"><em>Hình 3: Tìm policy cần gắn</em></p>
   Policy SSM cho phép quản trị EC2 private bằng Session Manager, tránh mở SSH public.


4. **Tạo policy quyền ứng dụng**

   Mở phần tạo policy hoặc thêm inline policy. Sau đó, nhập JSON quyền đọc artifact S3, đọc/ghi upload S3 và đọc secret.

   ![Tạo policy quyền ứng dụng](/images/5-Workshop/5.4-data-access/iam-role-04.png)

   <p class="image-caption"><em>Hình 4: Tạo policy quyền ứng dụng</em></p>
   Policy chỉ cấp quyền trên bucket, prefix và secret của SportBooking; không cấp `s3:*` hoặc `secretsmanager:*` trên toàn tài khoản.


5. **Đặt tên và tạo policy**

   Đặt policy name `sportbooking-ec2-app-policy`. Sau đó, rà soát danh sách quyền. Tiếp theo, chọn **Create policy**.

   ![Đặt tên và tạo policy](/images/5-Workshop/5.4-data-access/iam-role-05.png)

   <p class="image-caption"><em>Hình 5: Đặt tên và tạo policy</em></p>
   Tên rõ ràng giúp khi audit có thể biết policy này phục vụ EC2 app SportBooking.


6. **Quay lại bước chọn permissions**

   Làm mới danh sách policy sau khi tạo policy mới. Sau đó, tìm policy `sportbooking-ec2-app-policy`.

   ![Quay lại bước chọn permissions](/images/5-Workshop/5.4-data-access/iam-role-06.png)

   <p class="image-caption"><em>Hình 6: Quay lại bước chọn permissions</em></p>
   Sau khi danh sách được làm mới, policy vừa tạo đã xuất hiện và được chọn cho role.


7. **Gắn policy vào role**

   Chọn policy tùy chỉnh. Sau đó, xác nhận có cả policy SSM và policy ứng dụng. Tiếp theo, chọn **Next**.

   ![Gắn policy vào role](/images/5-Workshop/5.4-data-access/iam-role-07.png)

   <p class="image-caption"><em>Hình 7: Gắn policy vào role</em></p>
   Role cần đồng thời quyền vận hành qua SSM và quyền runtime của ứng dụng.


8. **Đặt tên role**

   Nhập role name `sportbooking-ec2-role`. Sau đó, thêm mô tả vai trò của role.

   ![Đặt tên role](/images/5-Workshop/5.4-data-access/iam-role-08.png)

   <p class="image-caption"><em>Hình 8: Đặt tên role</em></p>
   Role này được chọn làm IAM instance profile trong Launch Template và được tách biệt với các role của dịch vụ khác.


9. **Rà soát và tạo role**

   Kiểm tra trust policy là EC2. Sau đó, kiểm tra danh sách permission policy. Tiếp theo, chọn **Create role**.

   ![Rà soát và tạo role](/images/5-Workshop/5.4-data-access/iam-role-09.png)

   <p class="image-caption"><em>Hình 9: Rà soát và tạo role</em></p>
   Lần rà soát cuối xác nhận trusted entity là EC2 và role có đủ hai loại quyền: quyền vận hành và quyền ứng dụng.


10. **Xác nhận role đã sẵn sàng**

   Tìm role `sportbooking-ec2-role` trong danh sách. Sau đó, mở role và kiểm tra permissions.

   ![Xác nhận role đã sẵn sàng](/images/5-Workshop/5.4-data-access/iam-role-10.png)

   <p class="image-caption"><em>Hình 10: Xác nhận role đã sẵn sàng</em></p>
   Role này đã được gắn vào Launch Template để mọi EC2 trong ASG nhận quyền cần thiết khi khởi động.




