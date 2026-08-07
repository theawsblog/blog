---
title: "Cross-Account ECS Telemetry Needs a Platform Boundary"
date: 2026-08-07
author: "Emiliano Montesdeoca"
description: "A centralized AWS Distro for OpenTelemetry gateway can reduce per-task overhead and close observability gaps across multi-account ECS environments, including Windows .NET workloads."
tags:
  - aws
  - ecs
  - observability
  - opentelemetry
  - devops
  - dotnet
source: "https://aws.amazon.com/blogs/containers/centralize-cross-account-amazon-ecs-telemetry-with-an-adot-gateway/"
draft: false
---

A collector sidecar is a friendly pattern right up until an organization has hundreds of ECS tasks. Then the same convenience becomes a platform tax: every task carries collector CPU and memory, every account carries configuration, and every update becomes another rollout.

The problem is even sharper for Windows .NET Framework workloads. A Linux collector sidecar is not a universal answer for an IIS application running on Windows. You either create a separate collection strategy or accept a hole in the telemetry map.

The AWS Containers Blog's [cross-account ECS telemetry pattern](https://aws.amazon.com/blogs/containers/centralize-cross-account-amazon-ecs-telemetry-with-an-adot-gateway/) moves collection into a dedicated observability account. Workloads send OpenTelemetry Protocol data over private connectivity to a centralized AWS Distro for OpenTelemetry gateway, which exports traces to AWS X-Ray and metrics and logs to CloudWatch.

That is more than a collector deployment. It is an observability platform boundary.

## Sidecar versus gateway

The sidecar model has a real advantage: the collector sits next to the application and is easy to understand. For a small service, that local ownership may be exactly right. The cost appears when the pattern is copied everywhere.

A collector per task multiplies baseline resource usage. It also multiplies configuration drift. A change to batching, sampling, authentication, or metric dimensions must reach every task definition and every account. Windows workloads introduce another constraint because the application may not be able to host the same Linux sidecar at all.

A centralized gateway reverses those trade-offs:

- one fleet to patch and configure,
- one endpoint for workloads across accounts,
- one place to control batching and export behavior,
- a shared path for Linux and Windows applications.

The price is network and platform complexity. You need private routing between workload VPCs and the observability VPC, capacity planning for the gateway, and a clear ownership model for telemetry arriving from many accounts.

## Windows .NET is the forcing function

For a Windows .NET Framework application hosted in IIS, the gateway is not just a cost optimization. It can be the practical way to reach the destination telemetry systems without running a Linux collector beside the application.

The source pattern configures OpenTelemetry instrumentation in the Windows image and points the OTLP exporter at the gateway's internal network load balancer. IIS worker processes need the right machine-level environment variables and instrumentation registration. That detail is easy to miss because a Linux container tutorial can make environment configuration look universal.

The useful design principle is to keep application instrumentation consistent while moving platform-specific collection into the shared gateway. The application emits standard OTLP. The gateway owns export, batching, dimensions, and destination-specific details.

## The failure mode that wastes a day

Centralization does not make networking disappear. One common failure is a target health-check loop: ECS tasks send telemetry to the load balancer, but the collector targets never become healthy. The security group allows client CIDRs but blocks health checks originating from the load balancer subnets.

Allow the health-check path explicitly. Test the gateway's health endpoint from the load balancer perspective, then test OTLP reachability from a workload VPC. A collector can be running and still be unreachable, which makes task logs unnecessarily confusing.

Preserve source identity as well. The gateway should add fallback metadata only when the workload did not provide it. If the collector overwrites account, cluster, or service attributes, all telemetry starts looking as if it came from the observability account. Centralization is useful only when the origin remains trustworthy.

## Choose the layer that is missing

The AWS article distinguishes collection from visualization. CloudWatch cross-account observability can give operators a shared view of telemetry that is already being collected. It does not solve an instrumentation or ingestion gap. An ADOT gateway operates earlier, at the collection boundary.

That distinction keeps teams from deploying a gateway to solve the wrong problem. If every workload already emits reliable telemetry and the pain is searching across accounts, use the native cross-account viewing features. If applications cannot collect consistently, or you need one governed exporter configuration, the gateway is the better fit.

## A practical rollout

I would start with one observability account, two Availability Zones, and one representative Linux service plus one Windows .NET service. Before onboarding more accounts:

1. Establish Transit Gateway or equivalent private routing and verify non-overlapping CIDR ranges.
2. Run the gateway behind an internal load balancer and alarm on unhealthy targets.
3. Send a test trace, metric, and log from each operating-system family.
4. Check that account, cluster, service, and environment dimensions survive the gateway.
5. Limit CloudWatch metric dimensions to fields you actually query; high-cardinality metadata becomes an ingestion bill quickly.
6. Add a second collector task and exercise a replacement while telemetry is flowing.

The source article estimates that a small centralized gateway can run in the low tens of dollars per month before telemetry ingestion costs. The exact number will vary, but the broader lesson is stable: compare one governed fleet with the cumulative cost of per-task collectors, not with a zero-cost imaginary sidecar.

A centralized ADOT gateway is not automatically the right answer for every ECS estate. It is a strong answer when multi-account scale, Windows .NET support, and consistent collection are the actual constraints. Treat it as platform infrastructure, preserve source identity, and test the failure path before calling observability complete.