---
title: Interrupt Catcher
linkTitle: IC
---

The Interrupt Catcher is the entry-point for any tenant to get help,
ask questions and raise issues. It's our interface with our Tenants.

The MT-SRE Team member with the Interrupt Catcher responsibility can
be reached out via the @mt-sre-ic Slack handle, in
the #forum-managed-tenants channel.

## Coverage

Each working day has 15 hours of IC "Follow The Sun" coverage:

* APAC: From 4:30 to 9:30 UTC
* EMEA: From 9:30 to 14:30 UTC
* NASA: From 14:30 to 19:30 UTC

Work items generated outside that time-frame will be picked up
in the next FTS shift.

PagerDuty Schedule: <https://redhat.pagerduty.com/schedules#PM3YCH1>

## Response Time

| Service Level Indicator (SLI)                        |  SLO Time   | MT-SRE "Goal Time" |
| ---------------------------------------------------- | :---------: | :----------------: |
| MR's to managed-tenants repositories                 |   24 FTSH*  |      4 FTSH*       |
| #forum-managed-tenants Slack messages, misc. support | best effort |      4 FTSH*       |

*FTSH: Follow The Sun working Hours

## Responsibilities

* Review Merge Requests created by the MT-SRE Tenants on the
  "Surfaces" repositories (listed in the next section)
* Respond to the alerts in the #sd-mt-sre-alert Slack channel
* Respond to incidents from PagerDuty
* Respond to questions and requests from the MT-SRE tenants in
  the #forum-managed-tenants Slack channel
* Engage on incidents with Addons that are on-boarding to the
  MT-SRE
* Handover the outstanding work to the next IC

## Surfaces

* Slack channels:
  * `#sd-mt-sre-info`
  * `#mt-cs-sre-teamchat`
  * `#mt-cs-sre-teamhandover`
  * `#forum-managed-tenants`
* Mailing lists:
  * managed-tenants@redhat.com
* Git Repositories:
  * <https://gitlab.cee.redhat.com/service/managed-tenants-bundles>
      (not automated)
  * <https://gitlab.cee.redhat.com/service/managed-tenants>
      (tenants-related MRs, partially automated)
  * <https://gitlab.cee.redhat.com/service/managed-tenants-manifests>
      (fully automated)
  * <https://gitlab.cee.redhat.com/service/managed-tenants-sops>
      (tenants-related MRs, partially automated)
