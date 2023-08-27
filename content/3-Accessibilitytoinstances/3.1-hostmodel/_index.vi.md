---
title : "Đưa mô hình vào endpoint"
date :  "`r Sys.Date()`" 
weight : 1 
chapter : false
pre : " <b> 3.1. </b> "
---


1. Quay lại với Jupiter notebook,
- Nén tệp trọng số  thành *model.tar.gz*
    
      mkdir export && mkdir export/Servo
      
      mv yolov5l_saved_model export/Servo/1
      
      !tar -czvf model.tar.gz export/
        
2. Sau đó đưa mô hình lên S3 Bucket. Trường“bucket” là tên bucket mà bạn đã tạo lúc trước ở phần 1.2.
        
      ```jsx
      from sagemaker import s3
      
      bucket = "<s3://BUCKET/NAME>"
      # example: bucket = "s3://ml-bucket-123"
      prefix = "yolov5/custom-endpoint"
      model_data = s3.S3Uploader.upload("model.tar.gz", bucket + "/" + prefix)
      ```
        
3. Sau đó ta chạy đoạn mã triển khai mô hình *yolov5.* trên endpoint. Ở đây, tôi chọn instance *ml.m5.xlarge*. Đợi 4-5 phút cho hoàn tất.
        
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
  
  Khi triển khai xong, mở Amazon SageMaker > Endpoints bạn sẽ thấy endpoint ta đã tạo.
  
  ![](../../images/saved/018.png)
        