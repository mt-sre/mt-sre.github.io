---
title: Configure Vault Access
linkTitle: Vault Access
---

## Background

Vault access is required by teams at paths like `mt-sre/tenants/<addon-name>/secrets/*` to store their secrets
which are referenced by their `addon.yaml`s.
For this, vault access needs to be set up for that team.

Vault auth configuration is managed
via [app-interface](https://gitlab.cee.redhat.com/gathomas/app-interface#manage-vault-configurations-via-app-interface).

Multiple components have to be set up to provide vault access to the team. We will define the vault policy, oidc
permission,
role and then assign the role to the members of the team.

### Vault Policy

First we want to create a vault policy that allows access to the path that the team will store their secrets, i.e.
`mt-sre/tenants/<addon-name>/secrets/*`. A vault policy is essentially a path with associated permissions (verbs).
See the [HashiCorp Vault Policy documentation](https://developer.hashicorp.com/vault/docs/concepts/policies) for more
information.

To create the policy we follow
Create a file under `data/services/vault.devshift.net/config/prod/policies` with the
name `mt-sre-tenants-<team-name>-secrets-policy.yaml`
with the following structure:

```yaml
---
$schema: /vault-config/policy-1.yml

labels:
  service: vault.devshift.net

name: "mt-sre-tenants-<team-name>-secrets-policy"
instance:
  $ref: /services/vault.devshift.net/config/instances/devshift-net.yml
rules: |
  path "mt-sre/metadata" {
    capabilities = ["list"]
  }
  path "mt-sre/metadata/tenants" {
    capabilities = ["list"]
  }
  path "mt-sre/metadata/tenants/<team-name>" {
    capabilities = ["list"]
  }
  path "mt-sre/data/tenants/<team-name>/secrets/*" {
    capabilities = ["list", "create", "read", "update", "delete"]
  }
  path "mt-sre/metadata/tenants/<team-name>/secrets/*" {
    capabilities = ["list", "create", "read", "update", "delete"]
  }

```

Take whatever subset of the rules above are needed.
For example, under `data/services/vault.devshift.net/config/prod/policies/mt-sre-tenants-odf-secrets-policy.yml`, the
content is:

```yaml
---
$schema: /vault-config/policy-1.yml

labels:
  service: vault.devshift.net

name: "mt-sre-tenants-odf-secrets-policy"
instance:
  $ref: /services/vault.devshift.net/config/instances/devshift-net.yml
rules: |
  path "mt-sre/metadata/tenants/odf/secrets/*" {
    capabilities = ["create", "read", "update", "delete", "list"]
  }
  path "mt-sre/data/tenants/odf/secrets/*" {
    capabilities = ["create", "read", "update", "delete", "list"]
  }
```

See
the [app-interface documentation about creating policies](https://gitlab.cee.redhat.com/service/app-interface#manage-vault-policies-vault-configpolicy-1yml)
for more information.

### OIDC Permission

Next we will create an OIDC Permission to wrap the vault policy we just created.

Create a file under `data/dependencies/vault/permissions/oidc/prod` with the
name `mt-sre-tenants-<team-name>-oidc-permission.yaml`
with the following structure:

```yaml
---
$schema: /access/oidc-permission-1.yml

labels:
  service: vault.devshift.net

name: mt-sre-tenants-<team-name>-oidc-permission
description: mt-sre-tenants-<team-name>-oidc-permission vault devshift oidc permission

instance:
  $ref: /services/vault.devshift.net/config/prod/devshift-net.yml

service: vault
vault_policies:
  - $ref: /services/vault.devshift.net/config/prod/policies/mt-sre-tenants-<team-name>-secrets-policy.yml
```

For example, under `data/dependencies/vault/permissions/oidc/prod/github-app-sre-vault-odf-team.yaml`
(the difference in file name is for legacy reasons), the content is:

```yaml
---
$schema: /access/oidc-permission-1.yml

labels:
  service: vault.devshift.net

name: github-app-sre-vault-odf-team-oidc-vault
description: github-app-sre-vault-odf-team vault devshift oidc permission

instance:
  $ref: /services/vault.devshift.net/config/prod/devshift-net.yml

service: vault
vault_policies:
  - $ref: /services/vault.devshift.net/config/prod/policies/mt-sre-tenants-odf-secrets-policy.yml
```

### Role

We now have to include this permission in a role that we can assign to the team members: *Note* this is an app-interface
role, not to be confused with a vault role.

Under `data/teams/<team-name>/roles` create a file called `app-sre-vault-<team-name>.yml` with the following contents:

```yaml
---
$schema: /access/role-1.yml

labels: { }
name: app-sre-vault-odf

permissions: [ ]

oidc_permissions:
  - $ref: /dependencies/vault/permissions/oidc/prod/mt-sre-tenants-<team-name>-oidc-permission.yaml
```

For example, the contents of  `data/teams/odf-managed-service/roles/app-sre-vault-odf.yml` is:

```yaml
---
$schema: /access/role-1.yml

labels: { }
name: app-sre-vault-odf

permissions: [ ]

oidc_permissions:
  - $ref: /dependencies/vault/permissions/oidc/prod/github-app-sre-vault-odf-team.yml
```

### Adding Users

Now the users just need to be added with the role we just created.
Under `data/teams/<team-name>/users` create files called `<redhat-username>.yml` with the following contents:

```yaml
---
$schema: /access/user-1.yml

labels: { }

name: <Your name>
org_username: <org username>
github_username: <github username>
quay_username: <quay username>

roles: ## "roles" link your user to a certain role. Role in this case would point to the app-sre/your-team on Github.
  - $ref: /teams/<team-dir-name>/roles/app-sre-vault-<team-name>.yml ## path to the roles file you created in the last step
```

For example, the contents of `data/teams/odf-managed-service/users/dbindra.yml` is:

```yaml
---
$schema: /access/user-1.yml

labels: { }

name: Dhruv Bindra
org_username: dbindra
github_username: bindrad
quay_username: dbindra

roles:
  - $ref: /teams/odf-managed-service/roles/app-sre-vault-odf.yml
```
