---
title: "Scheme file for process setup"
layout: single
sidebar:
  nav: "setup_processes"
excerpt: "Once the database is set up, you should stop using the postgres superuser. All subsequent operations — including registering processes — use an ordinary database user defined in a simpler scheme file."
permalink: /setup_processes/scheme_file/
author_profile: false
date: 2026-06-09
last_modified_at: 2026-06-09
---

Once you have set up the database you should avoid using the postgres superuser for managing access and running processes. Instead you log in as one of the database users defined during [database setup][setup_db] or using the process <span class='process'>manage_user</span>.

The scheme file for using the database — as opposed to creating it — no longer needs information about the postgres database cluster. It only needs a `project_path` and user credentials for the database (`user_project`).

## Scheme file structure

The minimal scheme file for process registration (and all other post-setup operations) looks like this:

```json
{
  "project_path": "./xspatula",
  "user_project": {
    "user_name": "ini_cat5",
    "password": "armchair"
  }
}
```

If you want to change the default settings for execute, verbosity, overwrite and delete you need to add the process object from its default settings (the example below shows the default settings):

```json
{
  "project_path": "./xspatula",
  "user_project": {
    "user_name": "ini_cat5",
    "password": "armchair"
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

The `user_name` and `password` refer to an ordinary database user (not a postgres pg_user). This user must exist in the `community.user` table and must have sufficient privileges — a stratum-5 user is recommended for setup operations. For details on the different options of setting up users see [Setup Community][setup_community].

## Using a .netrc file

You can replace `user_name` and `password` with a `user_netrc_id` to store credentials in a [.netrc file][netrc]:

```json
{
  "project_path": "./xspatula",
  "user_project": {
    "user_netrc_id": "xspatula"
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

The corresponding entry in `~/.netrc`:

```
machine xspatula login ini_cat5 password armchair
```

The `machine` value is an arbitrary identifier you choose — the framework reads the `login` and `password` from the matching `.netrc` entry.

## Key differences from the setup_db scheme file

| Property | setup_db scheme | Post-setup scheme |
|---|---|---|
| `postgresdb` block | Required (superuser, host, port, db name) | Not present |
| `user_project` block | Not present | Required |
| `db_users` array | Required (defines pg_users) | Not present |
| User type | Postgres superuser | Ordinary database user |

See the [scheme file reference][scheme_file_ref] in the framework documentation for a full description of all scheme file options.

## Input files

| File | Purpose |
|---|---|
| `scheme_xspatula_local_use.json` | Example scheme file for post-setup operations; uses `ini_cat5` user with explicit password |

[setup_db]: ../../setup_db/

[netrc]: ../../setup_db/netrc/

[scheme_file_ref]: ../../framework/scheme_file/

[setup_community]: ../../setup_community/
