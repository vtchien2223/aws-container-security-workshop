---
title: "5.Dọn dẹp Tài nguyên"
weight: 50
---

Chúc mừng bạn đã hoàn thành workshop!

Đây là bước cuối cùng nhưng rất quan trọng: dọn dẹp tất cả các tài nguyên AWS đã tạo để tránh phát sinh chi phí không mong muốn.

{{% notice warning %}}
Hãy thực hiện các bước sau theo đúng thứ tự để tránh các lỗi phụ thuộc (dependency errors).
{{% /notice %}}

### 1. Dừng và Xóa môi trường ECS
1.  **Cập nhật Service về 0:**
    * Truy cập **Amazon ECS** -> **Clusters** -> `portfolio-cluster`.
    * Chọn tab **Services**, tick chọn `portfolio-service` và nhấn **Update**.
    * Thay đổi **Desired tasks** từ `1` về `0`. Nhấn **Update**.
    * Chờ cho đến khi task biến mất.
2.  **Xóa Service:**
    * Sau khi task đã dừng, tick chọn `portfolio-service` một lần nữa và nhấn **Delete**.
3.  **Xóa Cluster:**
    * Quay lại trang **Clusters**, chọn `portfolio-cluster` và nhấn **Delete**.
4.  **Hủy đăng ký Task Definition:**
    * Ở menu bên trái, chọn **Task Definitions**.
    * Nhấn vào `portfolio-task`. Với mỗi phiên bản (revision), hãy nhấn **Actions** -> **Deregister**.

### 2. Xóa Pipeline và các tài nguyên liên quan
1.  **Xóa Pipeline:**
    * Truy cập **AWS CodePipeline**, chọn `Security-Pipeline` và nhấn **Delete**.
2.  **Xóa CodeBuild Project:**
    * Truy cập **AWS CodeBuild** -> **Build projects**.
    * Chọn `CodeBuild-Project` và nhấn **Delete build project**.
3.  **Xóa ECR Repository:**
    * Truy cập **Amazon ECR**.
    * Nhấn vào repository `my-portfolio-app`.
    * Chọn tất cả các image bên trong và nhấn **Delete**.
    * Quay ra ngoài, chọn repository `my-portfolio-app` và nhấn **Delete**.

### 3. Xóa các tài nguyên Giám sát & Khắc phục
1.  **Xóa EventBridge Rule:**
    * Truy cập **Amazon EventBridge** -> **Rules**.
    * Xóa rule `RemediateUnrestrictedSSH`.
2.  **Xóa Config Rule:**
    * Truy cập **AWS Config** -> **Rules**.
    * Xóa rule `custom-check-unrestricted-ports`.
3.  **Tắt các dịch vụ An ninh:**
    * Truy cập **Amazon GuardDuty** -> **Settings** -> **Disable GuardDuty**.
    * Truy cập **AWS Security Hub** -> **Settings** -> **General** -> **Disable Security Hub**.
4.  **Xóa Lambda Functions:**
    * Truy cập **AWS Lambda**.
    * Xóa các hàm: `VulnerabilityCheckFunction`, `SGRuleRemediator`, `CheckUnrestrictedPortsFunction`.

### 4. Xóa Mạng và các IAM Role
1.  **Xóa VPC:**
    * Truy cập dịch vụ **VPC**.
    * Chọn **Your VPCs**, tick chọn `FirstCloudJourney-vpc`.
    * Nhấn **Actions** -> **Delete VPC**. AWS sẽ tự động xóa các tài nguyên phụ thuộc như Subnet, Route Table, Internet Gateway.
2.  **Xóa Security Group:**
    * Nếu Security Group `FirstCloudJourney-SG` chưa được xóa cùng VPC, hãy vào **Security Groups** và xóa nó thủ công.
3.  **Xóa IAM Roles:**
    * Truy cập **IAM** -> **Roles**.
    * Tìm và xóa tất cả các role bạn đã tạo trong workshop (tên thường chứa `Pipeline`, `CodeBuild`, `Lambda`, `ECS`).

Sau khi hoàn thành các bước này, tài khoản của bạn đã được dọn dẹp sạch sẽ.