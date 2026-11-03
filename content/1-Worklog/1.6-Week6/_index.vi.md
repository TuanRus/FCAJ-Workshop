---
title: "Worklog Tuần 6"
date: 2026-11-03
weight: 6
chapter: false
pre: " <b> 1.6. </b> "
---

### Mục tiêu tuần 6:

* Hiểu về tích hợp ứng dụng trên AWS: SQS, SNS, EventBridge và Step Functions
* Thực hành xây dựng kiến trúc hướng sự kiện (event-driven)

### Các công việc đã triển khai trong tuần:
| Thứ | Công việc | Ngày bắt đầu | Ngày hoàn thành | Nguồn tài liệu |
| --- | --------- | ------------ | --------------- | -------------- |
| 2   | - Học về nhắn tin với SQS và SNS | 19/10/2026 | 19/10/2026 | <https://000077.awsfcaj.com/> |
| 3   | - Học về định tuyến sự kiện với EventBridge | 20/10/2026 | 20/10/2026 | <https://000054.awsfcaj.com/> |
| 4   | - Học về workflow với Step Functions | 21/10/2026 | 21/10/2026 | <https://000047.awsfcaj.com/> |
| 5   | - **Thực hành: xây dựng kiến trúc hướng sự kiện** <br>&emsp; + Bước 1: tạo SNS topic và SQS queue, đăng ký queue vào topic <br>&emsp; + Bước 2: tạo EventBridge rule để định tuyến sự kiện <br>&emsp; + Bước 3: tạo Lambda xử lý tin nhắn từ SQS <br>&emsp; + Bước 4: gửi sự kiện thử và kiểm tra luồng xử lý <br>&emsp; + Bước 5: dọn dẹp tài nguyên sau lab | 22/10/2026 | 22/10/2026 | <https://000054.awsfcaj.com/><br><https://000083.awsfcaj.com/> |
| 6   | - Ôn tập kiến thức tích hợp ứng dụng, tiếp tục project | 23/10/2026 | 23/10/2026 | |

### Kết quả đạt được tuần 6:

* Phân biệt được SQS (hàng đợi) và SNS (publish/subscribe)
* Biết dùng EventBridge để định tuyến sự kiện và Step Functions để điều phối workflow
* Đã dựng được kiến trúc hướng sự kiện đơn giản

### Khó khăn & cách giải quyết:

* Đã có ý tưởng nhưng gặp khó khăn trong công cuộc hiện thực hoá: chi phí của project cao hơn chi phí trước khi giải quyết vấn đề

### Kế hoạch tuần tới:

* Tìm hiểu về Containers và Orchestration: Docker, ECS và ECR theo [roadmap học AWS với Free Tier](https://000001.awsfcaj.com/vi/9-roadmap-h%E1%BB%8Dc-aws-v%E1%BB%9Bi-free-tier/)
