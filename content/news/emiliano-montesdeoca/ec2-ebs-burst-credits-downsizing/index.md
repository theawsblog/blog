---
title: "Before downsizing EC2, simulate the EBS burst budget"
date: 2026-06-17
author: "Emiliano Montesdeoca"
description: "AWS shows how to simulate EBS burst credits before downsizing EC2 instances, a practical cost-optimization step that avoids turning compute savings into storage throttling."
tags:
  - aws
  - ec2
  - ebs
  - cost-optimization
  - performance
source: "https://aws.amazon.com/blogs/compute/simulating-amazon-ec2-ebs-burst-credits-before-downsizing-an-instance/"
draft: false
---

Rightsizing EC2 usually starts with CPU and memory. That is necessary, but it is not enough.

The AWS Compute Blog post on [simulating Amazon EC2 EBS burst credits before downsizing an instance](https://aws.amazon.com/blogs/compute/simulating-amazon-ec2-ebs-burst-credits-before-downsizing-an-instance/) is a good reminder that storage performance can be the hidden reason a "safe" downsize becomes a production incident.

## What changed

The article walks through a simulation approach for burstable EBS-optimized instance performance. Instead of looking only at average utilization, it pulls EBS read and write metrics from CloudWatch, compares them with the baseline and burst ceiling of a target instance type, and simulates whether IOPS or throughput credits would run out after downsizing.

The important metrics include:

- EBS read and write bytes,
- EBS read and write operations,
- EBS I/O and byte balance percentages,
- instance EBS IOPS and throughput exceeded checks.

The outcome is more useful than a simple utilization chart: it tells you whether the target instance can absorb the actual I/O pattern without draining credits and throttling.

## Why builders should care

Cost optimization often fails when it optimizes one dimension in isolation. A smaller instance can look perfect from a CPU perspective but have lower EBS baseline performance. If the workload depends on burst credits during business hours, the downsize may save compute cost while increasing latency, queue depth, or database wait time.

This matters especially for databases, analytics workers, search nodes, and file-heavy applications. It also matters when downsizing reduces memory. Less memory can mean less cache, which can increase disk I/O after the move.

The practical lesson: do not approve a downsize until the storage path has been modeled.

## The trade-offs

The simulation described by AWS is intentionally conservative when using maximum statistics over five-minute periods. That is useful for safety, but it can overstate credit drain. If the conservative simulation passes, confidence is high. If it fails, the answer is not automatically "do nothing." It may mean you need higher-resolution data, a different target instance, gp3 tuning, or application-level changes.

Also, burst credits are not a performance strategy. They are a buffer. If normal business traffic depends on sustained bursting, the workload is under-provisioned for its real behavior.

## What to do next

For each EC2 rightsizing candidate, add a storage check to the decision process:

1. Pull at least two weeks of CloudWatch EBS metrics, four if the workload has monthly cycles.
2. Check whether the current instance already hits EBS exceeded metrics.
3. Compare peak and sustained I/O against the target instance baseline and burst ceiling.
4. Simulate credit balance for both IOPS and throughput.
5. Monitor `EBSByteBalance%`, `EBSIOBalance%`, and exceeded checks after the change.

This is the kind of small operational habit that prevents cost work from damaging reliability. A good rightsizing recommendation should say not only "CPU is low," but also "storage credits survive the target shape."
