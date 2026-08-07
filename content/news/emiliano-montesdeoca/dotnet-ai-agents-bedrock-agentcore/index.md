---
title: "Beyond the SDK Demo: Building Production-Ready .NET Agents with AgentCore"
date: 2026-08-07
author: "Emiliano Montesdeoca"
description: "Moving a .NET AI agent from an SDK sample to Amazon Bedrock AgentCore means making deliberate choices about streaming, sessions, testing, deployment, and portability."
tags:
  - aws
  - dotnet
  - bedrock
  - agentcore
  - ai
source: "https://aws.amazon.com/blogs/developer/building-and-deploying-net-ai-agents-with-amazon-bedrock-agentcore/"
draft: false
---

The first .NET AI agent is usually easy to demonstrate. Register a model client, add a handler, send a prompt, print the answer. The production version is where the interesting decisions begin: how do sessions stay separate, how do responses stream, how do you test without AWS credentials, and what exactly happens when the runtime scales or restarts?

The AWS Developer Tools Blog's [AgentCore guide for .NET](https://aws.amazon.com/blogs/developer/building-and-deploying-net-ai-agents-with-amazon-bedrock-agentcore/) introduces `AWS.AgentCore.Hosting` and the surrounding tooling for those concerns. The library "handles the operational concerns: scaling, session routing, health checking, and providing managed capabilities like conversation memory." That is useful, but a hosting library cannot choose the boundaries of your application for you.

## Choose an API shape that leaves room

The package supports a source-generator experience and an extension-method experience in ASP.NET Core. The choice looks like syntax. It is really a choice about how much control the application will need around the handler.

If the agent is a small, stable endpoint, generated wiring can keep the code compact. If the agent will need custom middleware, request correlation, policy checks, metrics, or dependency injection behavior, the extension method gives the team more room to shape the host. The source article describes middleware that can intercept every invocation. That is exactly where I would put cross-cutting concerns rather than scattering them through prompt handlers.

Keep the handler thin either way. Put business operations, model selection, and external calls behind services that can be tested without the AgentCore runtime. Then the hosting layer remains an integration boundary, not the place where the entire application lives.

## Streaming changes the contract

Returning a complete string is comfortable, but it makes users wait for the slowest part of the generation. AgentCore supports returning `IAsyncEnumerable<string>` so tokens can be streamed as they are produced. For conversational applications, the difference between first token latency and total response latency is visible immediately.

Streaming also changes failure handling. A model call, memory lookup, or downstream tool can fail after the response has started. The caller cannot receive a new HTTP status code at that point, so the protocol needs a clear way to represent an error in the stream. Cancellation matters as well: when the browser disconnects, the handler should stop generating and release work instead of continuing to pay for a response nobody will read.

Test partial output, cancellation, slow clients, and a failure after the first token. A streaming demo proves that bytes arrive. A production test proves that the system behaves when the stream is interrupted.

## Session IDs are security boundaries

AgentCore Memory combines a configured Memory ID with a session ID to load and store conversation history. That makes the session ID part of your security model, not just a convenience parameter.

Two users must never share a session by accident. A missing session ID should fail clearly or create a deliberately scoped new session; it should not quietly fall back to a shared default. Add tests for two independent sessions, repeated calls in the same session, missing identifiers, and attempts to reuse another user's identifier.

The local emulator and `AWS.AgentCore.Testing` package are valuable here. Use them to verify session behavior in CI without deploying the agent or requiring live AWS credentials. Memory integration is exactly the kind of feature that appears correct in a happy-path demo and fails in a multi-user system.

## Deployment is still architecture

The `dotnet aws deploy` workflow makes deployment approachable, but it also hides choices. Container architecture, memory, VPC access, request header allowlists, and private service connectivity all affect runtime behavior.

Read the generated resources and make those choices explicit. If the agent needs a private database or internal API, test the VPC path from the deployed runtime rather than assuming the local Aspire application proves it. If the service has a meaningful cold-start budget, measure it on the actual architecture you plan to run.

Native AOT can reduce startup time, but it changes the development experience. Source-generated serialization and explicit service resolution may be required where flexible reflection-based binding previously worked. Decide whether cold-start reduction is worth that constraint for this workload instead of enabling AOT because it is available.

## Keep portability in the business layer

AgentCore is a managed runtime, but the agent's business logic can remain portable. Treat the handler as an adapter around services that know how to perform domain operations. Avoid spreading AgentCore-specific session and transport calls through every class.

That design gives you a useful fallback: the same domain service can be hosted in another ASP.NET Core application, tested in isolation, or moved to another container platform if the operational requirement changes. Portability is not free, but it is much cheaper when the boundary is intentional from the beginning.

## My pre-production checklist

Before the first real users arrive, I would require:

- separate-session tests and a clear session ID ownership model,
- streaming tests covering cancellation and mid-response failures,
- emulator-backed integration tests in CI,
- health and correlation telemetry around every invocation,
- a documented deployment and rollback path,
- a decision on JIT versus Native AOT based on measured startup data.

The AgentCore hosting library removes a lot of plumbing. That is its value. The remaining work is the architecture around the plumbing: state, failure, identity, and operations. The SDK demo gets you to the first response. Those decisions are what get the agent to production.