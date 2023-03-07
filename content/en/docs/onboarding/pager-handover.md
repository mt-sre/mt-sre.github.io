---
title: Pager Handover
weight: 50
images: []
toc: true
---

## Transition Overview

This stage focuses on the viability of the service. If, after an agreed amount of time of
being in production(defaults to a 4-week rolling window), the service meets its SLOs then
it will be considered viable by SRE and the pager will be transferred to the Addon SRE
team.

## Viability Steps

- The service runs in production and SLO data is collected.
- An Addon SRE reviews the SLOs and SOPs.
- If the SLOs are met, the service will be transitioned to Addon SRE.
- Any critical alerts will be routed to the Addon SRE 24x7 PagerDuty escalation policy.

## Further Links

For information on how to set up the PagerDuty integration for your addon,
the [PagerDuty](../creating-addons/monitoring/pagerduty_integration.md) section.
