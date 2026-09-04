---
title:  "Setup AI4SH DB"
layout: single
sidebar:
  nav: "ai4sh"
excerpt: "Setting up a database like the AI4SoilHealth database in the framework is just an extension of the general principles for setting up a database introduced in the framework introduction section. What is additionally required is the definition of schemas and tables that  and their connections."
permalink: /ai4sh/setup_ai4sh_db
author_profile: false
date:   2026-03-22 06:13:03 +0200
last_modified_at:   2026-03-22 06:13:03 +0200
---

Setting a database like the AI4SoilHealth database in the framework is just an extension of the general principles for setting up a database introduced in the [framework introduction section][framework]. What is additionally required is the definition of schemas and tables and their connections.

## Prerequisits

To create your own database using the framework, you must have completed the installations of:

- [Visual Studion code (VScode)][vscode] (if you want to use the Jupyter notebook interface),
- [postgreSQL][postgres] (unless you have access to a server with postgres installed), and/or
- [Anaconda][anaconda] or any other application for defining virtual python environments,

If you want make life easier and save your server logins in one place, you can also setup a a hidden [.netrc][netrc] solution.

## Setup AI4SH database

To setup the AI4SH database the following steps are required:

1. clone or download the AI4SH database project from GitHub,
2. edit the path to your project,
3. edit the superuser credentials to match your postgreSQL installation,
4. define the pg_users you want to include in your AI4SH database,
5. edit the path in the Jupyter notebook setup_db, and
6. run the notebook setup_db.

### Clone or download the AI4SH database project

The project for setting up the AI4SH database is available on [GitHub][github] as a private repository. To access the project you need to send a request to join that repo and then click [here][github_framework_project_ai4sh_database]. [TO BE COMPLETED]

To use the project without having to bother too much about paths, clone the content of the repo to your users home directory and you should get a new directory with the (OS independent) path:

```
~/framework_project_ai4sh_database/
```

If you opt for storing the project under some other path, you need to adjust the paths in the framework notebook to the syntax of your operating system. The instructions in this document will illustrate how to setup the database if you put it in your home directory.

## Edit the file scheme_ai4sh_setup_db.json

The scheme file for setting up the AI4SH database, <span class='file'>scheme_ai4sh_setup_db.json</span> that comes with the project has to be edited before you can run the database setup. The scheme file itself is in the root folder of the cloned or dowloaded repository. Locate the scheme file and then edit:

- the path to the project (only if you changed the project internal hierarchical folder system),
- host, port, db and postgreSQL super user credentials, and
- pg_users to include with the setup.

### Edit the path to the project

The project is by default located directly under the folder where the [scheme][scheme_file] file itself resides and is defined using a relative path:

```
"project_path": "./ai4sh_db"
```

If you split the project hierarchical folder arrangement by putting your scheme file and the project in different locations, you have to change the project_path in the scheme file. If not you can just leave the default project_path.

### Edit the postgreSQL super user credentials

You must have access to a postgreSQL cluster where you are superuser with rights to create new databases. In the [scheme][scheme_file] file <span class='file'>scheme_ai4sh_setup_db.json</span>, change the definitions for these children of the object postgresdb to your settings:

```
"postgresdb": {
    "host": "localhost",
    "port": 5432,
    "db": "ai4sh",
    "user_name": "your_postgres_superuser",
    "password": "your_postgres_superuser_password"
```

You can also put the login credentials in a [.netrc][netrc] file and replace the user_name and password objects with an object for reading your credentials using netrc:

```
"postgresdb": {
    "host": "localhost",
    "port": 5432,
    "db": "ai4sh",
    "host_netrc_id": "your_netrc_machine_code"
```   

### Edit the pg_users to include with setup

In the scheme file you can also add pg_users to include in the database when setting it up. There are 8 default pg_users included in the project for setting up the AI4SH database defined in the prepared scheme file:

```
    "db_users": [
      {
        "user_id": "community_admin",
        "password": "guessing-rubble-garden-opera",
        "role": "community_admin"
      },
      {
        "user_id": "login_evaluation",
        "password": "hippodrome-bicycle-concert-shuttle",
        "role": "login_evaluation"
      },
      {
        "user_id": "user_cat_0",
        "password": "tablecloth-summerleaf-riverbasin-vacuumcleaner",
        "role": "user_cat_1"
      },
      {
        "user_id": "user_cat_1",
        "password": "secret-parsimony-archipelago-hedgehog",
        "role": "user_cat_1"
      },
      {
        "user_id": "user_cat_2",
        "password": "sailing-courageous-upsidedown-castle",
        "role": "user_cat_2"
      },
      {
        "user_id": "user_cat_3",
        "password": "rollerscates-forever-skyline-coconut",
        "role": "user_cat_3"
      },
      {
        "user_id": "user_cat_4",
        "password": "superfluid-altruistic-guitarplayer-climatechange",
        "role": "user_cat_4"
      },
      {
        "user_id": "user_cat_5",
        "password": "fireplace-olympicgames-grassroot-luminescence",
        "role": "user_cat_5"
      }
    ]
```

These 8 roles have hardcoded privileges in the framework package. You can change that in the python module

```
./setup/src_setup/lib_setup/setup_db.py
```
[TG TODO: THIS SHOULD BE CHANGED TO A JSON FILE THAT IS READ INSTEAD]

## Edit the Jupyter notebook setup_db

Return to the notebook interface for running the framework and the notebook

```
./setup/setup_db.ipynb
```

In that notebook edit the second code block (Scheme file) to point to the scheme file you edited above. If you saved the whole repository under your user home directory, the single code line in that block should be:

```
scheme_file = '~/framework_project_ai4sh_database/scheme_ai4sh_setup_db.json'
```

If you did not change any other names or paths you do not need to edit any of the other code blocks in the notebook, the name of the [job file][job_file] will still be job_setup_db.json.

## Run the notebook

If you did setup the [Jupyter notebook][notebook] and created the [Anaconda][anaconda] virtual environment for the framework, you can now setup the entire AI4SH database by running the notebook setup_db.

## Edit the database structure

The [next document][next] in this section explains the AI4SH database in detail and how you can edit the database structure and internal linkages.


## Acknowledgments and Funding

This work was partly done as part of the AI4SoilHealth project, funded by the European Union's Horizon Europe Research and Innovation Programme under Grant Agreement No. 101086179.

_Funded by the European Union. The views expressed are those of the authors and do not necessarily reflect those of the European Union or the European Research Executive Agency._


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

[github]: https://github.com/

[github_framework_project_ai4sh_database]: https://github.com/AI4SH/framework_project_ai4sh_database
