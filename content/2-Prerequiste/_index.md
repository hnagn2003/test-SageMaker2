---
title : "Preparation "
date : "`r Sys.Date()`"
weight : 2
chapter : false
pre : " <b> 2. </b> "
---
First we need prepare the environment for inferences task.
We have a SageMaker notebook for deploying the endpoints, run the code, intergrate the sevices and many other tasks,...
We also need a S3 bucket to receive input image. It also play as the storage of our pretrained model too.

### Content
  - [Create SageMaker notebook](2.1-createsagemakernotebook/)
  - [Create S3 bucket](2.2-creates3bucket/)
  - [Modify IAM role](2.3-modifyiamrole/)