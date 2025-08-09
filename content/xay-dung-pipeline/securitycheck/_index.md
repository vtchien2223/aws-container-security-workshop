---
title: "2.3 Tạo Cổng An ninh"
weight: 23
---

Bây giờ pipeline đã có thể tự động build image, chúng ta sẽ tạo một "cổng kiểm soát an ninh" bằng AWS Lambda. Nếu image có lỗ hổng nghiêm trọng, pipeline sẽ tự động dừng lại, ngăn chặn việc triển khai các ứng dụng có rủi ro.

### A. Tạo IAM Role cho Lambda
{{% notice info %}}
Hàm Lambda cần có quyền để đọc kết quả quét từ Amazon Inspector và báo cáo trạng thái (thành công/thất bại) về cho CodePipeline.
{{% /notice %}}

1.  Truy cập dịch vụ **IAM** -> **Roles** -> **Create role**.
2.  **Trusted entity**: `AWS service`, **Use case**: `Lambda`.
3.  **Add permissions**: Tìm và đính kèm 2 policy sau:
    * `AmazonInspector2ReadOnlyAccess`
    * `AWSCodePipeline_FullAccess`
4.  **Role name**: Đặt tên là `VulnerabilityCheckLambdaRole` và hoàn tất việc tạo role.

### B. Tạo Lambda Function
1.  Truy cập dịch vụ **AWS Lambda** -> **Create function**.
2.  **Function name**: `VulnerabilityCheckFunction`.
3.  **Runtime**: Chọn `Python 3.11` (hoặc mới hơn).
4.  **Permissions**: Chọn `Use an existing role` và chọn role `VulnerabilityCheckLambdaRole` bạn vừa tạo.
![](/images/cr-lamda.png)
5.  **Thêm Code:** Sao chép và dán toàn bộ đoạn code Python sau vào trình soạn thảo code của Lambda.
    {{% notice tip %}}
    Đoạn code này sẽ nhận artifact từ pipeline, tìm ra image vừa được build, sau đó gọi Amazon Inspector để kiểm tra các lỗ hổng có mức độ `CRITICAL` hoặc `HIGH`.
    {{% /notice %}}
    ```python
    import boto3
    import json
    import logging
    import zipfile
    import os

    logger = logging.getLogger()
    logger.setLevel(logging.INFO)

    inspector_client = boto3.client('inspector2')
    codepipeline_client = boto3.client('codepipeline')
    s3_client = boto3.client('s3')

    def get_image_details_from_artifact(job_data):
        try:
            artifact_data = job_data['inputArtifacts'][0]
            s3_location = artifact_data['location']['s3Location']
            bucket_name = s3_location['bucketName']
            object_key = s3_location['objectKey']

            local_zip_path = "/tmp/artifact.zip"
            s3_client.download_file(bucket_name, object_key, local_zip_path)

            extract_dir = "/tmp/artifact_extracted"
            with zipfile.ZipFile(local_zip_path, 'r') as zip_ref:
                zip_ref.extractall(extract_dir)
            
            imagedefinitions_path = os.path.join(extract_dir, 'imagedefinitions.json')
            with open(imagedefinitions_path, 'r') as f:
                imagedefs = json.load(f)
                
            image_uri = imagedefs[0]['imageUri']
            repo_name = image_uri.split('@')[0].split('/', 1)[1]
            image_digest = image_uri.split('@')[1]
            
            return repo_name, image_digest
        except Exception as e:
            logger.error(f"Failed to get image details from artifact: {str(e)}")
            raise

    def lambda_handler(event, context):
        job_id = event['CodePipeline.job']['id']
        job_data = event['CodePipeline.job']['data']
        
        try:
            repo_name, image_digest = get_image_details_from_artifact(job_data)
            logger.info(f"Checking vulnerabilities for image {repo_name} with digest {image_digest}")

            paginator = inspector_client.get_paginator('list_findings')
            pages = paginator.paginate(
                filterCriteria={
                    'awsAccountId': [{'comparison': 'EQUALS', 'value': context.invoked_function_arn.split(":")[4]}],
                    'ecrImageDigest': [{'comparison': 'EQUALS', 'value': image_digest}]
                }
            )

            critical_or_high_findings = []
            for page in pages:
                for finding in page.get('findings', []):
                    if finding.get('severity') in ['CRITICAL', 'HIGH']:
                        logger.warning(f"Found {finding.get('severity')} finding: {finding.get('type')}")
                        critical_or_high_findings.append(finding)

            if critical_or_high_findings:
                failure_message = f"Pipeline stopped. Found {len(critical_or_high_findings)} CRITICAL/HIGH severity vulnerabilities."
                logger.error(failure_message)
                codepipeline_client.put_job_failure_result(
                    jobId=job_id,
                    failureDetails={'type': 'JobFailed', 'message': failure_message}
                )
            else:
                success_message = "Security scan passed. No CRITICAL/HIGH severity vulnerabilities found."
                logger.info(success_message)
                codepipeline_client.put_job_success_result(jobId=job_id)

        except Exception as e:
            logger.error(f"An error occurred: {str(e)}")
            codepipeline_client.put_job_failure_result(
                jobId=job_id,
                failureDetails={'type': 'JobFailed', 'message': f"Function error: {str(e)}"}
            )
            
        return 'Done'
    ```
    ![](/images/deploy-lamda.png)
6.  Nhấn **Deploy**. Sau đó, vào tab **Configuration** -> **General configuration** và tăng **Timeout** lên **1 phút**.

### C. Thêm Stage "Security_Check" vào Pipeline

1.  Quay lại pipeline `Security-Pipeline` và nhấn **Edit**.
2.  Sau stage `Build`, nhấn **+ Add stage** và đặt tên là `Security_Check`.
![](/images/stagte_securitycheck.png)
3.  Trong stage mới, **Add action group**.
4.  **Cấu hình Action:**
    * **Action name**: `Check_Vulnerabilities`.
    * **Action provider**: `AWS Lambda`.
    * **Input artifacts**: Chọn `BuildArtifact`.
    * **Function name**: Chọn `VulnerabilityCheckFunction`.
    ![](/images/add-action.png)
5.  **Save** lại pipeline.

### D. Cấp quyền cuối cùng cho Pipeline
{{% notice warning %}}
Đây là bước cuối cùng để hoàn thiện chuỗi quyền hạn. Đừng bỏ qua nó!
{{% /notice %}}

1.  Truy cập **IAM** -> **Roles**.
2.  Tìm **CodePipeline Service Role** (tên chứa `Security-Pipeline`).
3.  Đính kèm thêm policy: **`AWSLambda_FullAccess`**.

Sau khi hoàn thành, hãy nhấn **"Release change"** trên pipeline để chạy và kiểm tra toàn bộ quy trình.
![](/images/securitypipeline.png)