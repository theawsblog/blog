---
title: "Secure multi-tenant RAG needs authorization at retrieval time"
date: 2026-07-14
author: "Emiliano Montesdeoca"
description: "AWS shows how Amazon Bedrock Knowledge Bases and Verified Permissions can enforce runtime document access for intra-tenant RAG applications without duplicating every knowledge base."
tags:
  - aws
  - bedrock
  - rag
  - security
  - verified-permissions
source: "https://aws.amazon.com/blogs/architecture/secure-multi-tenant-rag-with-amazon-bedrock-and-verified-permissions/"
draft: false
---

RAG security is not solved when the documents are ingested. It is solved when the right user retrieves the right document at the right time.

The AWS Architecture Blog post on [secure multi-tenant RAG with Amazon Bedrock and Verified Permissions](https://aws.amazon.com/blogs/architecture/secure-multi-tenant-rag-with-amazon-bedrock-and-verified-permissions/) is useful because it focuses on runtime authorization. The pattern uses a shared Bedrock knowledge base with metadata filtering, while Amazon Verified Permissions decides which filter should be applied for the current user.

That is a practical middle ground for enterprises that need department-level access control inside one organization.

## What changed

The source article describes a defense-in-depth pattern for intra-tenant RAG. Instead of hardcoding retrieval filters in application code, the application evaluates Cedar policies in Verified Permissions and constructs the metadata filter passed to the Bedrock retrieve-and-generate flow.

That means access rules can change without redeploying the application. It also means authorization decisions can be audited outside the application code.

## Why builders should care

Many internal RAG systems start with a simple permission model: each department gets a folder, each folder maps to a filter, and the app passes the filter during retrieval. That works until executives need cross-department access, project teams form, policies change, or audit teams ask where decisions are recorded.

Externalized authorization helps because access logic becomes a policy layer, not a code branch.

## The trade-offs

The source article is clear about an important boundary: this is logical isolation, not hard infrastructure isolation. A shared knowledge base with metadata filters is not the same as a separate knowledge base per customer, account, or compliance boundary.

Use this pattern when users are inside the same tenant or organization and the goal is department, role, or project-level control. Do not use it as the only boundary between separate customers where a filter bug would create a regulatory incident.

Also design for failure. If the authorization service is unavailable, the safe default should be deny, not retrieve broadly.

## What to do next

Before building a RAG app, classify the isolation requirement:

- same organization, department-level permissions,
- separate business units with strong governance needs,
- external customers requiring hard tenant isolation,
- regulated data requiring infrastructure separation.

Then choose the architecture. A shared knowledge base plus Verified Permissions can be efficient for intra-tenant access. Dedicated knowledge bases or accounts may be necessary for hard boundaries.

The practical takeaway: RAG authorization belongs in the retrieval path. If the model can see data the user should not see, the answer is already compromised.
