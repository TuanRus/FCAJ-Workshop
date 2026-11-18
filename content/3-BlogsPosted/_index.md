---
title: "Blogs Posted"
date: 2024-01-01
weight: 3
chapter: false
pre: " <b> 3. </b> "
---

### [Overview of modernizing a .NET monolith on AWS](3.1-Blog1/)

Blog 1 is a summary from the AWS Architecture Blog that shares a path for moving a .NET Framework monolith to AWS in 5 phases (assess, stabilize and replatform, modernize .NET, modernize the database, re-architect). It is checked against current AWS documentation and was posted in the AWS Study Group (FCAJ) Facebook group.

### [Securing a legacy .NET application on AWS](3.2-Blog2/)

Blog 2 continues the target architecture from Blog 1 and covers how to secure a legacy .NET application after moving it to AWS, in 4 pillars: secrets management (Secrets Manager and rotation), IAM roles (task execution role and task role), authentication (Cognito for users, domainless gMSA with Active Directory for the workload), and network and data security (WAF, security groups, TLS, KMS).

### [Cost optimization for a .NET application on AWS](3.3-Blog3/)

Blog 3 completes the series with cost optimization for the .NET application after modernization, in 4 groups: cost visibility (tags, Cost Explorer, Budgets, Cost Anomaly Detection), compute (right-sizing, Graviton, Savings Plans, Fargate Spot), database (SQL Server edition, Developer Edition for dev/test, Reserved Instances), and network and logs (NAT gateway, VPC endpoints, log retention). It ends with the OrderServ case study, which cut compute cost by up to 64%.
