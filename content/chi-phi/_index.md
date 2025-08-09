---
title: "Phụ lục: Ước tính Chi phí"
weight: 60
---

Một trong những câu hỏi quan trọng nhất khi làm việc với cloud là "Nó tốn bao nhiêu tiền?". Dưới đây là phân tích chi phí ước tính cho các dịch vụ bạn đã sử dụng.

{{% notice info %}}
**Kết luận nhanh:** Nếu tài khoản AWS của bạn vẫn còn trong thời gian Gói miễn phí (Free Tier), chi phí cho toàn bộ workshop này gần như chắc chắn sẽ là **$0**.
{{% /notice %}}

### Phân tích chi phí từng dịch vụ

Hầu hết các dịch vụ chúng ta sử dụng đều có trong **AWS Free Tier**. Dưới đây là chi phí **sau khi** hết thời gian dùng thử hoặc vượt mức miễn phí.

| Dịch vụ | Cách tính phí | Chi phí ước tính (Môi trường nhỏ) |
| :--- | :--- | :--- |
| **AWS CodePipeline** | $1/pipeline hoạt động/tháng | **~$1.00 / tháng** |
| **AWS CodeBuild** | Theo số phút build | Gói miễn phí 100 phút/tháng thường là đủ. |
| **Amazon ECR** | Theo dung lượng lưu trữ (GB/tháng) | Rất thấp, chưa đến $0.01 / tháng. |
| **AWS Lambda** | Theo lượt gọi và thời gian chạy | Gói miễn phí 1 triệu lượt gọi/tháng là rất lớn. |
| **Amazon Inspector** | Theo số lượt quét image | Dùng thử miễn phí 15 ngày. |
| **Amazon ECS on Fargate** | Theo tài nguyên vCPU và Bộ nhớ (GB) theo giờ | Đây là chi phí chính nếu bạn chạy ứng dụng 24/7. Với task 0.5 vCPU & 1GB RAM, chi phí khoảng **~$0.74/ngày** (~$22/tháng). |
| **Security Hub, GuardDuty, Config** | Theo lượng dữ liệu và số lượt kiểm tra | Dùng thử miễn phí 30 ngày. Sau đó, chi phí cho tài khoản nhỏ thường chỉ khoảng **~$5-10/tháng** cho cả ba. |
| **Amazon S3**| Theo dung lượng lưu trữ (GB/tháng) | Rất thấp, gần như bằng $0. |

### Tổng kết chi phí

* **Trong thời gian workshop:** Chi phí là **$0** nhờ các gói dùng thử và miễn phí.
* **Nếu bạn giữ lại và chạy ứng dụng 24/7 sau workshop:** Chi phí chính sẽ là từ **ECS Fargate (~$22/tháng)** và các dịch vụ khác (~$1 cho Pipeline + ~$5 cho dịch vụ an ninh). Tổng cộng vào khoảng **dưới $30/tháng**.

{{% notice warning %}}
Hãy luôn nhớ thực hiện bước **Dọn dẹp Tài nguyên** sau khi hoàn thành workshop để tránh phát sinh chi phí ngoài ý muốn!.
{{% /notice %}}