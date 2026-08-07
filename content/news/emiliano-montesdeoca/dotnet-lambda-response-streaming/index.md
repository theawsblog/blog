---
title: "Response Streaming for .NET on Lambda Changes Perceived Latency"
date: 2026-08-07
author: "Emiliano Montesdeoca"
description: "Response streaming for .NET on AWS Lambda can deliver tokens and large responses earlier, but buffering, headers, timeouts, and disconnects become part of the API contract."
tags:
  - aws
  - dotnet
  - lambda
  - serverless
  - ai
source: "https://aws.amazon.com/blogs/developer/announcing-response-streaming-for-net-on-aws-lambda/"
draft: false
---

A response that takes five seconds can feel faster than a response that takes three seconds if the first one starts showing useful content immediately. That is the practical reason response streaming matters for .NET workloads on AWS Lambda.

The [AWS announcement](https://aws.amazon.com/blogs/developer/announcing-response-streaming-for-net-on-aws-lambda/) adds response streaming support for .NET Lambda functions. Instead of buffering the complete result and returning it once the handler finishes, the function can send bytes as they become available. That is especially useful for token generation, large exports, and responses that do not fit comfortably in the traditional buffer.

The feature is valuable. It also changes the failure model of the endpoint.

## Buffering hides time from the user

A model can generate a token at a time, but a buffered Lambda response makes the client wait for all of them. The same problem appears with a large CSV export: the function spends time building the response, and the caller sees nothing until serialization and buffering are complete.

Streaming changes that first-byte experience. With a `StreamWriter`, the handler writes a chunk and flushes it to the caller. For an LLM, that means the interface can render the first tokens while the model continues working.

The flush is important. A streaming API that never flushes is just a buffered API with extra ceremony. The source examples use explicit `FlushAsync` calls, and your implementation should choose a flush strategy based on the client and payload rather than assuming every write crosses the network immediately.

## Headers are a one-time decision

The API Gateway integration adds a constraint that is easy to miss: the response needs a prelude containing the HTTP status code and headers. Once the prelude has gone out, the handler cannot change the status code because a later exception occurs.

That means validation and setup should happen before the first response bytes. If a model call can fail, decide how the stream will represent that failure after headers have been sent. A JSON error body cannot magically turn a previously sent `200` into a `500`.

The API Gateway integration also needs the response-streaming invocation path and the correct transfer mode. If the function is configured for streaming but the gateway still buffers, the client receives none of the latency benefit. Test the deployed path, not only the handler in isolation.

## Timeouts and disconnects become visible

Streaming does not reset the Lambda timeout. A function with a 30-second timeout still has a 30-second budget, even if it flushes a token every second. API Gateway also needs its own timeout configuration and enough headroom for the integration to finish.

Client disconnects create a second operational concern. If the browser closes after token 40, Lambda may continue generating token 41 through 200 unless the handler observes cancellation or a failed flush. That work costs money and may keep a model invocation alive after the user has gone away.

Log the session or request ID, time to first byte, number of chunks sent, total duration, and the reason the stream ended. A normal completion, a client disconnect, and a model failure should not look identical in CloudWatch.

## ASP.NET Core is a useful path

For ASP.NET Core applications hosted on Lambda, the AWS hosting layer can enable response streaming with `EnableResponseStreaming`. Standard response helpers can continue to work, but the integration still needs to be configured and tested.

A minimal endpoint might write a line, flush periodically, and stop when the request is cancelled. Keep the stream format stable. For token output, newline-delimited events or a documented event format is easier for clients to process than arbitrary text fragments. For large exports, make the client aware that a partial response is not a valid completed file.

## Where the feature fits

The larger response ceiling is useful, but it should not become an excuse to send unbounded data through a synchronous Lambda request. A 200 MB limit does not make a long-lived download a good fit for every API. Use S3 for durable files and presigned delivery when that is the simpler contract.

I would prioritize streaming for interactive AI responses and incremental exports where first-byte latency matters. For normal CRUD endpoints, buffering remains easier to observe and retry.

## What to test before production

Test the full path with:

- a slow model or generator,
- a client that disconnects halfway through,
- an exception after the first flush,
- a response that approaches the integration limit,
- API Gateway timeout headroom,
- retries after a partial response.

Also measure perceived latency rather than only total duration. The point of streaming is not to make the model compute less. It is to deliver useful output sooner without turning partial output into an ambiguous API.

For .NET teams building serverless AI features, response streaming is a meaningful improvement. Just remember that once bytes leave the function, the response contract is already in motion. Headers, cancellation, observability, and client behavior need to be designed along with the `IAsyncEnumerable` or stream itself.