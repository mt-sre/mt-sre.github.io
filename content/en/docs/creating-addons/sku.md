---
title: "SKU"
date: 2022-12-15T00:53:51+01:00
description: >
  How to request a SKU for your addon.
---

**NOTE** MT-SRE do not influence SKU creation/priorities. You must work with SDB directly for this.

## Requesting a SKU

To request a SKU, please complete the following steps:

* Determine a unique quota ID for the addon. This should be
  lowercase with dashes and of the format `addon-<addon-name>`. For example:
  `addon-prow-operator`
* Create a JIRA Request at [Service Dev Team B](https://issues.redhat.com/projects/SDB/)
  with the subject `Request for new Add-On SKU in OCM` and the following information:
  * Add-On name.
  * Add-On owner.
  * Requested Add-On unique quota ID.
  * Additional information that would help qualify the ask, including goals,
      timelines, etc., you might have in mind.
* You will need at least your PM and the OCM PM's to sign off before the SKU
  is created. We expect to resolve these requests within 7 working days.

## Requesting SKU Attributes Changes

From time to time you may want to update some SKU fields like supported cloud providers, quota cost, product support, etc.
To do this:

* Create a JIRA Request at [Service Dev Team B](https://issues.redhat.com/projects/SDB/)
* Ping the ticket in #service-development-b Slack channel (@sd-b-team is the handle)
* This requires an update to be committed in-code in AMS, then deployed to stage and eventually prod (allow up to 7 working days).

## Current Status

To check current SKUs and attributes, see
[OCM Resource Cost Mappings](https://docs.google.com/spreadsheets/d/1HGvQnahZCxb_zYH2kSnnTFsxy9MM49vywd-P0X_ISLA/edit#gid=1479779350).
