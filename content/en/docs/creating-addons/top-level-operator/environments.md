---
title: "Environments"
date: 2022-12-15T00:53:51+01:00
---

## Mandatory environments

Add-ons are normally deployed to two environments:

- `ocm stage`: development/testing - All add-ons **must** deploy to this environment before being released to production.
- `ocm production`: once the deployment in stage has been accepted/approved/reviewed it can be promoted to production via `/lgtm` by your SRE team.


We recommend the `ocm stage` and `ocm production` add-on metadata be as similar as possible.

## SLOs

`ocm stage` have no SLO and operates with best effort support from Add-on SRE, SREP, and App-SRE
`osd stage cluster` have no SLO and operates with best effort support from Add-on SRE, SREP, and App-SRE
`ocm production` environments are subject to App-SRE [SLOs](https://gitlab.cee.redhat.com/app-sre/contract/-/blob/master/README.md#appsre-service-level-objectives).
`osd production cluster` environments are subject OSD [SLOs](https://source.redhat.com/groups/public/openshiftplatformsre/wiki/faq_openshift_dedicated_service_level_objectives).

## Additional Environments (via duplicate add-ons)
Some add-on providers have had use cases which require additional add-on envs. While we only have `ocm stage` and `ocm prod`,
managed-tenants may be leveraged to deploy to an additional add-on (like edge or internal). Today we don't recommend
this practice due to the need to clone all add-on metadata which increases the risk for incorrect metadata going to
production/customer clusters.

If you need to do the above, please reach out to your assigned SRE team for guidance first.
