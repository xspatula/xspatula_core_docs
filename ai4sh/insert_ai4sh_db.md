---
title:  "Insert initial data to AI4SH DB"
layout: single
sidebar:
  nav: "ai4sh"
excerpt: "The framework database requires some initial records to be defined directly at setup to function. Optionally it is also possible to insert initial users and some other default settings."
permalink: /ai4sh/insert_ai4sh_db
author_profile: false
date:   2026-03-22 06:13:03 +0200
last_modified_at:   2026-03-22 06:13:03 +0200
---

The framework database requires some initial records to be defined directly at setup to function. Optionally it is also possible to insert initial users and some other default settings.

## Prerequisites

Inserting initial data is an integrated part of the database [setup][db_setup], and thus rely on the same prerequisites.

## Insert table records

The framework process <span class='sub_process'>table_insert</span> can be used as part of the setup process to insert table records. This process in not available as a process outside the database setup. The process requires three parameters (arguments):

- schema (the name of the schema),
- table (the name of the table), and
- command (object that have 2 arrays as children, columns and values).

The column array lists the columns in the table and the values is an array of array that lists the actual values to insert.

The example below shows the command for inserting data to the table <span class='db_table'>process.root_process</span>:

```
{
  "process": [
    {
      "process_id": "table_insert",
      "overwrite": false,
      "delete": false,
      "parameters": {
        "schema": "process",
        "table": "root_process",
        "command": {
          "columns": [
            "root_process_id",
            "title",
            "label",
            "creator"
          ],
          "values": [
            [
              "manage_process",
              "Manage database defined process",
              "Mangaging a processes requires data on all parameters and their type and default values",
              "thomasg"
            ]
          ]
        }
      }
    },
```


The individual elements in the array can include any syntax that is compatible with postgreSQL.

If you want to change the definition on a table in the framework database, open the process file defining the table you want to change and edit the schema, table, or the elements in the command array. Then set overwrite to true and rerun the notebook setup_db, this will create, or drop and recreate, the table. Remember to reset overwrite to false once your table has the structure you wanted.

## Acknowledgments and Funding

This work was partly done as part of the AI4SoilHealth project, funded by the European Union's Horizon Europe Research and Innovation Programme under Grant Agreement No. 101086179.

_Funded by the European Union. The views expressed are those of the authors and do not necessarily reflect those of the European Union or the European Research Executive Agency._


[db_setup]: ./setup_ai4sh_db

[framework]: ../setup_db/

[vscode]: ../framework/vscode/

[postgres]: ../setup_db/postgres/

[anaconda]: ../setup_db/anaconda/

[netrc]: ../setup_db/netrc/

[notebook]: ../framework/notebook/

[scheme_file]: ../framework/scheme_file/

[scheme_pg_user]: ../framework/scheme_file/#define-pg_users

[job_file]: ../framework/job_file/

[pilot_file]: ../framework/pilot_file/

[process_file]: ../framework/process_file/

[schemas_tables]: ../setup_db/schemas_tables/

[schemas_tables_default_processes]: ../setup_db/schemas_tables/#default-processes

[framework_synopsis_user]: ../setup_db/schemas_tables/#default-human-users

[framework_table_insert]: ../setup_db/schemas_tables/#table-insert

[github]: https://github.com/

[github_framework_project_ai4sh_database]: https://github.com/AI4SH/framework_project_ai4sh_database
