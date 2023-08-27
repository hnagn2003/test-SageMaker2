---
title : "Create Lambda layers using Docker"
date : "`r Sys.Date()`"
weight : 1
chapter : false
pre : " <b> 4.1 </b> "
---


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
    
    ![](../../images/lambda/006.png)
    ![](../../images/lambda/007.png)
    ![](../../images/lambda/008.png)
    
2. Pull and build the docker image
    
    ```jsx
    !docker build --tag aws-lambda-layers:latest Docker
    ```
    
    This progess may take some time to finish.
    
    ![](../../images/lambda/buildimage.png)
    
3. Then run Lambda layers container
    
    ```jsx
    !docker run --rm -it -v $(pwd):/layers aws-lambda-layers cp /packages/cv2-python37.zip /layers
    ```
    
4. Upload OpenCV modules to S3
    
    ```jsx
    !aws s3 cp cv2-python37.zip s3://ml-bucket-123/yolov5/artifacts/cv2-python37.zip
    ```
    