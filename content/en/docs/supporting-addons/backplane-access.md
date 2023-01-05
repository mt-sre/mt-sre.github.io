---
title: Getting Backplane Access
linkTitle: Backplane Access
---

## Backplane

Backplane is the system used to provide access to the fleet of Openshift clusters. It creates ssh tunnels and modifies your local `~/.kube/config`.

### Getting access

1. [Install ocm CLI](https://github.com/openshift-online/ocm-cli#installation)
2. [Follow the instructions here](https://source.redhat.com/groups/public/openshiftplatformsre/wiki/backplane_user_documentation)
3. Make sure your user is part of the [`sd-mtsre` Rover group](https://rover.redhat.com/groups/group/sd-mtsre).
4. Wait for `https://gitlab.cee.redhat.com/service/authorizedkeys-builder` to sync your ssh key onto the fleet of Openshift clusters
5. [Install backplane CLI](https://gitlab.cee.redhat.com/service/backplane-cli#install) or use the [PKGBUILD](https://gitlab.consulting.redhat.com/red-hat-arch-linux-users/backplane-cli).
