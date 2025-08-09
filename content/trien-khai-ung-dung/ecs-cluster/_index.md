---
title: "3.1 Tạo ECS Cluster"
weight: 31
---

ECS Cluster là một nhóm logic các tài nguyên, là "khu đất" nơi các ứng dụng container của bạn sẽ được vận hành.

{{% notice info %}}
Chúng ta sẽ sử dụng **AWS Fargate**, một công nghệ serverless cho container. Điều này có nghĩa là bạn không cần phải quản lý bất kỳ máy chủ (server) nào.
{{% /notice %}}

### Điều kiện tiên quyết: Service-Linked Role
{{% notice tip %}}
Lần đầu tiên sử dụng ECS, bạn có thể cần tạo một Service-Linked Role. Đây là vai trò đặc biệt cho phép ECS quản lý các tài nguyên thay mặt bạn. Nếu bạn gặp lỗi ở bước tạo cluster, hãy quay lại và tạo role này trong IAM với use case là "ECS - Service Linked Role".
{{% /notice %}}

### Hướng dẫn tạo Cluster

1.  Truy cập dịch vụ **Amazon ECS** trong AWS Console.
2.  Ở menu bên trái, chọn **Clusters**.
3.  Nhấn nút **Create cluster**.
4.  **Cluster name**: Đặt một cái tên, ví dụ: `portfolio-cluster`.
5.  **Infrastructure**: Hãy chắc chắn rằng **AWS Fargate (serverless)** đã được chọn.
6.  Nhấn **Create**.
![](/images/cr-ecs.png)
Quá trình này rất nhanh. Sau khi cluster được tạo thành công, bạn đã có một "ngôi nhà" sẵn sàng cho ứng dụng.