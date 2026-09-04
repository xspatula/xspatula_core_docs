---
title:  "Defining schemas and tables"
layout: single
sidebar:
  nav: "setup_db"
excerpt: "In the Xspatula framework, schemas and tables are defined as part of the database setup. Only the superuser can create, update or delete schemas and tables."
permalink: /setup_db/schemas_tables/
author_profile: false
date:   2026-03-15 16:13:03 +0200
last_modified_at:   2026-03-29 21:43:13 +0200
---

In the Xspatula framework, schemas and tables are defined as part of the database setup. As long as you do not set overwrite or delete to true, no data will be lost. But if you set overwrite to true, existing data in the table you overwrite will be lost. If you set delete to true the table is removed and all existing data lost.

## Create schema

Defining a schema is simple, you just create a process file, or edit the default JSON process file for creating schema with the path ./schema_v10_sql.json:

```
{
  "process": [
    {
      "process_id": "create_schema",
      "parameters": {
        "schema": "observation"
      }
    },
    {
      "process_id": "create_schema",
      "parameters": {
        "schema": "observation_utility"
      }
    },
    {
      "process_id": "create_schema",
      "parameters": {
        "schema": "utility"
      }
    },
    {
      "process_id": "create_schema",
      "parameters": {
        "schema": "process"
      }
    },
    {
      "process_id": "create_schema",
      "parameters": {
        "schema": "community"
      }
    }
  ]
}
```

## Create table

To create a table is more complicated and requires properly structured Standard Query Language (SQL) syntax. The syntax is not evaluated by the framework but is executed as it is given by the user. If there are errors in the syntax the script will crash with a rudimentary feedback on the error.

Framework default process for _create_table_, example schema.table community.user, command objects ending with '_id' or '_code' denote foreign keys from other tables; the hashtag comments are added as explanations and should not be included in production files.
```
{
  "process": [
    {
      "process_id": "create_table",
      "overwrite": false, # added for security so the table is not overwritten by mistake
      "delete": false, # added for security so the table is not deleted by mistake
      "parameters": {
        "schema": "community", # target schema for this table
        "table": "user", # name of table
        "command": [
          "id SERIAL", # automatically incremented value giving all users a unqique id
          "organisation_id INTEGER REFERENCES community.organisation (id)", # the id of the organisation for this user, 0 = no organisation
          "user_name VARCHAR UNIQUE", # unique user name
          "password VARCHAR", # bcrypt hash of the user's password, generated with setup/hash_password.py - never plain text
          "stratum_code SMALLINT DEFAULT 1", # stratum_code is the privilege level for the user, 1 = few rights
          "first_name VARCHAR", # first name of user
          "middle_name VARCHAR", # middle name of user
          "last_name VARCHAR", # last (family) name of user
          "email VARCHAR UNIQUE", # email of user (must be unique)
          "email_alt VARCHAR", # alternative email of user
          "address1 VARCHAR", # physical address of user (first line)
          "address2 VARCHAR", # physical address of user (second line)
          "postal_address VARCHAR", # postal address of user
          "postal_zip_code VARCHAR", # postal zip code of user
          "state VARCHAR", # state the user is associated with
          "territory_id INTEGER REFERENCES utility.territory (id)", # id of territory, where territory must be defined in another table if used
          "telephone VARCHAR", # telephone number of user
          "department VARCHAR", # department the user
          "section VARCHAR",
          "position VARCHAR",
          "create_timestamp TIMESTAMP DEFAULT CURRENT_TIMESTAMP(1)", # automatic timestamp of registering in the database
          "last_update_timestamp TIMESTAMP DEFAULT CURRENT_TIMESTAMP(1)", # automatic timestamp of last activity
          "status_code INTEGER DEFAULT 10", # status_codes include active, passive, dormant, ended etc
          "PRIMARY KEY (user_name)"
        ]
      }
    }  
  ]
}
```

## Table insert

While the intention is to build processes that interact with the database, inserting, updating and deleting records, there is a bypass function available when setting up the database to directly insert values in the database. Among other things, this allows defining default users for the database at setup, eliminating the need for using the superuser to login for creating the first users. Thus the default processes defined for setting up the database include the creation of some default users:
- a user for handling other users,
- a user with stratum 5 privileges to access all data, and
- Jane Doe (the example user in the documentation) as a stratum 1 (restricted) user.

None of these users are linked to any organisation ("organisation_id": 0).

You can use the suggested users to test setting up the database. Then delete the whole database and define your own users and recreate the database.

The `password` value for each user below is **not** the plain-text password anyone would type in — it's a bcrypt hash. Before editing this file with your own users, choose a plain-text password for each one and run it through `setup/hash_password.py` (see [hashing user passwords][setup_db_hash_passwords] on the Setup DB page), then paste the printed hash in place of the plain text. The values shown below (`$2b$12$...`) are illustrative placeholders, not real hashes.

```
{
  "process": [
    {
      "process_id": "table_insert",
      "overwrite": false,
      "delete": false,
      "parameters": {
        "schema": "community",
        "table": "user",
        "command": {
          "columns": [
            "organisation_id",
            "email",
            "email_alt",
            "password",
            "first_name",
            "last_name",
            "user_name",
            "territory_id",
            "stratum_code"
          ],
          "values": [
            [
              "0",
              "user_manager_email@example.com",
              "user_manager_email_alt@example.com",
              "$2b$12$examplehashforusermanagerxxxxxxxxxxxxxxxxxxxxxxxxxxxx",
              "user",
              "manager",
              "user_manager",
              "0",
              "6"
            ],
            [
              "0",
              "cat_5_email@example.com",
              "cat_5_email_alt@example.com",
              "$2b$12$examplehashforinicat5userxxxxxxxxxxxxxxxxxxxxxxxxxxxx",
              "ini_cat_5_user",
              "Ini",
              "Cat",
              "0",
              "5"
            ],
            [
              "0",
              "jane_doe@example.com",
              "jane_doe_alt@example.com",
              "$2b$12$examplehashforjanedoexxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx",
              "Jane",
              "Doe",
              "jane_doe",
              "0",
              "1"
            ]
          ]
        }
      }
    }
  ]
}
```

## Default processes

The most important insert process is the adding of the root and sub processes that define the structure for adding any other process. All definitions related to defining processes are in the schema _process_ holding the following tables:

- root_process (for grouping processes)
- process (actual processes linked to framework script functions)
- process_parameter (parameters expected and required for running a process)
- process_parameter_set_value (if a text parameter can only take a on a set of distinct values)
- process_parameter_minmax (if a numerical parameter can only take on a limited range)
- process_parameter_schema_table (the target schema.table for a parameter to be written to the database)
- process_parameter_permission (if the parameter is allowed to be updated or deleted)
- process_parameter_inherit (if a parameter can default to a value copied from an existing database record)
- process_parameter_auto_name (if a parameter's value is auto-generated by concatenating other parameters in the same process)

The handling of processes is fairly complex. If you are in for a deeper understanding please have a look in the JSON process files setting up and inserting the data for defining processes:

```
./setup/zzz/xspatula/setup_db/json_core/process/processes_v10_sql.json
./setup/zzz/xspatula/setup_db/json_core/process/processes_records_v10_sql.json
```

For the full column-level reference of every `process` schema table, and how each maps to a registration option, see the [process schema tables][define_process_tables] reference in Setup processes. For how `processes_records_v10_sql.json` itself bootstraps `add_root_process`/`add_process` and the stratum levels required to use them, see [Bootstrapping process management][manage_process].

## Input files

| File | Purpose |
|---|---|
| `db_xspatula_core_setup.txt` | Pilot file; defines the execution order for all 10 process files below |
| `schema_v10_sql.json` | Creates the three core schemas: `utility`, `community` and `process` |
| `utility_territory_v10_sql.json` | Creates the `utility.territory` table for ISO country codes |
| `community_user_categories_v10_sql.json` | Creates the `community.user_categories` table defining privilege strata |
| `community_user_categories_records_v10_sql.json` | Inserts the predefined user category records into `community.user_categories` |
| `community_organisation_v10_sql.json` | Creates the `community.organisation` table |
| `community_organisation_records_v10_sql.json` | Inserts a default organisation record — edit before running |
| `community_user_v10_sql.json` | Creates the `community.user` table with all user columns including `stratum_code` |
| `community_user_records_v10_sql.json` | Inserts default users including `ini_cat_5_user` and `jane_doe` — edit before running; `password` values must be bcrypt hashes from `setup/hash_password.py`, not plain text |
| `processes_v10_sql.json` | Creates the `process` schema tables for defining framework processes |
| `processes_records_v10_sql.json` | Inserts root process records enabling all other process management |

[vscode]: https://code.visualstudio.com

[postgres]: ../postgres/

[anaconda]: ../anaconda/

[netrc]: ../netrc/

[download_anaconda]: https://www.anaconda.com/download

[install_anaconda]: https://www.anaconda.com/docs/getting-started/anaconda/install/overview

[setup_db_hash_passwords]: ../#hashing-user-passwords

[define_process_tables]: ../../setup_processes/define_process/#process-schema-tables

[manage_process]: ../manage_process/
