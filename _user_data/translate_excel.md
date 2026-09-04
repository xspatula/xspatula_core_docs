---
title: "Translate Excel data to JSON"
layout: single
sidebar:
  nav: "add_user_data"
excerpt: "The translate_tabular_data process reads a spreadsheet and generates a JSON process file where each row becomes one process command. This is step 1 of the user data pipeline."
permalink: /user_data/translate_excel/
author_profile: false
date: 2026-06-09
last_modified_at: 2026-06-09
---

The `translate_tabular_data` process reads an Excel or CSV file and generates a JSON process file where each row becomes one process command entry. This is step 1 of the user data pipeline — it does not write anything to the database.

## Process definition file

The translation is controlled by a JSON process file. The example for the territory data looks like this:

```json
{
  "process": [
    {
      "process": "translate_tabular_data",
      "parameters": {
        "process": "manage_territory",
        "tabular_data_path": "../../utility/excel/territory.xlsx",
        "dst_path": "../../utility/manage_process"
      }
    }
  ]
}
```

| Parameter | Description |
|---|---|
| `process` | The target process name that will be written into each entry of the output JSON file |
| `tabular_data_path` | Path to the source spreadsheet (relative to the process file itself) |
| `dst_path` | Destination directory for the generated JSON output file |

Paths can be relative (as above) or absolute.

## Source spreadsheet

The spreadsheet `territory.xlsx` contains one row per territory. Each column name maps to a parameter of the target process (`manage_territory`). For the territory example the columns are:

- `name` — territory name in lowercase (e.g. `afghanistan`)
- `display_name` — formatted territory name (e.g. `Afghanistan`)
- `iso_code_a2` — ISO 3166-1 alpha-2 code (e.g. `af`)
- `iso_code_a2_ext` — extended alpha-2 code (same as `iso_code_a2` for most territories)

The spreadsheet column names must match the parameter names of the target process exactly.

## Generated output

After running the translate step, the framework writes a JSON file to the `dst_path` directory. The filename is derived from the `process` parameter. For the territory example the output is `manage_territory.json` with one entry per row:

```json
{
  "process": [
    {
      "root_process_id": "import_tabular_data",
      "process": "manage_territory",
      "delete": false,
      "overwrite": false,
      "parameters": {
        "name": "afghanistan",
        "display_name": "Afghanistan",
        "iso_code_a2": "af",
        "iso_code_a2_ext": "af"
      }
    },
    {
      "root_process_id": "import_tabular_data",
      "process": "manage_territory",
      "delete": false,
      "overwrite": false,
      "parameters": {
        "name": "åland islands",
        "display_name": "Åland Islands",
        "iso_code_a2": "ax",
        "iso_code_a2_ext": "ax"
      }
    },
    ...
  ]
}
```

## Running step 1 in the notebook

Open `project_example/import_data/load_utility_data.ipynb`. Step 1 (code block 3) runs the translate process:

```python
#%%script false --no-raise-error
process_file = 'import_data/utility/process/territory.json'

structured_process_D, scheme_params_D = Initiate_process(notebook_path, scheme_file, process_file)

if structured_process_D is not None:

    Run_process(structured_process_D, scheme_params_D)
```

After this block completes, the file `utility/manage_process/manage_territory.json` is created and ready for the insert step.

## Input files

| File | Purpose |
|---|---|
| `territory.xlsx` | Source spreadsheet; ~200 rows of national territory data with ISO 3166-1 codes |
| `territory.json` | Process definition for the translate step; specifies the target process, source spreadsheet path and output directory |

## Output files

| File | Purpose |
|---|---|
| `manage_territory.json` | Process definitions for inserting territory data. |
