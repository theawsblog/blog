---
title: "Running pgvector on Aurora is a production operations decision"
date: 2026-06-25
author: "Emiliano Montesdeoca"
description: "AWS guidance on pgvector in Amazon Aurora PostgreSQL highlights that vector search is not only a model feature; it needs indexing, memory, partitioning, and observability discipline."
tags:
  - aws
  - aurora
  - postgresql
  - pgvector
  - ai
  - data
source: "https://aws.amazon.com/blogs/database/running-pgvector-in-production-on-amazon-aurora-postgresql/"
draft: false
---

It is easy to prototype vector search. It is harder to operate it after users, documents, embeddings, and retrieval patterns start changing every day.

The AWS Database Blog post on [running pgvector in production on Amazon Aurora PostgreSQL](https://aws.amazon.com/blogs/database/running-pgvector-in-production-on-amazon-aurora-postgresql/) is useful because it moves the conversation away from "can I store embeddings?" and toward "can I keep this retrieval system healthy?"

## What changed

The source article covers operational practices for pgvector workloads on Aurora PostgreSQL: choosing index types and distance functions, managing HNSW behavior, using quantization and partitioning, sizing memory, and monitoring the signals that show when the vector store is drifting out of shape.

That is the right level of discussion for production RAG systems. The database is not just a place to put vectors. It is part of the user-facing latency, relevance, and cost profile.

## Why builders should care

Aurora PostgreSQL with pgvector is attractive because many teams already understand PostgreSQL. They can keep relational data, metadata filters, access patterns, and embeddings close together. That reduces architecture sprawl for early and mid-sized AI applications.

But familiarity can hide risk. Vector indexes have different maintenance behavior than normal B-tree indexes. Embedding dimensions affect memory. Update and delete patterns can degrade index quality. Query filters can change recall and latency. The database may need to serve both transactional and retrieval traffic.

If you treat pgvector like a small column type, production will teach you otherwise.

## The trade-offs

The main decision is managed abstraction versus self-managed control.

Aurora PostgreSQL with pgvector gives control over schema, SQL, transactions, and tuning. That is valuable when retrieval is tightly coupled to application data. Amazon Bedrock Knowledge Bases or other managed retrieval systems reduce operational burden, which can be better when the team does not need direct database-level control.

There is no universal winner. Choose pgvector on Aurora when PostgreSQL integration is a real product advantage. Choose a more managed path when the team mostly wants ingestion, embedding, retrieval, and scaling handled for them.

## What to do next

Before putting pgvector-backed retrieval into production, define operational checks:

- index type and distance metric per use case,
- expected vector count and growth rate,
- memory needed to keep hot indexes healthy,
- update and deletion behavior,
- query latency percentiles under realistic filters,
- recall evaluation for representative prompts,
- vacuum and maintenance expectations,
- fallback behavior when retrieval fails or gets slow.

Also separate prototype metrics from production metrics. A demo with 10,000 documents says little about a system with millions of vectors, concurrent users, and evolving embeddings.

The practical takeaway is simple: pgvector on Aurora can be a strong architecture choice, but only if the team is ready to operate vector search as a database workload, not as a model configuration checkbox.
