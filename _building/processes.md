---
title: "Adding a new process"
layout: single
sidebar:
  nav: "building"
excerpt: "A new root process needs a JSON registration, a line in your pilot file, and a hand-written Python handler — nothing about the wiring between them is automatic beyond one database lookup."
permalink: /building/processes/
author_profile: false
date: 2026-08-24
last_modified_at: 2026-08-24
---

Every process documented elsewhere on this site — `translate_tabular_data`, `manage_territory`, `manage_organisation` — is registered under one of two root processes the framework ships by default (`translate_data`, `manage_table_data`). A real project outgrows those. This page walks through adding an entirely new root process, using a generic worked example: a `report` root process with two sub-processes, `report_summary` and `report_chart`.

**The two layers are worth keeping straight from the start:** registering a process (writing it into the database's `process` table) is a completely separate step from what runs when that process is actually called. Confusing the two is the most common way this goes wrong.

## Layer 1: registering the process

This is exactly the mechanism already documented in [Define and register a process][setup_processes_define] — nothing new here, just applied to a root process that doesn't exist yet. A JSON file with one `add_root_process` block and one `add_process` block per sub-process:

```json
{
  "process": [
    {
      "process": "add_root_process",
      "overwrite": false,
      "parameters": {
        "root_process": "report",
        "title": "Reporting",
        "label": "Root for generating reports from stored data"
      }
    },
    {
      "process": "add_process",
      "overwrite": false,
      "parameters": {
        "root_process": "report",
        "process": "report_summary",
        "min_user_stratum": 1,
        "title": "Summary report",
        "label": "Generate a summary report for a dataset."
      },
      "nodes": [
        {
          "parent": "process",
          "element": "parameters",
          "parameter": [
            {
              "parameter": "dataset_name",
              "parameter_type": "text",
              "required": true,
              "default_value": "",
              "hint": "Name of the dataset to summarize"
            }
          ]
        }
      ]
    },
    {
      "process": "add_process",
      "overwrite": false,
      "parameters": {
        "root_process": "report",
        "process": "report_chart",
        "min_user_stratum": 1,
        "title": "Chart report",
        "label": "Generate a chart for a dataset."
      },
      "nodes": [
        {
          "parent": "process",
          "element": "parameters",
          "parameter": [
            {
              "parameter": "dataset_name",
              "parameter_type": "text",
              "required": true,
              "default_value": "",
              "hint": "Name of the dataset to chart"
            },
            {
              "parameter": "chart_type",
              "parameter_type": "text",
              "required": false,
              "default_value": "bar",
              "hint": "Kind of chart to draw"
            }
          ]
        }
      ]
    }
  ]
}
```

Place this under your own `setup/zzz/<project>/setup_processes/json_<project>/report/report_v10_sql.json`, add a line for it to your setup-processes pilot file (the root process must come before its sub-processes — within one JSON file, as above, that ordering is handled for you), and run `setup_processes.ipynb`. This writes rows into the database's `process` and `root_process` tables — nothing runs yet, it's purely registration.

### Updating a process

If you discover an error in your process definition, or change the type or name of a parameter, it is safe to user `overwrite: true` option. This will recreate the registered process and is a very useful tool when developing the database.

### Deleting a process

To delete a process registered in the database set the `delete: true` option. If you set both `overwrite: true` and `delete: true`, overwriting takes precedence.

### Changing the name of the process itself

If you want to change the name of an existing process, start by using the `delete: true` option and run the notebook, then rename the process and set `delete: false` and rerun the notebook.

## Layer 2: what actually runs

This is the part that's entirely hand-written, and it's worth being explicit about that: **there is no decorator, registry, or naming convention that automatically connects a registered process name to Python code.** You write both connections yourself.

### How a registered process finds your code at runtime

When a job actually calls `report_summary`, the framework looks up that process name in the database and reads back its `root_process` — this one lookup is the only automatic part of the whole chain. Everything downstream is a plain `if`/`elif` chain you maintain by hand, in two places:

**1. Your project's `process.py`** dispatches by `root_process`. Compare this to the two branches (`translate_data`, `manage_table_data`) the shipped `src/example_project/process.py` already has — you add one more:

```python
from src.<project>.report import Process_report

...

if root_process == 'translate_data':
    ...

elif root_process == 'manage_table_data':
    ...

elif root_process == 'report':
    report_C = Process_report(process_S, pg_session_C)
    report_C._Sub_process(key)
```

**2. A handler module you write** (`src/<project>/report.py`), with its own `if`/`elif` chain — this time by exact process name, not root process — calling one private method per registered sub-process:

```python
class Process_report(Get_schema_table):

    def __init__(self, process_S, pg_session_C):
        self.process_S = process_S
        self.pg_session_C = pg_session_C

    def _Sub_process(self, _json_file_key):

        if self.process_S.process.process == 'report_summary':
            self._Report_summary()

        elif self.process_S.process.process == 'report_chart':
            self._Report_chart()

    def _Report_summary(self):
        '''Implementation goes here.'''

    def _Report_chart(self):
        '''Implementation goes here.'''
```

The method names (`_Report_summary`, `_Report_chart`) are a convention you're free to choose — the framework doesn't derive them from the process name automatically. Match them to whatever you wrote in the `if`/`elif` chain and that's the whole contract.

If this handler needs project-specific SQL beyond the generic framework helpers, this is also where you'd construct `self.pg_<project>_C` — see [Setting up the database][building_database]'s postgres-extension section.

## What you created

Three things, matching the three items above: one JSON registration file, one pilot-file line, and one Python module — plus the one-line edit to your existing `process.py`. See [Postgres — summary of edits][building_postgres] for the consolidated list alongside the database-extension pattern from the previous page.

[setup_processes_define]: /setup_processes/define_process/
[building_database]: /building/database/
[building_postgres]: /building/postgres/
