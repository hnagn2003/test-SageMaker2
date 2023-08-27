---
title : "Kích hoạt trigger Lambda khi có ảnh đầu vào"
date :  "`r Sys.Date()`" 
weight : 3
chapter : false
pre : " <b> 4.3 </b> "
---

0. **Mở Lambda function bạn đã tạo và tạo trigger**

![](../../images/lambda/trigger1.png)
![](../../images/lambda/trigger2.png)

Trigger này sẽ phản hồi với sự kiện đến, cụ thể là 1 ảnh được tải lên S3

![](../../images/lambda/trigger3.png)


1. Mở [Buckets](https://console.aws.amazon.com/s3/buckets), điều hướng đến Amazon S3 console và chọn bucket lúc trước ta tạo.
    
    ![](../../images/lambda/trigger5.png)


    
2. Mở thư mục **Yolov5/**
    
    ![](../../images/lambda/trigger6.png)

    
3. Chọn **Upload**, chọn **Add file** tải lên 1 tệp nho nhỏ nào đó.
    
    ![](../../images/lambda/trigger7.png)
    
    Chọn **Upload**
    
    ![](../../images/lambda/trigger8.png)

    
4. Để kiểm tra xem trigger có hoạt động không, ta dùng CloudWatch Logs
    
    - Mở [CloudWatch](https://console.aws.amazon.com/cloudwatch/home). Đảm bảo đúng Region.
    
    - Chọn **Logs**, sau đó chọn **Log groups**.
    - Chọn log group của function của ta
        
        ![](../../images/lambda/trigger10.png)
        
    - Dưới **Log streams**, Chọn log stream gần nhất
        
        ![](../../images/lambda/trigger9.png)
        
    - Nếu hàm của bạn được gọi để phản hồi với trigger của Amazon S3, bạn sẽ nhìn thấy tệp `001.png` tôi đã tải lên.
        
        ![](../../images/lambda/trigger11.png)
        
