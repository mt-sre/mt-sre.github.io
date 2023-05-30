---
title: Migrate to Imagesets
linkTitle: Migrate to Imagesets
---

All addons will eventually have to use imagesets. To start using imagesets, follow the steps below:

1. In the `managed-tenants-bundles` repository, create a merge request making a no-op change in your addon's directory
   *and*
   add the name of your addon to the list of imageset enabled addons (`IMAGESET_ENABLED_ADDONS`) in the `Makefile`,
   see [IMAGE_SET_ADDONS](https://gitlab.cee.redhat.com/service/managed-tenants-bundles/-/blob/main/Makefile#L13).

   - See [this MR](https://gitlab.cee.redhat.com/service/managed-tenants-bundles/-/merge_requests/608)
     as an example.
   - The `managed-tenants-bundles` pipeline will then create an imageset for your addon in the environment(s) specified
     in `managed-tenants-bundles/addons/<addon-name>/config/main.yaml` i.e. create an
     imageset file in `addons/<addon-name>/addonimagesets/<environment>` in the `managed-tenants` repository. To get an
     imageset into production, please
     see [promote an imageset to production](content/en/docs/release-process/promote-to-production.md).

2. In the `managed-tenantes` repository, create a merge request replacing `indexImage: ...`
   with `addonImageSetVersion: latest`
   in the `addons/<addon-name>/metadata/<environment>/addon.yaml` file(s).

   - See [this MR](https://gitlab.cee.redhat.com/service/managed-tenants/-/merge_requests/2903) as an example.
