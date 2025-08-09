---
title: "1.3 Chuẩn bị Mạng (VPC & Security Group)"
weight: 12
---

Trước khi triển khai ứng dụng, chúng ta cần chuẩn bị "mảnh đất" và "hàng rào" cho nó. Trong AWS, đó chính là Virtual Private Cloud (VPC) và Security Group.

{{% notice info %}}
Việc tạo trước các tài nguyên mạng này giúp chúng ta dễ dàng lựa chọn chúng ở các bước sau và đảm bảo ứng dụng được triển khai trong một môi trường mạng an toàn, được kiểm soát.
{{% /notice %}}

### A. Tạo VPC (FirstCloudJourney-vpc)

Nếu bạn đã có một VPC mặc định (default VPC), bạn có thể sử dụng nó. Tuy nhiên, việc tạo một VPC mới cho workshop sẽ giúp bạn quản lý tài nguyên một cách tách biệt.

1.  Truy cập dịch vụ **VPC** trong AWS Console.
2.  Ở menu bên trái, chọn **Your VPCs** và nhấn **Create VPC**.
3.  Chọn tùy chọn **"VPC and more"**.
4.  **Name tag auto-generation**: Đặt một tên, ví dụ: `FirstCloudJourney-vpc`.
5.  **Number of Availability Zones (AZs)**: Chọn `2`.
6.  **Number of public subnets**: `2`.
7.  **Number of private subnets**: `2`.
8.  **NAT gateways**: Chọn `none`.
9.  Nhấn **Create VPC**. Quá trình này có thể mất vài phút.

### B. Tạo Security Group (FirstCloudJourney-SG)

Security Group hoạt động như một "tường lửa ảo" cho các tài nguyên của bạn, kiểm soát lưu lượng truy cập vào và ra.

1.  Vẫn trong dịch vụ **VPC**, ở menu bên trái, dưới mục "Security", chọn **Security Groups**.
2.  Nhấn **Create security group**.
3.  **Basic details**:
    * **Security group name**: `FirstCloudJourney-SG`.
    * **Description**: `Security group for workshop application`.
    * **VPC**: Chọn VPC bạn vừa tạo (`FirstCloudJourney-vpc`).
4.  **Inbound rules (Luật truy cập vào)**:
    * Nhấn **Add rule**.
    * **Type**: `Custom TCP`.
    * **Port range**: `8080`. (Đây là port mà ứng dụng Node.js của chúng ta sẽ chạy).
    * **Source**: `Anywhere-IPv4` (`0.0.0.0/0`).
    * Nhấn **Add rule**.
    * **Type**: `SSH`.
    * **Port range**: `22`. (để phục vụ việc kiểm tra sau này).
    * **Source**: `Anywhere-IPv4` (`0.0.0.0/0`).
    {{% notice tip %}}
    Chúng ta cố tình tạo ra một rule "không an toàn" này ngay từ đầu. Ở bước 4, chúng ta sẽ xây dựng một cơ chế tự động phát hiện và xóa rule này đi.
    {{% /notice %}}
    {{% notice warning %}}
    Việc mở port ra `Anywhere` giúp chúng ta dễ dàng truy cập ứng dụng demo. Trong môi trường thực tế, bạn nên giới hạn Source IP ở những địa chỉ tin cậy.
    {{% /notice %}}
5.  Giữ nguyên phần "Outbound rules" mặc định (cho phép tất cả traffic đi ra).
6.  Nhấn **Create security group**.

**Kết quả:** Bạn đã có sẵn một môi trường mạng hoàn chỉnh (`FirstCloudJourney-vpc`) và một "hàng rào" (`FirstCloudJourney-SG`) đã mở sẵn "cánh cổng" ở port `8080`, sẵn sàng cho việc triển khai ứng dụng ở bước 3.