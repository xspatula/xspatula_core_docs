---
title:  "Setup Xspatula DB"
layout: single
sidebar:
  nav: "setup_db"
excerpt: "The first step in building up an Xspatula framework is to setup a postgreSQL database. You can also use Xspatula just for setting up a database without using the framework for something else."
permalink: /setup_db/
author_profile: false
date:   2026-03-15 16:13:03 +0200
last_modified_at:   2026-11-08
---

The first step in building up an Xspatula framework is to setup a postgreSQL database. You can also use Xspatula just for setting up a database without using the framework for something else.

## Prerequisits

To build your own postgreSQL database you might need to install the following applications/files:

- [Visual Studion code (VScode)][vscode] (if you want to use the Jupyter notebook interface),
- [postgreSQL][postgres] (unless you have access to a machine with postgres installed),
- [Anaconda][anaconda] or any other application for defining virtual python environments, and/or
- A [.netrc][netrc] file that defines your credentials for setting up new databases (to protect your login credentials).

**Shortcut with a premade database.** If someone gives you a dump of an existing Xspatula database, you do not need to build it. Restore the dump with `setup/dump_restore_db.ipynb` and skip the schema, table and process setup. The same notebook backs up your own database. See [Dump and restore][dump_restore].

## Hidden files and security

**No password ever goes in a scheme file or a seed JSON file.** Scheme files live in the project folder and get committed with it, and a password committed to a public repository is compromised, even after deletion. Every credential lives in `~/.netrc` and is referenced from the scheme file by a `*_netrc_id`. Two sets of hidden files hold credentials:

### .netrc

A [.netrc][netrc] file is a hidden file in your home directory holding one line per credential:

```
machine my_pg_superuser login postgres_superuser password <long-random-password>
```

_machine_ is the id you reference from the scheme file, _login_ and _password_ are the credentials. See [.netrc and credentials][netrc] for the full example block, the rules and the pitfalls.

Also make sure PostgreSQL actually checks passwords: several installs default to `trust` in `pg_hba.conf`. See [Securing authentication][postgres_hba].

### .environment

When a user tries to login to the framework database, a postgreSQL session is opened and the user credentials are read from the database. If the user and password are found, the database is instead opened from an environment (.env) file reflecting the rights of the user logging in. When you setup an Xspatula database, the framework automatically creates .env files for different categories of postgres users (pg_user in postgres jargon). The framework includes some default pg_users, but you most probably want to define your own. All the hidden .env files will by default be saved in the framework python package path (gitignored, chmod 600, but still plaintext):

```
./src/postgres/environment
```

All environment files contain the following entries:

```
DB_HOST=localhost
DB_NAME=my_database
DB_USER=db_user
DB_PASSWORD=db_user_password
```

These credentials are automatically loaded when a user is recognised in the system and then used for accessing the database. The default .env files created by the framework when setting it up include:

- community_admin (for handling users),
- login_evaluation (used for checking all login attempts),
- user_cat_1
- user_cat_2
- user_cat_3
- user_cat_4
- user_cat_5

The different user categories (user_cat_1 to user_cat_5) have varying rights using the database, with 1 have the least and 5 the most.

## Notebook interface

The framework database can be managed through two of the Jupyter notebooks included in the online repository:

- ./setup/setup_db.ipynb
- ./setup/delete_db.ipynb

To learn more, see the documents on [notebooks][notebook] and [VScode][vscode].

### Scheme file

Running Xspatula requires a _scheme_ file that defines the user credentials for accessing the database and can also define default settings for execute, overwrite and delete. The _scheme_ file also links to the library of JSON command files to run. To actually interact with the framework database, the _scheme_ file points to a _job_file_ (project) that links to the _process file(s)_ to run.

The _scheme_ file and the JSON command files can be located anywhere on your system. The default setting is to store the library of files inside the framework itself - referencing it as a relative path in the _scheme_ file. This allows you to initially run the framework without consideration of paths and operating systems while testing it. Once you understand the system you can put the _scheme_ file and the JSON command structure in any other relative or absolute path.

#### Scheme file for database setup

The scheme file for setting up your database defines the postgres superuser that will own the database, the pg_users, and optionally the initial (bootstrap) application user. All three refer to entries in `~/.netrc`; no password appears in the file. The shipped template is `setup/zzz/scheme_template_setup.json`:

```json
{
  "project_path": "./my_project",
  "postgresdb": {
    "host": "localhost",
    "port": 5432,
    "db": "my_database",
    "host_netrc_id": "my_pg_superuser",
    "dump_path": "~/xspatula/db_dump/my_database.dump",
    "db_users": [
      { "user_netrc_id": "community_admin",  "role": "community_admin" },
      { "user_netrc_id": "login_evaluation", "role": "login_evaluation" },
      { "user_netrc_id": "user_cat_1",       "role": "user_cat_1" },
      { "user_netrc_id": "user_cat_2",       "role": "user_cat_2" },
      { "user_netrc_id": "user_cat_3",       "role": "user_cat_3" },
      { "user_netrc_id": "user_cat_4",       "role": "user_cat_4" },
      { "user_netrc_id": "user_cat_5",       "role": "user_cat_5" }
    ]
  },
  "initial_user": {
    "user_netrc_id": "my_initial_user",
    "email": "first.user@example.com",
    "email_alt": "first.user.alt@example.com",
    "first_name": "First",
    "last_name": "User",
    "organisation_id__organisation_name": "my_organisation",
    "territory_id__territory_name": "my_territory",
    "stratum_code": "5"
  },
  "process": [
    {
      "execute": true,
      "verbose": 1,
      "overwrite": false,
      "delete": false
    }
  ]
}
```

- `host_netrc_id`: the postgres superuser that creates and owns the database.
- `db_users[].user_netrc_id`: one pg_user each; the netrc `login` becomes the role name (an explicit `"user_id"` overrides it), and `role` picks the predefined role type. All entries are resolved before anything is created; if one fails, nothing is set up.
- `initial_user`: the first `community.user`, see [Setting up the initial user][initial_user].
- `process.delete` must be `false` to set up a database.

The matching `.netrc` block is on the [.netrc and credentials][netrc] page.

#### Default (human) users

The postgres superuser defined in the _scheme_ file for setting up your new database will become both the owner and a superuser for your new database. The default processes defined for setting up a new database include the definition of ordinary user(s) (not pg_users) in the database. The bootstrap user's row uses the `"{initial_user}"` placeholder and gets its identity and password from `initial_user` and `.netrc` (see below). The other example users (e.g. _jane_doe_) ship with fixed bcrypt hashes; remove them or replace their hashes before using the database for real. The seed file is

```
./setup/zzz/xspatula/setup_db/json/community/user_records_v10_sql.json.
```

For ordinary login (not database setup) the use scheme points `user_project.user_netrc_id` at the same `.netrc` entry as `initial_user`, so there is nothing extra to set up.

For details on including default users as part of the database setup, see the document on [defining schemas and tables][schemas_tables].

#### Passwords for application users

The `password` column in `community.user` stores a bcrypt hash, never plain text. You don't hash anything by hand for the first user: add an `initial_user` block to the setup scheme (shown above) and put the placeholder `"{initial_user}"` as the bootstrap row's `password` in `user_records_v10_sql.json`. At setup the framework reads the password from `.netrc`, hashes it in memory and fills the row. See [Setting up the initial user][initial_user].

`setup/hash_password.py` is only needed to reset a password on a **live** database (`UPDATE community.user SET password = '<hash>' WHERE user_name = '...';`) or to seed additional, non-bootstrap users.

#### Scheme file for database delete

The framework package allows deleting either parts of, or the whole database with a single process. The delete scheme needs a project_path, the postgres database cluster and your superuser netrc id. You must also set the process object delete to true:

```
{
  "project_path": "./xspatula", # Absolute or relative path to the root folder of your project for deleting a database
  "postgresdb": {
    "host": "localhost", # The host of your database
    "port": 5432, # The port that your database is attached to
    "db": "name_of_db_to_delete", # The name of the postgreSQL database to delete
    "host_netrc_id": "my_pg_superuser" # .netrc entry for the postgres superuser
  },
  "process": [ # Default settings for all processes called from this scheme file
    {
      "execute": true,
      "verbose": 1,
      "overwrite": false,
      "delete": true # Must be set to true to delete a database
    }
  ]
}
```

Deleting a database never needs pg_user passwords. Roles belong to the server, are shared by every database on it, and are not dropped with a database, so `db_users` is not needed in the delete scheme.

### Job file (project file)

The _scheme_ file object <span class='job_file'>project_path</span> defines the path to the root directory of your actual project to run. A project is built up using a hierarchy of command files, with a JSON _job_file_ at the top. In the _job_file_ there are three options for linking to the _process_files_ you want to run:

- directly to a single JSON _process_file_,
- defining an array of JSON _process_files_, or
- via a simple text _pilot_file_ listing the _process_files_ to run.

The default setting in the Xspatula GitHub online repository is linking to a _pilot_file_.

```
{
  "process": {
    "job_folder": "setup_db",
    "process_sub_folder": "json",
    "pilot_file": "db_setup.txt"
    ]
  }
}
```

See the document on [job files][job_file] for details.

### Pilot file

A pilot file is a simple text file, where empty lines and lines starting with a hash-tag (#) are ignored when executing. The advantage with a pilot file is that you can write notes and turn individual process files on and off by using a hash-tag. The example below shows the process files that must be run to setup any Xspatula dataset using the framework default settings.

```
#################################################
##### DEFINE DB SCHEMAS AND TABELS #####
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

To learn more, see the document on [pilot file][pilot_file].

### Process file

All JSON _process_files_ are built from the same objects and arrays. The example below shows the process for creating schemas, a process that is as simple as it gets in the framework. The example also shows how it is possible to stack any number of processes in a single _process_file_.

```
{
  "process": [
    {
      "process_id": "create_schema",
      "parameters": {
        "schema": "observation"
      }
    },
    ...
    ...
    {
      "process_id": "create_schema",
      "parameters": {
        "schema": "community"
      }
    }
  ]
}
```

To learn more, see the document on [process file][process_file].

#### Define the community accessing the database

The default setting is to create a schema called community, including the following tables:

- user_categories
- organisation
- user
- user_media
- user_activity

The table user and the columns _user_name_, _password_, and _stratum_code_ are searched each time a user is trying to login. The _stratum_code_ defines which .env file a user is entering the database with and thus the privileges that that user has in the database. If you change the name of either the schema, the table or these columns, you need to update the python source code accordingly.

For details on defining tables and columns, see the document on [Defining schemas and tables](../setup_db/schemas_tables/#create-table).

#### Community organisations and users

The _process_ files `"community_organisation_records_v10_sql.json"` and `"community_user_records_v10_sql.json"` directly inserts records in the community schema tables organisation and user. The users inserted will get immediate access to the created database.

Note that this is a different password to the ones in the postgres pg_user _scheme_ file shown above: the pg_user password authenticates the *database connection* itself, while `community.user.password` authenticates an individual *application login* and is stored as a bcrypt hash, computed automatically for the bootstrap user as described in [passwords for application users](#passwords-for-application-users). This setup-time route is meant for your initial, default users. Once the database is running, organisations and users can also be added by non-technical staff filling in an Excel sheet — see [Setup Community][setup_community].

The figure below illustrates the content and connections for the schema community after setting up the Xspatula framework.

[![Schema community tables and connections]({{ "/assets/media/xspatula_db_community_schema.png" | relative_url }})]({{ "/assets/media/xspatula_db_community_schema.png" | relative_url }})

#### Processes

The process_file processes_v10_sql.json defines the table structure for definition of processes that the framework should include. It defines the following tables:

- root_process (for agglomerating a family of sub_processes),
- process (actual processes that the framework can run),
- process_parameter (definition of all parameters required by a specific process),
- process_parameter_set_value (if a text parameter can take on only a predefined set of values),
- process_parameter_minmax (if a numerical parameter is restricted to a range)
- process_parameter_schema_table (the target table for any particular parameter),
- process_parameter_permission (if a set parameter is allowed to be updated or deleted),
- process_parameter_inherit (if a parameter can default to a value copied from an existing database record), and
- process_parameter_auto_name (if a parameter's value is auto-generated by concatenating other parameters in the same process).

For the full column-level reference of every `process` schema table, see the [process schema tables][define_process_tables] reference in Setup processes.

The figure below illustrates the content and connections for the schema process after setting up the Xspatula framework.

[![Schema process tables and connections]({{ "/assets/media/xspatula_db_process_schema.png" | relative_url }})]({{ "/assets/media/xspatula_db_process_schema.png" | relative_url }})

#### Initial process records

Running processes_records_v10_sql.json inserts the root process and processes required for managing all other processes in the framework.

#### Turning on auditing

`setup_db.ipynb` has a second, optional cell after the main "Setup database" cell you've just walked through: **Apply audit triggers**. It is worth understanding what each cell does before you run either:

- **The "Setup database" cell**, as a side effect of running it, always scans every table's definition for an inline `audit` key and (re)writes a set of audit-trigger configuration files to disk — one per schema — plus a matching pilot file. This is pure file bookkeeping: it never touches the database and creates no audit objects, so running it changes nothing about whether auditing is actually active.
- **The "Apply audit triggers" cell** reads those freshly-written files and applies them to the database: it creates an `audit` schema with a `logged_actions` table, a shared trigger function, and one trigger per audited table. This is the step that actually switches auditing on. It's optional — skip it and your database simply has no audit triggers — and safe to (re-)run any time, including repeatedly, since it always applies the complete current configuration rather than only what changed.

A table opts into auditing by carrying an `audit` key next to its `parameters` in its `create_table` process block:

```json
"audit": {
  "INSERT": true,
  "UPDATE": true,
  "DELETE": true
}
```

No `audit` key (or one with every value `false`) means that table simply isn't audited — there's no other configuration step required. The cell looks like this in the notebook:

```
## Apply audit triggers (optional, separate step)

Every table's audit setting is declared inline on its own create_table process
block via an "audit" key. Running the cell above already assembled the
per-schema audit trigger config files and a matching pilot file
(db_<project>_audit.txt) from those keys, purely as files - it did not touch
the database.

Run the cell below any time after the cell above - immediately, later, or
repeatedly - to actually apply that config to the database.
```
```python
audit_job_file = 'job_setup_audit.json'
Initiate_audit(notebook_path, scheme_file, audit_job_file)
```

Full instructions for using the audit system once it's switched on — what gets logged, how to query it, and which tables are worth auditing on `INSERT` versus only `UPDATE`/`DELETE` — are in the [Auditing][auditing] collection.

### Project file hierarchy

The directory structure for the project defined in the scheme_file and the job_file above looks like this:

```
.
├── scheme_setup.json # scheme_file
└── xspatula # project root directory (can be put anywhere - path given in scheme_file)
    ├── job_setup_db.json # job file (should be in a directory under the project root - path given in scheme_file)
    ├── setup_db # Should be a directory under the project root - path given in object job_folder in the job_file
        ├── db_setup.txt # pilote_file - name given as object pilot_file in the job_file
        └── json # directory with process_files - path given as object process_sub_folder in job_file
            ├── schema_v10_sql.json # process_file listed in pilot_file
            ├── community_organisation_v10_sql.json
            ├── community_organisation_records_v10_sql.json
            ├── community_user_v10_sql.json
            ├── community_user_records_v10_sql.json
            ├── processes_v10_sql.json
            ├── processes_records_v10_sql.json
            ...
```

## Input files

| File | Purpose |
|---|---|
| `scheme_local_setup.json` | Scheme file for database setup; defines postgres superuser, pg_users and initial_user (as netrc ids) and default process settings |
| `job_setup_db.json` | Job file for database setup; points to the pilot file and process file directory |
| `db_setup.txt` | Pilot file; lists the 10 JSON process files to execute in order for a full database setup |

[vscode]: ../framework/vscode/

[postgres]: ./postgres/

[anaconda]: ./anaconda/

[netrc]: ./netrc/

[postgres_hba]: ./postgres/#securing-authentication-pg_hbaconf

[initial_user]: {{ "/building/initial_user/" | relative_url }}

[notebook]: ../framework/notebook/

[scheme_file]: ../framework/scheme_file/

[job_file]: ../framework/job_file/

[pilot_file]: ../framework/pilot_file/

[process_file]: ../framework/process_file/

[schemas_tables]: ./schemas_tables/

[dump_restore]: ./dump_restore/

[auditing]: {{ "/auditing/" | relative_url }}

[setup_community]: {{ "/setup_community/" | relative_url }}

[define_process_tables]: ../setup_processes/define_process/#process-schema-tables
