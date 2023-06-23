---
title: "managed-tenants Repository"
date: 2022-12-15T00:53:51+01:00
description: >
    managed-tenants Repository.
---

Addons are deployed through GitOps pipelines. Most of the configuration for Addons can
be found in the
[managed-tenants Repository](https://gitlab.cee.redhat.com/service/managed-tenants)
. See the
[create an addon](https://gitlab.cee.redhat.com/service/managed-tenants/-/blob/main/docs/tenants/create_an_addon.md)
documentation page for a good starting point.

## Structure

The `addons` directory contains one subdirectory per Add-On that contains a `metadata` subdirectory.
The `metadata` subdirectory contains one subdirectory per environment, which contains an Add-On metadata file
for the corresponding environment. For example:

```yaml
addons/ocs-converged
└── metadata
    ├── production
    │   └── addon.yaml
    └── stage
        └── addon.yaml
```

### addon imagesets

Our tooling now supports creating imagesets for addons. The imageset of a new version of
an addon, which sits inside a new directory called `addonimagesets/<environment>`. CPaaS/Tenants will need to create
a merge request in <https://gitlab.cee.redhat.com/service/managed-tenants/> in order to notify
MT-SRE of the availability of the new release. Each new addon version is a new file under the
earlier mentioned `addonimagesets/<environment>` directory.

See the [Imageset Specification](top-level-operator/imageset-specification.md) for more details about
the imageset file.

Example repository structure of an addon with imagesets:

```bash
mock-operator-with-imagesets
├── addonimagesets
│   └── stage
│       ├── mock-operator.v1.0.0.yml
│       ├── mock-operator.v1.0.2.yml
└── metadata
    └── stage
        └── addon.yaml
```

The `addon.yaml` file would then reference the required imageset version
using the attribute `addonImageSetVersion`. See the
[Metadata Specification](top-level-operator/addon-metadata-file.md) for details.

Ex:

```yaml
id: mock-operator
name: Mock Operator
description: This is a mock operator.
icon: mock
label: "api.openshift.com/addon-mock-operator"
enabled: true
addonOwner: Mock <mock@redhat.com>
quayRepo: quay.io/osd-addons/mock-operator
installMode: OwnNamespace
targetNamespace: mock-operator
namespaces:
  - mock-operator
namespaceLabels:
  monitoring-key: mock
namespaceAnnotations: {}
ocmQuotaName: addon-mock-operator
ocmQuotaCost: 1
testHarness: quay.io/mock-operator/mock-operator-test-harness
operatorName: mock-operator
hasExternalResources: true
defaultChannel: alpha
bundleParameters:
  useClusterStorage: "true"
addonImageSetVersion: 1.0.0
```

#### metadata

The `metadata` directory contains exclusively sub-directories, one per
environment.

The environment subdirectories host the corresponding `addons.yaml` metadata.
See the [Metadata Specification](top-level-operator/addon-metadata-file.md) for details.

#### bundles (deprecated)

The `bundles` directory has been moved to the
[managed-tenants-bundles repository](https://gitlab.cee.redhat.com/service/managed-tenants-bundles/-/blob/main/README.md)
.
