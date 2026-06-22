---
title: "EKS control plane egress through your VPC closes a real private-cluster gap"
date: 2026-06-22
author: "Emiliano Montesdeoca"
description: "Amazon EKS customer-routed control plane egress lets Kubernetes API server traffic use customer VPC routing, security controls, and private endpoints for webhooks and OIDC dependencies."
tags:
  - aws
  - eks
  - kubernetes
  - networking
  - security
source: "https://aws.amazon.com/blogs/containers/amazon-eks-now-supports-control-plane-egress-through-your-vpc/"
draft: false
---

Private EKS clusters have always had two sides: how clients and nodes reach the API server, and how the API server reaches things on behalf of Kubernetes. The first side was easier to reason about. The second side had awkward edges.

AWS has announced [customer-routed control plane egress for Amazon EKS](https://aws.amazon.com/blogs/containers/amazon-eks-now-supports-control-plane-egress-through-your-vpc/), which routes customer-controllable Kubernetes API server outbound traffic through your VPC. That includes admission webhook callbacks, OIDC discovery, and aggregate API server requests.

This is a practical feature for teams that need private webhooks, private identity providers, and auditable network paths.

## What changed

With the new `controlPlaneEgressMode` set to `CUSTOMER_ROUTED`, the Kubernetes API server uses an elastic network interface in your VPC for specific outbound flows. Those flows can then follow your routing, security groups, VPC endpoints, DNS, Network Firewall, PrivateLink, Direct Connect, and logging patterns.

EKS-managed service traffic still uses the EKS-managed path. The feature is scoped to customer-controllable API server egress, not every packet from the control plane.

One important detail: after a cluster uses `CUSTOMER_ROUTED`, the setting is immutable for the life of the cluster. That makes planning more important than experimentation on a random production cluster.

## Why it matters

Admission webhooks are often part of the security boundary. They validate images, enforce labels, inject sidecars, block risky configurations, and integrate with policy engines. If the API server can only call a public endpoint, teams end up exposing services that they would rather keep private.

The same issue appears with external OIDC identity providers. If the control plane must fetch discovery documents and JWKS over an internet-reachable path, the cluster is not as private as the architecture diagram suggests.

Customer-routed egress makes a cleaner design possible:

- webhook services can live behind internal load balancers,
- private DNS can resolve API server dependencies,
- VPC Flow Logs can show the path,
- SCPs can enforce the required egress mode across accounts,
- network teams can apply existing inspection and routing controls.

For regulated environments, the value is not only connectivity. It is evidence.

## Design considerations

This feature moves responsibility to your network design. If DNS, routes, endpoint policies, or security groups are wrong, API server calls can fail. That can break admissions, identity association, or aggregated APIs.

I would pay attention to four areas:

1. **DNS resolution.** The API server now depends on the DNS path available from your VPC configuration for those customer-controllable names.
2. **Webhook availability.** A private webhook outage can become a cluster-wide admission outage if failure policies are strict.
3. **Certificate trust.** Private does not always mean privately trusted. The source article notes that OIDC issuer certificates still need a publicly trusted chain.
4. **Cost and routing.** NAT gateways, cross-AZ paths, inspection appliances, and endpoints can add cost or latency if the path is not designed deliberately.

## What to do next

Start by identifying clusters that already use admission webhooks, external OIDC providers, or aggregate API servers. Those clusters have the most to gain.

For new clusters, decide whether `CUSTOMER_ROUTED` should be part of the baseline. For existing clusters, test in a non-production environment with the same webhook and identity dependencies before updating anything important.

Then build a failure test. Block the webhook endpoint, break DNS resolution, and confirm your cluster behavior matches your expectations. Network privacy is useful only if the failure modes are understood.

This EKS change does not make private Kubernetes automatic, but it removes a real architectural compromise. Builders now have a better way to align the control plane's outbound path with the same network rules they already apply to workloads.
