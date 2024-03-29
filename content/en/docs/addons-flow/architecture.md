---
title: "Addons Flow Architecture"
date: 2022-12-15T00:53:51+01:00
---

Add-Ons are Operators. As such, Add-Ons are installed using typical Operator
objects, like `Subscription`, `OperatorGroup` and `CatalogSource`.

To get those objects into the OpenShift clusters, we rely on
[OCM](https://cloud.redhat.com/openshift/) and
[Hive](https://github.com/openshift/hive). You can read more
about Hive in this [blog post](https://cloud.redhat.com/blog/openshift-hive-cluster-as-a-service).

## Deployment

Our input is the Add-On metadata file
(`managed-tenants/addons/<addon_name>/metadata/<stage|prod>/addon.yaml`)
and the corresponding bundles directories
(`managed-tenants-bundles/addons/<addon_name>/`).
With that in place, we will:

* Build the Operator catalog container image.
  * Push the catalog image to our organization repository in Quay.io.
* Generate a
  [SyncSet](https://github.com/openshift/hive/blob/master/docs/syncset.md#syncset-object-definition)
  with the Operator install objects.
  * Apply the SyncSet to Hive.
* Generate the OCM API payload.
  * Post the payload to OCM.

The Managed Tenants CI is in charge of processing the input and deploying all
the artifacts. This image shows the data flows:

![Data Flows](/addon-syncset-installation.png)

With that in place, OCM will present an "Add-Ons" tab, listing all the Add-Ons
that your organization has quota for. Example:

![Data Flows](/architecture_ocm_ui.png)

## Installation

When you click "Install" in the OCM Web UI, under the hood, OCM creates a
`SyncSet` [object](https://github.com/openshift/hive/blob/master/docs/syncset.md#syncset-object-definition)
in Hive. The `SyncSet` object references the cluster in which the addon was just
installed in the `clusterDeploymentRefs` field.

![Data Flows](/architecture_install_flow.png)
[excalidraw](https://excalidraw.com/#room=71d4b0273d4404dbbebf,Pk5KFYj9fFvXSvObY6juCA)

From there, OLM will take over, installing the Operator in the OpenShift
cluster. While OLM is installing the Operator, OCM will keep polling the
telemetry  data reported by the cluster, waiting for the `csv_succeeded=1`
metric from that Operator:

![Data Flows](/architecture_telemetry_wait.png)

At some point, when the Operator is fully installed, OCM will reflect that in
the Web UI:

![Data Flows](/architecture_telemetry_done.png)

## Addon Status Lifecycle

![Addon Status Lifecycle](/addon-status-ocm.png)

## Deprecated SelectorSyncSet Installation

![Data Flows](/architecture_data_flow.png)
When you click "Install" in the OCM Web UI, under the hood, OCM will apply
a label to the corresponding `ClusterDeployment` object in Hive.

That label is the same label used in the `SelectorSyncSet` as a `matchLabel`.

With the `ClusterDeployment` label now matching the `SelectorSyncSet` label,
Hive will apply all the objects in the `SelectorSyncSet` to the target cluster:
