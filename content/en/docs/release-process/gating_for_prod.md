---
title: Gating for Production
weight: 100
images: []
toc: true
---

![Agreements](/Addons-prod-view.png)

This process intends to allow any addon to get to PROD, but not be sold to customers. Before an addon is enabled for
customer consumption, various teams have to validate and agree on the content, product, and service.

The focus is on three areas:

### Service & Support

1. Layered SRE (if present): payload consistency and policy for upgrades,
testing, supportability, escalation paths + IMS; complements the Managed Tenants SRE
scope, but does not replace it.
1. SRE-P: OSD/ ROSA 3-R’s
1. CEE : SBR & context + Docs + IMS
1. OSD e2e: for CD Signal
1. OSD SRE: Ensure OSD sanity
1. Platform & Payload

### PM / Biz

1. OSD / ROSA
1. OCM
1. Addon / Product

### Spec & technical patterns

1. ServiceDev A : UX & CS
1. ServiceDev B : SKU, Quota, rBac - AMS
1. Managed Tenants SRE : consistency & Policy for payload delivery, upgrades, testing

## Indicating agreement

Each team in the scope above needs to explicitly indicate agreement for the go-live signal. This can be done via
a +1 / LGTM signal on the Merge Request that 'Releases Content for non-Red Hat consumption' or via a linked JIRA
where the SKU rules are requested to go live. The go-Live signal is normally defined as either when the SKUs are enabled
with AMS delivering quota reconcile against it, or it could just be a 'Free with OSD' / 'Free overall'.

## Get to Prod consideration

Promoting an addon to prod does not automatically mean it can be consumed by
a non-Red Hat person e.g. a customer. Therefore, a
reduced set of criteria applies for getting content and the addon itself
available to  Dev, QE, CEE, SRE, Docs and other associated teams.

Baseline criteria then needed for the promote-to-prod is reduced to:

1. Managed Tenants SRE agrees on consistency and policy
2. Passing thorough testing via [OSD e2e](https://github.com/openshift/osde2e) for CD Signal
3. SREP approve and sign off on early content landing in production
4. Dev team has signed off on content being consumed

As a fallout of this reduced criteria the corresponding
OSD cluster that lands this addon, gets a potentially lower Service assurance
from the various SRE and CEE teams.

## Approval and example data flow

![Flow](/Addons-Rel-flow.png)

The yellow box indicates short-lived, non-customer facing infra.

Primary Takeaway:

1. As a developer you can deliver content into Integration or Stage as long as you meet the basic technical requirements.
Other than the eng team and SRE baseline requirements there are no gates applied.
1. To get content available in Production OCM, a wider set of considerations must be met - including sign off
from Managed Tenants SRE and OSD-e2e that the platform is not being compromised, and no forward-looking
feature work is being exposed/exploited.
1. To release the product to customers, ALL criteria must be met, including, but not limited to,
demonstrated viability and support cover, service definitions and demonstrated SLIs,
before you can. Titles like Beta, Alpha, GA, Tech
Preview etc. are removed from the state of service, and are not something SRE participates in.
1. As a part of making your content available to customers, SRE will agree on capacity
and velocity numbers based things like cost of service, human and service toil etc.
The expectation is that there will be no more customers than the agreed capacity, and
the instance of the product will remain within the agreed velocity and service goals.
