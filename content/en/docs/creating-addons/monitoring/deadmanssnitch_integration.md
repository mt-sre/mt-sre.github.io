---
title: "Dead Man's Snitch Operator Integration"
linkTitle: "Dead Man's Snitch"
---

## Overview

[Dead Man's Snitch (DMS)](https://deadmanssnitch.com/) is essentially a constantly firing
prometheus alert and an external receiver (called a snitch) that will alert should the
monitoring stack go down and stop sending alerts. The generation of the snitch URLs is
done dynamically via the
[DMS operator](https://github.com/openshift/deadmanssnitch-operator),
which runs on hive and is owned by SREP. The snitch URL shows up in a [secret](#generated-secret).

## Usage

The Add-On metadata file (`addon.yaml`) allows you to provide
a `deadmanssnitch` field (see `deadmansnitch` field in
the Add-On metadata file
[schema documentation](https://github.com/mt-sre/managed-tenants-cli/blob/main/docs/tenants/zz_metadata_schema_generated.md)
for more information).
This field allows you to provide the required Dead Man's Snitch integration configuration.
A `DeadmansSnitchIntegration`resource is then created and applied to Hive alongside the Add-On
[SelectorSyncSet (SSS)](https://github.com/openshift/hive/blob/master/docs/syncset.md#selectorsyncset-object-definition).

### DeadmansSnitchIntegration Resource

The default DMS configurations which will be created if you specify the bare minimum
fields under 'deadmanssnitch' field in addon metadata:

```yaml
- apiVersion: deadmanssnitch.managed.openshift.io/v1alpha1
  kind: DeadmansSnitchIntegration
  metadata:
    name: addon-{{ADDON.metadata['id']}}
    namespace: deadmanssnitch-operator
  spec:
    clusterDeploymentSelector: ## can be overridden by .deadmanssnitch.clusterDeploymentSelector field in addon metadata
      matchExpressions:
      - key: {{ADDON.metadata['label']}}
        operator: In
        values:
        - "true"

    dmsAPIKeySecretRef: ## fixed
      name: deadmanssnitch-api-key
      namespace: deadmanssnitch-operator

    snitchNamePostFix: {{ADDON.metadata['id']}} ## can be overridden by .deadmanssnitch.snitchNamePostFix field in addon metadata

    tags: {{ADDON.metadata['deadmanssnitch']['tags']}} ## Required

    targetSecretRef:
      ## can be overridden by .deadmanssnitch.targetSecretRef.name field in addon metadata
      name: {{ADDON.metadata['id']}}-deadmanssnitch
      ## can be overridden by .deadmanssnitch.targetSecretRef.namespace field in addon metadata
      namespace: {{ADDON.metadata['targetNamespace']}}
```

### Examples of `deadmanssnitch` field in `addon.yaml`

```yaml
id: ocs-converged
....
....
deadmanssnitch:
  tags: ["ocs-converged-stage"]
....
```

```yaml
id: managed-odh
....
....
deadmanssnitch:
  snitchNamePostFix: rhods
  tags: ["rhods-integration"]
  targetSecretRef:
    name: redhat-rhods-deadmanssnitch
    namespace: redhat-ods-monitoring
....
```

```yaml
id: managed-api-service-internal
....
....
deadmanssnitch:
  clusterDeploymentSelector:
    matchExpressions:
    - key: "api.openshift.com/addon-managed-api-service-internal"
      operator: In
      values:
      - "true"
    - key: "api.openshift.com/addon-managed-api-service-internal-delete"
      operator: NotIn
      values:
      - 'true'
  snitchNamePostFix: rhoam
  tags: ["rhoam-production"]
  targetSecretRef:
    name: redhat-rhoami-deadmanssnitch
    namespace: redhat-rhoami-operator
```

### Generated Secret

A secrete will be generated (by default in the same namespace as your addon) with the `SNITCH_URL`.
Your add-on will need to pick up the generated secret in cluster and inject it into your
alertmanager config. Example of in-cluster created secret:

```yaml
kind: Secret
apiVersion: v1
metadata:
  namespace: redhat-myaddon-operator
  labels:
    hive.openshift.io/managed: 'true'
data:
  SNITCH_URL: #url like https://nosnch.in/123123123
type: Opaque
```

### Alert

Your alertmanager will need a constantly firing alert that is routed to DMS:
Example of an alert that always fires:

```yaml
- name: DeadManSnitch
  interval: 1m
  rules:
    - alert: DeadManSnitch
      expr: vector(1)
      labels:
        severity: critical
      annotations:
        description: This is a DeadManSnitch to ensure RHODS monitoring and alerting pipeline is online.
        summary: Alerting DeadManSnitch
```

## Route

Example of a route that forwards the firing-alert to DMS:

```yaml
- match:
    alertname: DeadManSnitch
  receiver: deadman-snitch
  repeat_interval: 5m
```

## Receiver

Example receiver for DMS:

```yaml
- name: 'deadman-snitch'
  webhook_configs:
  - url: '<snitch_url>?m=just+checking+in'
    send_resolved: false
```

{{% alert %}}
Before going live with the SRES SRE team, they will need to manually point the `tags: ["my-addon-production"]`
in the Service Delivery DMS account to their pagerduty service.
{{% /alert %}}

Please log a JIRA with your assigned SRE team to have this completed at least one week
before going live with the SRE team.

### Current Example

- [RHOAM addon: DMS CR template](https://gitlab.cee.redhat.com/service/managed-tenants/-/blob/09cf5112e7dc5588c14f158d6490f7f1e7051c6a/addons/managed-api-service-internal/metadata/production/deadmanssnitch.yaml.j2)
- [RHOAM addon: extraResources](https://gitlab.cee.redhat.com/service/managed-tenants/-/blob/09cf5112e7dc5588c14f158d6490f7f1e7051c6a/addons/managed-api-service-internal/metadata/production/addon.yaml#L40)
  field in `addon.yaml`
- [RHODS addon: alertmanager configuration](https://github.com/red-hat-data-services/odh-deployer/blob/cb48c55725fd32fdc89a5ff29517b3f4cc0d1f54/monitoring/prometheus/prometheus.yaml)
