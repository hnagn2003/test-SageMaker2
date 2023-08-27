---
title : "Cập nhật IAM Role"
date :  "`r Sys.Date()`" 
weight : 2 
chapter : false
pre : " <b> 2.2 </b> "
---

## Cập nhật IAM role

Role bạn tạo trước đó khi tạo SageMaker notebook chỉ cấp quyền cho các dịch vụ của SageMaker. Trong lab này chúng ta sẽ làm việc với Amazon S3 and Amazon Lambda, ta đi thêm 1 số chính sách vào role đó.

1. Mở **IAM role** từ SageMaker notebook instance của ta, lưu trữ ARN id này để về sau.
    
    ![](../../images/lambda/014-runpredict.png)
    ![](../../images/lambda/015.png)

2. Thêm quyền S3 and Lambda
    
    ![](../../images/lambda/003.png)
    ![](../../images/lambda/002.png)

3. Thêm trust policy cho Lambda
    
    
    Click **Edit trust policy**
    ![](../../images/lambda/006.png)
    Thêm đoạn mã này
    ```jsx
    {
        "Effect": "Allow",
        "Principal": {
            "Service": "lambda.amazonaws.com"
        },
        "Action": "sts:AssumeRole"
    }
    ```
    ![](../../images/lambda/008.png)
    Sau đó click **Update policy**