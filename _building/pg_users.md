---
title: "Defining pg_users and their environment files"
layout: single
sidebar:
  nav: "building"
excerpt: "pg_users are created only once, by the superuser, when a database is set up. Every ordinary community.user row logs in through whichever pg_user its stratum_code points to — decide your roster before that first run."
permalink: /building/pg_users/
author_profile: false
date: 2026-08-25
last_modified_at: 2026-08-25
---

Your project needs its own set of pg_users — the postgres-level roles ordinary logins actually run under — defined in your own setup scheme file. This page covers the mechanics of how that works and why it has to be decided up front.

## Where pg_users are defined

In the `postgresdb.db_users` array of your project's own setup scheme file, mirroring the shipped `setup/zzz/scheme_xspatula_local_setup.json`. The full JSON shape, the 8 predefined role names, and how to add one pg_user per role are already documented in [Scheme file][framework_scheme_file] — that page is the syntax reference; this one covers what happens once you run it.

## Only the superuser can create pg_users

[Scheme file][framework_scheme_file] already states this; here's concretely why it's true, not just asserted. Creating postgres roles (`Create_db_roles`) and writing their `.env` credential files (`Create_db_environment_dot`) both happen inside `Initiate_database` — the function driven by `setup_db.ipynb` — and only run after the operator has authenticated as the postgres superuser and confirmed the "Set up database?" prompt. No other code path in the framework creates a pg_user or rewrites an `.env` file. There's no way for an ordinary logged-in user, even a stratum-5 one, to add a pg_user later.

**Practical implication**: decide your full pg_user roster — which of the 8 roles you actually need — before your first `setup_db.ipynb` run. Adding one afterward means running database setup again as the superuser.

## How the .env files actually get created

One file per `db_users` entry, named `.<user_id>.env`, written under `src/postgres/environment/`:

```
DB_NAME=<db>
DB_USER=<user_id>
DB_PASSWORD=<password>
DB_HOST=<host>
DB_PORT=<port>
```

So a `db_users` entry with `"user_id": "user_cat_3"` produces `src/postgres/environment/.user_cat_3.env`. These files are generated, not hand-edited — re-running database setup regenerates them from whatever your scheme file's `db_users` array currently says.

## How an ordinary user links to a pg_user at login

`community.user.stratum_code` is turned into a pg_user name by plain string formatting — `stratum_code = 3` becomes the string `user_cat_3`, and from there the filename `.user_cat_3.env`. There's no lookup table and no validation: if no `.env` file exists for that exact stratum, login fails outright rather than falling back to something less privileged.

This applies identically no matter how the `community.user` row was created — seeded directly (see [Setting up the initial user][building_initial_user]) or self-registered through [Excel intake][building_add_users], which defaults `stratum_code` to `0` if the spreadsheet leaves it blank. Either way, **your pg_user roster has to include an entry for every stratum_code value any of your users will actually have**, decided before setup, per the section above.

Under the hood, logging in is a two-connection handoff: password verification always happens over a fixed `login_evaluation` connection, regardless of who's logging in. Only after that succeeds does a second, separate connection open using the stratum-specific `.env` file's credentials — that second connection is what the rest of the session actually runs under. This is why a missing `.env` file doesn't degrade gracefully: the password check can succeed while the follow-up connection has nothing to connect with.

## stratum_code vs. pg_user tier — two different layers

It's easy to conflate these, since the shipped defaults use the same numbers for both, but they're separate systems:

- **`min_user_stratum`** (covered in [Bootstrapping process management][manage_process]) is an application-level check — which processes a logged-in user is allowed to *call*.
- **The pg_user tier** a `stratum_code` resolves to is a postgres-level role — what the underlying database *connection* is actually permitted to do.

The shipped defaults happen to make these coincide at the top: stratum 5 resolves to `user_cat_5`, which is created `WITH SUPERUSER` — a real postgres superuser, not just "allowed to call `add_root_process`." Below that, privileges scale by tier: `user_cat_1` through `user_cat_4` get connect-only grants (no schema access) by default; `community_admin`/`login_evaluation` get their fixed `community` (and, for `login_evaluation`, `audit`) grants; `user_cat_5` alone gets explicit `utility`/`process` schema grants on top of its superuser status. If your project needs finer-grained access for tiers 1-4 — read access to your own domain schemas, say — see [Editing and revoking role grants](#editing-and-revoking-role-grants) below.

## Editing and revoking role grants

Every role's grants live in two JSON config files:

- `setup/src_setup/lib_setup/roles_grants.json` — one `GRANT`-style SQL template per role, with `{user}`/`{password}`/`{db}` placeholders.
- `setup/src_setup/lib_setup/revoke_privileges.json` — the matching `REVOKE`-style template per role, `{user}`/`{db}`.

Both files sit in the framework's own shared setup tooling, not under your project's `setup/zzz/<project>/` tree — that's deliberate, not an exception to "keep core files untouched" elsewhere in this collection. A postgres role is a single-cluster concept, not something namespaced per project the way schemas and tables are, so this is the framework's intended, supported place to customize it.

A representative entry from each file — `user_cat_1`'s connect-only pair:

```json
"user_cat_1": "CREATE USER {user} WITH LOGIN PASSWORD '{password}'; GRANT CONNECT ON DATABASE {db} TO {user};"
```
```json
"user_cat_1": "REVOKE CONNECT ON DATABASE {db} FROM {user};"
```

**Changing an existing role's grants**: edit its value in `roles_grants.json` (and update the matching `revoke_privileges.json` entry to match), then re-run `setup_db.ipynb`'s main "Setup database" cell as the superuser — nothing else. Role creation is idempotent and self-healing: each run hashes the role's configured grant SQL and compares it against a hash stored as a comment on the postgres role from the last run. Unchanged roles are skipped; a changed role has its old grants automatically revoked (via `revoke_privileges.json`) before the new grants are applied and the new hash stored. There's no manual `REVOKE`/`GRANT` to write and no need to rebuild the database — re-running when nothing changed is a safe no-op.

**Adding a brand-new role**: add a new key to both JSON files, then reference that role name in your scheme file's `db_users` entries via the `"role"` field (see [Scheme file][framework_scheme_file]).

**Keep both files' key sets in sync.** If a role exists in `roles_grants.json` but has no matching key in `revoke_privileges.json`, revoking it silently does nothing — no error, no print, the grants just stay in place.

## Input files

| File | Purpose |
|---|---|
| `setup/zzz/scheme_<project>_local_setup.json` | Your project's setup scheme file; its `postgresdb.db_users` array defines the pg_user roster |
| `src/postgres/environment/.<user_id>.env` | Generated, one per `db_users` entry — never hand-edit, re-running setup regenerates them |
| `setup/src_setup/lib_setup/roles_grants.json` | Framework-level; GRANT SQL template per role — edit to change or add a role's privileges |
| `setup/src_setup/lib_setup/revoke_privileges.json` | Framework-level; matching REVOKE SQL template per role — keep in sync with the file above |

[framework_scheme_file]: /framework/scheme_file/
[manage_process]: /setup_db/manage_process/
[building_initial_user]: /building/initial_user/
[building_add_users]: /building/add_users/
