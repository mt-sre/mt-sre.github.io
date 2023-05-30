---
title: Addon Requirements
---

Requirements for an add-on can be defined using the `addOnRequirements` field. This field takes an array of requirement
objects.

```yaml
addOnRequirements:
  - resource: 'cluster'
    id: 'my-cluster-req'
    data:
      cloud_provider.id: 'aws'
    enabled: true
```

#### resource

Requirements must have a `resource` field, and it must be a supported resource type, currently supported types:

* `cluster`
* `addon`
* `machine_pool`

#### id

Requirements must have an 'id'. It can be anything, but it must be unique per addon/resource pair.

#### enabled

Indicates if the requirement is enabled or not. To deprecate or disable a parameter set `enabled` to `false`, or
alternatively remove it from the requirements list.

#### data

Resource specific data that is used to supply constraints on individual resource fields.
A constraint can be added here for any resource field that is exposed by the publicly facing API of that particular
resource.
Data is a map, and the key is a resource field represented in dot notation e.g. ccs.enabled == {ccs: {enabled: true}},
the value is the value expected.

For a list of currently supported fields please refer to
the [metadata schema](https://github.com/mt-sre/managed-tenants-cli/blob/main/managedtenants/schemas/shared/addon_requirements.json)

## Compute Requirements

Compute requirements can be set on a cluster or machine pool resource type that enforces that the resource is capable of
providing the specified compute capacity (cpu or memory).

### compute.cpu

Amount of vCPUs required by your addon.

### compute.memory

Amount of memory in bytes required by your addon.

Example here would require the cluster to provide at lease '11' vCPUs and '25769803776' bytes or '24' GiB of memory
combined across all it's compute nodes.

```yaml
addOnRequirements:
  - resource: 'cluster'
    id: 'my-cluster-compute-req'
    data:
      compute.memory: 25769803776,
      compute.cpu: 11
    enabled: true
```

**Important Note**: Compute requirements, like all others, are only checking the relevant resource's (cluster/machine
pool)
specification within OCM, it does not check the current usage of the particular resource. If other workloads are already
using capacity it may still negatively impact your addons installation.

## Platform Requirements

Platform requirements can be set on a cluster if your addon only supports a particular version or versions of the
platform. Platform requirements can be specified as semantic version constraints.

### version.raw_id

The platform version(s) supported, a comma separated list of semantic version constraints using any of these
operators: ```"=","!=",">","<",">=","<=","~>"```

```yaml
addOnRequirements:
  - resource: 'cluster'
    id: 'my-cluster-req'
    data:
      version.raw_id: '>= 4.6, < 4.8, != 4.6.6'
    enabled: true
```

## Managed Service - specific requirements

AddOns that comprise a managed service have the option to specify **taint** and **label** requirements for machine pool
resources.
Declaring these requirements lets OCM know to build a machine pool with the taints/labels listed.

```yaml
 addOnRequirements:
   - id: mp-requirements
     resource: machine_pool
     enabled: true
     data:
       taints:
         - key: taint-key0
           value: taint-value0
           effect: NoSchedule
         - key: taint-key1
           value: taint-value1
           effect: PreferNoSchedule
         - key: taint-key2
           value: taint-value2
           effect: NoExecute
       labels:
         label0: value0
         label1: value1
         label2: value2
```

## Examples

### cluster

The target cluster must have a `cloud_provider.id` of `aws` (My AddOn only works on AWS)

```yaml
addOnRequirements:
  - resource: 'cluster'
    id: 'my-cluster-req'
    data:
      cloud_provider.id: 'aws'
    enabled: true
```

### addon

The `codeready-workspaces` addon must be installed

```yaml
addOnRequirements:
  - resource: 'addon'
    id: 'my-addon-req'
    data:
      id: 'codeready-workspaces'
    enabled: true
```

The code `codeready-workspaces` addon must be installed and in a `ready` state

```yaml
addOnRequirements:
  - resource: 'addon'
    id: 'my-addon-req'
    data:
      id: 'codeready-workspaces'
      state: 'ready'
    enabled: true
```

### machine_pool

A machine pool must exist. This does not include the `default` machine pool

```yaml
addOnRequirements:
  - resource: 'machine_pool'
    id: 'my-machine-pool-req'
    data: { }
    enabled: true
```

A machine pool with the id `my-machine-pool` must exist

```yaml
addOnRequirements:
  - resource: 'machine_pool'
    id: 'my-machine-pool-req'
    data:
      id: 'my-machine-pool'
    enabled: true
```

A machine pool must exist with instance type of `m5.xlarge` and a minimum `2` replicas. Th*is includes the `default`
machine pool.

```yaml
addOnRequirements:
  - resource: 'machine_pool'
    id: 'my-machine-pool-req'
    data:
      instance_type: 'm5.xlarge'
      replicas: 2
    enabled: true
```

### Multiple

```yaml
addOnRequirements:
  - resource: 'cluster'
    id: 'my-cluster-req'
    data:
      cloud_provider.id: 'aws'
      ccs.enabled: false
      product.id: 'osd'
      nodes.compute: 4
      nodes.compute_machine_type.id: 'm5.xlarge'
    enabled: true
  - resource: 'addon'
    id: 'my-addon-req'
    data:
      id: 'codeready-workspaces'
      state: 'ready'
    enabled: true
  - resource: 'machine_pool'
    id: 'my-machine-pool-req'
    data:
      id: 'addon-mas'
      instance_type: 'm5.xlarge'
      replicas: 2
    enabled: true
```

## Dynamically Set Requirements

To allow the user to dynamically set requirements at install time, a `resource_requirement` parameter
can be used. See [resource_requirement parameter](addon-parameters.md#resource_requirement).
