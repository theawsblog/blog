---
title: "Agent UIs need protocols, not only chat boxes"
date: 2026-07-17
author: "Emiliano Montesdeoca"
description: "AG-UI with Amazon Bedrock AgentCore shows how agent frontends can support generative UI, shared state, and human-in-the-loop workflows without coupling every frontend to one agent framework."
tags:
  - aws
  - bedrock
  - agents
  - frontend
  - ai
source: "https://aws.amazon.com/blogs/machine-learning/build-generative-ui-for-ai-agents-on-amazon-bedrock-agentcore-with-the-ag-ui-protocol/"
draft: false
---

A chat box is not enough for many agent workflows. Agents need to show charts, update state, ask for approval, stream intermediate results, and let users steer the task while it is running.

The AWS Machine Learning Blog post on [building generative UI for AI agents on Amazon Bedrock AgentCore with AG-UI](https://aws.amazon.com/blogs/machine-learning/build-generative-ui-for-ai-agents-on-amazon-bedrock-agentcore-with-the-ag-ui-protocol/) is useful because it treats agent user experience as a protocol problem.

## What changed

AG-UI defines a standard event protocol between agent backends and frontends. The source article shows AG-UI running with Amazon Bedrock AgentCore Runtime, the FAST starter template, Strands Agents, LangGraph, Cognito, Amplify, and CopilotKit.

The important idea is decoupling. The frontend should not need to know every detail of the agent framework. The agent should emit structured UI events that the frontend can render consistently.

## Why builders should care

As agents move beyond Q&A, the UI becomes part of the system's reliability and trust model.

A human approval step should be explicit. A generated chart should be tied to data. A shared todo canvas should update predictably. A long-running agent should expose progress instead of leaving the user guessing.

Protocols like AG-UI can make those interactions more portable across frameworks and frontends.

## The trade-offs

Generative UI introduces new failure modes. The agent can request the wrong component, produce inconsistent state, or ask for approval without enough context. Frontends need validation, permissions, and safe rendering rules.

Builders should define which UI events are allowed, which components are trusted, and how user actions flow back to the agent. Treat UI actions like tool calls: validate inputs, authorize state changes, and log important decisions.

Also avoid overbuilding. Some agents only need text. Use generative UI when interaction complexity justifies it.

## What to do next

Review agent workflows and identify where plain chat creates friction:

- approval gates,
- charts or tables,
- editing structured state,
- multi-step progress,
- choosing between alternatives,
- reviewing generated artifacts.

Those are good candidates for AG-UI-style events.

The practical takeaway: the next generation of agent applications will need richer interaction contracts. A protocol boundary between agent logic and frontend rendering makes that easier to build and safer to operate.
