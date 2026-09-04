---
title:  "Xspatula job file (project) structure"
layout: single
sidebar:
  nav: "framework"
excerpt: "A job file is a JSON file defining 1. a job folder where other files related to this job (project) are found, 2. the process_sub_folder in that job_folder where the actual process files are stored, and 3 one of the following three objects: pilot_file, pilot_list or process_file."
permalink: /framework/job_file/
author_profile: false
date:   2026-03-15 16:13:03 +0200
last_modified_at:   2026-03-15 16:13:03 +0200
---

A job file is a JSON file defining:
1. a job_folder where other files related to this job (project) are found,
2. the process_sub_folder in that job_folder where the actual process files are stored, and
3. one of the following three objects: pilot_file, pilot_list or process_file.

**NOTE** if you are running a single process file you do not need a job file, you can give the process file directly in the notebook. Process files can also list any number of processes, and if you only have a few processes to run, calling the process file directly from the notebook is probably easier. If your project is large and complex, using an intermediate job file allows you to run processes in logical batches.

Example of a job file linking to an external list of process files in a [pilot_file][pilot_file]:
```
{
  "process": {
    "job_folder": "setup_db",
    "process_sub_folder": "process_files",
    "pilot_file": "db_setup.txt"
}
```

Example of a job file linking to a list of internally defined process file:
```
{
  "process": {
    "job_folder": "setup_db",
    "process_sub_folder": "process_files",
    "pilot_list": [
      schema_v10_sql.json,
      processes_v10_sql.json,
      processes_records_v10_sql.json,
      utility_v10_sql.json,
      community_organisation_v10_sql.json,
      community_organisation_records_v10_sql.json
    ]
  }
}
```

Example of a job file linking to a single process file:
```
{
  "process": {
    "job_folder": "setup_db",
    "process_sub_folder": "process_files",
    "process_file": "schema_v10_sql.json"
  }
}
```

### Hierarchical path structure

The directory entered as the object job_folder must be directly under the project_path where the job file itself is saved. There is no option for changing a project hierarchy inside a project. You can, however, name the job_folder freely. The object process_sub_folder in the job file likewise always looks for a folder directly under the job_folder, and this is where you must store the process files that define what the framework should do. You can also freely name the process_sub_folder.

## Input files

| File | Purpose |
|---|---|
| `job_setup_db.json` | Job file for database setup; points to the `setup_db` job folder, `json_core` process sub-folder, and `db_xspatula_core_setup.txt` pilot file |
| `job_setup_processes.json` | Job file for process registration; points to the `setup_processes` job folder, `json_xspatula` process sub-folder, and `xspatula_setup_processes.txt` pilot file |

[pilot_file]: ../pilot_file/
