---
title: "PagerDuty Integration"
linkTitle: "PagerDuty"
---

The PagerDuty integration is configured in the `pagerduty` field in the
[addon.yaml metadata file](https://github.com/mt-sre/managed-tenants-cli/blob/main/docs/tenants/zz_metadata_schema_generated.md).
Given this configuration, a secret with the specified name is created in the
specified namespace by the [PagerDuty Operator](https://github.com/openshift/pagerduty-operator),
which runs on Hive. The secret contains the `PAGERDUTY_KEY`.
