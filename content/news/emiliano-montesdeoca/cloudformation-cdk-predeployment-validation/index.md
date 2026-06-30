---
title: "CloudFormation pre-deployment validation makes IaC failures cheaper"
date: 2026-06-30
author: "Emiliano Montesdeoca"
description: "AWS CloudFormation and CDK pre-deployment validation now runs on stack operations, helping builders catch quota, Config, and ECR issues before failed deployments waste time."
tags:
  - aws
  - cloudformation
  - cdk
  - devops
  - iac
source: "https://aws.amazon.com/blogs/devops/ship-infrastructure-faster-with-cloudformation-and-cdk-pre-deployment-validation-on-every-stack-operation/"
draft: false
---

The best deployment failure is the one that fails before it starts changing infrastructure.

AWS has announced that [CloudFormation pre-deployment validation now runs automatically on every stack create and update operation](https://aws.amazon.com/blogs/devops/ship-infrastructure-faster-with-cloudformation-and-cdk-pre-deployment-validation-on-every-stack-operation/). The same post also introduces new checks and a `cdk validate` workflow.

This is not as visually exciting as a new service. It is more useful than many new services.

## What changed

Pre-deployment validation now runs on CloudFormation stack operations without requiring a separate setup. The source article mentions new checks for service quota limits, AWS Config Recorder conflicts, and ECR repository delete readiness. It also describes operation-level control and CDK integration through `cdk validate`.

For builders, this shifts some failure detection earlier in the pipeline. Instead of discovering quota or environment conflicts halfway through deployment, teams can catch them before the operation commits to a path.

## Why it matters

Infrastructure failures are expensive because they consume time, create partial state, and distract teams from the real change they were trying to ship.

Quotas are a classic example. A template can be valid and still fail because the target account or Region does not have enough capacity for a resource type. A pre-deployment check that catches this earlier improves both developer experience and operational safety.

The CDK angle matters too. CDK users often think in application constructs while CloudFormation owns the final deployment. Bringing validation closer to the CDK workflow helps developers discover platform constraints before the change reaches a shared environment.

## The trade-offs

Validation is not a proof of success. It can catch known classes of problems, but it cannot guarantee the application will work, the dependency will be healthy, or the rollout will be safe.

Builders still need:

- template linting and policy checks,
- least-privilege review,
- drift detection,
- integration tests,
- deployment alarms,
- rollback or remediation plans,
- post-deploy smoke tests.

There is also a balance between strictness and velocity. If validation blocks legitimate emergency changes, teams will look for bypasses. The `DisableValidation` option should be treated like any other break-glass control: rare, logged, and reviewed.

## What to do next

Add `cdk validate` or equivalent CloudFormation validation to pull requests and pre-merge pipelines where it makes sense. Then track validation failures as a signal. If many teams hit the same quota or Config issue, the platform needs a baseline fix, not repeated ticket handling.

For production, make validation one gate in a broader release process. It should happen before deployment, but readiness should still be proven after deployment.

The practical takeaway: pre-deployment validation makes IaC failures cheaper and faster to understand. It does not remove the need for good deployment engineering, but it moves a useful class of mistakes to the left.
