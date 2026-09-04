---
title: "Setup Xspatula processes"
layout: single
sidebar:
  nav: "setup_processes"
excerpt: "After setting up the database, the next step is to register the processes that the framework should be able to run. Process registration uses the setup_processes notebook and a category-5 user — not the postgres superuser."
permalink: /setup_processes/
author_profile: false
date: 2026-06-09
last_modified_at: 2026-06-09
---

After setting up the Xspatula database, the next step is to register the processes that the framework should be able to run. Process definitions are stored in the database and the framework looks them up at runtime to validate parameters and execute the correct functions.

Process registration is handled by the notebook:

```
./setup/setup_processes.ipynb
```

## Prerequisites

Before running `setup_processes.ipynb`, the database must already exist. See the [Setup DB][setup_db] section for instructions on creating the database.

Process registration uses an ordinary database user — not the postgres superuser. The initial run of `setup_processes.ipynb` must thus use one of the users defined with the [database setup][setup_db]. The scheme file for this step therefore has a different structure from the one used during [database setup](http://127.0.0.1:4000/setup_db/schemas_tables/#table-insert). See the [Scheme file][scheme_file] page for details.

## File hierarchy

The same four-level hierarchy used for database setup applies here:

```
.
├── scheme_xspatula_local_use.json   # scheme file — project path + user credentials
└── xspatula/
    ├── job_setup_processes.json     # job file — points to job folder, sub-folder and pilot file
    └── setup_processes/
        ├── xspatula_setup_processes.txt   # pilot file — lists process JSON files to run
        └── json_xspatula/                 # process sub-folder
            ├── root_process/
            │   └── root_processes_v10_sql.json
            ├── translate/
            │   └── translate_tabular_data_v10_sql.json
            └── utility/
                └── territory_v10_sql.json
```

## Running setup_processes.ipynb

The notebook has the same block structure as all Xspatula notebooks. With code block 1 importing the required python packages and modules, and code block 2 setting the scheme file:

```python
scheme_file = './zzz/scheme_xspatula_local_use.json'
```

Code block 3 loads the job file and runs the processes:

```python
#%%script false --no-raise-error
job_file = 'job_setup_processes.json'

structured_process_D, scheme_params_D = Initiate_process(notebook_path, scheme_file, job_file)

if structured_process_D is not None:

    Run_process(structured_process_D, scheme_params_D)
```

## Default processes registered

The default pilot file registers processes in order:

1. **Root processes** — two root process groups (`manage_table_data` and `translate_data`) that parent the processes
2. **translate_tabular_data** — converts Excel or CSV files to Xspatula JSON process files in 2-step process
3. **insert_tabular_data** — converts Excel or CSV files to Xspatula JSON process files in 1-step process
4. **foregin_key** — inserts, updates or deletes foreign key records in `utility.foreign_key`
5. **manage_territory** — inserts, updates or deletes territory records in `utility.territory`
6. **manage_organisation** — inserts, updates or deletes organisation records in `community.organisation`
7. **manage_user** — inserts, updates or deletes user records in `community.user`

For details on defining your own processes, see [Define and register a process][define_process].

## Input files

| File | Purpose |
|---|---|
| `scheme_xspatula_local_use.json` | Scheme file for process registration; defines project path and a category-5 user login — no superuser credentials |
| `job_setup_processes.json` | Job file; points to the `setup_processes` job folder, `json_xspatula` sub-folder and `xspatula_setup_processes.txt` pilot file |
| `xspatula_setup_processes.txt` | Pilot file; lists the JSON process files to run in order |

[setup_db]: ../../setup_db/

[scheme_file]: /setup_processes/scheme_file/

[define_process]: /setup_processes/define_process/
