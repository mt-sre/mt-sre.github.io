---
title: Addon External Resources
---

Whenever an add-on creates a resource externally to the cluster (eg, S3, RDS Instances, etc.) The flag
`hasExternalResources` must be set to true in the addon metadata:

```yaml
...
hasExternalResources: true
...

```

## Addon Deletion

The Addon is responsible for all resources it creates and needs to clean them up when it is removed.

The full deletion process:

1. Addon deletion is triggered via the OCM UI or Cluster deletion.
2. A "Deletion" ConfigMap is created in the Addons Namespace. [1]
3. **The Addon cleans up its resources.**
4. **The Addon signals it is ready to be removed by removing its ClusterServiceVersion.**
5. OCM will wait for the `csv_succeeded` metric to transition to `false`. [2]
6. OCM will remove the `Subscription`, `CatalogSource`, and `Namespace` from the cluster.
7. The deletion process is complete.

Each Addon is responsible for implementing points 3. and 4. within one of their operator's controllers.

**[1] Deletion ConfigMap:**

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: <ADDON.metadata['id']>
  namespace: <ADDON.namespace>
  labels:
    <ADDON.metadata['label']>-delete: 'true'
```

**[2] csv_succeeded metric**
As Clusters Service and Hive is unaware of resources owned and controlled by the operator it relies on metrics to
evaluate the state of the add-on. Notably it uses `csv_succeeded{}` to determine the state, when an add-on is marked
for deletion, an absence of the `csv_succeeded{}` will allow Clusters Service and Hive remove any remaining SyncSets
associated with the add-on and clean the `Cluster Deployment` of the add-on

> **_NOTE:_** A label will be added to add-on namespace `{{ADDON.metadata['label']}}-delete: 'true'` on addon deletion.
> This is deprecated, add-ons should watch for existence of the "deletion ConfigMap" to trigger resource clean up.
