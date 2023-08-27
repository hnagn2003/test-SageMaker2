---
title : "Create SageMaker notebook"
date : "`r Sys.Date()`"
weight : 1
chapter : false
pre : " <b> 2.1 </b> "
---

1. **Create a notebook instance**:
- Name: ```ml-inference```
- Type: ```ml.c5.xlarge```

  ![](../../static/images/saved/00.png)
  ![](../../static/images/saved/000.png)
  ![](../../static/images/saved/001.png)
  ![](../../static/images/saved/002.png)

- In Permissions and encryption field, we create a role for this SageMaker

  ![](../../static/images/saved/003.png)
  ![](../../static/images/saved/004.png)
  ![](../../static/images/saved/005.png)



2. After creating notebook instance for a while, the status will be shown **InService**, then click - - **Open JupyterLab** to open jupiter SageMaker notebooks
    
    ![](../../static/images/saved/006.png)

    
  - Choose anaconda python3 environment
    
    ![](../../static/images/saved/006.png)

    
**3. With jupiter notebook**
    
We choose Anaconda Python3 environment
    ![](../../static/images/saved/007.png)
{{% notice info %}}
In Jupiter Notebook, we write code in the cell and press *Shift+Enter* to execute it
{{% /notice %}}
    ![](../../static/images/saved/008.png)
    
4. **First, we need to clone YOLOv5 source code for reference task**
    
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
    
    ![](../../static/images/saved/009.png)
    
    - The model will be loaded successfully
        
        ![](../../static/images/saved/009model.png)
        
    - **From now all our code will run on this notebook.**