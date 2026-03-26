---
title: "Understanding DynamoDB: Data Modeling for Real Workloads"
date: 2025-07-03
author: "Marco Rossi"
description: "Single-table design, partition keys, GSIs — making sense of DynamoDB data modeling and avoiding common pitfalls."
tags:
  - aws
  - dynamodb
  - databases
  - architecture
---

DynamoDB is deceptively simple to start with but hard to master. The wrong data model can make your application slow, expensive, or both. Let's get it right.

## The Core Concept: Key-Value at Scale

DynamoDB is a key-value store with an optional sort key. Every query must target a specific partition key. No partition key, no query — full scans are available but expensive.

```python
import boto3

dynamodb = boto3.resource('dynamodb')
table = dynamodb.Table('Orders')

# Efficient — targets a specific partition
response = table.query(
    KeyConditionExpression='PK = :pk AND begins_with(SK, :sk)',
    ExpressionAttributeValues={
        ':pk': 'CUSTOMER#123',
        ':sk': 'ORDER#'
    }
)
```

## Single-Table Design

Instead of one table per entity, you store all entities in a single table using composite keys:

| PK | SK | Data |
|---|---|---|
| CUSTOMER#123 | PROFILE | {name, email} |
| CUSTOMER#123 | ORDER#001 | {total, status} |
| CUSTOMER#123 | ORDER#002 | {total, status} |
| ORDER#001 | ITEM#A | {product, qty} |

This lets you fetch a customer and all their orders in a **single query**.

## Global Secondary Indexes (GSIs)

GSIs let you query the same data by different keys. Think of them as alternative views of your table.

```python
# GSI: query orders by status
response = table.query(
    IndexName='GSI1',
    KeyConditionExpression='GSI1PK = :pk',
    ExpressionAttributeValues={
        ':pk': 'STATUS#PENDING'
    }
)
```

> **Important**: Each GSI costs additional write capacity. Don't create GSIs you don't need.

## When to Use Single-Table vs. Multi-Table

Use single-table when you have well-defined access patterns and want to minimize the number of queries. Use multi-table when your access patterns are unpredictable or when you need full SQL-like flexibility — in that case, consider Amazon Aurora or Amazon RDS instead.

## Capacity Modes

- **On-Demand**: Pay per request. Great for unpredictable traffic.
- **Provisioned**: Set read/write capacity units. Cheaper at steady-state traffic.

## Summary

- Always start with your access patterns, not your entities
- Prefer single-table design for related entities queried together
- Use GSIs sparingly — each one costs money
- Use on-demand mode until you understand your traffic patterns
