---
title: "Proposal"
date: 2026-11-23
weight: 2
chapter: false
pre: " <b> 2. </b> "
---

# RoomLedger

## A Serverless AWS Platform for Boarding-House Management with Tamper-Proof Photo Records and AI Meter Reading

### 1. Executive Summary

In large Vietnamese cities, disputes between boarding-house landlords and tenants usually come down to two things: the deposit at move-out (was the room damaged?) and the monthly electricity and water bill (what did the meter really show?). Both sides often have only verbal agreements or a few photos sent over chat. RoomLedger is a serverless web application on AWS, deployed in ap-southeast-2 (Sydney), with three roles: administrator, landlord and tenant, sized for 31–51 users (one boarding house with 30–50 rooms). It stands out with two features: move-in photo records that are locked with Amazon S3 Object Lock once both sides confirm, and Amazon Bedrock (Claude Haiku 4.5) reading electricity and water meter photos to produce the monthly bill. Paper contracts are scanned, confirmed by the tenant and stored.

### 2. Problem Statement

### What's the Problem?

Move-in records are often missing, or they are just a few scattered photos on each person's phone, so at move-out the two sides have no shared evidence. Photos can be claimed to be taken later, taken in another room, or edited. Each month the landlord reads the meters alone, and tenants who never see the meter tend to doubt the numbers. Existing Vietnamese boarding-house apps (LOZIDO, Quản Lý Phòng Trọ Việt Nam) handle invoices, contracts and payment reminders well, but have no photo-based handover record and no AI. Foreign apps (Witnessed, PropCheckAI, DepositCam) record handover photos, but they do not manage boarding houses and do not require confirmation from both sides.

### The Solution

RoomLedger puts locked, two-party-confirmed photo records, AI meter reading and a move-out review into one flow together with contracts, monthly invoices and deposit settlement. The Angular web app is served from Amazon S3 through Amazon CloudFront, users sign in with Amazon Cognito, and API calls go through Amazon API Gateway protected by AWS WAF to AWS Lambda (.NET 8) running in private subnets of an Amazon VPC. Data is stored in Amazon RDS for PostgreSQL (Multi-AZ). The API Lambda writes business events to an S3 outbox bucket, and S3 forwards them to Amazon EventBridge. Photos are uploaded directly from the browser to Amazon S3 with a presigned URL and a SHA-256 checksum, flow through Amazon EventBridge and Amazon SQS to the photo-processing Lambda, and are copied to an evidence bucket with Object Lock when the record is locked. For meter photos, AWS Step Functions calls Amazon Bedrock to read the value; the landlord confirms it and the app issues the invoice. At move-out the landlord compares move-in and move-out photos of the same area side by side and enters deductions; the tenant accepts or disputes each item. AI does not compare photos.

### Benefits and Return on Investment

Landlord and tenant share one move-in record that both have confirmed and nobody can change, including the administrator. Every meter reading has a meter photo behind it, and the paper contract has a scan confirmed by both sides. Benefits are measured on demo data: time to record utilities for 10 rooms (manual versus photo + AI) and AI meter-reading accuracy. Production costs about 80.62 USD/month, roughly 1.6–2.7 USD per room per month. Almost all of it is fixed hourly cost (RDS Multi-AZ, the Cognito Interface endpoint, WAF), so growing from 31 to 51 users barely changes the bill. There is no NAT Gateway or load balancer, S3 traffic uses a free Gateway endpoint, and AI is only used for meter reading with a low-cost model. The app gives both sides a shared basis for negotiation; the project does not claim the record has legal standing in court.

### 3. Solution Architecture

RoomLedger uses a serverless architecture on AWS with a single production environment in ap-southeast-2 (Sydney). The API, photo-processing, post-confirmation and migrate Lambdas run in private subnets across two Availability Zones with no NAT Gateway and no load balancer. They connect to RDS over TLS with RDS IAM authentication and reach S3 through a Gateway endpoint. Instead of calling EventBridge directly, the API Lambda writes events to an S3 outbox bucket through the Gateway endpoint, and the outbox bucket sends them to EventBridge, so no EventBridge Interface endpoint is needed. The only Interface endpoint is Cognito (one per AZ), used by the post-confirmation Lambda to add users to groups. The AI and notification Lambdas run outside the VPC, so no Bedrock or SES endpoint is needed. All infrastructure is written in AWS CDK (C#) and deployed from GitHub to AWS CloudFormation through OIDC.

![RoomLedger Architecture](/images/2-Proposal/roomledger-architecture.png)

### AWS Services Used

- **Amazon CloudFront**: Serves the Angular web app from S3 and forwards API requests to API Gateway.
- **AWS WAF**: Regional Web ACL on API Gateway that blocks abnormal traffic and rate-limits requests, including requests that bypass CloudFront.
- **Amazon Cognito**: Two user pools, one for landlords and tenants, one for administrators with mandatory MFA.
- **Amazon API Gateway (REST API)**: API entry point with a Cognito authorizer.
- **AWS Lambda (.NET 8)**: API, photo processing, post-confirmation and migrate inside the VPC; AI meter reading and notifications outside the VPC.
- **Amazon EventBridge**: Receives events from the S3 outbox and incoming buckets and routes them (new photo, record locked, meter reading, settlement, notification).
- **Amazon SQS**: Photo queue in front of the photo-processing Lambda plus a dead-letter queue, absorbing bursts without losing events.
- **AWS Step Functions**: Orchestrates meter reading: calls the AI Lambda, retries on failure, then passes the result to the photo-processing Lambda, which saves it to RDS for the landlord to approve.
- **Amazon Bedrock (Claude Haiku 4.5)**: Reads electricity and water meter values from photos; the only place AI is used.
- **Amazon S3 and S3 Glacier Deep Archive**: Four buckets: incoming (deleted after 7 days), outbox (events, deleted after 7 days), derived (previews, PDFs) and evidence (Object Lock, encrypted with KMS); old evidence moves to Glacier Deep Archive.
- **Amazon RDS for PostgreSQL**: db.t4g.micro Multi-AZ, 20 GB gp2, encrypted, SSL enforced, IAM authentication.
- **Amazon VPC**: Private subnets in two AZs, Gateway endpoint for S3, Interface endpoint for Cognito in each AZ; no NAT Gateway.
- **AWS Systems Manager Parameter Store**: Model ID and prompt for the AI Lambda, changeable without redeploying.
- **AWS IAM, AWS KMS and AWS Secrets Manager**: Least-privilege role per Lambda; customer managed key for the evidence bucket; Secrets Manager only holds the RDS master password created by CDK, while Lambdas sign in with RDS IAM authentication.
- **Amazon GuardDuty and AWS CloudTrail**: Threat detection and audit trail.
- **Amazon SES**: Sends invite links, invoices and notifications.
- **Amazon CloudWatch, Amazon SNS and AWS Budgets**: Logs, dashboard, alarms emailed through SNS, and cost alerts.
- **AWS CloudFormation (AWS CDK) and GitHub Actions (OIDC)**: Infrastructure as code, deployed without stored AWS keys.

### Component Design

- **Web Application**: Angular UI for three roles; computes the SHA-256 of the original file before uploading to S3.
- **Contracts and Move-in Records**: Contract scans and condition photos are locked once both sides confirm; nobody can edit or delete them.
- **Utilities and Invoices**: AI reads meter values from photos and flags uncertain readings or readings lower than last month; the landlord confirms and sends the invoice.
- **Move-out**: Move-in and move-out photos of the same area are shown side by side; the landlord enters deductions, the tenant accepts or disputes each item, then the app settles the deposit and produces a termination PDF.
- **Access Control and Audit Log**: Each landlord sees only their own data and each tenant only their room; every action and every denied access is logged.
- **Events**: the API Lambda writes events to the S3 outbox (outbox pattern); EventBridge routes them to SQS, Step Functions and the notification Lambda.
- **Data**: EF Core code-first on PostgreSQL; idempotent migrations run through the migrate Lambda only at deploy time.

### 4. Technical Implementation

**Implementation Phases**

The project is built in four phases during the five implementation weeks:

- Foundation: CDK infrastructure (two-AZ VPC and endpoints, RDS, four S3 buckets, two Cognito user pools, API), CI/CD with OIDC, Angular skeleton for three roles, data model and access control. (Week 8)
- Contracts, Move-in and Invoices: properties, rooms, invite links, contract scans, photo records and locking, AI meter reading and monthly invoices. (Weeks 9–10)
- Move-out: side-by-side photo review, deductions, disputes, deposit settlement, termination PDF. (Week 11)
- Finalization: security tests, AI meter-reading evaluation, demo and workshop. (Week 12)

**Technical Requirements**

- Frontend: Angular, Angular Material and WebCrypto; desktop UI for landlords and administrators, mobile UI for tenants.
- Backend: .NET 8 on Lambda: ASP.NET Core and EF Core (Npgsql) for the API; separate Lambdas for photo processing (Magick.NET for HEIC to JPEG, EXIF), Bedrock calls through the AWS SDK for .NET, and PDF generation with QuestPDF.
- Infrastructure: AWS CDK (C#), GitHub Actions with OIDC, one team AWS account in ap-southeast-2 with AWS Budgets alerts.
- Dataset: move-in and move-out photos of two real rooms, about 30 meter photos (mechanical, digital, water) with known values, and a few sample contract scans.

### 5. Timeline & Milestones

**Project Timeline**

- Weeks 1–5: Learn AWS (core services, serverless, security, IaC) and complete the workshops.
- Weeks 6–7: Learn while shaping the idea: research the problem, compare existing products, design the architecture and UI, write the proposal.
- Weeks 8–12: Build the project:
  - Week 8: Foundation; original photos uploaded to S3 with checksum verification, all three roles can sign in.
  - Weeks 9–10: Contracts, move-in and invoices work end to end.
  - Week 11: Move-out and deposit settlement.
  - Week 12: Security tests, demo and final report.

### 6. Budget Estimation

The estimate was made with the AWS Pricing Calculator for Asia Pacific (Sydney), production running 24/7 across 2 AZs, 31–51 active users per month, Before Discount rates. Removing the EventBridge and Secrets Manager Interface endpoints (replaced by the S3 outbox and RDS IAM authentication) lowered the estimate from 99.60 to 80.62 USD/month, a saving of about 19 USD/month.

### Infrastructure Costs

- AWS Services:
  - Amazon RDS for PostgreSQL: $42.75/month (db.t4g.micro Multi-AZ 730 hours $37.23, 20 GB gp2 Multi-AZ storage $5.52)
  - Amazon VPC: $18.98/month (Cognito Interface endpoint in 2 AZs, 1,460 hours; 0.1 GB processed)
  - AWS WAF: $10.01/month (1 Web ACL $5.00, 5 rules $5.00, 20,000 requests)
  - Amazon CloudWatch: $4.64/month (1 dashboard $3.00, 15 alarms $1.50, about 0.2 GB of logs)
  - Amazon Bedrock (Claude Haiku 4.5): $1.49/month (input tokens $1.22, output tokens $0.27)
  - AWS KMS: $1.00/month (1 customer managed key, 400 requests)
  - AWS Secrets Manager: $0.80/month (2 secrets including the RDS master password, 200 API requests)
  - Amazon SQS: $0.26/month (650,000 requests, mostly Lambda long-polling the photo queue)
  - Amazon S3: $0.16/month (about 8 GB, evidence bucket on Intelligent-Tiering)
  - Amazon CloudFront: $0.14/month (1 GB data transfer, 20,000 HTTPS requests)
  - Amazon GuardDuty: $0.14/month (6,000 events, 0.1 GB of logs)
  - Data Transfer: $0.14/month (1.2 GB out to the internet)
  - AWS Lambda (ARM), API Gateway, SES, CloudTrail, EventBridge, SNS: $0.10/month (11,000 API requests, 100 emails)
  - Amazon Cognito: $0.00/month (31–51 MAU within the free tier)

Total: $80.62/month, $967.44/12 months

### 7. Risk Assessment

#### Risk Matrix

- Tenant uploads old photos or photos of another room at move-in: High impact, medium probability.
- AI misreads a meter: Medium impact, medium probability.
- Access-control bug between landlords: High impact, low probability.
- iPhone HEIC photos cannot be sent to Bedrock directly: Medium impact, high probability.
- AI model fails, is throttled or returns a wrong format: Medium impact, medium probability.
- Contract scan differs from the tenant's paper copy: High impact, low probability.

#### Mitigation Strategies

- Old or wrong photos: warn on EXIF capture date; the record is valid only after both sides confirm.
- Meter readings: always confirmed by a person; uncertain readings or readings lower than last month are flagged.
- Access control: checked per record in the API; covered by access-control tests.
- HEIC: the photo-processing Lambda converts to JPEG for analysis and keeps the original.
- AI model: JSON output against a schema, a fallback model, and model switching through Parameter Store without redeploying.
- Contract scan: the tenant must confirm before locking and can press "Not correct".

#### Contingency Plans

- If Bedrock is unstable for the demo, meter reading runs on a fixed demo photo set and the landlord enters values manually when AI cannot read them.
- If costs rise unexpectedly, AWS Budgets alerts the team to review and limit Bedrock calls.

### 8. Expected Outcomes

#### Technical Improvements:

Move-in photo records locked with S3 Object Lock and confirmed by both sides, with security tests for evidence locking, access control and WAF.
AI meter reading replaces manual recording, with a report on accuracy and time saved.

#### Long-term Value

A reusable pattern for tamper-proof photo evidence on AWS (checksum upload, Object Lock, two-party confirmation, audit log), fully defined in AWS CDK.
Future work: bank transfer reconciliation, photo incident reports during the tenancy, Zalo notifications and a mobile app.
