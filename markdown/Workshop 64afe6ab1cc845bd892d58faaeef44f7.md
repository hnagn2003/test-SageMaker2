# Workshop

# Induction

# Prerequiste

## Create SageMaker notebook

1. Create a notebook instance:
- Name: ml
- Type:

![Untitled](Workshop%2064afe6ab1cc845bd892d58faaeef44f7/Untitled.png)

img 00→005

1. After creating notebook instance for a while, the status will be shown **InService**, then click - - **Open JupyterLab** to open jupiter SageMaker notebooks
    
    006
    
    Choose anaconda python3 environment
    
    007
    
2. With jupiter notebook
    
    ```markdown
    {{% notice info %}}
    Write code in the cell and press *Shift+Enter* to execute it
    {{% /notice %}}
    ```
    
3. First, we need to clone YOLOv5 source code for reference task
    
    ```jsx
    !git clone https://github.com/ultralytics/yolov5
    ```
    
    Then open the folder yolov5
    
    ```jsx
    !git clone https://github.com/ultralytics/yolov5
    ```
    
    Install requirements modules and export weights file to folder *saved_model* for pretrained model deployment
    
    ```jsx
    !pip install -r requirements.txt tensorflow-cpu
    !python export.py --weights yolov5l.pt --include saved_model --nms
    ```
    
    009
    
    - The model will be loaded successfully
        
        009model
        
    - Don’t close the tab, we will work with it in the following instruction

## Create S3 bucket

Create a S3 bucket with name ```ml-bucket-123```, choose your right Region too.

010-013

## Modify IAM role

The role you create along with SageMaker in above only have SageMaker access. In the next steps we will work with S3 and Lambda, so we should add some policies to that role.

1. Open IAM role from your SageMaker notebook instance, save its ARN for later.
    
    014-r, 015
    
2. Add S3 and Lambda permission
    
    003, 002
    
3. Add trust policy for Lambda
    
    ```jsx
    {
        "Effect": "Allow",
        "Principal": {
            "Service": "lambda.amazonaws.com"
        },
        "Action": "sts:AssumeRole"
    }
    ```
    
    006, 008
    

# Deploy YOLOv5 on SageMaker endpoint

## Forward model weight to endpoint

1. At your Jupiter notebook,
    - Zip the exported model weights file
        
        ```jsx
        mkdir export && mkdir export/Servo
        ```
        
        ```jsx
        mv yolov5l_saved_model export/Servo/1
        ```
        
        ```jsx
        !tar -czvf model.tar.gz export/
        ```
        
    - Then upload the model to S3 Bucket. Where “bucket” field is your bucket name that you created in section 1.2
        
        ```jsx
        from sagemaker import s3
        
        bucket = "<s3://BUCKET/NAME>"
        # example: bucket = "s3://ml-bucket-123"
        prefix = "yolov5/custom-endpoint"
        model_data = s3.S3Uploader.upload("model.tar.gz", bucket + "/" + prefix)
        ```
        
    - After that, we deploy the model to the endpoint named *yolov5.* Here, I choose *ml.m5.xlarge* for the instance will be used for deploying*.* It will take about 5-10’ to complete.
        
        ```jsx
        import os
        import tensorflow as tf
        from tensorflow.keras import backend
        import sagemaker
        from sagemaker.tensorflow import TensorFlowModel
        
        role = sagemaker.get_execution_role()
        
        model = TensorFlowModel(model_data=model_data,
                                framework_version='2.8', role=role)
        
        INSTANCE_TYPE = 'ml.m5.xlarge'
        ENDPOINT_NAME = 'yolov5l'
        
        predictor = model.deploy(initial_instance_count=1,
                                    instance_type=INSTANCE_TYPE,
                                    endpoint_name=ENDPOINT_NAME)
        ```
        
        014
        
        When the deploying complete, open Amazon SageMaker > Endpoints and you will see the endpoints we created with above lines of code.
        
        018
        

## Test the endpoint

1. We will ensure that the endpoint work correctly by sending a blank image to the endpoint for inference.

```jsx
import numpy as np
import json

import boto3

ENDPOINT_NAME = 'yolov5l'
runtime= boto3.client('runtime.sagemaker')
modelHeight, modelWidth = 640, 640
blank_image = np.zeros((modelHeight,modelWidth,3), np.uint8)
data = np.array(blank_image.astype(np.float32)/255.)
payload = json.dumps([data.tolist()])
response = runtime.invoke_endpoint(EndpointName=ENDPOINT_NAME,
ContentType='application/json',
Body=payload)

result = json.loads(response['Body'].read().decode())
print('Results: ', result)
```

017-testendpoint.png

If you got that output then your endpoint run properly.

# **Set up Lambda**

Lambda doesn’t come with external libraries like OpenCV pre-built, therefore we need to build it before we can invoke the Lambda code.

## Create Lambda layers using Docker

1. Create Dockerfile
    
    We create a folder named Docker have the same folder level with folder yolov5. Open Docker folder, then upload this Dockerfile
    
    ```jsx
    FROM amazonlinux
    
    RUN yum update -y
    RUN yum install gcc openssl-devel bzip2-devel libffi-devel wget tar gzip zip zlib-devel make -y
    
    # Install Python 3.7
    WORKDIR /
    RUN wget https://www.python.org/ftp/python/3.7.12/Python-3.7.12.tgz
    RUN tar -xzvf Python-3.7.12.tgz
    WORKDIR /Python-3.7.12
    RUN ./configure --enable-optimizations
    RUN make altinstall
    
    # Install Python packages
    RUN mkdir /packages
    RUN echo "opencv-python" >> /packages/requirements.txt
    RUN mkdir -p /packages/opencv-python-3.7/python/lib/python3.7/site-packages
    RUN pip3.7 install -r /packages/requirements.txt -t /packages/opencv-python-3.7/python/lib/python3.7/site-packages
    
    # Create zip files for Lambda Layer deployment
    WORKDIR /packages/opencv-python-3.7/
    RUN zip -r9 /packages/cv2-python37.zip .
    WORKDIR /packages/
    RUN rm -rf /packages/opencv-python-3.7/
    ```
    
    006-008
    
2. Pull and build the docker image
    
    ```jsx
    !docker build --tag aws-lambda-layers:latest Docker
    ```
    
    This progess may take some time to finish.
    
    buildimage
    
3. Then run Lambda layers container
    
    ```jsx
    !docker run --rm -it -v $(pwd):/layers aws-lambda-layers cp /packages/cv2-python37.zip /layers
    ```
    
4. Upload OpenCV modules to S3
    
    ```jsx
    !aws s3 cp cv2-python37.zip s3://ml-bucket-123/yolov5/artifacts/cv2-python37.zip
    ```
    

buildimage.png

## **Create the Lambda function and attach the OpenCV layer**

1. We create [app.py](http://app.py) that runs the inference and gets the results. Upload this Python script to the folder contain yolov5 and Docker folders
    
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
    
    004, 009
    
2. Zip it and pass to S3
    
    ```jsx
    !zip app.zip app.py
    ```
    
    ```jsx
    !aws s3 cp app.zip s3://ml-bucket-123/yolov5/app.zip
    ```
    
    zipapp.png
    
3. Then create Lambda function and config it
    - Get your Lambda layer that contains cv2 ARN
    
    cv2.png
    
    cv2_2.png
    
    ```jsx
    !aws lambda create-function --function-name yolov5-lambda --handler app.lambda_handler --region ap-southeast-2 --runtime python3.7 --environment '{"Variables":{"BUCKET_NAME":"$BUCKET_NAME", "S3_KEY":"$S3_KEY"}}' --code S3Bucket=ml-bucket-123,S3Key="yolov5/app.zip" --role arn:aws:iam::974837768594:role/service-role/AmazonSageMaker-ExecutionRole-20230827T044926
    ```
    
    where
    
    - Region: config to your current region
    - Role: ARN your role you created in 1.3
    - Attach the OpenCV layer to the Lambda function
    
    ```jsx
    !aws lambda update-function-configuration --function-name yolov5-lambda --layers <cv2_ARN>
    ```
    
    where 
    
    - <cv2_ARN> : your cv2 ARN you copied

## Activate trigger Lambda for input image

1. Open Lambda function you have created and create the trigger

trigger1.png → trigger4.png

1. Open the [Buckets](https://console.aws.amazon.com/s3/buckets) page of the Amazon S3 console and choose the bucket you created earlier
    
    trigger5.png
    
2. Open folder **Yolov5/**
    
    trigger6.png
    
3. Choose **Upload**, choose **Add file** and upload a random tiny file 
    
    trigger7.png
    
    Choose **Upload**
    
    trigger8.png
    
4. To verify correct operation using CloudWatch Logs
    
    1. Open the [CloudWatch](https://console.aws.amazon.com/cloudwatch/home) console. Make sure about your Region
    
    1. Choose **Logs**, then choose **Log groups**.
    2. Choose the log group for your function
        
        trigger10.png
        
    3. 1. Under **Log streams**, choose the most recent log stream.
        
        trigger9.png
        
    4. If your function has been invoked correctly in response to your Amazon S3 trigger, you’ll see output similar to the following. The `CONTENT TYPE` you see depends on the type of file you uploaded to your bucket.
        
        trigger11.png
        

# Run inference

After all is up, we test the output from invoking Lambda function by uploading an image to S3 as a trigger to invoke Lambda and get the inference result.

1. Import requirement module
    
    ```jsx
    %matplotlib inline
    import matplotlib.pyplot as plt
    import numpy as np
    import cv2
    import json
    import boto3, botocore
    ```
    
    ```jsx
    def prep_image(img_path):
        img = cv2.imread(img_path)
        img = cv2.cvtColor(img, cv2.COLOR_BGR2RGB)
        height,width = img.shape[0], img.shape[1]
    
        top_pad = bot_pad = height % 640 // 2
        left_pad = right_pad = width % 640 // 2
    
        img_padded = cv2.copyMakeBorder(img, top_pad, bot_pad, left_pad, right_pad, cv2.BORDER_CONSTANT, value=[114,114,114])
        img_padded_and_resized = cv2.resize(img_padded,(640,640))
        #calculate border padding
    
        #img = cv2.resize(img, (4032,3040))
    
        #plt.imshow(img_padded_and_resized)
        return img_padded_and_resized
    ```
    
2. Upload an random image for reference
    
    ```jsx
    img = prep_image('test_image.jpg')
    ```
    
    ```jsx
    plt.figure(figsize=(10,10))
    plt.imshow(img)
    plt.axis('off')
    ```
    
    002
    
    ```jsx
    config = botocore.config.Config(read_timeout=500)
    runtime = boto3.client('runtime.sagemaker', config=config)
    ```
    
    ```jsx
    data = np.array(img.astype(np.float16)/255.)
    payload = json.dumps([data.tolist()])
    
    response = runtime.invoke_endpoint(EndpointName='yolov5l-demo2', ContentType='application/json', Body=payload)
    
    result = json.loads(response['Body'].read().decode())
    ```
    
    ```jsx
    indices = np.where(np.array(result['predictions'][0]['output_1']) > 0.5)
    xywh = np.array(result['predictions'][0]['output_0'])[indices]
    xywh[:,0] *= 640
    xywh[:,1] *= 640
    xywh[:,2] *= 640
    xywh[:,3] *= 640
    xywh = xywh.astype(int)
    
    scores = np.array(result['predictions'][0]['output_1'])[indices]
    classes = np.array(result['predictions'][0]['output_2'])[indices]
    ```
    
    ```jsx
    class_names = ['person', 'bicycle', 'car', 'motorcycle', 'airplane', 'bus', 'train', 'truck', 'boat', 'traffic light',
            'fire hydrant', 'stop sign', 'parking meter', 'bench', 'bird', 'cat', 'dog', 'horse', 'sheep', 'cow',
            'elephant', 'bear', 'zebra', 'giraffe', 'backpack', 'umbrella', 'handbag', 'tie', 'suitcase', 'frisbee',
            'skis', 'snowboard', 'sports ball', 'kite', 'baseball bat', 'baseball glove', 'skateboard', 'surfboard',
            'tennis racket', 'bottle', 'wine glass', 'cup', 'fork', 'knife', 'spoon', 'bowl', 'banana', 'apple',
            'sandwich', 'orange', 'broccoli', 'carrot', 'hot dog', 'pizza', 'donut', 'cake', 'chair', 'couch',
            'potted plant', 'bed', 'dining table', 'toilet', 'tv', 'laptop', 'mouse', 'remote', 'keyboard', 'cell phone',
            'microwave', 'oven', 'toaster', 'sink', 'refrigerator', 'book', 'clock', 'vase', 'scissors', 'teddy bear',
            'hair drier', 'toothbrush']
    ```
    
    ```jsx
    FONT = cv2.FONT_HERSHEY_SIMPLEX
    FONTSCALE = .6
    WHITE = (255, 255, 255)
    THICKNESS = 2
    for idx, rect in enumerate(xywh):
        img = cv2.rectangle(img,
                  (rect[0], rect[1]-5),
                  (rect[2], rect[3]), thickness=2, color = (255,0,0))
        
        class_idx = int(classes[idx])
        img = cv2.putText(img, 
                          f'{class_names[class_idx]}: {scores[idx]:0.3f}',
                          (rect[0],rect[1]),
                          FONT,
                          FONTSCALE,
                          WHITE,
                          THICKNESS)
    ```
    
    003
    
3. And show the result:
    
    ```jsx
    plt.figure(figsize=(10,10))
    plt.imshow(img)
    plt.axis('off')
    ```
    
    004
    

# Clean