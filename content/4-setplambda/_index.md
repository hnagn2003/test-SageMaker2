---
title : "Setup Lambda"
date : "`r Sys.Date()`"
weight : 4
chapter : false
pre : " <b> 4. </b> "
---

# Set up **Lambda**

**Lambda** doesn’t come with external libraries like OpenCV pre-built, therefore we need to build it before we can invoke the Lambda code.

   - [Create **Lambda layers** using **Docker**](4.1-createlambdalayer/)
   - [Create the **Lambda function** and attach the OpenCV layer](4.2-createslambdafunction/)
   - [Activate trigger Lambda for input image](4.3-activatetrigger/)