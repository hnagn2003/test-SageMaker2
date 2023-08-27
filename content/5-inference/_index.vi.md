---
title : "Chạy suy luận"
date :  "`r Sys.Date()`" 
weight : 5 
chapter : false
pre : " <b> 5. </b> "
---

Sau khi đã cài đặt xong mọi thứ, ta điểm thử đầu ra của việc gọi Lambda function bằng cách tải lên S3 một bức ảnh như 1 trigger để kích hoạt Lambda và nhận được kết quả suy luận từ mô hình.

1. Truy xuất các thư viện cần thiết
    
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
    
2. Tải lên một ảnh bất kì nào đó bạn muốn
    
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

    ![](../images/runinference/003.png)

    3. Kết quả như sao:
    
    ```jsx
    plt.figure(figsize=(10,10))
    plt.imshow(img)
    plt.axis('off')
    ```
    
    ![](../images/runinference/004.png)