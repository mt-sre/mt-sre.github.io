---
title: Feature Flags in ADO
linkTitle: Feature Flags
---

Currently, some ADO functionality, for example, addons plug and play, is hidden behind a feature flag.
Feature flags are specified in the `AddonOperator` resources under `.spec.featureflags`. This field is a string
which is a comma separated list of feature flags to enable. For addons plug and play and thus the addon package
to be enabled, the string `ADDONS_PLUG_AND_PLAY` has to be included in this field.
