---
title:  "Xspatula scheme file"
layout: single
sidebar:
  nav: "framework"
excerpt: "In the Xspatula framework a scheme file defines: 1. the project you want to run, 2. your login credentials for accessing the database, and 3. default settings used throughout the processing of all processes linked to the project. Scheme files are written in JSON format."
permalink: /framework/scheme_file/
author_profile: false
date:   2026-03-20 16:13:03 +0200
last_modified_at:   2026-03-29 21:24:00 +0200
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

The setting of the database credentials differs between when you setup or delete the database compared to all other processes. Setting up or deleting the database requires that you give the login credentials for the postgres server superuser with rights to create new databases. You must also give the host (server), port and name of the database to create when setting up or deleting a database. When setting up the database, the host, port and database name are saved in .env files (see below). Thus you do not need to state these when setting up or deleting the database.

Scheme file object postgresdb for setting up a new Xspatula framework database:
```
{
  "project_path": "./your_framework_directory",
  "postgresdb": {
    "host": "localhost",
    "port": 5432,
    "db": "xspatula",
    "user_name": "postgres_superuser",
    "password": "password_for_superuser"
  }
  ...
  ...
}
```

If you use a [.netrc][netrc] file, this instead becomes:

```
{
  "project_path": "./your_framework_directory",
  "postgresdb": {
    "host": "localhost",
    "port": 5432,
    "db": "xspatula",
    "host_netrc_id": "my_postgres_cluster"
  }
  ...
  ...
}
```

with a corresponding machine in the .netrc file:

```
machine my_postgres_cluster login postgres_superuser password password_for_superuser
```

The superuser used for setting up the new database will also become a superuser in the new database.

### Define pg_users

You can also add additional postgres users (pg_user in postgres jargon, which is not a human person but a key that a human user get access to if logging in with the correct credentials to the database you are creating directly in the scheme file. There are 8 predefined groups of pg_users (also called roles in the postgres jargon):

- community_admin [for administrating users],
- login_evaluation (user_cat_6) [for checking login credentials],
- user_cat_0 [for users that can only SELECT limited tables],
- user_cat_1,
- user_cat_2,
- user_cat_3,
- user_cat_4,
- user_cat_5 [for users who can SELECT, INSERT, UPDATE AND DELETE in most tables]

The privileges for the user categories listed above are granted and revoked from the JSON files `./setup/src_setup/lib_setup/roles_grants.json` and
`./setup/src_setup/lib_setup/revoke_privileges.json`.


You can change the user categories, their names and privileges to fit your own needs. But changing them will have downstream effects in other settings that you then must also accommodate. **Note** that only the superuser that owns the database can change the pg_users and their privileges.

The scheme file example below shows how to set up one pg_user per predefined pg_user group.
```
{
  "project_path": "./your_framework_directory",
  "postgresdb": {
    host": "localhost",
    "port": 5432,
    "db": "xspatula",
    "host_netrc_id": "my_postgres_cluster"
    "db_users": [
      {
        "user_id": "your_community_admin",
        "password": "first-difficult-password",
        "role": "community_admin"
      },
      {
        "user_id": "your_login_evaluation",
        "password": "second-difficult-password",
        "role": "login_evaluation"
      },
      {
        "user_id": "your_user_cat_0",
        "password": "third-difficult-password",
        "role": "user_cat_1"
      },
      {
        "user_id": "your_user_cat_1",
        "password": "fourth-difficult-password",
        "role": "user_cat_1"
      },
      {
        "user_id": "your_user_cat_2",
        "password": "fifth-difficult-password",
        "role": "user_cat_2"
      },
      {
        "user_id": "your_user_cat_3",
        "password": "sixth-difficult-password",
        "role": "user_cat_3"
      },
      {
        "user_id": "your_user_cat_4",
        "password": "seventh-difficult-password",
        "role": "user_cat_4"
      },
      {
        "user_id": "your_user_cat_5",
        "password": "eigth-difficult-password",
        "role": "user_cat_5"
      }
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

When an ordinary (human) user logs into the database, the user only has to give the user_name and password. All other information required (host, port, database name) are retrieved from the .env file. Thus the scheme file for working with projects in the database, once it is set up, only requires a project path and login credentials:

```
{
  "project_path": "~/your_framework_directory",
  "user_project": {
    "user_name": "database_user",
    "password": "database_user_password"
  }
}
```

If the user instead chooses to store the credentials in a .netrc file, the scheme file becomes even smaller:

```
{
  "project_path": "~/your_framework_directory",
  "user_project": {
    "user_netrc_id": "user_netrc_code"
  }
}
```

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
    "user_netrc_id": "user_netrc_code"
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
| `scheme_xspatula_local_setup.json` | Scheme file for creating a new database; defines postgres cluster credentials, the pg_users and default process settings |
| `scheme_xspatula_local_use.json` | Scheme file for using an existing database; defines a single project user — no superuser credentials required |

[notebook]: ../notebook/

[netrc]: ../../setup_db/netrc/
