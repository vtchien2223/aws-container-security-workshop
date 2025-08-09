---
title: "3.3 Tạo ECS Service & Chạy ứng dụng"
weight: 33
---

Đây là bước cuối cùng để khởi chạy ứng dụng. ECS Service sẽ nhận "bản thiết kế" (Task Definition) và ra lệnh cho "khu đất" (Cluster) thi công, đồng thời duy trì hoạt động của ứng dụng.

### Hướng dẫn tạo Service

1.  Truy cập **Amazon ECS** -> **Clusters** -> `portfolio-cluster`.
2.  Chọn tab **Services** và nhấn **Create**.
![](/images/gd-cr-cluster.png)
3.  **Deployment configuration**:
    * **Task definition family**: `portfolio-task`.
    * **Revision**: `latest`.
    * **Service name**: `portfolio-service`.
    * **Desired tasks**: Nhập số `1`. (Luôn duy trì 1 container của ứng dụng đang chạy).
    ![](/images/service-detail.png)
4.  **Networking**: {{% notice warning %}}Đây là phần chúng ta sẽ sử dụng các tài nguyên mạng đã tạo ở bước 1.{{% /notice %}}
    * **VPC**: Từ danh sách thả xuống, hãy tìm và chọn VPC bạn đã tạo: **`FirstCloudJourney-vpc`**.
    * **Subnets**: Trong danh sách, hãy chọn **2 public subnets** thuộc về `FirstCloudJourney-vpc`.
    * **Security Group**:
        * Chọn **Use an existing security group**.
        * Trong danh sách, tìm và chọn security group bạn đã tạo: **`FirstCloudJourney-SG`**.
    * **Public IP**: Chọn **`Enabled`**.
5.  Kéo xuống cuối trang và nhấn **Create**.
![](/images/networking-service.png)
### Kiểm tra ứng dụng

Sau vài phút, task của bạn sẽ chuyển sang trạng thái `RUNNING`.
1.  Trong tab **Tasks** của service, nhấn vào **Task ID** của task đang chạy.
![](/images/taskservices-running.png)
2.  Trong trang chi tiết của task, tìm đến mục **Network**.
3.  Sao chép địa chỉ **Public IP**.
![](/images/taskrunning-detail.png)
4.  Mở trình duyệt và truy cập: `http://<Public-IP-bạn-vừa-sao-chép>:8080`

**Chúc mừng! Trang web của bạn đã hoạt động trên Internet! Giờ đây bạn có thể truy cập ứng dụng từ bất kì đâu**