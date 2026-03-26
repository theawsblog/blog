---
title: "Building Event-Driven Architectures with AWS EventBridge and SQS"
date: 2025-10-08
author: "Marco Rossi"
description: "A practical walkthrough of designing and implementing event-driven microservices using EventBridge, SQS, and Lambda on AWS."
tags:
  - aws
  - microservices
  - eventbridge
  - sqs
  - architecture
---

Event-driven architecture gives you loose coupling, resilience, and the ability to scale services independently. But it also adds complexity. In this post, I'll show you how to keep that complexity manageable using **EventBridge** and **SQS** on AWS.

## Why EventBridge?

Amazon EventBridge is a serverless event bus that connects applications using events. It provides:

- **Event routing** with content-based filtering
- **Schema discovery** and registry
- **Built-in integrations** with 30+ AWS services
- **Archive and replay** for debugging

## Setting Up an Event Bus

```bash
aws events create-event-bus --name orders-bus
```

```python
import boto3
import json

eventbridge = boto3.client('events')

# Publish an event
eventbridge.put_events(
    Entries=[{
        'Source': 'com.myapp.orders',
        'DetailType': 'OrderCreated',
        'Detail': json.dumps({
            'orderId': 'ORD-001',
            'customerId': 'CUST-123',
            'total': 149.99
        }),
        'EventBusName': 'orders-bus'
    }]
)
```

## Defining Event Rules

Rules match incoming events and route them to targets:

```json
{
  "source": ["com.myapp.orders"],
  "detail-type": ["OrderCreated"],
  "detail": {
    "total": [{"numeric": [">=", 100]}]
  }
}
```

This rule only matches orders above $100 — powerful for building targeted processing pipelines.

## SQS as a Buffer

For high-throughput consumers, put SQS between EventBridge and Lambda:

EventBridge → SQS Queue → Lambda Consumer

This gives you:
- **Buffering** during traffic spikes
- **Dead-letter queues** for failed messages
- **Batching** for cost efficiency

```python
def handler(event, context):
    for record in event['Records']:
        body = json.loads(record['body'])
        detail = json.loads(body['detail'])
        print(f"Processing order {detail['orderId']}")
        # ... business logic ...
```

## Handling Failures

Configure a dead-letter queue (DLQ) on your SQS queue:

```bash
aws sqs set-queue-attributes \
  --queue-url https://sqs.us-east-1.amazonaws.com/123456789/orders-queue \
  --attributes '{"RedrivePolicy": "{\"deadLetterTargetArn\":\"arn:aws:sqs:us-east-1:123456789:orders-dlq\",\"maxReceiveCount\":\"3\"}"}'
```

After 3 failed processing attempts, messages land in the DLQ for inspection.

## Summary

- Use EventBridge for event routing and fan-out
- Use SQS as a buffer between producers and consumers
- Always configure dead-letter queues
- Start with simple point-to-point patterns before building complex choreographies
