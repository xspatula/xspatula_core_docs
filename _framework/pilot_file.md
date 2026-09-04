---
title:  "Xspatula pilot file"
layout: single
sidebar:
  nav: "framework"
excerpt: "A pilot file is an intermediate file linking a job file to multiple process files. You do not need to use a pilot file. It is, however, rather a convenient way to organise, comment and turn process files on or off."
permalink: /framework/pilot_file/
author_profile: false
date:   2026-03-15 16:13:03 +0200
last_modified_at:   2026-03-29 21:34:58 +0200
---

A pilot file is an intermediate file linking a [job file][job_file] to multiple [process files][process_file]. You do not need to use a pilot file. It is, however, rather a convenient way to organise, comment and turn individual process files on or off.

A pilot file must be stored directly under the project_path as defined in the [scheme file][scheme_file].

## Pilot file structure

A pilot file is simply an ordinary text file listing the process files to run. Empty lines are skipped and lines starting with a hashtag (#) are also ignored. This allows writing comments related to the listed process files, and by putting a hashtag before a process file you also exclude that process file form the defined chain of processes.

To run the framework with a Pilot file, the [job file][job_file] must include the object pilot_file with the name of the pilot_file to run as an attribute.

```
{
  "process": {
    "job_folder": "setup_db",
    "process_sub_folder": "jsonsql",
    "pilot_file": "db_setup.txt"
  }
}
```

Example pilot file:

```
#################################################
##### DEFINE XSPATULA DB SCHEMAS AND TABELS #####
#################################################

###===========================================###
### SCHEMAS ###
###===========================================###
#!!!!! The schemas must be defined before the tables

schema_v10_sql.json

###===========================================###
### GENERAL UTILITIES AND TERRITORIES###
###===========================================###
#!!!!! Territory codes are required by community

utility_v10_sql.json

utility_territory_v10_sql.json

###===========================================###
### COMMUNITY ###
###===========================================###
#!!!!! user categories required for adding users

community_user_categories_v10_sql.json

community_user_categories_records_v10_sql.json

#!!!!! organisations required for adding users
community_organisation_v10_sql.json

#!!!!! EDIT TO INCLUDE AT LEAST ONE DEFAULT ORGANISATION
community_organisation_records_v10_sql.json

#!!!!! users required for adding processes
community_user_v10_sql.json

#!!!!! EDIT TO INCLUDE AT LEAST ONE USER THAT WILL BE THE CREATOR OF THE PROCESSES
community_user_records_v10_sql.json

###===========================================###
### PROCESSES ###
###===========================================###
#!!!!! processes required for adding processes

processes_v10_sql.json

#!!!!! processes records adds the processes for adding all other processes
processes_records_v10_sql.json

...
```

## Input files

| File | Purpose |
|---|---|
| `db_xspatula_core_setup.txt` | Pilot file for database setup; lists the 10 JSON process files to run in order (schemas → tables → users → processes) |
| `xspatula_setup_processes.txt` | Pilot file for process registration; lists the 3 JSON process files to run in order (root processes → translate → territory) |

[scheme_file]: ../scheme_file

[job_file]: ../job_file/

[process_file]: ../process_file/
