---
title: "Athena and Macie"
date: 2021-10-03T09:16:26+09:00
draft: false
author: Torres
tags: ["Athena", "Macie"]
categories: ["AWS"]
toc:
  enable: true
  auto: true
share:
  enable: true
comment:
  enable: true
featuredImage: "/images/posts/common/AWS_Logo.svg"
featuredImagePreview: "/images/posts/common/AWS_Logo.svg"
---

## Athena

### What is Athena

**Athena is a interactive query service which enables you to analyse and query data located in S3 using standard SQL.**

- Serverless, nothing to provision, pay per query / per TB scanned
- No need to set up complex Extract/Transform/Load(ETL) process
- Works directly with data stored in S3



### Athena Use Case

#### What can Athena be used for?

- Query log files stored in S3: e.g. ELB logs, S3 access logs etc
- Generate business reports on data stored in S3
- Analyse AWS cost nad usage reports
- Run queries on click-stream data

## Macie

### What is PII(Personally Identifiable Information)?

- Personal data used to establish an indevidual’s identity
- Could be exploited by criminals, used in identity theft and financial fraud
- Home address, emal address
- Passport number, driver’s license number
- …

### What is Macie

**Security service which uses Machine Learning and NLP to discover, classify and protect sensitive data stored in S3**

- Uses AI to recognise if your S3 objects contain sensitive data such as PII
- Dashboards, reporting and alerts
- Works directly with data stored in S3
- Can also analyze CloudTrail logs
- Great for PCI-DSS and preventing ID theft

### Macie Exam Tips

**Remember What is Macie used for:**

- Uses AI to analyze data in S3 and helps identify PII
- Analyse CloudTrail logs for suspicious API activity
- Dashboards, Reports and Alerting
- PCI-DSS compliance and preventing ID theft
