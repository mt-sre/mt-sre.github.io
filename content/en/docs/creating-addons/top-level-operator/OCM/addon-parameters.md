---
title: Addon Parameters
---

Parameters can be defined for an add-on using the addOnParameters field, this field takes an array of parameter objects.

```yaml
- id: "my-param"
  name: "My Param"
  description: "My Param Description"
  value_type: string
  validation: '^myparam-[A-Za-z0-9]$'
  required: false
  editable: false
  enabled: true
  default_value: "my default"
```

#### Value Type

Parameters must have a `value_type` field, and it must be a supported value type, currently supported types :

* `string`
* `boolean`
* `number`
* `cidr`
* `resource`
* `resource_requirement`

> **_NOTE:_** All values are ultimately stored as strings, but additional validation is handled by clusters service to
> ensure they are supported by their corresponding type e.g boolean is a boolean value, number is an integer or float
> etc..

#### 'resource'

A resource type parameter is used to allow addons to set values to consume from resource quota stored in the account
manager service (AMS).
The 'id' of the parameter is the quota resource name and the 'value' is a number representing the amount the
installation of this addon will consume from the total of that resource.

**Example:**

```yaml
  - id: "addon-my-addon"
    name: "AddOn Flavour"
    description: "Flavour of AddOn, small, medium or large"
    value_type: resource
    required: true
    editable: true
    enabled: true
    options:
      - name: "Small SKU"
        value: "1"
      - name: "Medium SKU"
        value: "5"
      - name: "Large SKU"
        value: "10"
    default_value: "5"
```

The above would require quota in AMS to exist with a resource name of `addon-my-addon`, and depending on which option
the user selects at installation
time (1, 5 or 10), that value is deducted from the total available.

Details of quota and resources available can be found from the following endpoint:

```bash
ocm get /api/accounts_mgmt/v1/organizations/{myorg}/quota_cost  --parameter=search="quota_id LIKE 'addon%'" --parameter=fetchRelatedResources=true
```

#### 'resource_requirement'

A `resource_requirement` parameter is used to introduce a set of cluster requirements as part of the parameter options.
See [Add-on Requirements](addon-requirements.md) for information on what requirements can be set.
These cluster requirements are applied as addon resources, and therefore, they will consume the quota stored in the
account manager service (AMS).
When the addon is installed the `requirements` present in the option that match the parameter value will be used
for the addon, as if they were defined directly in the addon itself as `addOnRequirements`. A `resource_requirement`
parameter
can be used with or without `addOnRequirements` being defined. If both are used, `requirements` will inherit any
requirements
set in `addOnRequirements`. However, the same requirement cannot be set in both `addOnRequirements` and `requirements`.
In this case the addon would not be able to be installed.

Only one `resource_requirement` parameter is allowed per addon, and options with `requirements` are only allowed for
`resource_requirement` type parameters.

**Example:**

```yaml
- id: "addon-my-addon"
  name: "AddOn Flavour and Requirements"
  description: "Flavour of AddOn, small, medium or large, with requirements"
  value_type: resource_requirement
  required: true
  editable: true
  enabled: true
  options:
    - name: "Small SKU"
      value: "1"
      requirements:
        - resource: 'machine_pool'
          id: 'my-machine-pool-req'
          data:
            instance_type: 'm5.xlarge'
            replicas: 2
          enabled: true
    - name: "Medium SKU"
      value: "5"
      requirements:
        - resource: 'machine_pool'
          id: 'my-machine-pool-req'
          data:
            instance_type: 'm5.2xlarge'
            replicas: 4
          enabled: true
    - name: "Large SKU"
      value: "10"
      requirements:
        - resource: 'machine_pool'
          id: 'my-machine-pool-req'
          data:
            instance_type: 'm5.16xlarge'
            replicas: 5
          enabled: true
  default_value: "5"
```

The above would require quota in AMS to exist with a resource name of `addon-my-addon`, and depending on which option
the user selects at installation
time (1, 5 or 10), that value is deducted from the total available.

Details of quota and resources available can be found from the following endpoint:

```bash
ocm get /api/accounts_mgmt/v1/organizations/{myorg}/quota_cost  --parameter=search="quota_id LIKE 'addon%'" --parameter=fetchRelatedResources=true
```

#### Required

Parameters can be set as "required" meaning that they must have a value set in order for the add-on to be installed.

**Note:** In the case of boolean values this does not mean they need to be true (checked in the UI), since a true or
false value can both satisfy the OCM "required" requirement.
If however you want to enforce a true value (must be checked in the UI), you can add a validation to your parameter
like the following:

```yaml
  - id: "agree-to-something"
    name: "Agree to Something Important"
    description: "Agree to Something Important"
    value_type: boolean
    validation: "^true$"
    required: true
    editable: false
    enabled: true
```

#### Editable

Parameters set `editable: false` can not be modified after it has been set, and any request to modify it from its
original
value will result in an error. Alternatively setting `editable: true` allows the value to be modified after the fact.

**Note:** Please be cautious when using `editable: false` with `required: false`. With this configuration, it is
possible to create the addon
without setting the given parameter. Then, because the parameter cannot be modified _after it has been set_, it is still
possible
for the parameter value to be changed (set for the first time) after installation.

#### Editable Direction

Parameters can have `editable_direction: ["up"|"down"]` set to restrict upscale/downscale operations on options. For
example:

```json
 "parameters": {
    "items": [
      {
        "name": "size",
        "description": "Default editable direction is unrestricted.",
        "id": "size",
        "value_type": "string",
        "required": true,
        "editable": true,
        "editable_direction": "up"
        "enabled": true,
        "options": [
          {
            "name": "0 TiB",
            "value": "0"
          },
          {
            "name": "4 TiB",
            "value": "4",
            "rank": 4
          },
          {
            "name": "20 TiB",
            "value": "20",
            "rank": 20
          }
        ]
      }
    ]
  }
```

Add-on installations with this configuration would be able to upscale from 4 TiB to 20 TiB, but they would not be able
to downscale from 4 TiB to 0 TiB.

#### Validation

Custom validations can also be added to an addon by means of the `validation` field. This field should be a regular
expression, and any values given for this parameter must match the validation criteria, any request containing a value
that doesn't will result in a validation error. Validation is also used when setting `default_value`

#### Validation Error Message

Should a parameter not pass the validation criteria, OCM's default behavior is to return the regular expression where
the mismatch occurred. A custom message can be returned instead by setting the `validation_err_msg` field.

#### Default Value

`default_value` can be set to provide a placeholder value for the parameter in the portal. Note that `default_value` is
passed as a `string` regardless of the parameter type.
In particular boolean defaults are passed as `"True"` or `"False"` and **not** their YAML representations (`true`
, `false`).

#### Options

`options` can be set to provide a list of allowed values for this parameter, these will be presented in a dropdown
select component in the portal.

**Example:**

```yaml
  - id: "sku"
    name: "SKU"
    description: "SKU"
    value_type: string
    options:
      - name: "SKU1"
        value: "sku1"
      - name: "SKU2"
        value: "sku2"
    required: true
    editable: false
    enabled: true
```

If specifying `editable_direction`, it is recommended to add numerical `rank` values to the options (see above Editable
Direction for an example).

#### Conditions

`conditions` can be set to put conditions on when the parameter is to be included for a particular cluster.
If the cluster conditions for the parameter are not met by the target cluster, the parameter will be ignored, and will
not be displayed to the end user in the UI.

**Example:**

```yaml
  - id: "aws-sts-arn"
    name: "AWS STS ARN"
    description: "AWS STS ARN (Only required for AWS STS clusters)"
    value_type: string
    conditions:
      - resource: "cluster"
        data:
          aws.sts.enabled: true
          cloud_provider.id: 'aws'
    required: true
    editable: false
    enabled: true
```

#### Enabled

The aim is to not remove parameters after they have been added to the metadata. To deprecate or disable a parameter
set `enabled` to `false`

### Accessing parameter values

To access parameter values in cluster, an add-on parameters secret `addon-<<add_on_id>>-parameters` is created in
the `target namespace` of the add-on on the target cluster. This secret contains all the parameter information given at
installation time and is automatically updated as and when parameter values are updated.

```bash
oc get secret/addon-<<add_on_id>>-parameters -n <<target ns>> -o json | jq '.data | map_values(@base64d)'
{
  "my-param": "my value",
}
```
