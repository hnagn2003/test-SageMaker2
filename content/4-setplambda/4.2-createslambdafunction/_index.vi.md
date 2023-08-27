---
title : "Tạo S3 Bucket"
date :  "`r Sys.Date()`" 
weight : 2 
chapter : false
pre : " <b> 4.2 </b> "
---


1. Tạo một file Python ```app.py``` để kết nối các dịch vụ và đưa ra kết quả suy luận.
    
    ```jsx
    app.py
    ```
    
    ```jsx
    import os, logging, json, time, urllib.parse
    import boto3, botocore
    import numpy as np, cv2
    
    logger = logging.getLogger()
    logger.setLevel(logging.INFO)
    client = boto3.client('lambda')
    
    # S3 BUCKETS DETAILS
    s3 = boto3.resource('s3')
    BUCKET_NAME = "<NAME OF S3 BUCKET FOR INPUT IMAGE>"
    IMAGE_LOCATION = "<S3 PREFIX TO IMAGE>/image.png"
    
    # INFERENCE ENDPOINT DETAILS
    ENDPOINT_NAME = 'yolov5-demo'
    config = botocore.config.Config(read_timeout=80)
    runtime = boto3.client('runtime.sagemaker', config=config)
    modelHeight, modelWidth = 640, 640
    
    # RUNNING LAMBDA
    def lambda_handler(event, context):
        key = urllib.parse.unquote_plus(event['Records'][0]['s3']['object']['key'], encoding='utf-8')
        
        # INPUTS - Download Image file from S3 to Lambda /tmp/
        input_imagename = key.split('/')[-1]
        logger.info(f'Input Imagename: {input_imagename}')
        s3.Bucket(BUCKET_NAME).download_file(IMAGE_LOCATION + '/' + input_imagename, '/tmp/' + input_imagename)
    
        # INFERENCE - Invoke the SageMaker Inference Endpoint
        logger.info(f'Starting Inference ... ')
        orig_image = cv2.imread('/tmp/' + input_imagename)
        if orig_image is not None:
            start_time_iter = time.time()
            # pre-processing input image
            image = cv2.resize(orig_image.copy(), (modelWidth, modelHeight), interpolation = cv2.INTER_AREA)
            data = np.array(image.astype(np.float32)/255.)
            payload = json.dumps([data.tolist()])
            # run inference
            response = runtime.invoke_endpoint(EndpointName=ENDPOINT_NAME, ContentType='application/json', Body=payload)
            # get the output results
            result = json.loads(response['Body'].read().decode())
            end_time_iter = time.time()
            # get the total time taken for inference
            inference_time = round((end_time_iter - start_time_iter)*100)/100
        logger.info(f'Inference Completed ... ')
    
        # OUTPUTS - Using the output to utilize in other services downstream
        return {
            "statusCode": 200,
            "body": json.dumps({
                "message": "Inference Time:// " + str(inference_time) + " seconds.",
                "results": result
            }),
        }
    ```
    Tải file này lên thư mục chứa thư mục **Docker** và **yolov5**
    ![](../../images/lambda/004.png)
    ![](../../images/lambda/009.png)
    
2. Nén và vứt lên S3
    
    ```jsx
    !zip app.zip app.py
    ```
    
    ```jsx
    !aws s3 cp app.zip s3://ml-bucket-123/yolov5/app.zip
    ```
    
    ![](../../images/lambda/zipapp.png)
    
3. Sau đó tạo Lambda function and config it
    - Get your Lambda layer that contains cv2 ARN
  
    ![](../../images/lambda/cv2.png)
    
    - and copy the ARN
    ![](../../images/lambda/cv2_2.png)

    - Next we create Lambda function
    
    ```jsx
    !aws lambda create-function --function-name yolov5-lambda --handler app.lambda_handler --region ap-southeast-2 --runtime python3.7 --environment '{"Variables":{"BUCKET_NAME":"$BUCKET_NAME", "S3_KEY":"$S3_KEY"}}' --code S3Bucket=ml-bucket-123,S3Key="yolov5/app.zip" --role arn:aws:iam::974837768594:role/service-role/AmazonSageMaker-ExecutionRole-20230827T044926
    ```
    
    you will need modify some your own information
    
      - Region: config to your current region
      - Role: ARN your role you created in 1.3
      - Attach the OpenCV layer to the Lambda function

    - Ta đi gắn OpenCV layer vào Lambda function
    
    ```jsx
    !aws lambda update-function-configuration --function-name yolov5-lambda --layers <cv2_ARN>
    ```
    
    where 
    
    - <cv2_ARN> : your cv2 ARN you copied
![](../../images/lambda/010.png)

4. Ta sẽ thấy function được tạo trong bảng điều khiển
![](../../images/lambda/011.png)