---
title: "2.2 Xây dựng Pipeline"
weight: 22
---

Trong phần này, chúng ta sẽ tạo ra "bộ não" của hệ thống: một **AWS CodePipeline** hoàn chỉnh. Chúng ta sẽ xây dựng thủ công thay vì dùng template để kiểm soát mọi chi tiết. Pipeline này sẽ tự động lấy mã nguồn, build image và đẩy lên ECR.

### A. Tạo Pipeline và Stage "Source"

1.  Truy cập dịch vụ **AWS CodePipeline** và nhấn **Create pipeline**.
2.  **Pipeline settings**:
    * **Pipeline name**: `Security-Pipeline`.
    * **Service role**: Chọn `New service role`.
    {{% notice info %}}
    Lựa chọn này sẽ tự động tạo một IAM Role cho pipeline (ví dụ: `AWSCodePipelineServiceRole-...-Security-Pipeline`). Chúng ta sẽ cần cấp thêm quyền cho role này ở các bước sau.
    {{% /notice %}}
3.  **Source stage**:
    * **Source provider**: Chọn `GitHub (Version 2)`.
    * **Connection**: Kết nối đến tài khoản GitHub của bạn.

    #### Hướng dẫn kết nối tới GitHub
    {{% notice tip %}}
    Nếu đây là lần đầu bạn kết nối, hãy làm theo các bước sau.
    {{% /notice %}}
    1. Nhấn vào nút **Connect to GitHub**.
    2. Một cửa sổ pop-up sẽ hiện ra. Đặt tên cho kết nối (ví dụ: `my-github-connection`) và nhấn **Connect to GitHub**.
    3. Bạn sẽ được chuyển đến trang GitHub để xác thực. Hãy đăng nhập và cấp quyền (Authorize) cho **AWS Connector for GitHub**.
    4. Tiếp theo, bạn sẽ cài đặt ứng dụng AWS. Ở phần **Repository access**, bạn có thể chọn `All repositories` hoặc `Only select repositories` (và chỉ chọn repo của workshop).
    5. Sau khi cài đặt, cửa sổ pop-up sẽ tự đóng và kết nối của bạn sẽ được chọn trong CodePipeline.
    
    * **Repository name**: Chọn repository của bạn (`my-container-security-app`).
    * **Branch**: `main`.
    ![](/images/source-stage.png)
4.  Nhấn **Next**.

### B. Tạo Stage "Build"

1.  Trên màn hình "Add build stage".
3.  **Cấu hình build:**
    * **Action provider**: `Other build provider` - > `AWS CodeBuild`.
    * **Input artifacts**: Chọn `SourceOutput` (đây là sản phẩm từ stage Source).
4.  **Tạo CodeBuild Project:**
    * Nhấn vào nút **Create project**. Một cửa sổ mới sẽ hiện ra.
    * **Project name**: `CodeBuild-Project`.
    * **Environment**: Giữ nguyên các lựa chọn mặc định (Managed image, Amazon Linux, Standard...).
    * **Privileged**: Mở rộng Additional configuration -> tick chọn Privileged
    ![](/images/privileged.png)
    {{% notice warning %}}**BẮT BUỘC** phải tick chọn ô này để CodeBuild có quyền build Docker image.{{% /notice %}}
    * **Service role**: `New service role`.
    * **Environment variables (Quan trọng)**: Cuộn xuống và thêm chính xác 3 biến sau. Đây là bước cực kỳ quan trọng để `buildspec.yml` có thể hoạt động.
        * **Name**: `IMAGE_REPO_NAME`, **Value**: `my-portfolio-app` (tên ECR repo bạn đã tạo ở bước 2.1).
        * **Name**: `AWS_ACCOUNT_ID`, **Value**: [ID tài khoản AWS của bạn, ví dụ: 12345678].
        * **Name**: `AWS_DEFAULT_REGION`, **Value**: [Region bạn đang sử dụng, ví dụ: ap-southeast-1].
        ![](/images/enviroment.png)
    * **Buildspecs**: Chọn Use a buildspec file.
    * Nhấn **Continue to CodePipeline**.
5.  Nhấn **Next**.
6.  Nhấn **Skip Test stage and Deploy stage**.
7.  Nhấn **Create Pipeline**.

### C. Cấp quyền IAM (Bước bắt buộc)

{{% notice warning %}}
Pipeline vừa tạo sẽ thất bại ở lần chạy đầu tiên vì các IAM role mới được tạo tự động không có đủ quyền. Chúng ta sẽ cấp quyền cho chúng ngay bây giờ để pipeline hoạt động đúng.
{{% /notice %}}

1.  Truy cập dịch vụ **IAM** -> **Roles**.
2.  **Cấp quyền cho CodeBuild Role:**
    * Tìm role có tên chứa `CodeBuild-Project`.
    * Đính kèm policy: **`AmazonEC2ContainerRegistryPowerUser`**. Policy này cho phép CodeBuild đăng nhập và đẩy image lên ECR.
3.  **Cấp quyền cho CodePipeline Role:**
    * Tìm role có tên chứa `Security-Pipeline`.
    * Đính kèm policy: **`AWSCodeBuildDeveloperAccess`**. Policy này cho phép CodePipeline ra lệnh khởi động các job trong CodeBuild.

Sau khi cấp quyền, bạn có thể vào lại pipeline và nhấn "Release change" để bắt đầu một lượt build thành công.