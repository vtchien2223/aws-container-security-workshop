---
title: "2.Xây dựng Pipeline CI & Quét an ninh"
weight: 20
---

Trong bước này, chúng ta sẽ bắt đầu làm việc trên AWS Console để xây dựng phần cốt lõi của hệ thống: một quy trình Tích hợp Liên tục (CI) có khả năng tự động quét lỗ hổng bảo mật.

**Mục tiêu của phần 2:**
1.  **Tạo kho chứa image (ECR):** Một nơi an toàn để lưu trữ các container image.
2.  **Xây dựng Pipeline (CodePipeline & CodeBuild):** Tự động hóa việc lấy code, build image và đẩy lên ECR.
3.  **Tạo Cổng An ninh (Lambda):** Tự động kiểm tra kết quả quét và dừng pipeline nếu phát hiện rủi ro.

Khi hoàn thành bước này, bạn sẽ có một pipeline CI/CD có khả năng tự bảo vệ, ngăn chặn các image không an toàn đi tiếp đến giai đoạn triển khai.