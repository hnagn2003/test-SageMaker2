---
title : "Tạo kết nối đến máy chủ EC2 Private"
date :  "`r Sys.Date()`" 
weight : 2 
chapter : false
pre : " <b> 3.2. </b> "
---
1. Ta phải đảm bảo rằng endpoint đã hoạt động bằng cách gửi cho nó một tấm ảnh trắng và bảo nó chạy suy luận.

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


2. Nếu ra kết quả như thế này thì endpoint của bạn đã làm việc đúng cách.

![](../../images/saved/017-testendpoint.png)
