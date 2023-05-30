---
title: Addon Validation
linkTitle: Addon Validation
---

## Getting Started

Start with creating/activating a virtual environment:

```bash
make init
```

View Makefile documentation

```bash
make help
```

Run test suite:

```bash
make test
```

## pre-commit

### Install on your developer machine

This is intended for developers, and will trigger hooks every time `git commit` is run:

```bash
pip install pre-commit
pre-commit install
```

### Run in virtualenv

This is intended for CI systems, to validate PRs/MRs:

```bash
make pre-commit
```

## Environment "load" Check

Before proceeding, make sure to install the `managedtenants` CLI tool using `pip`

```bash
pip install managedtenants-cli
```

The code repository for the `managedtenants` CLI can be found at <https://github.com/mt-sre/managed-tenants-cli>

Using the CLI, check the content of the `addons` directory for a given
environment:

```bash
$ managedtenants --environment=stage --addons-dir addons load
Loading stage...
Loading stage OK
```

That minimalistic `OK` means that all the `addons` are valid for the `stage`
environment.

## Task API

If you want to write additional checks for the `addons` content, you can do so
using the `Task` API. Start creating a new file in the `tasks/check/` directory.

Example:

`tasks/check/40-demo_task.py`

```python
from managedtenants import Task


class DummyCheck(Task):
    def run(self):
        assert self.addon.metadata['id'] is not None
```

- The `self.addon` object abstracts the entire content of each `addon`.
- The `Task` base class is the flag for the runner to execute this class.
- The `run` method is the entrypoint. Feel free to create any number of
  helper methods in the same class.

To run the check against all the addons:

```bash
$ managedtenants --environment=stage --addons-dir addons --dry-run run tasks/check/40-demo_task.py
Loading stage...
Loading stage OK
== TASKS =====================================
tasks/check/40-demo_task.py:DummyCheck:codeready-workspaces-operator:stage...
tasks/check/40-demo_task.py:DummyCheck:codeready-workspaces-operator:stage OK
tasks/check/40-demo_task.py:DummyCheck:codeready-workspaces-operator-qe:stage...
tasks/check/40-demo_task.py:DummyCheck:codeready-workspaces-operator-qe:stage OK
tasks/check/40-demo_task.py:DummyCheck:dba-operator:stage...
tasks/check/40-demo_task.py:DummyCheck:dba-operator:stage OK
tasks/check/40-demo_task.py:DummyCheck:integreatly-operator:stage...
tasks/check/40-demo_task.py:DummyCheck:integreatly-operator:stage OK
tasks/check/40-demo_task.py:DummyCheck:prow-operator:stage...
tasks/check/40-demo_task.py:DummyCheck:prow-operator:stage OK
```

To run the check against a selected addon:

```bash
$ managedtenants --environment=stage --addons-dir addons --dry-run run tasks/check/40-demo_task.py::integreatly-operator:
Loading stage...
Loading stage OK
== TASKS =====================================
tasks/check/40-demo_task.py:DummyCheck:integreatly-operator:stage...
tasks/check/40-demo_task.py:DummyCheck:integreatly-operator:stage OK
```

All the checks included in `tasks/check/` are expected to pass on Pull
Request check.

## Addon Metadata Operator

There are also checks defined in the [Addon Metadata Operator Repository](https://github.com/mt-sre/addon-metadata-operator)
in the [addon-metadata-operator/pkg/validator/ directory](https://github.com/mt-sre/addon-metadata-operator/tree/master/pkg/validator).
These checks are run by in the `mtcli validate` subcommand.
