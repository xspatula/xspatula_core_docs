---
title: "Scheme file for process setup"
layout: single
sidebar:
  nav: "setup_processes"
excerpt: "Once the database is set up, you should stop using the postgres superuser. All subsequent operations — including registering processes — use an ordinary database user defined in a simpler scheme file."
permalink: /setup_processes/scheme_file/
author_profile: false
date: 2026-06-09
last_modified_at: 2026-11-08
---

Once you have set up the database you should avoid using the postgres superuser for managing access and running processes. Instead you log in as one of the database users defined during [database setup][setup_db] or using the process <span class='process'>manage_user</span>.

The scheme file for using the database — as opposed to creating it — no longer needs information about the postgres database cluster. It only needs a `project_path` and a `.netrc` reference for the user (`user_project.user_netrc_id`). No password goes in the scheme file.

## Scheme file structure

The minimal scheme file for process registration (and all other post-setup operations), as in the shipped template `setup/zzz/scheme_template_use.json`:

```json
{
  "project_path": "./my_project",
  "user_project": {
    "user_netrc_id": "my_initial_user"
  }
}
```

The corresponding entry in `~/.netrc`:

```
machine my_initial_user   login first_user         password <long-random-9>
```

The `machine` value is an identifier you choose; the framework reads `login` and `password` from the matching entry. Use the same entry as the setup scheme's `initial_user` block (see [Setting up the initial user][initial_user]): the user created at setup is the one you log in with now.

`user_netrc_id` alone is enough. `Project_login` fills `user_project.user_name` from the logged-in `community.user` record (it is used as `creator` when registering processes), so you don't need to repeat it.

The user is an ordinary application user (not a postgres pg_user). It must exist in the `community.user` table and must have sufficient privileges — a stratum-5 user is recommended for setup operations. For details on the different options of setting up users see [Setup Community][setup_community].

If you want to change the default settings for execute, verbosity, overwrite and delete, add the process object (the example shows the default settings):

```json
{
  "project_path": "./my_project",
  "user_project": {
    "user_netrc_id": "my_initial_user"
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

**Legacy:** `"user_name"` + `"password"` in `user_project` still works but puts a plaintext password in the scheme file. Don't commit it; the [pre-commit hook][netrc_guard] blocks it.

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
| `setup/zzz/scheme_template_use.json` | Use scheme template; `user_project.user_netrc_id` only |
| `~/.netrc` | Holds the user's login and password; chmod 600 |
| `setup/git_hooks/pre-commit` | Blocks commits of plaintext passwords in JSON; install manually |

[setup_db]: ../../setup_db/

[netrc]: ../../setup_db/netrc/

[scheme_file_ref]: ../../framework/scheme_file/

[setup_community]: ../../setup_community/

[initial_user]: ../../building/initial_user/

[netrc_guard]: ../../setup_db/netrc/#guard-rails-in-the-repository
