---
title : "Introduction"
date :  "`r Sys.Date()`" 
weight : 1 
chapter : false
pre : " <b> 1. </b> "
---
**Amazon SageMaker** is a full managed service for building, testing and deploying your machine learning model.
 - Enable more people to innovate with ML through a choice of tools—IDEs for data scientists and no-code interface for business analysts.
 - Access, label, and process large amounts of structured data (tabular data) and unstructured data (photo, video, geospatial, and audio) for ML.
 - Reduce training time from hours to minutes with optimized infrastructure. Boost team productivity up to 10 times with purpose-built tools.
 - Automate and standardize MLOps practices and governance across your organization to support transparency and auditability.

For complex deep learning machine models with large weights, when we need to run inference for those models, **Amazon SageMaker endpoints** offer a scalable and cost-optimized solution for deploying models.

**YOLOv5**, provided by **Ultralytics**, is the most popular open-source deep learning model for object detection tasks.

In this lab, we will deploy pretrained YOLOv5 model using SakeMaker endpoint and invoke the endpoint using Lambda. The SageMaker notebook call the model from S3 Bucket.

Uploading an image to the S3 will trigger Lambda function. OpenCV will go with Lambda layers for running the inference and produce the object detection results.

![](../images/workshop-cicd.png)

