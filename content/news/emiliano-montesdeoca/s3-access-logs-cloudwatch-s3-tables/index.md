---
title: "Faster S3 access log queries make storage security more usable"
date: 2026-06-29
author: "Emiliano Montesdeoca"
description: "AWS shows how CloudWatch and S3 Tables can make S3 access logs easier to query, which helps builders turn storage audit data into operational and security signals."
tags:
  - aws
  - s3
  - cloudwatch
  - security
  - observability
source: "https://aws.amazon.com/blogs/storage/query-amazon-s3-access-logs-instantly-with-cloudwatch-and-s3-tables/"
draft: false
---

S3 access logs are valuable, but value delayed often becomes value ignored. If logs are difficult to query, teams use them only after something has already gone wrong.

The AWS Storage Blog post on [querying Amazon S3 access logs instantly with CloudWatch and S3 Tables](https://aws.amazon.com/blogs/storage/query-amazon-s3-access-logs-instantly-with-cloudwatch-and-s3-tables/) is interesting because it focuses on making access data easier to use in everyday operations.

## What changed

The article walks through delivering S3 access log data to CloudWatch Logs and S3 Tables, then using those destinations for dashboards, alarms, and queries. The practical shift is from "logs exist somewhere" to "logs are queryable enough to support investigation and monitoring."

For builders, that means S3 access patterns can become part of normal observability instead of a separate forensic workflow.

## Why it matters

S3 is often where the most important data lives. Application logs, analytics exports, customer files, model artifacts, backups, and operational reports all end up in buckets. Yet many teams still monitor bucket configuration more carefully than bucket access behavior.

Queryable access logs help answer questions that matter:

- Which principals are reading sensitive prefixes?
- Did access spike after a deployment?
- Are clients getting unexpected 403 or 404 responses?
- Which workloads are driving request cost?
- Did a suspicious IP enumerate objects?
- Are lifecycle or replication assumptions visible in traffic?

When these questions are cheap to answer, teams ask them earlier.

## The trade-offs

More logging is not free. S3 access log delivery, CloudWatch ingestion, query volume, table storage, retention, and dashboards all have cost implications. The right design depends on the bucket's risk and traffic profile.

I would not send every low-value development bucket into a high-retention analytics pipeline. I would prioritize buckets that contain customer data, security logs, production artifacts, financial exports, backups, or AI training and retrieval data.

Also, access logs are only one layer. They should complement CloudTrail data events, IAM Access Analyzer, Macie, GuardDuty, S3 Storage Lens, and application-level audit logs. Different signals answer different questions.

## What to do next

Pick a production bucket that matters and define three operational queries before building anything. For example:

1. top readers by prefix over the last hour,
2. denied requests by principal and source IP,
3. unusual data transfer volume compared with baseline.

Then build the logging path and dashboard around those questions. Add alarms only where the signal is actionable; noisy storage alarms quickly become invisible.

The main takeaway is that storage observability should be designed for regular use, not just incident response. Faster S3 access log queries make that much more realistic.
