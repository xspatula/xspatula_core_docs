---
title:  "Setup Xspatula DB"
layout: single
sidebar:
  nav: "setup_db"
excerpt: "The first step in building up an Xspatula framework is to setup a postgreSQL database. You can also use Xspatula just for setting up a database without using the framework for something else."
permalink: /setup_db/
author_profile: false
date:   2026-03-15 16:13:03 +0200
last_modified_at:   2026-03-29 20:45:30 +0200
---

The first step in building up an Xspatula framework is to setup a postgreSQL database. You can also use Xspatula just for setting up a database without using the framework for something else.

## Prerequisits

To build your own postgreSQL database you might need to install the following applications/files:

- [Visual Studion code (VScode)][vscode] (if you want to use the Jupyter notebook interface),
- [postgreSQL][postgres] (unless you have access to a machine with postgres installed),
- [Anaconda][anaconda] or any other application for defining virtual python environments, and/or
- A [.netrc][netrc] file that defines your credentials for setting up new databases (to protect your login credentials).

## Hidden files and security

To create secure login credentials for the postgreSQL superuser that you must have for building the framework database, and for other users accessing the database, two sets of hidden files holding the credentials can be used.

### .netrc

A [.netrc][netrc] file is a hidden file in your home directory where you can store login credential to postgreSQL or any other server resource. You do not need to use a .netrc file to setup or run the Xspatula framework, you can also give your login and password in plain text in a _scheme_ file as explained below. If you want to use a .netrc file, create a file named .netrc in the home directory of your own machine and enter the credentials by stating _machine_, _login_ and _password_:

```
machine postgresql login <superuser> password <passphrase>
```

Where _machine_ is the code word you send to .netrc for the resource you want to access, and _login_ and _password_ are the credentials on the resource you are trying to enter.

### .environment

When a user tries to login to the framework database, a postgreSQL session is opened and the user credentials are read from the database. If the user and password are found, the database is instead opened from an environment (.env) file reflecting the rights of the user logging in. When you setup an Xspatula database, the framework automatically creates .env files for different categories of postgres users (pg_user in postgres jargon). The framework includes some default pg_users, but you most probably want to define your own. All the hidden .env files will by default be saved in the framework python package path:

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
- user_cat_0
- user_cat_1
- user_cat_2
- user_cat_3
- user_cat_4
- user_cat_5

The different user categories (user_cat_0 to user_cat_5) have varying rights using the database, with 0 have the least and 5 the most.

## Notebook interface

The framework database can be managed through two of the Jupyter notebooks included in the online repository:

- ./setup/setup_db.ipynb
- ./setup/delete_db.ipynb

To learn more, see the documents on [notebooks][notebook] and [VScode][vscode].

### Scheme file

Running Xspatula requires a _scheme_ file that defines the user credentials for accessing the database and can also define default settings for execute, overwrite and delete. The _scheme_ file also links to the library of JSON command files to run. To actually interact with the framework database, the _scheme_ file points to a _job_file_ (project) that links to the _process file(s)_ to run.

The _scheme_ file and the JSON command files can be located anywhere on your system. The default setting is to store the library of files inside the framework itself - referencing it as a relative path in the _scheme_ file. This allows you to initially run the framework without consideration of paths and operating systems while testing it. Once you understand the system you can put the _scheme_ file and the JSON command structure in any other relative or absolute path.

#### Scheme file for database setup

The scheme file for setting up your database must define both the postgres superuser that will own the database and at least one user that is then used for setting up processes and adding other users. Explained in more detail below.

In the example of a _scheme_ file for setting up a database below I have added some comments indicated by a hashtag (#). These comments are not allowed in a production version of a _scheme_ file.

```
{
  "project_path": "./xspatula", # Absolute or relative path to the root folder of your project, e.g. for setting up a database
  "postgresdb": { # Defines the postgres database cluster and your superuser credentials allowing you to create a new database
    "host": "localhost", # The host of your database cluster
    "port": 5432, # The port that your database cluster is attached to
    "db": "name_of_db_to_create", # The name of the postgreSQL database to create
    "host_netrc_id": "my_host_db_netrc_access_code", # The login credentials for your overaraching postgres database
    "user_name": "your_superuser_for_postgres", # Only required if host_netrc_id not stated
    "password": "your_superuser_password_for_postgres", # Only required if host_netrc_id not stated
    "db_users": [ # array of pg_users to add as users in the new postgres database and their credentials
      {
        "user_id": "community_admin", # pg_user for community administration
        "password": "guessing-rubble-garden-opera", # password for pg_user community_admin
        "role": "community_admin" # predefined (hardcoded) type of pg_user
      },
      {
        "user_id": "login_evaluation", # pg_user for checking any login attempt to the database (very restricted)
        "password": "hippodrome-bicycle-concert-shuttle", # password for pg_user login_evaluation
        "role": "login_evaluation" # predefined (hardcoded) type of pg_user
      },
      {
        "user_id": "user_cat_0", # most restricted database pg_user
        "password": "tablecloth-summerleaf-riverbasin-vacuumcleaner", # password for pg_user user_cat_0
        "role": "user_cat_1" # predefined (hardcoded) type of pg_user
      },
      ...
      ...
      {
        "user_id": "user_cat_5", # most permissive database pg_user
        "password": "fireplace-olympicgames-grassroot-luminescence", # password for pg_user user_cat_5
        "role": "user_cat_5" # predefined (hardcoded) type of pg_user
      }
    ]
  },
  "process": [ # Default settings for all processes called from this scheme file
    {
      "execute": true,
      "verbose": 1,
      "overwrite": false,
      "delete": false # Must be set to false to setup a database
    }
  ]
}
```

#### Default (human) users

The postgres superuser defined in the _scheme_ file for setting up your new database will become both the owner and a superuser for your new database. The default processes defined for setting up a new database include the definition of ordinary user(s) (not pg_users) in the database. The default user (Jane Doe) in the online repository has the user name _jane_doe_ and the password is _hello-xspatula_. It is recommended that you change the names and passwords of the default users in the file

```
./setup/zzz/xspatula/setup_db/json_core/community/user_records_v10_sql.json.
```

You must then also change the login credentials in the subsequent _scheme_ files for ordinary login (not database setup). You can also always login with the superuser that you used for setting up the database.

For details on including default users as part of the database setup, see the document on [defining schemas and tables][schemas_tables].

#### Hashing user passwords

The `password` column in `community.user` does not store plain text. It stores a bcrypt hash, and the framework verifies a login attempt by hashing the entered password and comparing it against that stored hash — it never compares plain text directly. This means the `"hello-xspatula"` shown above for Jane Doe is the password you'd actually type in at login, not what belongs in the JSON file: the JSON must hold the hash produced by a small command-line tool the framework ships with, `setup/hash_password.py`.

To generate a hash, open a terminal, `cd` into your local copy of the framework (the directory containing the `setup/` folder), activate your Anaconda environment if you're using one, and run:

```
python3 setup/hash_password.py
```

You'll be prompted twice, with hidden input:

```
Password to hash:
Repeat password:
```

The tool prints a bcrypt hash that looks like `$2b$12$KIXQ7z3n...` (yours will differ). Copy that whole string — it's what you paste into the `password` value in `community_user_records_v10_sql.json`, not the password itself.

You can also pass the password directly as an argument:

```
python3 setup/hash_password.py 'some-password'
```

This is quicker but leaves the plain-text password visible in your shell history and process list, so prefer the no-argument, prompted form on any machine you share with others.

Do this once per default user before running `setup_db.ipynb`, and again any time you want to reset a user's password by hand-editing the database (`UPDATE community.user SET password = '<hash>' WHERE user_name = '...';`).

#### Scheme file for database delete

The framework package allows deleting either parts of, or the whole database with a single process. To do that the _scheme_ file must still have a project_path, define the postgres database cluster and your superuser credentials allowing you to delete the database. You must also set the process object delete to true:

```
{
  "project_path": "./xspatula", # Absolute or relative path to the root folder of your project for deleting a database
  "postgresdb": {
    "host": "localhost", # The host of your database
    "port": 5432, # The port that your database is attached to
    "db": "name_of_db_to_delete", # The name of the postgreSQL database to delete
    "user_name": "your_superuser_for_postgres", # Only required if host_netrc_id not stated
    "password": "your_superuser_password_for_posgres", # Only required if host_netrc_id not stated
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
    "process_sub_folder": "json_core",
    "pilot_file": "db_xspatula_core_setup.txt"
    ]
  }
}
```

See the document on [job files][job_file] for details.

### Pilot file

A pilot file is a simple text file, where empty lines and lines starting with a hash-tag (#) are ignored when executing. The advantage with a pilot file is that you can write notes and turn individual process files on and off by using a hash-tag. The example below shows the process files that must be run to setup any Xspatula dataset using the framework default settings.

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

Note that this is a different password to the ones in the postgres pg_user _scheme_ file shown above: the pg_user password authenticates the *database connection* itself, while `community.user.password` authenticates an individual *application login* and must be pre-hashed as described in [hashing user passwords](#hashing-user-passwords). This setup-time route is meant for your initial, default users. Once the database is running, organisations and users can also be added by non-technical staff filling in an Excel sheet — see [Setup Community][setup_community].

The figure below illustrates the content and connections for the schema community after setting up the Xspatula framework.

<figure>
  <img src="{{ '/assets/media/xspatula_db_community_schema.png' | relative_url }}" alt="Schema community tables and connections">
</figure>

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

<figure>
  <img src="{{ '/assets/media/xspatula_db_process_schema.png' | relative_url }}" alt="Schema process tables and connections">
</figure>

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
        ├── db_xspatula_core_setup.txt # pilote_file - name given as object pilot_file in the job_file
        └── json_core # directory with process_files - path given as object process_sub_folder in job_file
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
| `scheme_xspatula_local_setup.json` | Scheme file for database setup; defines postgres superuser credentials, pg_users and default process settings |
| `job_setup_db.json` | Job file for database setup; points to the pilot file and process file directory |
| `db_xspatula_core_setup.txt` | Pilot file; lists the 10 JSON process files to execute in order for a full database setup |

[vscode]: ../framework/vscode/

[postgres]: ./postgres/

[anaconda]: ./anaconda/

[netrc]: ./netrc/

[notebook]: ../framework/notebook/

[scheme_file]: ../framework/scheme_file/

[job_file]: ../framework/job_file/

[pilot_file]: ../framework/pilot_file/

[process_file]: ../framework/process_file/

[schemas_tables]: ./schemas_tables/

[auditing]: /auditing/

[setup_community]: /setup_community/

[define_process_tables]: ../setup_processes/define_process/#process-schema-tables
