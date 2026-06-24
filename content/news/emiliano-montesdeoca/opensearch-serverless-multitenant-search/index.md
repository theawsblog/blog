---
title: "OpenSearch Serverless next generation changes the economics of tenant isolation"
date: 2026-06-24
author: "Emiliano Montesdeoca"
description: "Amazon OpenSearch Serverless next-generation architecture makes collection-per-tenant search more practical with scale-to-zero compute and regional endpoint routing."
tags:
  - aws
  - opensearch
  - search
  - serverless
  - multitenancy
source: "https://aws.amazon.com/blogs/big-data/implement-multi-tenant-search-with-amazon-opensearch-serverless-next-generation/"
draft: false
---

Multi-tenant search has always forced an uncomfortable trade-off: isolate tenants cleanly and pay for too much infrastructure, or pool tenants together and accept more operational and security complexity.

The AWS Big Data Blog post on [multi-tenant search with Amazon OpenSearch Serverless next generation](https://aws.amazon.com/blogs/big-data/implement-multi-tenant-search-with-amazon-opensearch-serverless-next-generation/) is important because it changes that cost model. Scale-to-zero compute and regional endpoint routing make collection-per-tenant designs more realistic.

## What changed

OpenSearch Serverless next-generation architecture lets collection groups scale compute to zero, while storage charges still apply. AWS also added a per-account, regional endpoint that can route requests to collections using headers such as `x-amz-aoss-collection-name`.

That means a SaaS application can keep a cleaner collection-per-tenant model without managing a separate endpoint and connection pool for every tenant.

## Why builders should care

Tenant isolation is easier to discuss in architecture reviews than to pay for in production.

A collection-per-tenant design gives strong isolation boundaries for data, workload behavior, encryption, lifecycle, and noisy-neighbor risk. But if each tenant carries a minimum always-on compute cost, the design breaks down for long-tail tenants.

Scale-to-zero compute makes the model more practical for SaaS platforms with many tenants that search occasionally. The regional endpoint also simplifies application routing. Instead of maintaining many endpoints, the application can target one endpoint and route by collection header.

## The trade-offs

Collection-per-tenant is not automatically the best design.

For very small tenants with similar access patterns, pooled indexes may still be simpler. For very large tenants, dedicated collections or even separate domains may be appropriate. For regulated tenants, encryption, access policy, and audit boundaries may matter more than cost.

Builders should also design the tenant mapping layer carefully. A regional endpoint reduces endpoint sprawl, but the application still needs a reliable mapping from tenant ID to collection name or ID. That mapping becomes part of the security boundary.

Operational questions remain:

- How are collections created and deleted?
- What happens when a dormant tenant becomes active?
- How are per-tenant quotas enforced?
- How are index templates and mappings rolled out safely?
- How are tenant-level costs reported?

## What to do next

For SaaS search workloads, revisit the tenancy model. Compare pooled, collection-per-tenant, and hybrid approaches using real tenant distribution, not an average tenant.

A practical path is:

1. Put high-volume tenants in dedicated collections.
2. Keep small tenants pooled or grouped if isolation requirements allow it.
3. Use collection-per-tenant where security, compliance, or noisy-neighbor risk justifies the boundary.
4. Automate collection lifecycle and mapping changes from the start.

The takeaway is that OpenSearch Serverless is making stronger isolation less expensive. That does not remove design work, but it gives builders more room to choose isolation for good reasons instead of avoiding it because of minimum compute cost.
