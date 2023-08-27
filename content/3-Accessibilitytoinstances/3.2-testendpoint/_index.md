---
title : "Test the endpoint"
date : "`r Sys.Date()`"
weight : 2
chapter : false
pre : " <b> 3.2. </b> "
---
# Test the endpoint

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


2. If you got this output then your endpoint run properly.

![](images/saved/017-testendpoint.png)
