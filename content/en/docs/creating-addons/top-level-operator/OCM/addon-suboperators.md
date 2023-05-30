---
title: Addon Sub-Operators
---

To support umbrella add-on operators, a list of sub operators can be defined in the add-on metadata using the
`subOperators` field. This field takes an array of operator objects:

```yaml
subOperators:
  - operator_name: 'foo'
    operator_namespace: 'bar'
    enabled: true
```

> **_NOTE:_** The `subOperator` `operator_name` and `operator_namespace` is used by OCM when defining the current
state and status of an Add-on. Eg, if one operator is in a `failed` state, the add-on status will be `failed`.
The life cycle of the `subOperators` relies on the add-on operator itself and **not** OCM.

### Operator Name

`operator_name` is a required field in the operator object.

### Operator Namespace

`operator_namespace` is a required field in the operator object.

### Enabled

Indicates if the operator is enabled or not. To deprecate or disable an operator set `enabled` to `false`, or
alternatively remove it from the sub operators list.
