---
title: "Giới thiệu Workshop"
---

## Tổng quan Xây dựng Container Security Pipeline trên AWS
{{% notice info %}}
Trong trang này, chúng ta sẽ tìm hiểu về mục tiêu của workshop **Xây dựng Container Security Pipeline trên AWS**, cách thức hoạt động và các thành phần quan trọng sẽ được sử dụng.
{{% /notice %}}

Workshop này sẽ cung cấp một hướng dẫn thực hành chi tiết để xây dựng một quy trình an ninh tự động cho ứng dụng container. Bắt đầu từ mã nguồn, chúng ta sẽ tự động hóa việc build, quét lỗ hổng, triển khai và cuối cùng là giám sát, phản ứng với các mối đe dọa trong môi trường đang chạy.

---
## Các thành phần chính

### **AWS CodePipeline & CodeBuild (CI/CD)**
{{% notice info %}}
**CodePipeline** là dịch vụ điều phối quy trình CI/CD, trong khi **CodeBuild** là dịch vụ build được quản lý hoàn toàn, giúp biên dịch mã nguồn, chạy kiểm thử và tạo ra các gói phần mềm sẵn sàng để triển khai.
{{% /notice %}}
* **CodePipeline** hoạt động như một "đạo diễn", kết nối các giai đoạn khác nhau (Source, Build, Deploy) thành một luồng công việc liền mạch.
* **CodeBuild** đóng vai trò là "công nhân", thực thi các lệnh được định nghĩa trong file `buildspec.yml` để tạo ra các sản phẩm (artifacts), trong trường hợp này là một Docker image.

{{% notice tip %}}
Luôn sử dụng các biến môi trường (Environment variables) trong cấu hình CodeBuild để truyền các thông tin cấu hình thay vì "hardcode" trực tiếp vào file `buildspec.yml`.
{{% /notice %}}

### **Amazon ECR & Inspector (Lưu trữ & Quét an ninh)**
{{% notice info %}}
**Amazon Elastic Container Registry (ECR)** là một dịch vụ registry Docker image được quản lý hoàn toàn, giúp bạn dễ dàng lưu trữ, quản lý và triển khai container image. **Amazon Inspector** là một dịch vụ quản lý lỗ hổng, có khả năng tự động quét các tài nguyên AWS để tìm các lỗ hổng phần mềm và các rủi ro ngoài ý muốn.
{{% /notice %}}
* **ECR** cung cấp một kho chứa riêng tư và an toàn cho các Docker image của bạn.
* **Inspector** tích hợp với ECR để thực hiện "Quét khi đẩy lên" (Scan on push), tự động phân tích các lớp của image để tìm các lỗ hổng đã biết trong hệ điều hành và các thư viện phần mềm (CVEs).

{{% notice warning %}}
**Cảnh báo:** Một image có thể an toàn hôm nay nhưng lại phát hiện lỗ hổng vào ngày mai khi có một CVE mới được công bố. Vì vậy, việc thiết lập cơ chế quét lại định kỳ cho các image đang được sử dụng là cực kỳ quan trọng.
{{% /notice %}}

### **Amazon ECS & Fargate (Môi trường Vận hành)**
{{% notice info %}}
**Amazon Elastic Container Service (ECS)** là một dịch vụ điều phối container hiệu suất cao, hỗ trợ các container Docker và cho phép bạn dễ dàng chạy và co giãn các ứng dụng được container hóa trên AWS. **AWS Fargate** là một công nghệ serverless cho container, cho phép bạn chạy chúng mà không cần quản lý các máy chủ.
{{% /notice %}}
* **Cluster**: Là một nhóm logic các tài nguyên nơi các task của bạn sẽ chạy.
* **Task Definition**: Hoạt động như một "bản thiết kế" cho ứng dụng của bạn, định nghĩa image nào cần dùng, tài nguyên CPU/bộ nhớ, port mapping...
* **Service**: Chịu trách nhiệm duy trì số lượng mong muốn các bản sao (instance) của một Task Definition đang chạy trong cluster.

### **Security Hub, GuardDuty & Config (Giám sát & Tuân thủ)**
{{% notice info %}}
Đây là bộ ba dịch vụ cốt lõi giúp bạn giám sát tình trạng an ninh và tuân thủ của tài khoản AWS.
{{% /notice %}}
* **Security Hub**: Hoạt động như một "bảng điều khiển trung tâm", tổng hợp các cảnh báo và phát hiện từ nhiều dịch vụ khác nhau vào một nơi duy nhất.
* **AWS Config**: Là một "thanh tra viên" liên tục theo dõi và ghi lại các thay đổi cấu hình của tài nguyên, đồng thời đánh giá chúng dựa trên các quy tắc (rules) bạn đặt ra.
* **Amazon GuardDuty**: Là một "hệ thống phát hiện mối đe dọa thông minh", sử dụng máy học để phân tích các log hoạt động và phát hiện các hành vi đáng ngờ.

{{% notice tip %}}
Bật các dịch vụ này ngay từ đầu trong một tài khoản AWS mới là một thông lệ tốt nhất để có cái nhìn tổng quan về tình trạng an ninh ngay lập tức.
{{% /notice %}}

### **AWS Lambda & EventBridge (Tự động hóa Phản ứng)**
{{% notice info %}}
**AWS Lambda** cho phép bạn chạy code mà không cần cung cấp hay quản lý máy chủ. **Amazon EventBridge** là một dịch vụ bus sự kiện serverless, giúp kết nối các ứng dụng với nhau bằng cách sử dụng các sự kiện.
{{% /notice %}}
* Chúng ta sử dụng bộ đôi này để xây dựng một cơ chế tự động khắc phục sự cố.
* **EventBridge** sẽ "lắng nghe" các cảnh báo từ Security Hub hoặc Config.
* Khi nhận được một cảnh báo khớp với mẫu định sẵn, nó sẽ kích hoạt một hàm **Lambda**.
* Hàm **Lambda** sẽ thực thi logic được lập trình sẵn để tự động sửa lỗi (ví dụ: xóa một rule không an toàn trong Security Group).

{{% notice warning %}}
**Lưu ý về Bảo mật:** IAM Role được gán cho các hàm Lambda khắc phục tự động phải tuân thủ nghiêm ngặt nguyên tắc đặc quyền tối thiểu. Chỉ cấp cho nó những quyền thực sự cần thiết để thực hiện nhiệm vụ sửa lỗi.
{{% /notice %}}
