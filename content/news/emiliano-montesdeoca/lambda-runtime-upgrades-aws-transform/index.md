---
title: "Lambda runtime upgrades need campaigns, not reminders"
date: 2026-06-20
author: "Emiliano Montesdeoca"
description: "AWS Transform custom can help teams upgrade Lambda runtimes at scale, but the durable improvement is treating runtime changes as governed modernization campaigns."
tags:
  - aws
  - lambda
  - modernization
  - developer-tools
  - operations
source: "https://aws.amazon.com/blogs/compute/upgrading-lambda-function-runtimes-at-scale-with-aws-transform-custom/"
draft: false
---

Lambda runtime upgrades are easy to postpone because the function still works today. Then a deprecation date becomes a support risk, a security exception, or a rushed platform campaign.

The AWS Compute Blog post on [upgrading Lambda function runtimes at scale with AWS Transform custom](https://aws.amazon.com/blogs/compute/upgrading-lambda-function-runtimes-at-scale-with-aws-transform-custom/) is useful because it treats runtime upgrades as a repeatable modernization workflow, not a calendar reminder.

## What changed

AWS Transform custom can analyze codebases, identify runtime upgrade risk, apply transformations, update dependencies or configuration, and validate against exit criteria. The source article focuses on Lambda runtime upgrades, including cases where code changes are required, such as moving callback-style Node.js handlers toward async patterns.

For platform teams, the more important piece is campaign management. A large organization rarely has one repository and one owner. It has hundreds of functions across many accounts, teams, IaC stacks, and CI/CD systems.

## Why builders should care

Runtime upgrades fail when ownership is unclear.

An application team may not know a function exists. A platform team may see the deprecated runtime but not understand the deployment path. Security may see the exposure but not own the code. The result is a spreadsheet-driven migration with too many manual exceptions.

A transformation tool can help, but only if it plugs into an operating model:

- inventory functions and repositories,
- map functions to owners,
- assess code and dependency risk,
- create pull requests or transformation branches,
- validate with tests and alarms,
- deploy with safe rollout and rollback,
- track completion centrally.

That is a campaign, not a one-off task.

## The trade-offs

Automated transformation is not a substitute for test coverage. It increases the value of tests because it can make changes faster than humans can manually review every edge case.

Before trusting a runtime upgrade, teams should check:

- unit and integration tests for handler behavior,
- dependency compatibility with the target runtime,
- infrastructure definitions that set the runtime,
- packaging and build pipeline changes,
- observability after deployment,
- alias or version-based rollback strategy.

For platform teams, the other trade-off is centralization. Pushing changes across many repositories can be efficient, but service owners still need to understand and approve behavior changes. The best model is usually centralized orchestration with distributed ownership.

## What to do next

Start with inventory. Use AWS Config, Trusted Advisor, CLI scripts, tags, or deployment metadata to find functions approaching deprecation. Then group them by owner, runtime, business criticality, and test readiness.

For low-risk functions with good tests, an automated transform can move quickly. For high-risk functions with poor tests, the first transformation should probably add documentation, tests, or alarms before changing the runtime.

The lesson from this AWS post is broader than Lambda. Modernization work becomes manageable when it is continuous, visible, and validated. If runtime upgrades are still handled as emergency cleanup, the tooling can help, but the process needs upgrading too.
