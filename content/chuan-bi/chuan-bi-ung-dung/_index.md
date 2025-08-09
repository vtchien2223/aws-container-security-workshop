---
title: "1.1 Chuẩn bị Mã nguồn"
weight: 11
---

Trong phần này, chúng ta sẽ chuẩn bị mã nguồn ban đầu cho workshop. Thay vì viết code từ đầu, bạn sẽ sao chép (clone) một dự án mẫu đã được chuẩn bị sẵn, sau đó đẩy nó lên repository GitHub của chính bạn để bắt đầu làm việc.

### Bước 1: Tạo một Repository mới trên GitHub
{{% notice info %}}
Mỗi người tham gia cần có một repository riêng để CodePipeline có thể kết nối và theo dõi các thay đổi.
{{% /notice %}}

1.  Truy cập vào tài khoản GitHub của bạn.
2.  Nhấn vào dấu **`+`** ở góc trên bên phải, sau đó chọn **`New repository`**.
3.  **Repository name**: Đặt một tên bất kỳ, ví dụ: `my-container-security-app`.
4.  Để repository ở chế độ **Public**.
5.  Nhấn **Create repository**.

### Bước 2: Clone Repository mẫu về máy tính
{{% notice info %}}
Chúng ta sẽ sao chép toàn bộ mã nguồn và các file cấu hình đã được chuẩn bị sẵn từ một repository mẫu.
{{% /notice %}}

1.  Mở một cửa sổ terminal hoặc PowerShell trên máy tính của bạn.
2.  Chạy lệnh `git clone` sau để tải mã nguồn mẫu về:
    ```bash
    git clone https://github.com/vtchien2223/demo-app.git
    ```
3.  Sau khi lệnh chạy xong, một thư mục mới có tên `demo-app` sẽ xuất hiện. Hãy di chuyển vào thư mục đó:
    ```bash
    cd demo-app
    ```

### Bước 3: Đẩy mã nguồn lên Repository mới của bạn
{{% notice info %}}
Bây giờ, chúng ta sẽ kết nối thư mục mã nguồn ở máy bạn với repository mới bạn vừa tạo trên GitHub và đẩy code lên đó.
{{% /notice %}}

1.  Trong terminal, chạy lệnh sau để thay đổi "địa chỉ" remote từ repo mẫu thành repo của bạn. 
    ```bash
    git remote set-url origin <URL_repo_moi_cua_ban>
    ```
    *Ví dụ:*
    ```bash
    git remote set-url origin https://github.com/abc/my-container-security-app.git
    ```
2.  Cuối cùng, đẩy toàn bộ mã nguồn lên repository mới của bạn:
    ```bash
    git push -u origin main
    ```

Bây giờ, bạn đã có một bản sao của dự án trên repository GitHub của riêng mình và sẵn sàng để bắt đầu workshop.