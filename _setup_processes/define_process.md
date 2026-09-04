---
title: "Define and register a process"
layout: single
sidebar:
  nav: "setup_processes"
excerpt: "A process is registered in the database by creating a JSON file with add_root_process or add_process entries and adding it to the pilot file. Running setup_processes.ipynb then inserts the process definitions into the database."
permalink: /setup_processes/define_process/
author_profile: false
date: 2026-06-09
last_modified_at: 2026-06-09
---

A process must be registered in the database before it can be called from a [process file][process_file]. Registration stores the process name, parameter definitions and access constraints in the `process` schema tables. By default only stratum 5 users have the rights to register new processes.

## Process hierarchy

Every process belongs to a root process. You must register the root process before registering the actual processes under it.

### Registering a root process

Use `add_root_process` to create a new root process group:

```json
{
  "process": [
    {
      "process": "add_root_process",
      "overwrite": false,
      "parameters": {
        "root_process": "manage_table_data",
        "title": "Manage table data",
        "label": "Root for processes for inserting, updating and deleting table data"
      }
    }
  ]
}
```

#### Root process parameters

A root process is registered with only 3 parameters:
- root_process, the name of the root process,
- title, a short title,
- label, a short description

### Registering a process

Use `add_process` to register a process under an existing root. The `nodes` array defines each parameter the process accepts:

```json
{
  "process": [
    {
      "process": "add_process",
      "overwrite": false,
      "parameters": {
        "root_process": "manage_table_data",
        "process": "manage_territory",
        "min_user_stratum": 5,
        "title": "Manage territory",
        "label": "Insert, update or delete a territory, using ISO 3166 naming convention."
      },
      "nodes": [
        {
          "parent": "process",
          "element": "parameters",
          "parameter": [
            {
              "parameter": "name",
              "parameter_type": "text",
              "required": true,
              "default_value": "",
              "hint": "Territory name",
              "schema_table": {
                "schema": "utility",
                "table": "territory",
                "write": true
              },
              "permission": {
                "update": false,
                "delete": false
              }
            }
          ]
        }
      ]
    }
  ]
}
```
#### Process parameters and nodes

A process is registered with 5 parameters:
- root_process, the name of the root process to which the process belongs,
- process, the name of process
- min_user_stratum, minimum user privilege level required to run the process
- title, a short title,
- label, a short description

Parameters required for running the process being defined are listed in the `nodes` array. Each node object must include objects for `parent` and `element`, that can be used for creating nested parameter settings. The parameters accepted by a process are defined in the array `parameter`, where each entry must contain the following objects:
- parameter, the name of the parameter
- parameter_type, the data type of the parameter
- required, true if compulsory, false if a default value exists
- default_value, the default value to use if required is set to false and no custom value is set
- hint, short clue on the objective of the parameter

In addition parameters that are directly translated to records in the database must also have a `schema_table` block that links a parameter to a target table in the database. Such parameters also have a `permission` block that defines if a parameter can be updated or deleted.

#### Process parameters data type

The following data types are accepted as values for the parameter `parameter_type` — this is the complete, enforced set (see [Bootstrapping process management][manage_process] for where it's defined):
- text
- real
- int
- intlist
- textlist
- bool

If a parameter represents an array, it's the parameter's *name* that carries an `_array` suffix (e.g. `substance_array`), not the `parameter_type` value — the suffix on the name is what triggers the framework's array handling.

#### Optional process parameter blocks

There are four additional blocks that can be used for [defining process options][define_process_options]:
- [set value][define_process_options_set_value], lists values accepted as input
- [minmax][define_process_options_minmax], defines the range accepted for numerical values
- [inherit][define_process_options_inherit], copies value from existing database record
- [auto naming][define_process_options_auto_naming], assigns an automatic value

## Process schema tables

Everything described above is stored in nine tables in the `process` schema. You'll rarely query these directly — the notebook and JSON files do that for you — but knowing what each table holds helps when troubleshooting a registration:

| Table | Objective |
|---|---|
| `root_process` | Groups related processes into a named root category (e.g. `manage_table_data`) that sub-processes register under. |
| `process` | Registers each individual process: its name, parent root process, and the minimum user privilege stratum (`min_user_stratum`) required to run it. |
| `process_parameter` | Defines every parameter a process accepts — type, whether required, default value, and hint text. |
| `process_parameter_set_value` | Restricts a parameter to a predefined [set of accepted values][define_process_options_set_value]. |
| `process_parameter_minmax` | Restricts a numerical parameter to a [min/max range][define_process_options_minmax]. |
| `process_parameter_schema_table` | Links a parameter to the target `schema.table` it writes to in the database (the `schema_table` block). |
| `process_parameter_permission` | Records whether a parameter's underlying column may be updated or deleted once set (the `permission` block). |
| `process_parameter_inherit` | Lets a parameter [default to a value copied][define_process_options_inherit] from an existing database record. |
| `process_parameter_auto_name` | Stores the [concatenation pattern][define_process_options_auto_naming] used to auto-generate a parameter's value from other parameters in the same process. |

## Adding the process file to the pilot file

Once you have created the JSON process file, add it to the pilot file `xspatula_setup_processes.txt`:

```
# My new process group
root_process/my_root_processes.json

my_process/my_process_v10_sql.json
```

Lines starting with `#` are comments and are skipped. The root process must appear before any sub-processes that reference it.

## Running setup_processes.ipynb

Open `setup/setup_processes.ipynb` and run all three code blocks. The notebook reads the pilot file and registers each process in the database in the listed order. Use `verbose: 2` in the scheme file if you want to see the parameter details as they are inserted.

## Default processes registered

The three JSON files included in the default setup illustrate the pattern described above:

| File | What it registers |
|---|---|
| `root_processes_v10_sql.json` | Two root process groups: `manage_table_data` and `translate_data` |
| `translate_tabular_data_v10_sql.json` | The `translate_tabular_data` sub-process under `translate_data` |
| `territory_v10_sql.json` | The `manage_territory` sub-process under `manage_table_data` |

## Input files

| File | Purpose |
|---|---|
| `xspatula_setup_processes.txt` | Pilot file; lists the JSON process files to register, in execution order |
| `root_processes_v10_sql.json` | Registers the `manage_table_data` and `translate_data` root process groups |
| `translate_tabular_data_v10_sql.json` | Registers the `translate_tabular_data` sub-process for converting spreadsheet data to JSON |
| `territory_v10_sql.json` | Registers the `manage_territory` sub-process for inserting and updating territory records |

[process_file]: ../../framework/process_file/
[define_process_options]: ../process_options
[define_process_options_set_value]: ../process_options#set-value
[define_process_options_minmax]: ../process_options#minmax-ranges
[define_process_options_inherit]: ../process_options#inherit
[define_process_options_auto_naming]: ../process_options#automatic-naming
[manage_process]: ../../setup_db/manage_process/
