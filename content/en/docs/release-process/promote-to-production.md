---
title: "Promote an Imageset to Production"
linkTitle: "Promote to Production"
---

Before starting, please make sure you have successfully switched your addon to imagesets in stage (and integration if
applicable),
see [here](content/en/docs/creating-addons/top-level-operator/migrate-to-imagesets.md).

When you have thoroughly tested a version of your addon in stage and would like to
promote it to production, follow these steps:

1. Create a merge request in `managed-tenants` copying the imageset file from `addons/<addon-name>/addonimagesets/stage`
   and
   pasting it in `addons/<addon-name>/addonimagesets/production`.
2. If this is the first time doing this i.e. your addon is not using imagesets in production yet, there is one more
   step: In the same
   merge request, replace `indexImage: ...` with `addonImageSetVersion: latest` in the
   `addons/<addon-name>/metadata/production/addon.yaml` file.

Because the `addon.yaml` file refers to the latest imageset (`addonImageSetVersion: latest`),
this newer version you just moved will be picked up and deployed.

## Versioning with imagesets

Currently, the only supported value for `addonImageSetVersion` is `latest`.
This is because further work is pending to pin an addon's version in the OCM API.
Attempting to lower addon versions by removing imageset files will not have any effect.
Though it is possible to update an addon version in-place by modifying the corresponding
imageset this is discouraged. It is preferred to "fix-forward" with new versions even in
the event that the fix is just a reversion to a previous stable version.
