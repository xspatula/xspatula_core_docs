---
title: "Setting up the initial user"
layout: single
sidebar:
  nav: "building"
excerpt: "Before you can register any process for your new project, you need at least one working login. The initial user is created automatically at setup from one .netrc entry: the framework reads the password, bcrypt-hashes it in memory and fills a placeholder row in the seed file."
permalink: /building/initial_user/
author_profile: false
date: 2026-08-24
last_modified_at: 2026-11-08
---

Registering processes for your new project — the JSON registration in [Adding a new process][building_processes], and inserting through any registered process at all — requires an already-logged-in `community.user`. There's no user until you seed one. `community.user.password` stores a bcrypt hash, not plain text, but you never handle the hash: the framework computes it at setup time from your `.netrc`. No password and no hash goes into the repository.

## Step 1: add a .netrc entry

Add one line to `~/.netrc` (see [.netrc and credentials][netrc] for the rules):

```
machine my_initial_user   login first_user         password <long-random-9>
```

Generate the password with `python3 -c "import secrets; print(secrets.token_urlsafe(18))"`. The `login` becomes the user's `user_name`; if it contains `@`, it is used as `email` instead.

## Step 2: put the placeholder in the seed row

In your project's user-seed file, `setup/zzz/<project>/setup_db/json/community/user_records_v10_sql.json`, set the bootstrap user's `password` value to the placeholder:

```json
"{initial_user}"
```

Use **one** placeholder row per seed file. Every placeholder row gets the same identity, so two would collide. Other rows with real bcrypt hashes are left untouched (in the shipped core example, `ini_cat5` uses the placeholder, `user_manager` and `jane_doe` keep their hashes).

## Step 3: add initial_user to the setup scheme

Add a top-level `initial_user` block (a sibling of `postgresdb`) to your setup scheme file, as in `setup/zzz/scheme_template_setup.json`:

```json
"initial_user": {
  "user_netrc_id": "my_initial_user",
  "email": "first.user@example.com",
  "email_alt": "first.user.alt@example.com",
  "first_name": "First",
  "last_name": "User",
  "organisation_id__organisation_name": "my_organisation",
  "territory_id__territory_name": "my_territory",
  "stratum_code": "5"
}
```

Only `user_netrc_id` is required. Every other key overrides the matching column of the seed row (email, names, organisation, territory, stratum_code).

## Step 4: run setup_db.ipynb

Run `setup_db.ipynb`'s main "Setup database" cell as normal — the same run that creates your schemas and tables from [Setting up the database][building_database]. At setup:

- `Resolve_initial_user()` (`setup_db_initiate.py`) reads `.netrc` and bcrypt-hashes the password in memory;
- `Apply_initial_user()` (`setup_db.py`) fills the placeholder row before it is inserted.

If the scheme has no `initial_user`, the placeholder row is **skipped** with a warning. The placeholder is never stored as a password.

## Step 5: log in with the same netrc id

Point the use scheme at the same entry, as in `setup/zzz/scheme_template_use.json`:

```json
{
  "project_path": "./my_project",
  "user_project": {
    "user_netrc_id": "my_initial_user"
  }
}
```

`user_name` is filled from the logged-in `community.user` record, so `user_netrc_id` alone is enough. One `.netrc` entry, nothing to remember. You can now register the processes covered in [Adding a new process][building_processes].

## If the database already exists

The seed only inserts rows that don't exist yet. To add or reset a password on a **live** database, generate a hash with the CLI and update the row directly:

```bash
cd /path/to/xspatula_core
conda activate <your-environment-name>
python3 setup/hash_password.py
```

You are prompted twice with hidden input (`Password to hash:` / `Repeat password:`). It prints a bcrypt hash such as `$2b$12$KIXQ7z3n...` (different every time, even for the same password). Then:

```sql
UPDATE community.user SET password = '<hash>' WHERE user_name = '...';
```

and update the password in `.netrc` to match. Avoid passing the password as an argument (`hash_password.py 'some-password'`): it lands in your shell history and process list.

`hash_password.py` is also the route for seeding additional, non-bootstrap users with fixed hashes.

## More detail

[Bootstrap user][setup_community_bootstrap_user] (Setup Community) covers the same "chicken-and-egg" problem in the context of registering later users through the Excel-intake workflow once this first one exists.

## Input files

| File | Purpose |
|---|---|
| `~/.netrc` | Holds the `my_initial_user` entry; chmod 600 |
| `setup/zzz/scheme_template_setup.json` | Setup scheme template with the `initial_user` block |
| `setup/zzz/scheme_template_use.json` | Use scheme template; logs in with the same netrc id |
| `setup/zzz/<project>/setup_db/json/community/user_records_v10_sql.json` | Seed file; bootstrap row has `"password": "{initial_user}"` |
| `setup/git_hooks/pre-commit` | Blocks commits of plaintext passwords in JSON; install manually |
| `setup/hash_password.py` | Optional; only for live-database resets and extra seeded users |

[building_processes]: {{ "/building/processes/" | relative_url }}
[building_database]: {{ "/building/database/" | relative_url }}
[netrc]: {{ "/setup_db/netrc/" | relative_url }}
[setup_community_bootstrap_user]: {{ "/setup_community/bootstrap_user/" | relative_url }}
