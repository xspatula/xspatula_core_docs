---
title: "Edit or delete a process"
layout: single
sidebar:
  nav: "setup_processes"
excerpt: "To update a process registration, set overwrite to true in the process JSON file and re-run setup_processes.ipynb. To remove a process from the database, set delete to true."
permalink: /setup_processes/edit_process/
author_profile: false
date: 2026-06-09
last_modified_at: 2026-06-09
---

Process definitions registered in the database can be updated or deleted by modifying the relevant JSON process file and re-running `setup_processes.ipynb`.

## Updating a process

To update an existing process registration, set `overwrite` to `true` in the JSON entry:

```json
{
  "process": [
    {
      "process": "add_process",
      "overwrite": true,
      "parameters": {
        "root_process": "manage_table_data",
        "process": "manage_territory",
        "min_user_stratum": 3,
        "title": "Manage territory",
        "label": "Updated description."
      },
      "nodes": [
        ...
      ]
    }
  ]
}
```

With `overwrite: true` the framework replaces the existing process registration with the new definition. The `overwrite` flag in the [scheme file][scheme_file] also applies globally — a process-level setting takes precedence.

## Deleting a process

To delete a process from the database, set `delete` to `true`:

```json
{
  "process": [
    {
      "process": "add_process",
      "delete": true,
      "parameters": {
        "root_process": "manage_table_data",
        "process": "manage_territory"
      }
    }
  ]
}
```

Deleting a root process removes it and all sub-processes registered under it. Only set `delete: true` when you are sure the process is no longer used anywhere in the framework.

## Re-running the notebook

After editing the JSON file, open `setup/setup_processes.ipynb` and run all three code blocks. The notebook re-reads the pilot file and processes the updated entries.

If you only want to run a subset of the process files, comment out the ones you do not need in the pilot file with a `#` before re-running:

```
# root_process/root_processes_v10_sql.json  # skip — already registered

# translate/translate_tabular_data_v10_sql.json  # skip

utility/territory_v10_sql.json  # run only this one
```

## Input files

| File | Purpose |
|---|---|
| `xspatula_setup_processes.txt` | Pilot file; comment out lines to run only the process files that need updating |
| `root_processes_v10_sql.json` | Root process definitions — set `overwrite` or `delete` here to modify root groups |
| `translate_tabular_data_v10_sql.json` | `translate_tabular_data` process definition |
| `territory_v10_sql.json` | `manage_territory` process definition |

[scheme_file]: ../scheme_file/
