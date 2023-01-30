---
title: ROMS
weight: 20
images: []
toc: true
---

Please start at the ROMS documentation on the Source
[here](https://source.redhat.com/communities/communities_of_practice/cross_cutting_co/cloud_services_cop/wiki/draft_how_to_get_started_with_roms_repeatable_onboarding_for_managed_services)
to get an overview of managed services and ROMS.

### Kickoff Overview

The Service Owners starts [ROMS Checklist](https://docs.google.com/spreadsheets/d/1qgK3_Yt3wRTyCqxVICrobDNmwMwjoSDPrQvy6vbTPYI/edit#gid=1141875805)
(including SAG review). SRES Architects will evaluate the service, determine whether it's suitable and viable.
If accepted it will receive a prioritization rating from the SRES Architects, and finally assigned to an SRE team by the
Addon SRE Manager.

**IMPORTANT**: As an addon provider you can self-service onto OCM stage via the addon flow and start prototyping
before completing ROMS. You are free to experiment and kick the tires! The only restrictions in place are the available
support forums to the Addon SRE team. These are limited to [#forum-managed-tenants](https://coreos.slack.com/archives/C01L46M0FQC)
and to the weekly `SD: Layered Products Sync` call (You can request an invitation via #forum-managed-tenants)

### Kickoff Steps

- Service Owner creates an Epic in the [SDE](https://issues.redhat.com/projects/SDE) board.
  - Service Owner will request sign-off from SRES Architects: [Paul Bergene](https://rover.redhat.com/people/profile/pbergene),
      [Jaime Melis](https://rover.redhat.com/people/profile/jmelis) and [Karanbir Singh](https://rover.redhat.com/people/profile/kasingh).
      Note that the preferred communication channels are the JIRA or the [#sd-org channel](https://app.slack.com/client/T027F3GAJ/CCNL6UJ9W/details/top).
  - Service Owner includes reference to the [SRES Onboarding Questionnaire](https://docs.google.com/document/d/1TjGSJdyIfsZ_90ea_bWEpj8y1pOjpq6tq5xnHrn5K68/edit)
       (part of ROMS) in the epic.
  - The epic should also reference a Service Definition Document. Service Definition examples: [OSD](https://www.openshift.com/products/dedicated/service-definition),
       [OCS](https://docs.google.com/document/d/1AEx6INWMAf3iyE9P_daS4XwaL5Pl2JXHp1jg-OMPhXI/edit#heading=h.vz6sfx5le5cl),
       [RHODS](https://docs.google.com/document/d/1MKLhNu6ALmuA4UdoqfbvEbLrpGSDcSRaKvMTQX28jdU/edit), [RHOAM](https://access.redhat.com/articles/5534341)
- SRES Architects and the Addon SRE team lead will review the epic, discuss with the Service Owners and accept or reject it.
  SRES Architects will also assign a priority.
- Addon SRE lead will scope out the work needed to onboard the service into Addon SRE.
