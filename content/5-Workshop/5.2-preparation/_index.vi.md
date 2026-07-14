---
title : "Chuẩn bị"
date : 2024-01-01
weight : 2
chapter : false
pre : " <b> 5.2. </b> "
aliases:
  - /vi/5-workshop/5.2-prerequiste/
---

#### Nội dung đã chuẩn bị

Trước khi khởi tạo tài nguyên AWS, kiểm tra source code, công cụ triển khai, quyền truy cập và các thông số được sử dụng xuyên suốt dự án. Ở giai đoạn này chỉ xác nhận ứng dụng có thể build; artifact chưa được upload vì S3 bucket chưa được tạo.

#### Điều kiện cần

| Hạng mục | Yêu cầu |
|---|---|
| AWS Region | Sử dụng thống nhất `us-east-1`. |
| AWS CLI | Credential được cấu hình với quyền thao tác trên VPC, EC2, S3, IAM, Secrets Manager, RDS, ALB, ACM, Route 53, CloudWatch và SNS. |
| Source code | Project `SportBookingSystem` build được bằng Maven Wrapper. |
| Domain | Có quyền quản lý domain `younglilliu.id.vn` để đổi name server và tạo record DNS. |
| Công cụ kiểm thử | PowerShell, trình duyệt, `curl.exe`, `nslookup`; Session Manager đã được sử dụng sau khi EC2 khởi tạo. |

{{% notice info %}}
Toàn bộ tài nguyên trong phạm vi báo cáo được triển khai cùng Region `us-east-1`. Việc thống nhất Region giúp hạn chế lỗi sai endpoint, ARN và certificate khi cấu hình HTTPS cho ALB.
{{% /notice %}}

#### Kiểm tra khả năng build ứng dụng

Mở Windows PowerShell tại thư mục source code và chạy:

```powershell
cd D:\22DTHD5\J2EE\SportBookingSystem
.\mvnw.cmd clean package -DskipTests
dir target\*.jar
```

Kết quả đã ghi nhận:

- Lệnh Maven kết thúc thành công.
- Thư mục `target` có file `SportBooingSystem-0.0.1-SNAPSHOT.jar`.
- Không có lỗi biên dịch hoặc lỗi thiếu dependency.

{{% notice tip %}}
Lần build này được dùng để kiểm tra source code trước khi tạo hạ tầng. Bản jar phát hành được build lại và upload ngay sau khi artifact bucket được tạo ở phần `5.4.1`; thao tác upload không phụ thuộc IAM Role của EC2.
{{% /notice %}}

#### Thông số đã sử dụng khi triển khai

| Thông số | Giá trị |
|---|---|
| Domain ứng dụng | `https://sport.younglilliu.id.vn` |
| Artifact bucket | `sportbooking-artifacts-3stars-us-east-1` |
| Artifact object | `releases/SportBooingSystem-0.0.1-SNAPSHOT.jar` |
| Upload bucket | `sportbooking-uploads-3stars-us-east-1` |
| Secret ID | `sportbooking/prod/app-env` |
| Database name | `sport_booking` |
| App port | `8080` |
| Health check path | `/actuator/health` |

{{% notice warning %}}
Access key, DB password, OAuth client secret và token không được lưu trực tiếp trong source code, file jar hoặc ảnh báo cáo. Các giá trị nhạy cảm được đưa vào AWS Secrets Manager tại phần `5.4.3`.
{{% /notice %}}


