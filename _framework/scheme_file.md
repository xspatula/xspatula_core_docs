---
title:  "Xspatula scheme file"
layout: single
sidebar:
  nav: "framework"
excerpt: "In the Xspatula framework a scheme file defines: 1. the project you want to run, 2. your login credentials for accessing the database, and 3. default settings used throughout the processing of all processes linked to the project. Scheme files are written in JSON format."
permalink: /framework/scheme_file/
author_profile: false
date:   2026-03-20 16:13:03 +0200
last_modified_at:   2026-11-08
---

In the Xspatula framework a scheme file defines:
1. the project to run,
2. login credentials for accessing the database, and
3. default settings used throughout the processing of all processes linked to the project.

Scheme files are written in JSON format.

A good practice is to define one scheme file per project, where the first project should be setting up the database, and the second to setup the processes that the framework should be able to run. If you want to delete parts of, or the whole, database, that should also be a separate project.

The scheme file, and thus project, to run is set directly in the Jupyter [notebook][notebook] or in the command line if you run the framework with a command line interface (CLI).

## Scheme file structure

Schemes files are written as JSON objects, with objects defining the project directory, default process settings and user login credentials for the postgreSQL database.

### project path

All scheme files must have an object that links to a project directory. The object _project_path_ can be either a relative or an absolute path, where the start of the relative path is the root of the notebook itself.

Example of a project with a relative path pointing to a sub directory to the folder where the notebook itself is located:

```
"project_path": "./your_framework_directory",
```

You can also put the project in your home directory and use an operating system independent syntax for designating the path:

```
"project_path": "~/your_framework_directory",
```

For the last alternative, to state an absolute path, you need to adhere to the path definition syntax of your operating system (example is for MacOS):

```
"project_path": "/Users/my_user/your_framework_directory",
```

### Default process settings

A scheme file can point to any number of processes and instead of setting general instructions in each process you can give these general settings in the scheme file itself. If the same argument is given in both the scheme file and in an individual process, the individual process setting takes precedence. The general arguments that can be set are illustrated in the example below:

```
  "process": [
    {
      "execute": true,
      "verbose": 1,
      "overwrite": false,
      "delete": false
    }
  ]
```

#### Verbosity

The framework is built to have 4 levels of verbosity:
- 0: only critical errors reported,
- 1: also successful steps are reported,
- 2: details on process settings are additionally reported, and
- 3: full verbosity.

#### Overwrite and delete

Overwrite and delete are risky to set, the framework will replace existing records and files with new versions if you set overwrite to true, and delete existing records and files if you set delete to true. If both overwrite and delete are to true, overwrite has precedence.

### Setup database credentials

**No password goes in a scheme file.** Every credential lives in `~/.netrc` and the scheme file references it by a `*_netrc_id`. See [.netrc and credentials][netrc].

The setting of the database credentials differs between when you setup or delete the database compared to all other processes. Setting up or deleting the database requires the postgres server superuser with rights to create new databases, plus the host (server), port and name of the database. When setting up the database, the host, port and database name are saved in .env files (see below), so later use schemes don't need them.

Scheme file object postgresdb for setting up or deleting an Xspatula framework database:

```json
{
  "project_path": "./my_project",
  "postgresdb": {
    "host": "localhost",
    "port": 5432,
    "db": "my_database",
    "host_netrc_id": "my_pg_superuser"
  }
  ...
}
```

with a corresponding machine in the .netrc file:

```
machine my_pg_superuser   login postgres_superuser password <long-random-1>
```

The superuser used for setting up the new database will also become a superuser in the new database.

**Legacy:** `"user_name"` + `"password"` in `postgresdb` instead of `host_netrc_id` still works, but prints `⚠️ plaintext password in scheme file`. Don't use it in any file you commit.

### Define pg_users

You also define the postgres users (pg_user in postgres jargon: not a human person but a key that a human user gets access to when logging in with the correct credentials) for the database you are creating directly in the scheme file. There are 7 predefined pg_users (also called roles in the postgres jargon):

- community_admin [for administrating users],
- login_evaluation [for checking login credentials],
- user_cat_1,
- user_cat_2,
- user_cat_3,
- user_cat_4,
- user_cat_5 [for users who can SELECT, INSERT, UPDATE AND DELETE in most tables]

The privileges and role attributes for the user categories listed above are granted and revoked from the JSON files `./setup/src_setup/lib_setup/roles_grants.json` and
`./setup/src_setup/lib_setup/revoke_privileges.json`.

You can change the user categories, their names and privileges to fit your own needs. But changing them will have downstream effects in other settings that you then must also accommodate. **Note** that only the superuser that owns the database can change the pg_users and their privileges.

The scheme file example below (from `setup/zzz/scheme_template_setup.json`) sets up one pg_user per predefined role. Each `user_netrc_id` points to a `.netrc` entry; its `login` becomes the role name (an explicit `"user_id"` overrides it):

```json
{
  "project_path": "./my_project",
  "postgresdb": {
    "host": "localhost",
    "port": 5432,
    "db": "my_database",
    "host_netrc_id": "my_pg_superuser",
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

Every setup run re-applies each role's password (from `.netrc`) and attributes (from `roles_grants.json`). To change a pg_user password, edit `.netrc` and re-run setup. See [pg_users][pg_users].

The legacy form `{"user_id": ..., "password": ..., "role": ...}` still works but prints a plaintext-password warning.

### initial_user

The optional top-level `initial_user` block (a sibling of `postgresdb`) creates the first `community.user` at setup. The bootstrap row in the seed file `user_records_v10_sql.json` has `"password": "{initial_user}"`; at setup the framework reads the password from `.netrc`, bcrypt-hashes it in memory and fills the row:

```json
"initial_user": {
  "user_netrc_id": "my_initial_user",
  "email": "first.user@example.com",
  "first_name": "First",
  "last_name": "User",
  "organisation_id__organisation_name": "my_organisation",
  "territory_id__territory_name": "my_territory",
  "stratum_code": "5"
}
```

- The netrc `login` becomes `user_name` (or `email` if it contains `@`).
- Every other key overrides the matching column of the seed row.
- Without `initial_user` in the scheme, the placeholder row is skipped with a warning.
- Use one placeholder row per seed file.

The same netrc entry is used for logging in afterwards (see [Login credentials](#login-credentials)). Full walkthrough: [Setting up the initial user][initial_user].

#### .env files

If you include pg_users (JSON object db_user) in the scheme for setting up your database, corresponding environment (.env) files will be automatically created in the framework. A .env file contain the credentials for a pg_user and is read by the script first when someone is logging in to open the database as a _login_evaluation_ pg_user. If the user trying to login is registered as (an ordinary human) user in the database, and the password (for that particular human user) is correct, the database checks which pg_user category that particular person has access to, and reads the postgreSQL credentials from the corresponding .env file. For example, if a user with rights at level 3 (levels are called _startum_code_ in the predefined database setup), the script will read the pg_user credentials for a category 3 access to the database and then open the database for the user with corresponding rights.

All the hidden .env files will by default be saved under the framework python package path
`./src/postgres/environment`.

Environment files contain the following entries:

```
DB_HOST=localhost
DB_NAME=my_database
DB_USER=db_user
DB_PASSWORD=db_user_password
```

### Login credentials

When an ordinary (human) user logs into the database, all information on host, port and database name is retrieved from the .env file. The scheme file for working with projects in the database, once it is set up, only requires a project path and a `.netrc` reference (template: `setup/zzz/scheme_template_use.json`):

```json
{
  "project_path": "./my_project",
  "user_project": {
    "user_netrc_id": "my_initial_user"
  }
}
```

`user_netrc_id` alone is enough: `user_name` is filled from the logged-in `community.user` record. The legacy `"user_name"` + `"password"` pair in `user_project` still works, but never commit it.

In the example above no default settings are included, the system will then set the following default values:

```
"execute": true,
"verbose": 1,
"overwrite": false,
"delete": false
```

To define explicit default settings, the scheme file could for example look like this:

```
{
  "project_path": "~/your_framework_directory",
  "user_project": {
    "user_netrc_id": "my_initial_user"
  },
  "process": [
    {
      "execute": true,
      "verbose": 2,
      "overwrite": false,
      "delete": false
    }
  ]
}
```

As noted above, if any of the default values appear in an individual process JSON command that setting takes precedence over the default settings in the scheme file.

## Input files

| File | Purpose |
|---|---|
| `setup/zzz/scheme_template_setup.json` | Setup scheme template; superuser, 7 pg_users and initial_user, all via netrc ids |
| `setup/zzz/scheme_template_use.json` | Use scheme template; `user_project.user_netrc_id` only |
| `~/.netrc` | All credentials; chmod 600 |
| `setup/git_hooks/pre-commit` | Blocks commits of plaintext passwords in JSON; install manually |

[notebook]: ../notebook/

[netrc]: ../../setup_db/netrc/

[pg_users]: ../../building/pg_users/

[initial_user]: ../../building/initial_user/
