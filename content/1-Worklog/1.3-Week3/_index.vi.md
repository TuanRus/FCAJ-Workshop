---
title: "Worklog Tuần 3"
date: 2026-11-03
weight: 3
chapter: false
pre: " <b> 1.3. </b> "
---

### Mục tiêu tuần 3:

* Hiểu các khái niệm về mạng trên AWS: VPC, Security Group, NACL, Internet Gateway và NAT Gateway
* Thực hành xây dựng kiến trúc mạng nhiều tầng (multi-tier)

### Các công việc đã triển khai trong tuần:
| Thứ | Công việc | Ngày bắt đầu | Ngày hoàn thành | Nguồn tài liệu |
| --- | --------- | ------------ | --------------- | -------------- |
| 2   | - Học khái niệm VPC và cách thiết lập <br>&emsp; + Dải CIDR, subnet public/private, route table | 28/09/2026 | 28/09/2026 | <https://000003.awsfcaj.com/> |
| 3   | - Tham dự sự kiện **AWS Cloud Day And AI Day Hanoi - Watch party** | 29/09/2026 | 29/09/2026 | |
| 4   | - Học về Security Group và NACL, so sánh sự khác nhau <br>- Học về Internet Gateway và NAT Gateway | 30/09/2026 | 30/09/2026 | <https://000003.awsfcaj.com/><br><https://000092.awsfcaj.com/> |
| 5   | - **Thực hành: xây dựng kiến trúc mạng nhiều tầng** <br>&emsp; + Bước 1: tạo VPC, subnet public và subnet private <br>&emsp; + Bước 2: gắn Internet Gateway và cấu hình route table cho subnet public <br>&emsp; + Bước 3: tạo NAT Gateway để subnet private ra được Internet <br>&emsp; + Bước 4: tạo Security Group cho từng tầng (web, ứng dụng) và kiểm tra kết nối giữa các tầng <br>&emsp; + Bước 5: dọn dẹp tài nguyên sau lab | 01/10/2026 | 01/10/2026 | <https://000092.awsfcaj.com/> |
| 6   | - Tham dự sự kiện **Fireside Chat with Dr. Werner Vogels** | 02/10/2026 | 02/10/2026 | |

### Kết quả đạt được tuần 3:

* Hiểu cách thiết kế mạng trên AWS: VPC, subnet public/private, route table
* Phân biệt được Security Group (cấp instance, stateful) và NACL (cấp subnet, stateless)
* Biết vai trò của Internet Gateway và NAT Gateway
* Đã dựng được kiến trúc mạng nhiều tầng bằng VPC

### Khó khăn & cách giải quyết:

* Tuy tiếp cận được nhiều studycase của các công ty lớn nhưng vì thời gian quá ngắn nên không hiểu hết được

### Kế hoạch tuần tới:

* Tìm hiểu về IAM: user, group, role, policy và cài đặt MFA theo [roadmap học AWS với Free Tier](https://000001.awsfcaj.com/vi/9-roadmap-h%E1%BB%8Dc-aws-v%E1%BB%9Bi-free-tier/)
