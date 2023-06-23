---
title: Subscription Config
linkTitle: Subscription Config
---

## Configs for addons

Additional configs maybe supplied to an addon using the `config` attribute present in either the addon metadata file
or the addon imageset file. The attributes defined here are passed to the subscription OLM object.
Read more about subscription
config [here](https://github.com/operator-framework/operator-lifecycle-manager/blob/master/doc/design/subscription-config.md)
.

We support two ways of defining subscription configs for addons:

- Version specific subscription config through the respective addon imageset file. This config takes precedence.
- A default subscription config through the addon metadata file (`addon.yaml` file)

Currently, only the following sub-attributes are supported under `config`:

- `env` : Defines an array of key value pairs (with the field `name` and `value` respectively).
  The env field defines a list of Environment Variables that must exist in all containers in the Pod created by OLM
  (values defined here will overwrite existing environment variables of the same name).

Example:
Lets say in stage environment you want your addon to have a more verbose logging. Assuming that your addon sets the
logging level according to the value set in the environment variable `LOGGING_LEVEL`, you'd have the
following `stage/addon.yaml` file:

```yaml
id: reference-addon
name: Reference Addon
description: Reference Addon is a real Addon, created to validate and demonstrate the Addons Flow.
link: https://github.com/openshift/reference-addon
icon: kLdODtkgbUXm7Htgt9scAMOODX6Tz2BImmdt/KGASyjvzlhfvjLU8wf0ZLk3JuMh4LOBxAgFJ5gDCGl31+DiW1syaLCjCDyBQHzIvQcM0CDLVq4MKAo3EMaluU4xB9C2Ys7K
label: api.openshift.com/addon-reference-addon
enabled: true
addonOwner: MT-SRE Team <sd-mt-sre@redhat.com>
quayRepo: quay.io/osd-addons/reference-addon
testHarness: quay.io/miwilson/addon-samples
installMode: OwnNamespace
targetNamespace: redhat-reference-addon
namespaces:
- redhat-reference-addon
ocmQuotaName: addon-reference-addon
ocmQuotaCost: 1
operatorName: reference-addon
defaultChannel: alpha
bundleParameters: {}
namespaceLabels: {}
namespaceAnnotations: {}
indexImage: quay.io/osd-addons/reference-addon-index@sha256:b9e87a598e7fd6afb4bfedb31e4098435c2105cc8ebe33231c341e515ba9054d
config:
env:
- name: LOGGING_LEVEL
  value: "VERBOSE"
```
