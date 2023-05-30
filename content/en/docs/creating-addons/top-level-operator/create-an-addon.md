---
title: Create an Addon
linkTitle: Create an Addon
weight: 20
---


## Shipping your addon through the `addon.yaml` manifest

Your addon operator will be available through
a [`CatalogSource`](https://olm.operatorframework.io/docs/concepts/crds/catalogsource/) CR in OSD. For this to work, you
need to:

- Build a catalog image available at `https://quay.io/osd-addons/<addon_name>-index`
- Build bundle images available at `https://quay.io/osd-addons/<addon_name>-bundle`

Both the [`managed-tenants`](https://gitlab.cee.redhat.com/service/managed-tenants/) and
the [`managed-tenants-bundles`](https://gitlab.cee.redhat.com/service/managed-tenants-bundles/) repositories will help
you in accomplishing this:

### Imageset

1. Create an initial merge request to `managed-tenants` with this file `addons/<addon_name>/metadata/stage/addon.yaml`.
    - Set `addonImageSetVersion: latest`
    - So that your addon can use the SyncSet flow from the beginning, please also
      set `syncsetMigration: "shortcircuit whitelist"`.
    - Look at
      the [reference-addon](https://gitlab.cee.redhat.com/service/managed-tenants/-/blob/main/addons/reference-addon/metadata/stage/addon.yaml)
      ,
      the [jsonschema documentation](https://github.com/mt-sre/managed-tenants-cli/blob/main/docs/tenants/zz_metadata_schema_generated.md)
      ,
      and the [validation PR checks](addon-metadata-file.md#existing-pr-checks-for-addon-metadata) for help.
2. Submit your bundles to the `managed-tenants-bundles` repository, respecting the directory structure. Specify that
   your
   addon will use imagesets by adding it to the list of imageset enabled addons (`IMAGESET_ENABLED_ADDONS`) in
   the `Makefile`,
   see [IMAGE_SET_ADDONS](https://gitlab.cee.redhat.com/service/managed-tenants-bundles/-/blob/main/Makefile#L13).
    - [Creating an operator Bundle](https://olm.operatorframework.io/docs/tasks/creating-operator-bundle/).
    - [Example MR for dbaas-operator without IMAGESET_ENABLED_ADDONS](https://gitlab.cee.redhat.com/service/managed-tenants-bundles/-/merge_requests/105)
      .
3. The `managed-tenants-bundles` pipeline will automatically open a new MR to the `managed-tenants` repository,
   adding a new imageset.

### Index Image (deprecated)

1. Create an initial MR to `managed-tenants` with this file `addons/<addon_name>/metadata/stage/addon.yaml`.
   - Set `indexImage: quay.io/osd-addons/<addon_name>` temporarily. It will be overridden automatically later.
   - Look at
     the [reference-addon](https://gitlab.cee.redhat.com/service/managed-tenants/-/blob/main/addons/reference-addon/metadata/stage/addon.yaml)
     ,
     the [jsonschema documentation](https://github.com/mt-sre/managed-tenants-cli/blob/main/docs/tenants/zz_metadata_schema_generated.md)
     ,
     and the [validation PR checks](addon-metadata-file.md#existing-pr-checks-for-addon-metadata) for help.
   - [Example MR for dbaas-operator](https://gitlab.cee.redhat.com/service/managed-tenants/-/merge_requests/1270).
2. Submit your bundles to the `managed-tenants-bundles` repository, respecting the directory structure.
   - [Creating an operator Bundle](https://olm.operatorframework.io/docs/tasks/creating-operator-bundle/).
   - [Example MR for dbaas-operator](https://gitlab.cee.redhat.com/service/managed-tenants-bundles/-/merge_requests/105).
3. The `managed-tenants-bundles` pipeline will automatically open a new MR to the `managed-tenants` repository,
   modifying the `indexImage: <...>` field with your catalog image.

See the [addon repository structure doc](content/en/docs/creating-addons/managed-tenants-repository.md#structure) for
more information.
