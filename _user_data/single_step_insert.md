---
title: "Single-step insert (no translate step)"
layout: single
sidebar:
  nav: "add_user_data"
excerpt: "The insert_tabular_data process translates a spreadsheet and inserts the records into the database in one step, with no manual hand-off between a separate translate and insert step."
permalink: /user_data/single_step_insert/
author_profile: false
date: 2026-08-24
last_modified_at: 2026-08-24
---

The `insert_tabular_data` process does the same translation as `translate_tabular_data` — reading an Excel or CSV file and turning each row into a process command — but then immediately applies the result to the database, in the same step. There's no generated JSON file to hand off to a second, manually-run process: one notebook cell reads the spreadsheet and the records exist in the database.

This route is **INSERT-only by construction**: an existing record (matched on its natural key, the same lookup the two-step route's `manage_territory` step uses) is left untouched — never updated, never duplicated. That makes it safe to re-run the same cell as many times as you like, for example after adding new rows to the spreadsheet.

## Which route should I use?

- Use **single-step insert** when you just want the rows in your spreadsheet to end up in the database, and you don't need to inspect or hand-edit the generated JSON, or ever update existing rows through this route.
- Use the **two-step route** ([Translate excel][translate_excel] then [Insert data][insert_data]) when you need to update existing records, your source is already a JSON file with no spreadsheet at all, or you want to inspect or hand-edit the generated JSON before it's applied to the database.

Both routes can be used side by side — nothing about setting one up disables the other.

## Process definition file

The parameters are identical to `translate_tabular_data`'s — only the process name differs:

```json
{
  "process": [
    {
      "overwrite": true,
      "process": "insert_tabular_data",
      "parameters": {
        "process": "manage_territory",
        "tabular_data_path": "import_data/utility/excel/territory.xlsx",
        "dst_path": "import_data/utility/general/insert_process/staging"
      }
    }
  ]
}
```

| Parameter | Description |
|---|---|
| `process` | The target process name that will be written into each entry of the generated staging file |
| `tabular_data_path` | Path to the source spreadsheet |
| `dst_path` | Destination directory for the generated staging JSON file |

The generated staging file is a debug/audit trail, not something you're meant to hand-edit — unlike the two-step route's generated file, it's read and applied to the database in the same run it's written in.

## Project file structure

The single-step route adds a parallel set of files alongside the ones [Add user data][user_data] already documents — it never touches the existing `process/`/`manage_process/` folders:

```
project_example/import_data/utility/
├── insert_process/
│   └── territory.json          # process definition for the single-step route
├── general/insert_process/staging/
│   └── manage_territory.json   # generated AND applied immediately by this route
├── job_insert_utility.json     # job file for the single-step route
└── insert_utility.txt          # pilot file for the single-step route
```

### Job file

```json
{
  "process": {
    "job_folder": "import_data/utility",
    "process_sub_folder": "insert_process",
    "pilot_file": "insert_utility.txt"
  }
}
```

`process_sub_folder` points at `insert_process` — the folder searched for per-table process definitions for this route, distinct from `process/` (two-step translate) and `manage_process/` (two-step generated output).

### Pilot file

Like any [pilot file][pilot_file], `insert_utility.txt` lists one process file per table you want this route to cover, and a `#` in front of a line disables it for that run:

```
###===================###
### UTILITIES ###
###===================###

territory.json

# Add more tables the same way — one insert_process/<table>.json per line
# foreign_key.json
```

## Running it in the notebook

Open `project_example/import_data/insert_utility_data.ipynb` — the single-step sibling to `load_utility_data.ipynb`. Its imports and scheme-file cells are identical to that notebook; the difference is the cell that actually runs the process:

```python
#%%script false --no-raise-error
job_file = 'import_data/utility/job_insert_utility.json'

structured_process_D, scheme_params_D = Initiate_process(notebook_path, scheme_file, job_file)

if structured_process_D is not None:

    Run_process(structured_process_D, scheme_params_D)
```

Note this cell passes a **job file**, not a raw process file the way the two-step notebook's cells do — the job file is what resolves to the pilot list of per-table `insert_process/*.json` files above, so a single cell can cover every table the pilot file lists.

## Verifying the data

Same as the two-step route — connect to the database directly:

```sql
SELECT id, name, display_name, iso_code_a2
FROM utility.territory
ORDER BY name
LIMIT 10;
```

## Input files

| File | Purpose |
|---|---|
| `territory.xlsx` | Source spreadsheet; same file the two-step route reads |
| `job_insert_utility.json` | Job file; points at the `insert_process` folder and the pilot file below |
| `insert_utility.txt` | Pilot file; lists which `insert_process/<table>.json` files to run, in order |
| `insert_process/territory.json` | Process definition for the single-step route; specifies the target process, source spreadsheet path and staging output directory |

[translate_excel]: ../translate_excel/

[insert_data]: ../insert_data/

[user_data]: ../

[pilot_file]: ../../framework/pilot_file/
