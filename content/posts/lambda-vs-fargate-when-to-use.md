---
title: "Lambda vs. Fargate: When to Use Which"
date: 2025-08-14
author: "Alice Nguyen"
description: "Lambda and Fargate are both serverless, but they solve different problems. Here's a principled guide on when each one shines."
tags:
  - aws
  - serverless
  - lambda
  - fargate
---

Lambda and Fargate both let you run code without managing servers. But they have fundamentally different execution models. Choosing wrong can cost you money and developer time.

## What Is Lambda?

Lambda runs functions in response to events. You write a handler, deploy it, and AWS handles everything else.

```python
def handler(event, context):
    name = event.get('name', 'World')
    return {
        'statusCode': 200,
        'body': f'Hello, {name}!'
    }
```

No servers. No containers. Just code triggered by events — API Gateway, S3, SQS, EventBridge, and more.

## What Is Fargate?

Fargate runs containers without managing EC2 instances. You define a task with CPU and memory, and AWS provisions the compute.

```yaml
# task-definition.json
{
  "family": "my-api",
  "cpu": "512",
  "memory": "1024",
  "containerDefinitions": [
    {
      "name": "api",
      "image": "123456789.dkr.ecr.us-east-1.amazonaws.com/my-api:latest",
      "portMappings": [{"containerPort": 8080}]
    }
  ]
}
```

## When Lambda Shines

**Event-driven workloads** — Processing S3 uploads, SQS messages, or DynamoDB streams.

**Spiky or unpredictable traffic** — Scale from zero to thousands of concurrent executions in seconds.

**Short-lived tasks** — Anything that completes in under 15 minutes.

## When Fargate Wins

**Long-running services** — APIs, WebSocket servers, or background workers that need to stay alive.

**Consistent high traffic** — At steady-state traffic, Fargate is often cheaper than Lambda.

**Complex dependencies** — If your application needs a specific runtime, system libraries, or sidecar containers.

## Cost Comparison

At low and spiky traffic, Lambda is almost always cheaper. At sustained high throughput (millions of requests per hour), Fargate with Savings Plans can be 50-70% cheaper than Lambda.

## My Take

Start with Lambda for new services. Move to Fargate if you outgrow Lambda's execution model or need finer cost control at scale. The two can coexist — many production architectures use both.
