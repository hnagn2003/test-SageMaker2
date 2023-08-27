---
title : "Deploy YOLOv5 on SageMaker endpoint"
date : "`r Sys.Date()`"
weight : 3
chapter : false
pre : " <b> 3. </b> "
---

After initializing the environment, we need to create an endpoint so the Lambda layer can read the upload image and run the inference on it.

### Content:
  - [Forward model weight to endpoint](3.1-hostmodel/)
  - [Test the endpoint](3.2-testendpoint/)