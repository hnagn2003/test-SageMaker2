+++
title = "Dọn dẹp tài nguyên  "
date = 2021
weight = 6
chapter = false
pre = "<b>6. </b>"
+++

Chúng ta sẽ tiến hành các bước sau để xóa các tài nguyên chúng ta đã tạo trong bài thực hành này.

#### Xóa EC2 instance

1. Đi tới [Lambda management console](https://ap-southeast-2.console.aws.amazon.com/lambda/home?region=ap-southeast-2#/discover)
   + Click **Founction**.
   + Tick **yolov5-lambda**
   + Click **Action**.
   + Click **Delete function**.

2. Lambda > Layers
   + Click **cv2**.
   + Click **Delete**.

#### Xóa S3 bucket

1. Truy cập [System Manager - Session Manager service management console](https://console.aws.amazon.com/systems-manager/session-manager).
   + Click the **Preferences** tab.
   + Click **Edit**.
   + Lăn xuống
   + Trong mục **S3 logging**.
   + Bỏ tick **Enable** để ngừng ghi hoạt động.
   + Lăn xuống
   + Click **Save**.

2. Đi tới [S3 service management console](https://s3.console.aws.amazon.com/s3/home)
   + Click vào S3 bucket ta đã tạo trong lab này. (Example: **ml-bucket-123** )
   + Click **Empty**.
   + Nhập **permanently delete**, và click **Empty** để xóa các vật thể trong bucket.
   + Click **Exit**.

3. Sau khi xóa hết vật thể, click **Delete**

4. Nhập tên bucket, sau đó click **Delete bucket** để xóa bucket.

#### Delete SageMaker notebook

1. Tới [Amazon SageMaker management console](https://ap-southeast-2.console.aws.amazon.com/sagemaker/home?region=ap-southeast-2#/landing)
+ Trong tabs bên trái, chọn **Notebook instances**
+ Tick notebook ta đã tạo. (Example: *ml-inference* )
+ Click **Actions**
+ Chọn **Stop** và đợi 1 lúc
+ Sau khi notebook hoàn toàn dừng lại, click **Actions**, chọn **Delete**

