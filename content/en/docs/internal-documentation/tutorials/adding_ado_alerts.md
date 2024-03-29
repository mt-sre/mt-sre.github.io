---
title: Adding ADO alerts
linkTitle: Adding ADO alerts
---

This guide describes how to add alerts for addon-operator in the OSD RHOBS tenant.

## Background

Since the addon-operator doesn’t have a specific tenant in RHOBS with a service account,
the addon-operator metrics are scraped to the OSD tenant with its own service account,
so instead of adding rules using [obsctl cli](https://github.com/observatorium/obsctl) and
syncing them using [obsctl-reloader](https://github.com/rhobs/obsctl-reloader)
SRE-P has a repo called [rhobs-rules-and-dashboards](https://gitlab.cee.redhat.com/service/rhobs-rules-and-dashboards)
which, based on the tenant, will automatically sync rules defined in the repo to app-interface.

## Overview

We need to have the following prerequisites:

If the tenant is not defined in the repo, we need to follow the process defined
[here](https://gitlab.cee.redhat.com/service/rhobs-rules-and-dashboards/-/tree/main/rules#registering-a-new-tenant)
which explains how to register the tenant in app-interface and how to configure obsctl-reloader
to sync rules for the particular tenant.

The OSD tenant is already registered and the obsctl-reloader configuration is already
defined [here](https://gitlab.cee.redhat.com/service/app-interface/-/blob/master/data/services/rhobs/observatorium-mst/cicd/saas.yaml#L127)
(we should see osd in **MANAGED_TENANTS** parameter in the **observatorium-mst-common** named
item and secrets are defined [here](https://gitlab.cee.redhat.com/service/app-interface/-/blob/master/data/services/rhobs/observatorium-mst/namespaces/app-sre-stage-01/observatorium-mst-stage.yml#L79))

## Steps

we can define rules for the addon-operator metrics in the [rhobs-rules-and-dashboards](https://gitlab.cee.redhat.com/service/rhobs-rules-and-dashboards)
repo in the [rules/osd](https://gitlab.cee.redhat.com/service/rhobs-rules-and-dashboards/-/tree/main/rules/osd)
folder (the tenants are added as individual folders) and tests for the prometheus rules are
defined in the [tests/rules/osd](https://gitlab.cee.redhat.com/service/rhobs-rules-and-dashboards/-/tree/main/test/rules/osd)
folder.
The suggested file naming conventions are described [here](https://gitlab.cee.redhat.com/service/rhobs-rules-and-dashboards/-/tree/main/rules#one-folder-per-targeted-rhobs-tenant).

tldr: create file name with `.prometheusrule.yaml` as a suffix,
add tenant label to the `PrometheusRule` object.

### Adding promrules

For creating a PoC alert we have selected the `addon_operator_addon_health_info` metric
since it basically explains the addon health for a particular `version` and `cluster_id`.
The metrics for creating alerts can be decided and the promql queries
can be tested out from [promlens stage](https://promlens.stage.devshift.net)
or [promlens prod](https://promlens.devshift.net).
The metrics are scraped from addon-operator to the
observatorium-mst-stage (stage) / observatorium-mst-prod (prod) datasources.

Sample `addon_operator_addon_health_info` metric data:

```yaml
addon_operator_addon_health_info{_id="08d94ae0-a943-47ea-ac29-6cf65284aeba", container="metrics-relay-server", endpoint="https", instance="10.129.2.11:8443", job="addon-operator-metrics", name="managed-odh", namespace="openshift-addon-operator", pod="addon-operator-manager-7c9df45684-86mh4", prometheus="openshift-monitoring/k8s", receive="true", service="addon-operator-metrics", tenant_id="770c1124-6ae8-4324-a9d4-9ce08590094b", version="0.0.0"}
```

This particular metric gives information about
`version`, `cluster_id (_id)` and `addon_name (name)` which can be used to create the alert.

- We should ignore the metrics with version `"0.0.0"`
- The addon health is `"Unhealthy"` if no version exists for the particular
addon with [value 1](https://github.com/openshift/addon-operator#monitoring-and-metrics)
- In theory, the latest version for the particular addon
should have the value of 1. If not, then the addon is `"Unhealthy"`

The prometheus rule for the addon_operator_addon_health_info metric is defined [here](https://gitlab.cee.redhat.com/service/rhobs-rules-and-dashboards/-/blob/main/rules/osd/addon-operator-addon-health-info-rules.prometheusrule.yaml)

```yaml
expr: (count by (name, _id) (addon_operator_addon_health_info{version!="0.0.0"})) - (count by (name,_id) (addon_operator_addon_health_info{version!="0.0.0"} == 0)) == 0
```

Explanation:

We aggregate all metrics on `name` and `_id` and count the non-0 value metrics.
If the count is 0 (implying there has been no non-0 value metrics), raise the alert.

### Writing tests for the promrules

First create a `<alertname>.promrulestests.yaml` file in the test/rules/<tenant>/ folder,
It is advised to test out different scenarios for the edge cases so
that the expr defined in the rules/<tenant> folder will raise alert if say the addon is `"Unhealthy"`.
Try out different scenarios by defining different series as
input to the test rules such as [here](https://gitlab.cee.redhat.com/service/rhobs-rules-and-dashboards/-/blob/main/test/rules/osd/addon-operator-addon-health-info-rules.prometheusrulestests.yaml#L8).

The tests can be validated by running: `make test-rules`
in the rhobs-rules-and-dashboards directory.
NOTE: Make sure that the tests are defined in the tests/rules/<tenant> folder

Since the pagerduty config is defined on a per tenant basis which is `osd tenant` in this case,
the alert will be triggered and paged to SRE-P,
so a proper runbook should be added redirecting the alert to lp-sre folks.

The runbook links/ SOP links can be validated by running: `make check-runbooks`
