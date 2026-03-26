---
title: "What's New in AWS: 2025 Highlights"
date: 2025-11-15
author: "Alice Nguyen"
description: "A deep dive into the most impactful AWS launches of 2025 — from new compute instances to serverless improvements and AI/ML services."
tags:
  - aws
  - reinvent
  - serverless
  - compute
---

AWS continues to ship at a relentless pace. In this post, we'll walk through the launches that matter most to builders shipping production workloads.

## Graviton4-Powered Instances

The new Graviton4 instances deliver up to **40% better performance** over Graviton3 for compute-intensive workloads. The R8g and M8g families are now generally available.

```bash
aws ec2 run-instances \
  --instance-type m8g.xlarge \
  --image-id ami-0abcdef1234567890 \
  --count 1
```

In our benchmarks, migrating from x86 M6i to Graviton4 M8g resulted in a **30% cost reduction** with equal or better performance.

## Lambda Improvements

Lambda now supports up to **10 GB of ephemeral storage** and **SnapStart for Python and Node.js** runtimes, not just Java. Cold starts for SnapStart-enabled functions dropped by **80%** in our tests.

```python
# handler.py — takes advantage of SnapStart pre-initialization
import boto3

# Heavy clients initialized at snapshot time
dynamodb = boto3.resource('dynamodb')
table = dynamodb.Table('Orders')

def handler(event, context):
    response = table.get_item(Key={'orderId': event['orderId']})
    return response['Item']
```

## Amazon Bedrock Enhancements

Amazon Bedrock added support for custom model imports, guardrails for responsible AI, and a new **Agents for Bedrock** capability that lets you build multi-step AI workflows without orchestration code.

```python
import boto3

bedrock = boto3.client('bedrock-runtime')

response = bedrock.invoke_model(
    modelId='anthropic.claude-3-sonnet-20240229-v1:0',
    body='{"prompt": "Summarize this document...", "max_tokens": 500}'
)
```

## Summary

AWS in 2025 focused on performance (Graviton4), developer experience (SnapStart expansion), and AI-native capabilities (Bedrock Agents). If you're running production on AWS, these updates are worth evaluating immediately.
