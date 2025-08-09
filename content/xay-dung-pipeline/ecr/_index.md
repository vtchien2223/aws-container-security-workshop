---
title: "2.1 Tạo ECR Repository"
weight: 21
---

Bước đầu tiên trên AWS là tạo một kho chứa riêng tư và an toàn để lưu trữ các Docker image của chúng ta. Dịch vụ này được gọi là Amazon Elastic Container Registry (ECR).

{{% notice info %}}
Chúng ta sẽ tích hợp Amazon Inspector ngay tại bước này để mọi image mới được đẩy lên ECR sẽ được tự động quét lỗ hổng.
{{% /notice %}}

### Hướng dẫn chi tiết

1.  Truy cập **AWS Management Console** và tìm dịch vụ **Amazon ECR**.
2.  Nhấn vào nút **Create repository**.
3.  Cấu hình các thông số sau:
![](/images/ecr-create.png)
    * **Repository name**: Đặt một cái tên, ví dụ: `demo-app`.
    * **Tag immutability**: Chọn `Immutable`. Việc này giúp ngăn chặn việc ghi đè lên các image tag đã có, là một thông lệ bảo mật tốt.
    * **Scan on push**: **Bật (Enable)** tùy chọn này. Đây chính là bước tích hợp **Amazon Inspector**.
4.  Nhấn **Create repository**.
![](/images/ecr-done.png)
**Kết quả:** Bạn đã có một kho chứa an toàn, sẵn sàng nhận image từ pipeline và tự động quét lỗ hổng cho chúng.