---
title : "Modify IAM Role"
date : "`r Sys.Date()`"
weight : 3
chapter : false
pre : " <b> 2.3 </b> "
---

## Modify IAM role

The role you create along with SageMaker in above only have SageMaker access. In the next steps we will work with S3 and Lambda, so we should add some policies to that role.

1. Open **IAM role** from your SageMaker notebook instance, save its ARN for later.
    
    ![](images/lambda/014-runpredict.png)
    ![](images/lambda/015.png)
2. Add S3 and Lambda permission
    
    ![](images/lambda/003.png)
    ![](images/lambda/002.png)

3. Add trust policy for Lambda
    
    
    Click **Edit trust policy**
    ![](images/lambda/006.png)
    Add this script
    ```jsx
    {
        "Effect": "Allow",
        "Principal": {
            "Service": "lambda.amazonaws.com"
        },
        "Action": "sts:AssumeRole"
    }
    ```
    ![](images/lambda/008.png)
    Then **Update policy**