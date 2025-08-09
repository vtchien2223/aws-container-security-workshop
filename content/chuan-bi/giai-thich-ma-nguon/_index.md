---
title: "1.2 Giải Thích Mã nguồn"
weight: 11
---

### **Giải thích Cấu trúc Mã nguồn**
Dự án bạn vừa clone về bao gồm các tệp tin cốt lõi sau:

#### **`Dockerfile`**

{{% notice info %}}
Đây là "bản thiết kế" để đóng gói ứng dụng Node.js vào một Docker image.
{{% /notice %}}
![Dockerfile minh họa](/images/dockerfile.png)
Nó định nghĩa các bước tuần tự:
* Bắt đầu từ một image Node.js gọn nhẹ (`node:18-alpine`).
* Cài đặt các thư viện cần thiết từ `package.json`.
* Sao chép toàn bộ mã nguồn vào bên trong image.
* Khai báo rằng container sẽ lắng nghe trên port `8080`.
* Đặt lệnh khởi động mặc định là `node server.js`.

#### **`buildspec.yml`**
{{% notice info %}}
![buildspec minh họa](/images/buildspec.png)
Đây là "bản hướng dẫn" dành cho dịch vụ AWS CodeBuild. Nó ra lệnh cho CodeBuild phải thực hiện những gì.
{{% /notice %}}
File này được chia làm các giai đoạn:
* **`pre_build`**: Chứa các lệnh cần chạy trước khi build, ví dụ như đăng nhập vào Amazon ECR.
* **`build`**: Chứa các lệnh chính để build, trong trường hợp này là `docker build` và `docker tag`.
* **`post_build`**: Chứa các lệnh chạy sau khi build thành công, ví dụ như `docker push` để đẩy image lên ECR và tạo ra file `imagedefinitions.json`.
* **`artifacts`**: Khai báo các tệp tin "đầu ra" của quá trình build để các bước sau trong pipeline có thể sử dụng.