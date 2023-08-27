+++
title = "Clean up resources"
date = 2022
weight = 6
chapter = false
pre = "<b>6. </b>"
+++

We will take the following steps to delete the resources we created in this exercise.

#### AWS Lambda resouces

1. Go to [Lambda management console](https://ap-southeast-2.console.aws.amazon.com/lambda/home?region=ap-southeast-2#/discover)
   + Click **Founction**.
   + Tick **yolov5-lambda**
   + Click **Action**.
   + Click **Delete function**.

2. Lambda > Layers
   + Click **cv2**.
   + Click **Delete**.

#### Delete S3 bucket

1. Access [System Manager - Session Manager service management console](https://console.aws.amazon.com/systems-manager/session-manager).
   + Click the **Preferences** tab.
   + Click **Edit**.
   + Scroll down.
   + In the section **S3 logging**.
   + Uncheck **Enable** to disable logging.
   + Scroll down.
   + Click **Save**.

2. Go to [S3 service management console](https://s3.console.aws.amazon.com/s3/home)
   + Click on the S3 bucket we created for this lab. (Example: *ml-bucket-123* )
   + Click **Empty**.
   + Enter **permanently delete**, then click **Empty** to proceed to delete the object in the bucket.
   + Click **Exit**.

3. After deleting all objects in the bucket, click **Delete**

4. Enter the name of the S3 bucket, then click **Delete bucket** to proceed with deleting the S3 bucket.

#### Delete SageMaker notebook

1. Go to [Amazon SageMaker management console](https://ap-southeast-2.console.aws.amazon.com/sagemaker/home?region=ap-southeast-2#/landing)
+ In to left tab, choose **Notebook instances**
+ Tick notebook we created for this lab. (Example: *ml-inference* )
+ Click **Actions**
+ Choose **Stop** and wait for a while
+ After notebook stops completely, click **Actions**, choose **Delete**
