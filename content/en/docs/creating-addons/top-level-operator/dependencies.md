---
title: "Dependencies"
date: 2022-12-15T00:53:51+01:00
---

This document describes the supported implementation for Addon dependencies,
as signed-off by the Managed Tenants SRE Team.

## Dependencies Specification

* Addons must specify dependencies using the OLM dependencies feature,
  documented
  [here](https://docs.openshift.com/container-platform/4.7/operators/understanding/olm/olm-understanding-dependency-resolution.html)
* The dependencies must have the version pin-pointed. Ranges are not allowed.
* The dependencies must come from a *Trusted Catalog*. See the
  [Trusted Catalogs](#trusted-catalogs) section for details.

## Trusted Catalogs

The Addon and its dependencies must come from Trusted Catalogs. Trusted
Catalogs are those with content published by the Managed Services Pipelines,
implemented by CPaaS, or by the Managed Tenants SRE Team.

### Trusted Catalogs List

* Addon catalog: the catalog created by the Managed Tenants SRE Team, for the
  purpose of releasing the Addon. Dependency bundles can be shipped in the same
  catalog of the Addon. The Addon catalog is considered "trusted" for
  the dependencies it carries.
* Red Hat Operators catalog: the catalog content goes through the Managed
  Services Pipelines, same process to build some Addons themselves, just with
  a different release process. This catalog is considered "trusted" and can be
  used for dependencies.

### Including a Catalog in the Trusted List

* Make sure that the catalog is available on OSD and its content is released
  through the Managed Services Pipelines, implemented by CPaaS.
* Create a Jira ticket in the
  [MT-SRE Team backlog](https://issues.redhat.com/secure/CreateIssue.jspa?pid=12326231&issuetype=3),
  requesting the assessment of the OSD catalog you want to consider as
  "trusted".

## Issues

There's a feature request to the OLM Team to allow specifying the
CatalogSource used for the dependencies:
* [OLM-2249](https://issues.redhat.com/browse/OLM-2249)
