---
title: "4.2 Tự động Khắc phục sự cố"
weight: 42
---

Đây là bước cuối cùng và cũng là bước nâng cao nhất của workshop. Chúng ta sẽ xây dựng một cơ chế tự "chữa bệnh": khi một vấn đề bảo mật được phát hiện, hệ thống sẽ tự động sửa nó mà không cần con người can thiệp.

{{% notice info %}}
**Kịch bản:** Ở bước 1, chúng ta đã cố tình tạo ra một lỗi cấu hình trong `FirstCloudJourney-SG` bằng cách mở port 22 (SSH) ra toàn thế giới. Bây giờ là lúc để kiểm tra xem hệ thống tự động khắc phục của chúng ta có phát hiện và sửa lỗi đó hay không.
{{% /notice %}}

### A. Tạo Lambda Function để khắc phục lỗi

Đầu tiên, chúng ta cần một "công nhân" có khả năng sửa lỗi cấu hình Security Group.

#### 1. Tạo IAM Role cho Lambda
1.  Truy cập dịch vụ **IAM** -> **Roles** -> **Create role**.
2.  **Trusted entity**: `AWS service`, **Use case**: `Lambda`.
3.  **Add permissions**: Tìm và đính kèm 3 policy sau:
    * `AWSLambdaBasicExecutionRole` (để ghi log)
    * `AWSSecurityHubFullAccess` (để cập nhật trạng thái finding)
    * `AmazonEC2FullAccess` (để có thể sửa Security Group).
4.  **Role name**: `SGRuleRemediationRole` và hoàn tất việc tạo role.

#### 2. Tạo Lambda Function
1.  Truy cập dịch vụ **AWS Lambda** -> **Create function**.
2.  **Function name**: `SGRuleRemediator`.
3.  **Runtime**: `Python 3.11` (hoặc mới hơn).
4.  **Permissions**: Chọn `Use an existing role` và chọn role `SGRuleRemediationRole`.
5.  **Thêm Code:** Dán đoạn code Python sau vào và nhấn **Deploy**.
    ```python
    import boto3
    import json
    import logging

    logger = logging.getLogger()
    logger.setLevel(logging.INFO)

    ec2_client = boto3.client('ec2')
    securityhub_client = boto3.client('securityhub')

    def lambda_handler(event, context):
        logger.info(f"Received event: {json.dumps(event)}")
        
        # Xử lý sự kiện từ EventBridge được kích hoạt bởi AWS Config
        if event.get('source') == 'aws.config':
            try:
                sg_id = event['detail']['resourceId']
                compliance_type = event['detail']['newEvaluationResult']['complianceType']

                if compliance_type == 'NON_COMPLIANT':
                    logger.info(f"Detected NON_COMPLIANT state for Security Group: {sg_id}")
                    
                    # ---- HÀNH ĐỘNG KHẮC PHỤC ----
                    ec2_client.revoke_security_group_ingress(
                        GroupId=sg_id,
                        IpPermissions=[{
                            'IpProtocol': 'tcp',
                            'FromPort': 22,
                            'ToPort': 22,
                            'IpRanges': [{'CidrIp': '0.0.0.0/0'}]
                        }]
                    )
                    logger.info(f"Successfully removed SSH rule from Security Group: {sg_id}")

            except Exception as e:
                logger.error(f"Error processing Config event for {event.get('detail', {}).get('resourceId', 'N/A')}: {str(e)}")
                
        # Xử lý sự kiện từ Security Hub (dành cho các kịch bản khác)
        elif event.get('source') == 'aws.securityhub':
            for finding in event['detail']['findings']:
                # ... (code xử lý finding của Security Hub có thể được thêm vào đây)
                pass # Bỏ qua trong kịch bản này

        return {'statusCode': 200, 'body': json.dumps('Remediation complete.')}
    ```

### B. Tạo "Cảm biến" kích hoạt (EventBridge Rule)
{{% notice tip %}}
Chúng ta sẽ tạo một rule lắng nghe sự kiện trực tiếp từ **AWS Config** để có phản ứng nhanh và đáng tin cậy nhất.
{{% /notice %}}

1.  Truy cập **Amazon EventBridge** -> **Rules** -> **Create rule**.
2.  **Name**: `RemediateUnrestrictedSSH`.
3.  **Event pattern**:
    * **Event source**: `AWS events or EventBridge partner events`.
    * Cuộn xuống **Event pattern** -> nhấn **Edit pattern**.
    * Dán đoạn JSON sau vào. Nó chỉ định rõ rằng chúng ta chỉ quan tâm đến rule `restricted-common-ports` khi nó chuyển sang `NON_COMPLIANT`.
    ```json
    {
      "source": ["aws.config"],
      "detail-type": ["Config Rules Compliance Change"],
      "detail": {
        "messageType": ["ComplianceChangeNotification"],
        "configRuleName": ["restricted-common-ports"],
        "newEvaluationResult": {
          "complianceType": ["NON_COMPLIANT"]
        }
      }
    }
    ```
4.  **Target**: Chọn `Lambda function` -> `SGRuleRemediator`.
5.  Hoàn tất việc tạo rule.

### C. Kiểm tra Hệ thống
Bây giờ, chúng ta sẽ không cần tạo lỗi mới, mà chỉ cần ra lệnh cho hệ thống kiểm tra lại các tài nguyên đang có.

1.  Truy cập dịch vụ **AWS Config**.
2.  Ở menu bên trái, chọn **Rules**.
3.  Tick chọn vào rule bạn đã tạo (ví dụ: `custom-check-unrestricted-ports`).
4.  Nhấn **Actions** -> **Re-evaluate**.
    {{% notice tip %}}
    Thao tác này sẽ buộc "thanh tra viên" Config phải đi kiểm tra lại tất cả các Security Group ngay lập tức.
    {{% /notice %}}

### Quan sát Kết quả

1.  **Chờ khoảng 2-3 phút.**
2.  Trong thời gian đó, quy trình sau sẽ tự động diễn ra:
    * **AWS Config** sẽ chạy và phát hiện `FirstCloudJourney-SG` đang `Non-compliant`.
    * **EventBridge** sẽ bắt được sự kiện này.
    * **Lambda** `SGRuleRemediator` sẽ được kích hoạt và thực thi lệnh xóa rule.
3.  Sau khi chờ, hãy truy cập **EC2** -> **Security Groups** và chọn security group **`FirstCloudJourney-SG`**.
4.  Vào tab **Inbound rules** và làm mới (refresh) trang.

Rule cho **port 22 (SSH)** mà bạn đã tạo từ Phase 1 sẽ **tự động biến mất**. Điều này chứng tỏ hệ thống đã phát hiện và khắc phục thành công lỗ hổng cấu hình.

**Chúc mừng! Bạn đã hoàn thành toàn bộ nội dung kỹ thuật của workshop!**