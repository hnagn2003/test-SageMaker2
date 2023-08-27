---
title : "Forward model weight to endpoint"
date : "`r Sys.Date()`"
weight : 1
chapter : false
pre : " <b> 3.1 </b> "
---

## Forward model weight to endpoint

1. Back to your Jupiter notebook,
- Zip the exported model weights file
    
      mkdir export && mkdir export/Servo
      
      mv yolov5l_saved_model export/Servo/1
      
      !tar -czvf model.tar.gz export/
        
2. Then upload the model to S3 Bucket. Where “bucket” field is your bucket name that you created in section 1.2
        
      ```jsx
      from sagemaker import s3
      
      bucket = "<s3://BUCKET/NAME>"
      # example: bucket = "s3://ml-bucket-123"
      prefix = "yolov5/custom-endpoint"
      model_data = s3.S3Uploader.upload("model.tar.gz", bucket + "/" + prefix)
      ```
        
3. After that, we deploy the model to the endpoint named *yolov5.* Here, I choose *ml.m5.xlarge* for the instance will be used for deploying*.* It will take about 5-10’ to complete.
        
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

        
![](../../images/saved/014-runpredict.png)
  
  When the deploying complete, open Amazon SageMaker > Endpoints and you will see the endpoints we created with above lines of code.
  
  ![](../../images/saved/018.png)
        