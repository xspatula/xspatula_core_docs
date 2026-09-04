---
title: "Bootstrapping process management"
layout: single
sidebar:
  nav: "setup_db"
excerpt: "Before add_root_process or add_process can be called by anyone, they have to exist in the database — a special table_insert bootstraps them, and its stratum_code values decide who gets to register new processes at all."
permalink: /setup_db/manage_process/
author_profile: false
date: 2026-08-25
last_modified_at: 2026-08-25
---

Everything documented in [Define and register a process][define_process] — calling `add_root_process` and `add_process` to register your own processes — depends on those two processes already existing in the database. This page covers how they get there in the first place, and why the privilege level required to use each of them matters.

## Why this file is special

`processes_records_v10_sql.json` doesn't call `add_root_process` or `add_process` the normal way. It can't: at the point this file runs, during the superuser-driven initial database setup, neither process has been registered yet — nothing exists to call. Instead it uses the same `table_insert` bypass already covered in [Table insert][table_insert] (the mechanism that also seeds the default users) to write the necessary rows directly into the `process` schema's own tables.

## The ordering dependency

The root-process insert below sets `creator: "ini_cat5"` — and `root_process.creator` is a foreign key to `community.user.user_name`. That insert only succeeds if `ini_cat5` already exists as a row in `community.user`. This is exactly why the pilot file runs the [default-user records][setup_db_default_users] before `processes_records_v10_sql.json` — not an arbitrary ordering choice, but a hard requirement.

## The manage_process root process

One `root_process` row, inserted directly:

```json
{
  "process_id": "table_insert",
  "parameters": {
    "schema": "process",
    "table": "root_process",
    "command": {
      "columns": ["root_process", "title", "label", "creator"],
      "values": [
        ["manage_process", "Manage database defined process",
         "Mangaging a processes requires data on all parameters and their type and default values",
         "ini_cat5"]
      ]
    }
  }
}
```

Every process that lets you register other processes — `add_root_process` and `add_process` — belongs to this one root.

## The two processes, and why their privilege levels differ

Two `process` rows, both under `manage_process`, each with a different `min_user_stratum`:

```json
{
  "process_id": "table_insert",
  "parameters": {
    "schema": "process",
    "table": "process",
    "command": {
      "columns": ["root_process", "process", "min_user_stratum", "title", "label", "creator"],
      "values": [
        ["manage_process", "add_root_process", "5",
         "Add root process to database",
         "Root processes are containers for processes having similar input/output requirements",
         "ini_cat5"],
        ["manage_process", "add_process", "4",
         "Add sub process to database",
         "Adding a process requires data on all parameters and their type and default values",
         "ini_cat5"]
      ]
    }
  }
}
```

| Process | `min_user_stratum` | Why |
|---|---|---|
| `add_root_process` | 5 (highest tier) | Creating a new root-process category is the most structurally significant action available — a new root defines an entire family of processes. |
| `add_process` | 4 | Registering a sub-process under an *existing* root is one tier less privileged than creating the root itself. |

This is the literal mechanism that decides who can register new processes on your database: only a user logged in at stratum 5 or above can call `add_root_process`; stratum 4 is enough for `add_process`. Everything in [Define and register a process][define_process] assumes a user with at least one of these strata.

## The parameters behind add_root_process and add_process

The `process_parameter` rows inserted alongside the two processes above are, literally, the schema backing the JSON shapes `add_root_process`/`add_process` already accept — the same `root_process`/`title`/`label` and `root_process`/`process`/`min_user_stratum`/`title`/`label` fields shown in [Define and register a process][define_process], plus the `node`/`parameter` substructure that lets `add_process` accept an arbitrary list of parameters for the process being registered:

```json
["add_root_process", "process", "parameters", "root_process", "text", "True", "", "Root process name"],
["add_root_process", "process", "parameters", "min_user_stratum", "integer", "False", "5", "minimum user stratum for using the process"],

["add_process", "process", "parameters", "root_process", "text", "True", "", "Root process to which this process belongs"],
["add_process", "process", "parameters", "process", "text", "True", "", "The name of the process to add"],
["add_process", "process", "parameters", "min_user_stratum", "integer", "False", "4", "minimum user stratum for using the process"],
["add_process", "node", "parameter", "parameter", "text", "True", "", "Process node parameter id"],
["add_process", "node", "parameter", "parameter_type", "text", "True", "", "Process node parameter type"]
```

## The authoritative list of parameter_type values

`add_process`'s own `parameter_type` parameter (the one you fill in for each parameter of the process you're registering) is itself restricted to a fixed set of values, via a `process_parameter_set_value` insert:

```json
["add_process", "parameter_type", "node", "parameter", "text", "text or string"],
["add_process", "parameter_type", "node", "parameter", "real", "real number"],
["add_process", "parameter_type", "node", "parameter", "int", "integer number"],
["add_process", "parameter_type", "node", "parameter", "intlist", "list of integer numbers"],
["add_process", "parameter_type", "node", "parameter", "textlist", "list of text strings"],
["add_process", "parameter_type", "node", "parameter", "bool", "boolean"]
```

These six values — `text`, `real`, `int`, `intlist`, `textlist`, `bool` — are the complete, enforced set. This is the same list [Define and register a process][define_process] now documents; if the two ever look inconsistent, this file is the source of truth.

## Input files

| File | Purpose |
|---|---|
| `processes_v10_sql.json` | Creates the `process` schema tables (`root_process`, `process`, `process_parameter`, etc.) |
| `processes_records_v10_sql.json` | Bootstraps the `manage_process` root process and the `add_root_process`/`add_process` processes covered on this page |

[define_process]: ../../setup_processes/define_process/
[table_insert]: ../schemas_tables/#table-insert
[setup_db_default_users]: ../#default-human-users
