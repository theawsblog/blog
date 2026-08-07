---
title: "Agent Authorization Needs to Understand Time and Sequence"
date: 2026-08-07
author: "Emiliano Montesdeoca"
description: "Amazon Bedrock AgentCore temporal policies extend authorization beyond identity so agent actions can be checked against workflow history, freshness, approvals, and cumulative limits."
tags:
  - aws
  - bedrock
  - agentcore
  - security
  - ai
source: "https://aws.amazon.com/blogs/machine-learning/securing-ai-agents-with-temporal-policies-in-amazon-bedrock-agentcore/"
draft: false
---

Traditional authorization asks who is making a request and whether that principal can call a tool. That model is still necessary, but agents create a second question: what happened immediately before this request?

A single tool call may be harmless in isolation and dangerous in sequence. An agent can read an account, invent a different identifier, and pass it to a money-moving tool. It can reuse an old approval. It can execute many individually valid actions until the total crosses a business limit.

The AWS Machine Learning Blog's [temporal policy pattern for Amazon Bedrock AgentCore](https://aws.amazon.com/blogs/machine-learning/securing-ai-agents-with-temporal-policies-in-amazon-bedrock-agentcore/) addresses that gap by evaluating an action against the agent's session history. The important idea is not another policy syntax. It is that authorization for an agent needs to understand trajectory.

## Identity is not enough

A normal IAM check can answer whether an agent role is allowed to call `execute_trade`. It does not necessarily know whether the agent fetched a current price, received approval for this trade, or already spent the session's budget.

Temporal policies can express conditions such as:

- a profile lookup must happen before a write action,
- the identifier used by the next tool must match a value returned earlier,
- a price must be refreshed within a defined time window,
- cumulative spend must stay below a session limit,
- a human approval can be consumed only once,
- a capability expires after inactivity.

These are business and safety rules that depend on order, time, and accumulated state.

## Put the enforcement outside the agent

The source article emphasizes that the policies run at the AgentCore Gateway perimeter, outside the agent's own code. That placement is important. If the agent's prompt, planner, or tool wrapper is the only enforcement point, a model mistake or a code defect can bypass it.

The gateway still needs an accurate session boundary. Requests carry a session ID so the policy engine can associate actions with the right trajectory. Treat that ID as security-sensitive. A collision or a user-controlled identifier that is not scoped correctly can mix histories and produce the wrong authorization decision.

Keep policy decisions observable. For every deny, operators should be able to see which condition failed, which previous action was relevant, and whether the problem was a legitimate workflow edge case or an attempted unsafe sequence.

## Temporal rules are least privilege with context

Least privilege for agents is more precise than giving a role access to a list of tools. It can mean giving a tool access only after the agent has completed the required verification steps, for a limited time, and within a cumulative budget.

That changes how I would design an agent workflow. First map the destructive or irreversible actions. Then identify the facts that must be true before each action. Those facts become policy conditions rather than instructions in a system prompt.

For example, a support agent may read a customer record freely, but a refund tool could require a recent order lookup, a matching order ID, a maximum amount, and one human approval per refund. The agent can still plan and converse flexibly. The gateway controls the action boundary.

## Start in observation mode

Do not turn a complex temporal policy on in enforcement mode without seeing how real traffic behaves. Begin with a logging or observation mode if the integration supports it. Collect normal trajectories, denied-looking sequences, stale data cases, retries, and human handoffs.

Then add one requirement at a time:

1. Define the session boundary and identity mapping.
2. Inventory tool calls and mark which actions are consequential.
3. Write a policy for one sequence or freshness rule.
4. Replay legitimate and adversarial trajectories in tests.
5. Observe production-like traffic without blocking it.
6. Move the rule to enforcement after false positives are understood.

A policy that is too strict can push operators toward disabling the control. A policy that is too vague gives false confidence. Small, testable rules are easier to review and explain.

## What temporal policies do not solve

They do not make a tool safe by themselves. The tool still needs input validation, idempotency, resource scoping, and useful errors. They also do not verify that the model's natural-language explanation is true. They control whether an action is permitted under the recorded trajectory.

That distinction matters when a workflow includes external systems or asynchronous callbacks. Decide what evidence is authoritative, how long it remains fresh, and what happens when an upstream result is delayed or duplicated.

Identity-based access control remains the foundation. Temporal policies add the missing context for workflows where order and time change the meaning of an action.

For teams putting agents near customer data, financial operations, or production changes, that is a practical upgrade in security thinking: do not ask only who can call the tool. Ask what must have happened before the tool call can be trusted.