---
title: "ACM ACME support turns certificate automation into a governance problem"
date: 2026-06-30
author: "Emiliano Montesdeoca"
description: "AWS Certificate Manager now supports ACME for public certificates, giving teams a standard automation path while keeping domain control, audit, and policy centralized."
tags:
  - aws
  - acm
  - tls
  - security
  - operations
source: "https://aws.amazon.com/blogs/aws/automate-public-tls-certificate-issuance-with-acme-support-in-aws-certificate-manager/"
draft: false
---

Certificate automation is becoming less optional. Validity periods are getting shorter, and manual renewal processes are exactly the kind of quiet operational risk that eventually becomes an outage.

AWS has now added [ACME support for public certificates in AWS Certificate Manager](https://aws.amazon.com/blogs/aws/automate-public-tls-certificate-issuance-with-acme-support-in-aws-certificate-manager/). The headline is compatibility with ACMEv2 clients such as Certbot, cert-manager, and acme.sh. The bigger builder impact is that certificate issuance can become standardized without moving governance outside AWS.

## What changed

ACM now provides a managed ACME server endpoint for public certificates issued by Amazon Trust Services. Teams can point existing ACME clients at ACM, request certificates through the protocol they already know, and still have ACM visibility, CloudTrail logging, CloudWatch metrics, and certificate expiry notifications.

The source article also highlights two controls that matter in real organizations:

- domain scopes at the endpoint level,
- External Account Binding credentials for client registration.

That means a central PKI or platform team can validate domains once, restrict what clients can request, and avoid distributing DNS credentials to every application team.

## Why this matters

Most certificate incidents are not caused by cryptography being hard. They are caused by ownership being unclear.

One team owns DNS. Another owns the ingress controller. Another owns the application. Someone installed a certificate manually two years ago. Nobody remembers the renewal path until browsers start showing errors.

ACME support in ACM gives builders a cleaner operating model:

- Kubernetes teams can keep using cert-manager.
- VM and container teams can keep using common ACME clients.
- Platform teams can centralize issuance policy and audit.
- Security teams can see certificates in ACM instead of chasing external CA dashboards.

That is a good trade: keep the open protocol at the edge, centralize governance at the account or organization boundary.

## The architecture trade-offs

This does not remove the need for certificate lifecycle design.

First, decide who owns ACME endpoints. If every workload account creates its own endpoint with broad wildcard permissions, you have recreated the sprawl problem. A better pattern is to define endpoints per domain boundary, environment, or platform domain, then scope allowed names carefully.

Second, treat External Account Binding credentials like bootstrap secrets. They should be short-lived where possible, stored securely, and rotated when registration flows change.

Third, think about wildcard issuance. Wildcards are convenient, but they expand blast radius. Many production environments should prefer exact domains and subdomains unless there is a strong operational reason.

Finally, price and volume behavior matter. Moving high-churn ACME issuance into ACM should be costed, especially for systems that create many short-lived certificates across tenants.

## What builders should do next

If I were running a platform team, I would start with an inventory:

- Which public certificates are already in ACM?
- Which are issued by external ACME providers?
- Which workloads use cert-manager or custom renewal scripts?
- Which teams still depend on manual DNS validation?

Then I would pilot ACM ACME with one non-critical domain and one existing ACME client. The goal is not only to issue a certificate. The goal is to prove the operating model: endpoint ownership, scope policy, audit trail, renewal alarms, and incident response.

The useful part of this launch is not that Certbot can talk to AWS. It is that certificate automation can finally be treated as platform governance instead of a collection of one-off scripts.
