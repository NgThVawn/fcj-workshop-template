---
title : "Khảo sát CloudFront"
date : 2024-01-01
weight : 4
chapter : false
pre : " <b> 5.6.4 </b> "
---

#### Kết quả khảo sát CloudFront

Thử cấu hình CloudFront với ALB làm origin để đánh giá phương án đặt CDN trước website. AWS Console trả về lỗi yêu cầu xác minh tài khoản trước khi tạo tài nguyên CloudFront mới, vì vậy distribution không được tạo và CloudFront không tham gia luồng truy cập đã kiểm thử.

{{% notice info %}}
DNS `sportbooking-alb-2038054256.us-east-1.elb.amazonaws.com` trong ảnh CloudFront thuộc cấu hình ALB thử nghiệm trước đó. Cấu hình hoàn thiện sử dụng `sportbooking-alb-1198360694.us-east-1.elb.amazonaws.com` và Route 53 trỏ trực tiếp đến ALB này.
{{% /notice %}}

**Quy trình thực hiện:**

1. **Ghi nhận lỗi xác minh tài khoản khi tạo CloudFront**

   Mở màn hình **Create distribution** và chọn ALB làm origin thử nghiệm. Sau đó, rà soát cache settings và security protections. Tiếp theo, ghi nhận thông báo tài khoản phải được xác minh trước khi thêm tài nguyên CloudFront.

   ![Ghi nhận lỗi xác minh tài khoản khi tạo CloudFront](/images/5-Workshop/5.6-domain-https/cloudfront-01.png)

   <p class="image-caption"><em>Hình 1: CloudFront không được tạo do tài khoản chưa hoàn tất xác minh</em></p>
   Việc khảo sát dừng tại màn hình này. Kiến trúc mục tiêu vẫn giữ CloudFront và WAF làm hướng mở rộng, không ghi nhận hai dịch vụ này là thành phần đã triển khai.
