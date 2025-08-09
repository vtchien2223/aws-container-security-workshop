---
title: "3.Triển khai ứng dụng"
weight: 30
---

Chúc mừng bạn đã xây dựng thành công một pipeline CI/CD an toàn! Giờ chúng ta đã có một container image "sạch", đã được quét và kiểm tra, bước tiếp theo là triển khai nó lên một môi trường để người dùng có thể truy cập.

**Mục tiêu của bước 3:**
1.  **Thiết lập Môi trường:** Tạo một "ngôi nhà" cho ứng dụng bằng Amazon ECS Cluster.
2.  **Tạo "Bản thiết kế":** Định nghĩa cách container sẽ chạy bằng ECS Task Definition.
3.  **Khởi chạy Ứng dụng:** Ra lệnh cho ECS khởi chạy và duy trì ứng dụng bằng ECS Service.

Trong bước này, bạn sẽ lần đầu tiên nhìn thấy ứng dụng của mình hoạt động trên internet!