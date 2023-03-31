---
title: Installing a specific version of an Addon in a staging environment
linkTitle: Installing a specific version of an Addon in a staging environment
---

Add-on services are typically installed using the OpenShift Cluster Manager web console, by selecting the specific addon from the **Add-ons** tab and clicking **Install**.
However, only the latest version of an addon service can be installed using the OpenShift Cluster Manager console.

In some cases, you might need to install an older version of an addon, for example, to test the upgrade of an add-on from one version to the next.
Follow this procedure to install a specific version of an addon service in a staging environment.

**IMPORTANT:**
Installing an addon service using this procedure is only recommended for testing upgrades in a staging environment and is not supported for customer-facing production workloads.

## Prerequisites

* You have the `version_select` capability added to your organization by creating a merge request to the [_ocm-resources respository_](https://gitlab.cee.redhat.com/service/ocm-resources).

    For more information about enabling the `version_select` capability, see [_organization YAML example_](https://gitlab.cee.redhat.com/service/ocm-resources/-/commit/35198592011391d4c296ff75d70a93a454104467) and [_merge request example_](https://gitlab.cee.redhat.com/service/ocm-resources/-/merge_requests/3271).

## Procedure

1. Create a JSON file with the addon service and addon version that you want to install. In this example, the JSON file is `install-payload.json`, the addon id is `reference-addon`, and the version we want to install is `0.6.7`.

    **Example**

    ```json
   {
     "addon": {
       "id": "reference-addon"
     },
     "addon_version": {
       "id": "0.6.7"
     }
   }
    ```

2. Set the `CLUSTER_ID` environment variable:

    ```bash
    export CLUSTER_ID=<your_cluster_internal_id>
    ```

3. Run the following API request to install the addon service:

    ```bash
    ocm post /api/clusters_mgmt/v1/clusters/$CLUSTER_ID/addons --body install-payload.json
    ```

4. Verify the addon installation:
    1. Log into your cluster:

        ```bash
        oc login
        ```

    2. Run the `oc get addons` command to view the addon installation status:

        ```bash
        $ oc get addons
        NAME              STATUS   AGE
        reference-addon   Pending  10m
        ```

    3. Optionally, run the `watch` command to watch the addon installation status:

        ```bash
        $ watch oc get addons
        NAME                 STATUS    AGE
        reference-addon      Ready     32m
        ```

    4. Review the addon service installation status and version:

        **Example**

        ```yaml
        $ oc get addons reference-addon -o yaml
       apiVersion: addons.managed.openshift.io/v1alpha1
       kind: Addon
       metadata:
         annotations:
         ...
         creationTimestamp: "2023-03-20T19:07:08Z"
         finalizers:
         - addons.managed.openshift.io/cache
         ...
       spec:
         displayName: Reference Addon
         ...
         pause: false
         version: 0.6.7
       status:
         conditions:
         - lastTransitionTime: "2023-03-20T19:08:10Z"
           message: ""
           observedGeneration: 2
           reason: FullyReconciled
           status: "True"
           type: Available
         - lastTransitionTime: "2023-03-20T19:08:10Z"
           message: Addon has been successfully installed.
           observedGeneration: 2
           reason: AddonInstalled
           status: "True"
           type: Installed
         lastObservedAvailableCSV: redhat-reference-addon/reference-addon.v0.6.7
         observedGeneration: 2
         observedVersion: 0.6.7
         phase: Ready
         ```

         In this example, you can see the addon version is set to `0.6.7` and `AddonInstalled` status is `True`.

5. If you do not want the addon service to automatically upgrade to the latest version after installation, delete the addon upgrade policy after installation is complete.

    1. List the upgrade policies:

        **Example**

        ```bash
        $ ocm get /api/clusters_mgmt/v1/clusters/$CLUSTER_ID/addon_upgrade_policies
        {
       "kind": "AddonUpgradePolicyList",
       "page": 1,
       "size": 1,
       "total": 1,
       "items": [
         {
          "kind": "AddonUpgradePolicy",
          "id": "991a69a5-ce33-11ed-9dda-0a580a8308f5",
          "href": "/api/clusters_mgmt/v1/clusters/22ogsfo8kd36bk280b6bqbi7l03micmm/addon_upgrade_policies/991a69a5-ce33-11ed-9dda-0a580a8308f5",
          "schedule": "0,15,30,45 * * * *",
          "schedule_type": "automatic",
          "upgrade_type": "ADDON",
          "version": "",
          "next_run": "2023-03-29T19:30:00Z",
          "cluster_id": "22ogsfo8kd36bk280b6bqbi7l03micmm",
          "addon_id": "reference-addon"
         }
        ]
        }
        ```

    2. Delete the addon upgrade policy:

        **Syntax**

        ```bash
        ocm delete /api/clusters_mgmt/v1/clusters/$CLUSTER_ID/addon_upgrade_policies/<addon_upgrade_policy_id>
        ```

        **Example**

        ```bash
        ocm delete /api/clusters_mgmt/v1/clusters/$CLUSTER_ID/addon_upgrade_policies/991a69a5-ce33-11ed-9dda-0a580a8308f5
        ```

    3. Verify the upgrade policy no longer exists:

        **Syntax**

        ```bash
        ocm get /api/clusters_mgmt/v1/clusters/$CLUSTER_ID/addon_upgrade_policies | grep <addon_upgrade_policy_id>
        ```

        **Example**

        ```bash
        ocm get /api/clusters_mgmt/v1/clusters/$CLUSTER_ID/addon_upgrade_policies | grep 991a69a5-ce33-11ed-9dda-0a580a8308f5
        ```

6. (Optional) If needed, recreate the addon upgrade policy manually.
    1. Create a JSON file with the addon upgrade policy information.

        **Example of automatic upgrade**

        ```json
        {
          "kind": "AddonUpgradePolicy",
          "addon_id": "reference-addon",
          "cluster_id": "$CLUSTER_ID",
          "schedule_type": "automatic",
          "upgrade_type": "ADDON"
        }
        ```

        **Example of manual upgrade**

        ```json
        {
          "kind": "AddonUpgradePolicy",
          "addon_id": "reference-addon",
          "cluster_id": "$CLUSTER_ID",
          "schedule_type": "manual",
          "upgrade_type": "ADDON",
          "version": "0.7.0"
        }
        ```

        In the example above, the schedule_type for the `reference-addon` is set to `manual` and the version to upgrade to is set `0.7.0`.
    1. Run the following API request to install the addon service:

        **Syntax**

        ```bash
        ocm post /api/clusters_mgmt/v1/clusters/$CLUSTER_ID/addon_upgrade_policies --body <your_json_filename>
        ```

        **Example**

        ```bash
        ocm post /api/clusters_mgmt/v1/clusters/$CLUSTER_ID/addon_upgrade_policies --body reference-upgrade-policy.json
        ```

    1. Verify the upgrade policy exists:

        **Syntax**

        ```bash
         ocm get /api/clusters_mgmt/v1/clusters/$CLUSTER_ID/addon_upgrade_policies | jq '.items[] | select(.addon_id=="<addon_id>")'
         ```

        **Example**

        ``` bash
        ocm get /api/clusters_mgmt/v1/clusters/$CLUSTER_ID/addon_upgrade_policies | jq '.items[] | select(.addon_id=="reference-addon")'
        ```

## Useful commands

* Get a list of available addons:

   ``` bash
   ocm get /api/clusters_mgmt/v1/addons | jq '.items[].id'
   ```

* Get a list of available versions to install for a given addon id:

   **Syntax**

   ```bash
   ocm get /api/clusters_mgmt/v1/addons/<addon-id>/versions | jq '.items[].id'
   ```

   **Example**

    ```bash
   $ ocm get /api/clusters_mgmt/v1/addons/reference-addon/versions | jq '.items[].id'
   "0.0.0"
   "0.1.5"
   "0.1.6"
   "0.2.2"
   "0.3.0"
   "0.3.1"
   "0.3.2"
   "0.4.0"
   "0.4.1"
   "0.5.0"
   "0.5.1"
   "0.6.0"
   "0.6.1"
   "0.6.2"
   "0.6.3"
   "0.6.4"
   "0.6.5"
   "0.6.6"
   "0.6.7"
   "0.7.0"
   ```
