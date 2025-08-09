---
title: "4.1 Giám sát An ninh & Tuân thủ"
weight: 41
---

Trong phần này, chúng ta sẽ kích hoạt một bộ các dịch vụ an ninh mạnh mẽ của AWS. Chúng hoạt động cùng nhau để cung cấp một cái nhìn 360 độ về tình trạng an ninh của tài khoản.

### A. AWS Security Hub (Bảng điều khiển trung tâm)
{{% notice info %}}
**Security Hub** hoạt động như một "bảng điều khiển an ninh trung tâm". Nó tổng hợp các phát hiện bảo mật từ nhiều dịch vụ khác nhau vào một nơi duy nhất.
{{% /notice %}}

1.  Truy cập dịch vụ **AWS Security Hub**.
2.  Nhấn **Enable Security Hub** (nếu đây là lần đầu bạn sử dụng).
3.  Ở menu bên trái, vào mục **Security standards**.
4.  Tìm và **Enable** tiêu chuẩn **"AWS Foundational Security Best Practices v1.0.0"**.
![](/images/enable-security.png)
### B. AWS Config (Thanh tra viên cấu hình)
{{% notice info %}}
**AWS Config** giống như một "thanh tra viên tự động". Nó liên tục kiểm tra cấu hình của các tài nguyên và so sánh với một "bộ quy tắc" (rules) bạn định nghĩa.
{{% /notice %}}

#### Kích hoạt Dịch vụ
1.  Truy cập dịch vụ **AWS Config**.
2.  Kích hoạt dịch vụ nếu đây là lần đầu sử dụng. Trong phần **Settings -> Edit**, hãy đảm bảo bạn đã chọn **"Record all supported resource types"** để Config theo dõi tất cả tài nguyên.
![](/images/record-allres.png)
#### Xây dựng Rule Tùy chỉnh (Custom Rule)
{{% notice tip %}}
Thay vì dùng các rule có sẵn (có thể khác nhau giữa các khu vực), chúng ta sẽ tự xây dựng một rule của riêng mình bằng AWS Lambda để có toàn quyền kiểm soát.
{{% /notice %}}

**1. Tạo IAM Role cho Lambda**
1.  Truy cập **IAM** -> **Roles** -> **Create role**.
2.  **Trusted entity**: Chọn `AWS service`.
3.  **Use case**: Chọn `Lambda`.
4.  **Add permissions**: Tìm và đính kèm 2 policy sau: `AWSConfigRulesExecutionRole` và `AmazonEC2ReadOnlyAccess`.
5.  **Role name**: `CustomConfigRuleLambdaRole` và hoàn tất việc tạo role.

**2. Tạo Lambda Function**
1.  Truy cập **AWS Lambda** -> **Create function**.
2.  **Name**: `CheckUnrestrictedPortsFunction`, **Runtime**: `Python 3.11+`, **Role**: Chọn `CustomConfigRuleLambdaRole`.
3.  Dán đoạn code sau vào và **Deploy**:
    ```python
    import json
    import boto3

    config_client = boto3.client('config')
    ec2_client = boto3.client('ec2')

    # Định nghĩa các port được coi là nguy hiểm nếu mở ra 0.0.0.0/0
    DANGEROUS_PORTS = [22, 3389] 

    def evaluate_compliance(configuration_item):
        sg_id = configuration_item['resourceId']
        
        try:
            response = ec2_client.describe_security_groups(GroupIds=[sg_id])
            ip_permissions = response['SecurityGroups'][0]['IpPermissions']
        except Exception as e:
            return 'NOT_APPLICABLE'

        for rule in ip_permissions:
            if rule.get('IpProtocol') != 'tcp': continue
            from_port = rule.get('FromPort', -1)
            to_port = rule.get('ToPort', -1)

            for ip_range in rule.get('IpRanges', []):
                if ip_range.get('CidrIp') == '0.0.0.0/0':
                    for port in DANGEROUS_PORTS:
                        if from_port <= port <= to_port:
                            return 'NON_COMPLIANT'
        return 'COMPLIANT'

    def lambda_handler(event, context):
        invoking_event = json.loads(event['invokingEvent'])
        configuration_item = invoking_event['configurationItem']
        compliance_status = evaluate_compliance(configuration_item)
        
        config_client.put_evaluations(
            Evaluations=[
                {
                    'ComplianceResourceId': configuration_item['resourceId'],
                    'ComplianceResourceType': configuration_item['resourceType'],
                    'ComplianceType': compliance_status,
                    'OrderingTimestamp': configuration_item['configurationItemCaptureTime']
                },
            ],
            ResultToken=event['resultToken']
        )
    ```

**3. Tạo Custom Rule trong AWS Config**
1.  Quay lại **AWS Config** -> **Rules** -> **Add rule**.
2.  Chọn **"Create custom Lambda rule"**.
3.  **Configure rule**:
    * **Name**: `custom-check-unrestricted-ports`.
    ![](/images/configure-rule.png)
    * **Lambda function ARN**: Dán ARN của hàm Lambda `CheckUnrestrictedPortsFunction`.
    ![](/images/arn-lamda.png)
4.  **Trigger**:
    * **Trigger type**: `Configuration changes`.
    * **Scope of changes** -> **Resources**: Chọn `EC2: SecurityGroup`.
    ![](/images/evaution-mode.png)
5.  Nhấn **Save**.

### C. Amazon GuardDuty (Hệ thống phát hiện xâm nhập)
{{% notice info %}}
**Amazon GuardDuty** là một "hệ thống an ninh thông minh" sử dụng máy học để phát hiện các hành vi đáng ngờ (ví dụ: quét cổng, giao tiếp với IP độc hại).
{{% /notice %}}

1.  Truy cập dịch vụ **Amazon GuardDuty**.
2.  Nhấn **Get Started** -> **Enable GuardDuty**. Việc kích hoạt chỉ đơn giản như vậy!

### D. AWS CloudTrail (Hộp đen an ninh)
{{% notice info %}}
**AWS CloudTrail** hoạt động như một "hộp đen" hay "camera an ninh", ghi lại **mọi hành động** (mọi lệnh gọi API) diễn ra trong tài khoản của bạn.
{{% /notice %}}

1.  Truy cập dịch vụ **AWS CloudTrail**.
2.  Ở menu bên trái, chọn **Event history**.
3.  Bạn có thể thấy một danh sách dài các sự kiện đã diễn ra. Đây là nguồn dữ liệu quan trọng nhất cho việc điều tra sự cố. Dịch vụ này được bật mặc định.
![](/images/cloudtrail-his.png)

**Kết quả:** Sau khi bật các dịch vụ trên, bạn đã có một hệ thống giám sát đa tầng. Tất cả các phát hiện từ Config và GuardDuty sẽ tự động được gửi về Security Hub.