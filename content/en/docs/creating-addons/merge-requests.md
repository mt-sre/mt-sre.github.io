---
title: Merge Requests
---

## Merge Request Review

## Stage

Merge Requests to stage can be self-serviced using the OWNERS file mechanism.

## Production

Any PR that promotes to prod will be manually reviewed to ensure that the addon
has been present in the staging environment previously, in order to ensure that
nothing goes directly to production without having had time in staging.

## FAQ

### Will my addon be automatically upgraded?

For that, the new version bundle directory has to contain a
`ClusterServiceVersion` manifest with the `replaces` field pointing to the version
it will be upgraded from:

```yaml
...
kind: ClusterServiceVersion
...
spec:
  version: 1.19.0
  replaces: example-operator.v1.18.0
  ...
```

### Can I change metadata fields?

By changing the metadata, the addon will be updated in the OCM API. That will
affect new installations, but no cleanup is done to the already installed
addons.

### If my addon fails testing in prod, how will I be notified?

Apart from the validations done here before publishing the addon, at the moment
all the real world testing in done externally to this repository, either (or
both) by using the OSD e2e framework or by assigning testing tasks to QA team.

### Can I push to the addon catalog quay repo?

No. The Quay.io organization is owned by the Add-Ons SRE team. All the pushes
there  are automated and executed after the code is merged, which happens
after all the tests are green.
