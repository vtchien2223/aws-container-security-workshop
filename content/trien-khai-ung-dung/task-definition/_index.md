---
title: "3.2 Tạo Task Definition"
weight: 32
---

Task Definition hoạt động như một "bản thiết kế" chi tiết cho ứng dụng của bạn. Nó chỉ định container image nào sẽ được sử dụng, cần bao nhiêu tài nguyên (CPU, bộ nhớ), mở port nào, v.v.

### Hướng dẫn tạo Task Definition

1.  Trong menu bên trái của ECS, chọn **Task definitions**.
2.  Nhấn **Create new task definition**.
3.  **Task definition family**: Đặt một tên họ cho các phiên bản, ví dụ `portfolio-task`.
4.  **Infrastructure requirements**:
    * **Launch type**: `AWS Fargate`.
    * **Operating system/Architecture**: `Linux/X86_64`.
5.  **Task size**:
    * **CPU**: Chọn `.5 vCPU`.
    * **Memory**: Chọn `1 GB`.
    ![](/images/cr-task.png)
6.  **Task roles**:
    * **Task execution role**: {{% notice info %}}
      Đây là quyền mà ECS cần để kéo (pull) image từ ECR và ghi logs ra CloudWatch.
      {{% /notice %}}
      Trong danh sách thả xuống, hãy tìm và chọn role có tên **`ecsTaskExecutionRole`**.

    {{% notice tip "Nếu `ecsTaskExecutionRole` không có sẵn trong danh sách?" %}}
    Đây là trường hợp thường gặp khi tài khoản mới sử dụng ECS. Hãy tạo nó một cách thủ công:
    1. Mở một tab mới và truy cập dịch vụ **IAM** -> **Roles** -> **Create role**.
    2. **Trusted entity**: Chọn `AWS service`.
    3. **Use case**: Trong danh sách thả xuống, chọn `Elastic Container Service`, sau đó ở dưới, chọn use case cụ thể là `Elastic Container Service Task`.
    4. **Add permissions**: Nhấn **Next**. Policy `AmazonECSTaskExecutionRolePolicy` cần thiết sẽ được tự động chọn sẵn cho bạn.
    5. **Name, review, and create**:
        * **Role name**: Đặt tên chính xác là `ecsTaskExecutionRole`.
        * Nhấn **Create role**.
    6. Quay lại tab tạo Task Definition, nhấn vào **biểu tượng làm mới (refresh)** bên cạnh danh sách. Bây giờ `ecsTaskExecutionRole` sẽ xuất hiện để bạn chọn.
    {{% /notice %}}

7.  **Container - 1**:
    * **Name**: `portfolio-container`.
    * **Image URI**: {{% notice warning %}}Đây là bước rất quan trọng. Hãy vào **Amazon ECR**, chọn repository của bạn, và sao chép **Image URI** của image có tag `latest`. Dán URI đó vào đây.{{% /notice %}}
    ![](/images/url-lastest.png)
    * **Port mappings**:
        * **Container port**: `8080`. (Port này phải khớp với port trong file `server.js`).
        ![](/images/container-1.png)
8.  Nhấn **Add** để thêm container.
9.  Cuối cùng, kéo xuống cuối trang và nhấn **Create**.