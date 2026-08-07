---
title: "ACM ACME Migration Is an Operations Project"
date: 2026-08-07
author: "Emiliano Montesdeoca"
description: "Moving existing ACME clients to AWS Certificate Manager requires renewal telemetry, quota planning, hybrid connectivity, and incident drills before certificate lifetimes shrink further."
tags:
  - aws
  - acm
  - tls
  - security
  - operations
source: "https://aws.amazon.com/blogs/security/automate-certificates-with-acme-support-in-aws-certificate-manager/"
draft: false
---

Certificate automation is becoming less optional. As certificate validity periods shrink, a large estate will move from occasional renewals to a steady stream of renewal events. For a team managing hundreds or thousands of certificates, that is not just a PKI design concern. It is an operations workload.

AWS Certificate Manager now supports ACME for public certificates. The [AWS Security Blog announcement](https://aws.amazon.com/blogs/security/automate-certificates-with-acme-support-in-aws-certificate-manager/) is useful for teams already using Certbot, cert-manager, acme.sh, or another ACME client. They can keep the protocol and client workflow while using ACM as the certificate service.

The easy interpretation is "point the client at a new endpoint." The safer interpretation is "plan a migration with an observable renewal path."

## The migration details matter

ACM uses External Account Binding credentials. Those are not the same thing as a generic username and password or an API token from another certificate authority. Your client configuration needs the EAB key identifier and MAC key, and those values should be treated as bootstrap secrets with a rotation and ownership story.

Domain scope is another gating point. An administrator needs to define which domains an ACME endpoint can issue for and whether subdomains or wildcards are allowed. That is a useful security control, but it changes the onboarding workflow. A new application domain now needs approval, validation, and a test issuance before the workload can rely on automated renewal.

The control plane also moves into AWS. If your ACME clients run in an on-premises data center, another cloud, or a private Kubernetes cluster, verify that they can reach the ACM endpoint over the intended network path. Test from the same runtime identity and network location the production client will use. A successful request from a laptop proves very little.

## Observe both issuance and installation

A certificate can be issued successfully and still not be serving the new certificate. That is why ACM-level telemetry is only half of the monitoring design.

Track endpoint operations and issuance failures through CloudTrail and CloudWatch. Alert on failure, not only on certificates that are already close to expiry. The earlier the failure is visible, the more time the team has to fix EAB credentials, domain scope, DNS validation, connectivity, or client configuration.

Then add workload-level checks. For cert-manager, monitor renewal and challenge status. For a VM or container, verify that the web server, ingress controller, or load balancer is actually serving the newly issued certificate. A simple external check comparing the served certificate with the expected ACM certificate can catch an installation hook that silently failed.

Use one correlation identifier or a common certificate name across the ACME client logs, CloudTrail events, ACM records, and endpoint health checks. That turns a renewal incident from a collection of disconnected clues into a timeline.

## Plan for bursts and quotas

Renewal events are not always evenly distributed. A cluster restart, a fleet rollout, or a shared certificate policy can cause many clients to request work at the same time. Rate limits can be reached before anyone notices that the renewal window is shrinking.

Separate environments or domain boundaries when that helps isolate failures. Development traffic should not consume the same endpoint capacity that production depends on. Stagger renewal schedules where the client allows it, and test DNS validation limits as well as ACM limits.

Do not wait for production volume to discover the quota model. Estimate the maximum number of concurrent requests, add retry behavior with backoff, and document which limits require a service quota request or a different endpoint design.

## Hybrid estates need a phased migration

Kubernetes teams can use cert-manager with ACM ACME, but each issuer still needs a clear credential and namespace ownership model. On-premises workloads need network access and a way to prove that the newly issued certificate was installed. During a migration, some domains will use ACM while others remain on the previous provider.

That is normal. Migrate by environment or criticality rather than changing every endpoint at once. Start with a non-critical domain, run the full issuance and installation path, then exercise renewal failure and recovery. Keep the old path available until the new one has survived a complete cycle.

## Drill the failure before the deadline

A renewal incident is a good candidate for a controlled game day:

1. Use a non-production endpoint and intentionally invalidate the EAB credential or stop the client.
2. Confirm the expected alarms and CloudTrail events appear.
3. Follow the recovery runbook to restore the credential, retry issuance, and install the certificate.
4. Verify the certificate served by the workload, not just the certificate stored in ACM.
5. Measure time to detect and time to recover.

The source article frames the work around increasingly short validity windows. That is the forcing function. Automation reduces manual effort, but only observability and rehearsal tell you whether the automation is dependable.

ACM ACME support is a good compatibility bridge for existing clients. Treat it as a platform migration with a measurable operating model, not as a one-line endpoint change.