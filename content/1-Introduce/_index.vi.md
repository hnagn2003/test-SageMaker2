---
title : "Giới thiệu"
date :  "`r Sys.Date()`" 
weight : 1 
chapter : false
pre : " <b> 1. </b> "
---


**Amazon SageMaker** là một dịch vụ quản lý đầy đủ cho việc xây dựng, thử nghiệm và triển khai mô hình học máy, học sâu.

- Cho phép cùng lúc nhiều người tham gia xây dựng mô hình học máy, thông qua các công cụ, môi trường phát triển tích hợp.

- Truy cập, gán nhãn và xử lý lượng lớn dữ liệu có cấu trúc (dữ liệu bảng) và dữ liệu không có cấu trúc (ảnh, video, địa lý và âm thanh) để sử dụng cho học máy.

- Giảm thời gian huấn luyện từ vài giờ xuống còn vài phút với cơ sở hạ tầng tối ưu hóa. Tăng năng suất của đội ngũ lên tới 10 lần với các công cụ được xây dựng đặc biệt.

- Tự động hóa và chuẩn hóa các ứng dụng MLOps và quản trị trong doanh nghiệp để hỗ trợ tính minh bạch và khả năng kiểm toán.
Đối với các mô hình học sâu phức tạp với trọng số lớn, khi chúng ta cần chạy suy luận cho những mô hình đó, Amazon SageMaker endpoints cung cấp một giải pháp triển khai mô hình có khả năng mở rộng và tối ưu chi phí.

**YOLOv5** của Ultralytics là mô hình học sâu mã nguồn mở phổ biến nhất cho các bài toán nhận dạng vật thể.

Trong bài thực hành này, chúng ta sẽ triển khai mô hình YOLOv5 được huấn luyện trước bằng cách sử dụng SakeMaker endpoint và gọi điểm cuối bằng Lambda. SageMaker sẽ lấy mô hình từ nơi chứa dữ liệu Amazon S3 bucket.

Việc tải lên một hình ảnh lên S3 sẽ kích hoạt Lambda function. OpenCV được tích hợp trên Lambda layers để sẽ thực hiện việc suy luận và tính toán kết quả phát hiện vật thể.

![](images/workshop-cicd.png)
