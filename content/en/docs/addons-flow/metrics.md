---
title: "Metrics"
---

## Addon Operator Metrics

Addon operator metrics can be found in [PromLens](https://promlens.devshift.net/)
in the `osd-observatorum-production` datasource.

The metrics are configured in the [lifecycler repo](https://gitlab.cee.redhat.com/service/lifecycler/)
and the [managed-cluster-config repo](https://github.com/openshift/managed-cluster-config).
See the following two merge requests for reference: [lifecycler MR](https://gitlab.cee.redhat.com/service/lifecycler/-/merge_requests/6/diffs)
and [managed-cluster-config MR](https://github.com/openshift/managed-cluster-config/pull/1281/files).

## `csv_succeeded` and `csv_abnormal` Metrics

`csv_succeeded` and `csv_abnormal` Metrics can be found in [production Thanos](https://infogw-proxy.api.openshift.com/)
and [stage Thanos](https://infogw-proxy.api.stage.openshift.com/).

It can be useful to query for `csv_succeeded` and `csv_abnormal` Metrics
by operator name, for example `csv_succeeded{name=~"ocs-operator.*"}`, or
by cluster id, for example `csv_succeeded{_id="049ea229-55dd-4e30-a2f0-87ae1dd37de6"}`.
