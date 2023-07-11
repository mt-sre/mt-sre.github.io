---
title:  Plug and Play Addon
linkTitle: Plug and Play Addon
---

## Package Operator

Package Operator is a Kubernetes Operator for packaging and managing a collection of arbitrary Kubernetes objects.

Each addon with a `packageOperator` defined in its `spec` will have a corresponding
[ClusterObjectTemplate](https://package-operator.run/docs/getting_started/api-reference/#clusterobjecttemplate).
The ClusterObjectTemplate is an API defined in Package
Operator, enabling users to create an object by templating a
 manifest and injecting values retrieved from other arbitrary source objects.
However, regular users typically do not need to interact with the `ClusterObjectTemplate`.
Instead, they can interact with the generated
[ClusterPackage](https://package-operator.run/docs/getting_started/api-reference/#clusterpackage)
manifest.

**Example of a `ClusterPackage` manifest:**

```shell
apiVersion: package-operator.run/v1alpha1
kind: ClusterPackage
metadata:
  name: <addon_name>
spec:
  image: <addon.spec.packageOperator>
   config:
    addonsv1:
      clusterID: a440b136-b2d6-406b-a884-fca2d62cd170
      deadMansSnitchUrl: https://example.com/test-snitch-url
      ocmClusterID: abc123
      ocmClusterName: asdf
      pagerDutyKey: 1234567890ABCDEF
      parameters:
        foo1: bar
        foo2: baz
      targetNamespace: pko-test-ns-00-req-apy-dsy-pdy
```

* The `deadMansSnitchUrl` and `pagerDutyKey` are obtained
from the ConfigMaps using their default names and locations.
**IMPORTANT:** To successfully inject the `deadMansSnitchUrl` and `pagerDutyKey` values into the `ClusterPackage` manifest,
you must keep the default naming scheme and location of the corresponding ConfigMaps.
See the [addons deadMansSnitch](https://mt-sre.github.io/docs/creating-addons/monitoring/deadmanssnitch_integration/)
and [addons pagerDuty](https://mt-sre.github.io/docs/creating-addons/monitoring/pagerduty_integration/)
documentation for more information.

* Additionally, all the values present in `.spec.config.addonsv1`
 can be injected into the objects within your packageImage.
 See the
[package operator](https://package-operator.run/docs/guides/packaging-an-application/#go-templates)
documentation for more information.

## Tenants Onboarding Steps

Although you can generate the packageImage yourself using the
[package operator](https://package-operator.run/docs/guides/packaging-an-application/#build--validate) documentation,
we recommend you use the Managed Tenants Bundles (MTB) facilities.

The following steps are an example of generating the packageImage for the reference-addon package using the MTB flow:

1. In the MTB repository, create a `package` directory and add the `manifests.yaml` inside the `package` directory.
See the following
 [merge request](https://gitlab.cee.redhat.com/service/managed-tenants-bundles/-/tree/main/addons/reference-addon/package)
 for an example.

2. The MTB CI creates the packageImage and the Operator Lifecycle Manager (OLM) Index Image as part of the team's
 [addon folder](https://gitlab.cee.redhat.com/service/managed-tenants-bundles/-/tree/main/addons/reference-addon).

3. The MTB CI creates a merge request to the
[managed-tenants repository](https://gitlab.cee.redhat.com/service/managed-tenants/-/blob/main/addons/reference-addon/addonimagesets/stage/reference-addon.v0.10.1.yaml#L24)
and adds a new AddonImageSet with the PackageImage and OLM Index images.
