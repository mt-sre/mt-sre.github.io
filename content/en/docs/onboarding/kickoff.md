---
title: Kickoff
weight: 100
images: []
toc: true
---

This document outlines the required process to onboard an addon into the addon flow and with an addon SRE team.

![Addon Onboarding](../images/addon_onboarding.png)

We have three distinct onboarding stages:

* Kickoff
* Onboarding
* SRE Transition Period


## 1. Kickoff

### Kickoff Overview

The Service Owners starts [ROMS](https://docs.google.com/spreadsheets/d/1qgK3_Yt3wRTyCqxVICrobDNmwMwjoSDPrQvy6vbTPYI/edit#gid=46944899)
(including SAG review). SRES Architects will evaluate the service, determine whether it's suitable and viable.
If accepted it will receive a prioritization rating from the SRES Architects, and finally assigned to Addon SRE by the
Addon SRE Manager.

**IMPORTANT**: As an addon provider you can self-service onto OCM stage via the addon flow and start prototyping
before completing ROMS. You are free to experiment and kick the tires! The only restrictions in place are the available
support forums to the Addon SRE team. These are limited to [#forum-managed-tenants](https://coreos.slack.com/archives/C01L46M0FQC)
and to the weekly `SD: Layered Products Sync` call (You can request an invite via #forum-managed-tenants)

### Kickoff Steps

- Service Owner creates an Epic in the [SDE](https://issues.redhat.com/projects/SDE) board.
    - Service Owner will request sign-off from SRES Architects: [Paul Bergene](https://rover.redhat.com/people/profile/pbergene), [Jaime Melis](https://rover.redhat.com/people/profile/jmelis) and [Karanbir Singh](https://rover.redhat.com/people/profile/kasingh). Note that the preferred communication channels are the JIRA or the [#sd-org channel](https://app.slack.com/client/T027F3GAJ/CCNL6UJ9W/details/top).
    - Service Owner includes reference to the [SRES Onboarding Questionnaire](https://docs.google.com/document/d/1TjGSJdyIfsZ_90ea_bWEpj8y1pOjpq6tq5xnHrn5K68/edit) (part of ROMs) in the epic
    - The epic should also reference a Service Definition Document. Service Definition examples: [OSD](https://www.openshift.com/products/dedicated/service-definition), [OCS](https://docs.google.com/document/d/1AEx6INWMAf3iyE9P_daS4XwaL5Pl2JXHp1jg-OMPhXI/edit#heading=h.vz6sfx5le5cl), [RHODS](https://docs.google.com/document/d/1MKLhNu6ALmuA4UdoqfbvEbLrpGSDcSRaKvMTQX28jdU/edit), [RHOAM] (https://access.redhat.com/articles/5534341)
- SRES Architects and the Addon SRE team lead will review the epic, discuss with the Service Owners and accept or reject it. SRES Architects will also assign a priority.
- Addon SRE lead will scope out the work needed to onboard the service into Addon SRE.

## 2. Onboarding

### Onboarding Overview

The Service Owners, with the assistance of the Addon SRE team, will deploy the service to production while the SLOs
are being implemented and fine-tuned. The Addon SRE onboarding team will work through the [Addon SRE Acceptance Criteria Checklist]([../process/sre_checkpoints.md](https://docs.google.com/document/d/1c0drua0YciULkEuYsTCAM1E_JJK-b2TgjTeFnjxh_6c/edit))
process and if successful, the Service Owners will be allowed to deploy to production. Once it's running in production,
the service can begin its Transition Period.

## 3. Transition Period

### Transition Overview

This stage focuses on the viability of the service. If, after an agreed amount of time (defaults to a 4-week rolling window),
the service meets its SLOs then it will be considered viable by SRE and the pager will be transferred to the Addon SRE team.

### Viability Steps

- The service runs in production and SLO data is collected.
- An Addon SRE reviews the SLOs and SOPs.
- If the SLOs are met, the service will be transitioned to Addon SRE.
- Any critical alerts will be routed to the Addon SRE 24x7 PagerDuty escalation policy.
