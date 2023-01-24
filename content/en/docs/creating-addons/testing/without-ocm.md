## Testing Addons on OCP

During the development process, it might be useful (and cheaper) to run your
addon on an OCP cluster.

You can spin up an OCP cluster on your local machine using
[CRC](https://cloud.redhat.com/openshift/install/crc/installer-provisioned).

OCP and OSD differ in one important aspect: OCP gives you full access, while
OSD restricts the administrative actions. But Managed Tenants will apply
resources as unrestricted admin to OSD, just like you can do with your OCP,
so OCP is a good OSD mockup for our use case.

By doing this, you're skipping:

- OCM and SKU management
- Hive

First, you have to build your catalog. Let take the `managed-odh` as example:

```bash
$ managedtenants --environment=stage --addons-dir addons --dry-run run --debug tasks/deploy/10_build_push_catalog.py:managed-odh
Loading stage...
Loading stage OK
== TASKS =======================================================================
tasks/deploy/10_build_push_catalog.py:BuildCatalog:managed-odh:stage...
 -> creating the temporary directory
 -> /tmp/managed-odh-stage-1bkjtsea
 -> generating the bundle directory
 -> generating the bundle package.yaml
 -> building the docker image
 -> ['docker', 'build', '-f', PosixPath('/home/apahim/git/managed-tenants/Dockerfile.catalog'), '-t', 'quay.io/osd-addons/opendatahub-operator:stage-91918fe', PosixPath('/tmp/managed-odh-stage-1bkjtsea')]
tasks/deploy/10_build_push_catalog.py:BuildCatalog:managed-odh:stage OK
tasks/deploy/10_build_push_catalog.py:PushCatalog:managed-odh:stage...
 -> pushing the docker image
 -> ['docker', '--config', '/home/apahim/.docker', 'push', 'quay.io/osd-addons/opendatahub-operator:stage-91918fe']
tasks/deploy/10_build_push_catalog.py:PushCatalog:managed-odh:stage OK
```

That command has built the image
`quay.io/osd-addons/opendatahub-operator:stage-91918fe` on your local machine.

You can inspect the image with:

```bash
$ docker run --rm -it --entrypoint "bash"  quay.io/osd-addons/opendatahub-operator:stage-91918fe -c "ls manifests/"
0.8.0  1.0.0-experiment  managed-odh.package.yml

$ docker run --rm -it --entrypoint "bash"  quay.io/osd-addons/opendatahub-operator:stage-91918fe -c "cat manifests/managed-odh.package.yml"
channels:
- currentCSV: opendatahub-operator.1.0.0-experiment
  name: beta
defaultChannel: beta
packageName: managed-odh
```

Next, you have to tag/push that image to some public registry repository of
yours:

```bash
$ docker tag quay.io/osd-addons/opendatahub-operator:stage-91918fe quay.io/<my-repository>/opendatahub-operator:stage-91918fe
$ docker push quay.io/<my-repository>/opendatahub-operator:stage-91918fe
Getting image source signatures
Copying blob 9fbc4a1ed0b0 done
Copying blob c4d8f7894b7d skipped: already exists
Copying blob 61598d8d1b24 skipped: already exists
Copying blob 38ada4bcd26f skipped: already exists
Copying blob d5fdf1f627c8 skipped: already exists
Copying blob 2bf094d88b12 skipped: already exists
Copying blob 8a6c7bacb5db done
Copying config 3088e48540 done
Writing manifest to image destination
Copying config 3088e48540 [--------------------------------------] 0.0b / 3.6KiB
Writing manifest to image destination
Writing manifest to image destination
Storing signatures
```

Now we have to apply the OpenShift resources that will install the operator
in the OCP cluster. You can use the `managedtenants` command to generate
the stage `SelectorSyncSet` and look at it for reference:

```bash
$ managedtenants --environment=stage --addons-dir addons --dry-run run --debug tasks/generate/99_generate_SelectorSyncSet.py
Loading stage...
Loading stage OK
== POSTTASKS ===================================================================
tasks/generate/99_generate_SelectorSyncSet.py:GenerateSSS:stage...
 -> Generating SSS template /home/apahim/git/managed-tenants/openshift/stage.yaml
tasks/generate/99_generate_SelectorSyncSet.py:GenerateSSS:stage OK
```

Here's the SelectorSyncSet snippet we are interested in:

```yaml
---
- apiVersion: hive.openshift.io/v1
  kind: SelectorSyncSet
  metadata:
    name: addon-managed-odh
  spec:
    clusterDeploymentSelector:
      matchLabels:
        api.openshift.com/addon-managed-odh: "true"
    resourceApplyMode: Sync
    resources:
      - apiVersion: v1
        kind: Namespace
        metadata:
          annotations:
            openshift.io/node-selector: ""
          labels: null
          name: redhat-opendatahub
      - apiVersion: operators.coreos.com/v1alpha1
        kind: CatalogSource
        metadata:
          name: addon-managed-odh-catalog
          namespace: openshift-marketplace
        spec:
          displayName: Managed Open Data Hub Operator
          image: quay.io/osd-addons/opendatahub-operator:stage-${IMAGE_TAG}
          publisher: OSD Red Hat Addons
          sourceType: grpc
      - apiVersion: operators.coreos.com/v1alpha2
        kind: OperatorGroup
        metadata:
          name: redhat-layered-product-og
          namespace: redhat-opendatahub
      - apiVersion: operators.coreos.com/v1alpha1
        kind: Subscription
        metadata:
          name: addon-managed-odh
          namespace: redhat-opendatahub
        spec:
          channel: beta
          name: managed-odh
          source: addon-managed-odh-catalog
          sourceNamespace: openshift-marketplace
```

Our OpenShift manifest to be applied to the OCP cluster looks as follows:

```yaml
kind: List
metadata: {}
apiVersion: v1
items:
  - apiVersion: v1
    kind: Namespace
    metadata:
      name: redhat-opendatahub
  - apiVersion: operators.coreos.com/v1alpha1
    kind: CatalogSource
    metadata:
      name: addon-managed-odh-catalog
    spec:
      displayName: Managed Open Data Hub Operator
      image: quay.io/<my-repository>/opendatahub-operator:stage-91918fe
      publisher: OSD Red Hat Addons
      sourceType: grpc
  - apiVersion: operators.coreos.com/v1alpha2
    kind: OperatorGroup
    metadata:
      name: redhat-layered-product-og
  - apiVersion: operators.coreos.com/v1alpha1
    kind: Subscription
    metadata:
      name: addon-managed-odh
    spec:
      channel: beta
      name: managed-odh
      source: addon-managed-odh-catalog
      sourceNamespace: openshift-marketplace
```

Finally, apply it to the OCP cluster:

```bash
$ oc apply -f manifest.yaml
Namespace/redhat-opendatahub created
CatalogSource/addon-managed-odh-catalog created
Subscription/addon-managed-odh created
OperatorGroup/redhat-layered-product-og created
```

Your operator should be installed in the cluster.
