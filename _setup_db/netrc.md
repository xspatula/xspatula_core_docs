---
title:  ".netrc and credentials"
layout: single
sidebar:
  nav: "setup_db"
excerpt: "Every password the Xspatula framework uses lives in one hidden file, ~/.netrc. Scheme files and seed JSON files only hold a reference (a *_netrc_id), never the password itself."
permalink: /setup_db/netrc/
author_profile: false
date:   2026-03-15 16:13:03 +0200
last_modified_at:   2026-11-08
---

Every password the Xspatula framework uses lives in one hidden file in your home directory, `~/.netrc`. Scheme files and seed JSON files only hold a reference to an entry in that file, a `*_netrc_id`, never the password itself.

## Three rules

1. **No password ever goes in a scheme file or a seed JSON file.** Every credential lives in `~/.netrc` and is referenced by a `*_netrc_id`.
2. **The database must actually check passwords.** `pg_hba.conf` must use `scram-sha-256` for TCP connections, and the configuration must be reloaded. Several common installs default to `trust`, which accepts *any* password. See [Securing authentication][postgres_hba].
3. **Hashing is automatic.** The bootstrap user's bcrypt hash is computed at setup time from `.netrc`. There is no manual hashing step and no hash in the repository. See [Setting up the initial user][initial_user].

**Why this matters.** Scheme files sit in the project folder and get committed with it. A password committed to a public repository is compromised, even if you delete it later: the git history and any clone or fork keep it. If that has happened to you, change every affected password now (edit `.netrc` and re-run setup, see [Changing a password](#changing-a-password)).

## Entry format

One entry per line, with all three keywords:

```
machine <netrc_id> login <user name> password <password>
```

- `machine` is the id you reference from the scheme file (e.g. `"host_netrc_id": "my_pg_superuser"`). It is an arbitrary label, not a host name.
- `login` is the user name on the resource. For pg_users it becomes the PostgreSQL role name.
- `password` is the password.

## Full example

The block below matches the shipped template `setup/zzz/scheme_template_setup.json`: one postgres superuser, the 7 pg_users and the initial (bootstrap) application user.

```
machine my_pg_superuser   login postgres_superuser password <long-random-1>
machine community_admin   login community_admin    password <long-random-2>
machine login_evaluation  login login_evaluation   password <long-random-3>
machine user_cat_1        login user_cat_1         password <long-random-4>
machine user_cat_2        login user_cat_2         password <long-random-5>
machine user_cat_3        login user_cat_3         password <long-random-6>
machine user_cat_4        login user_cat_4         password <long-random-7>
machine user_cat_5        login user_cat_5         password <long-random-8>
machine my_initial_user   login first_user         password <long-random-9>
```

Machine names are global for the whole PostgreSQL server. Roles are shared by every database on a server, so all projects on one server share the pg_user entries. Only the superuser entry and the initial user entry may differ per project.

## Rules for the file

- One entry per line.
- All three keywords `machine`, `login` and `password` on every line.
- No spaces and no `#` inside a password.
- Restrict access to yourself:

  ```bash
  chmod 600 ~/.netrc
  ```

Generate each password with:

```bash
python3 -c "import secrets; print(secrets.token_urlsafe(18))"
```

The output is 24 characters from `A-Z a-z 0-9 - _`, so it never contains a space or `#`.

## The netrc ids the framework reads

| Scheme key | Used for | Scheme file |
|---|---|---|
| `postgresdb.host_netrc_id` | postgres superuser that creates, deletes, dumps and restores the database | setup, delete |
| `postgresdb.db_users[].user_netrc_id` | one pg_user (role) each; `login` becomes the role name | setup |
| `initial_user.user_netrc_id` | the bootstrap application user in `community.user`; password is hashed at setup | setup |
| `user_project.user_netrc_id` | an application user logging in to an existing database | use |

The same `my_initial_user` entry serves both at setup (`initial_user`) and afterwards for login (`user_project`). One entry, nothing to remember.

## Changing a password

- **pg_user** (`db_users`): edit `.netrc`, re-run the setup cell of `setup_db.ipynb`. Every run re-applies the password to the existing role. See [pg_users][pg_users].
- **Postgres superuser**: `psql -U <superuser> -d postgres -c "\password <superuser>"`, then update `.netrc`.
- **Application user** (`community.user`): the seed only inserts rows that don't exist, so for an existing user use the live `UPDATE` route in [Setting up the initial user][initial_user_live], then update `.netrc`.

## Pitfalls

**One bad line breaks every lookup.** Python's `netrc` module parses the whole file, so a single malformed line makes every `*_netrc_id` fail, not just the one on that line. The most common mistake is a line missing the `password` keyword:

```
machine x login y secret          # wrong
machine x login y password secret # right
```

The framework error names only the file and line number. It does not print the offending token, because that token often is a password and would end up in the notebook output.

**An open editor can overwrite new entries.** If `.netrc` is open in an editor while setup tooling (or you, in a terminal) adds entries, saving from the editor writes back the old content. Close the file first, or reload it before saving.

## Guard rails in the repository

- `*_pswd.json` is gitignored. These are files that exist only to hold a plaintext password.
- Generated `.env` files under `src/postgres/environment/` are gitignored and chmod 600. They are still plaintext.
- A pre-commit hook blocks commits of JSON files with a plaintext `"password"` or `"pswd"` value. Empty values, the `{initial_user}` placeholder and bcrypt hashes (`$2…`) are allowed. It is not installed automatically. Install it once per clone:

  ```bash
  cp setup/git_hooks/pre-commit .git/hooks/ && chmod +x .git/hooks/pre-commit
  ```

## Legacy: plaintext in the scheme file

The framework still accepts `user_name` + `password` for the superuser and `user_id` + `password` in `db_users`, but prints `⚠️ plaintext password in scheme file`. Don't use it for anything you commit.

## Input files

| File | Purpose |
|---|---|
| `~/.netrc` | All credentials; chmod 600 |
| `setup/zzz/scheme_template_setup.json` | Setup scheme template; superuser, 7 pg_users and initial_user, all via netrc ids |
| `setup/zzz/scheme_template_use.json` | Use scheme template; `user_project.user_netrc_id` only |
| `setup/git_hooks/pre-commit` | Blocks commits of plaintext passwords in JSON; install manually |

[postgres_hba]: {{ "/setup_db/postgres/#securing-authentication-pg_hbaconf" | relative_url }}
[initial_user]: {{ "/building/initial_user/" | relative_url }}
[initial_user_live]: {{ "/building/initial_user/#if-the-database-already-exists" | relative_url }}
[pg_users]: {{ "/building/pg_users/" | relative_url }}
