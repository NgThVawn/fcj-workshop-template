---
title : "Xử lý sự cố"
date : 2024-01-01
weight : 4
chapter : false
pre : " <b> 5.7.4 </b> "
---

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
