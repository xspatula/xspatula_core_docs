---
title: "Defining pg_users and their environment files"
layout: single
sidebar:
  nav: "building"
excerpt: "pg_users are created by the superuser when a database is set up, with passwords from .netrc. Every setup run re-applies their passwords and attributes. Every ordinary community.user row logs in through whichever pg_user its stratum_code points to — decide your roster before that first run."
permalink: /building/pg_users/
author_profile: false
date: 2026-08-25
last_modified_at: 2026-11-08
---

Your project needs its own set of pg_users — the postgres-level roles ordinary logins actually run under — defined in your own setup scheme file. This page covers the mechanics of how that works and why it has to be decided up front.

## Where pg_users are defined

In the `postgresdb.db_users` array of your project's own setup scheme file, mirroring the shipped `setup/zzz/scheme_template_setup.json`. Each entry references a `.netrc` entry, never a password:

```json
"db_users": [
  { "user_netrc_id": "community_admin",  "role": "community_admin" },
  { "user_netrc_id": "login_evaluation", "role": "login_evaluation" },
  { "user_netrc_id": "user_cat_1",       "role": "user_cat_1" },
  ...
  { "user_netrc_id": "user_cat_5",       "role": "user_cat_5" }
]
```

The netrc `login` becomes the PostgreSQL role name; an explicit `"user_id"` overrides it. `Resolve_db_users()` (`setup_db_initiate.py`) resolves all entries in memory before anything is created, and only on setup (delete never needs pg_user passwords). It is fail-fast: if any entry can't be resolved, nothing is set up. The matching `.netrc` lines are on [.netrc and credentials][netrc]. Roles are server-wide, so all projects on one server share these entries.

The legacy form `{"user_id", "password", "role"}` still works but prints `⚠️ plaintext password in scheme file`.

The full JSON shape and the predefined role names are documented in [Scheme file][framework_scheme_file] — that page is the syntax reference; this one covers what happens once you run it.

## Only the superuser can create pg_users

[Scheme file][framework_scheme_file] already states this; here's concretely why it's true, not just asserted. Creating postgres roles (`Create_db_roles`) and writing their `.env` credential files (`Create_db_environment_dot`) both happen inside `Initiate_database` — the function driven by `setup_db.ipynb` — and only run after the operator has authenticated as the postgres superuser and confirmed the "Set up database?" prompt. No other code path in the framework creates a pg_user or rewrites an `.env` file. There's no way for an ordinary logged-in user, even a stratum-5 one, to add a pg_user later.

**Practical implication**: decide your full pg_user roster — which of the roles you actually need — before your first `setup_db.ipynb` run. Adding one afterward means running database setup again as the superuser (which is safe; see below).

## How the .env files actually get created

One file per `db_users` entry, named `.<user_id>.env`, written under `src/postgres/environment/`:

```
DB_NAME=<db>
DB_USER=<user_id>
DB_PASSWORD=<password>
DB_HOST=<host>
DB_PORT=<port>
```

So a `db_users` entry whose netrc login is `user_cat_3` produces `src/postgres/environment/.user_cat_3.env`. These files are generated, not hand-edited — re-running database setup regenerates them from your scheme file's `db_users` and your `.netrc`. They are written with chmod 600 and are gitignored, but they are still plaintext.

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

- `setup/src_setup/lib_setup/roles_grants.json` — one `CREATE USER … WITH …; GRANT …` SQL template per role, with `{user}`/`{password}`/`{db}` placeholders.
- `setup/src_setup/lib_setup/revoke_privileges.json` — the matching `REVOKE`-style template per role, `{user}`/`{db}`.

Both files sit in the framework's own shared setup tooling, not under your project's `setup/zzz/<project>/` tree — that's deliberate, not an exception to "keep core files untouched" elsewhere in this collection. A postgres role is a single-cluster concept, not something namespaced per project the way schemas and tables are, so this is the framework's intended, supported place to customize it.

A representative entry from each file — `user_cat_1`'s connect-only pair:

```json
"user_cat_1": "CREATE USER {user} WITH LOGIN PASSWORD '{password}'; GRANT CONNECT ON DATABASE {db} TO {user};"
```
```json
"user_cat_1": "REVOKE CONNECT ON DATABASE {db} FROM {user};"
```

**Passwords and attributes are re-applied on every setup run.** For every role that already exists, `Create_db_roles` (`setup_db.py`) runs `ALTER ROLE <user> WITH <attributes> PASSWORD <pw>`:

- **Password** comes from `.netrc`. **To change a pg_user password: edit `.netrc`, re-run the setup cell.** That's all. The password is sent as a bound parameter, never formatted into SQL.
- **Attributes**: `Role_attributes()` reads the `CREATE USER … WITH` clause in `roles_grants.json` and sets all of `SUPERUSER`, `CREATEDB`, `CREATEROLE`, `LOGIN`, `REPLICATION` and `BYPASSRLS` explicitly: listed → on, not listed → `NO…`. `roles_grants.json` is authoritative. A manual `ALTER ROLE … NOLOGIN` (or any other attribute change made in `psql`) is undone on the next setup run; make the change in `roles_grants.json` instead.

**Changing an existing role's grants**: edit its value in `roles_grants.json` (and update the matching `revoke_privileges.json` entry to match), then re-run `setup_db.ipynb`'s main "Setup database" cell as the superuser — nothing else. Each run hashes the role's configured grant SQL and compares it against a hash stored as a comment on the postgres role from the last run. Unchanged grants are skipped; changed grants are automatically revoked (via `revoke_privileges.json`) before the new grants are applied and the new hash stored. The password is not part of this hash. (Consequence: the first setup run after upgrading to this version re-applies all grants once. Harmless.) There's no manual `REVOKE`/`GRANT` to write and no need to rebuild the database.

**Adding a brand-new role**: add a new key to both JSON files, then reference that role name in your scheme file's `db_users` entries via the `"role"` field (see [Scheme file][framework_scheme_file]).

**Keep both files' key sets in sync.** If a role exists in `roles_grants.json` but has no matching key in `revoke_privileges.json`, revoking it silently does nothing — no error, no print, the grants just stay in place.

## Input files

| File | Purpose |
|---|---|
| `~/.netrc` | One entry per pg_user; the passwords live only here |
| `setup/zzz/scheme_template_setup.json` | Setup scheme template; copy to your project's setup scheme; `postgresdb.db_users` defines the pg_user roster |
| `src/postgres/environment/.<user_id>.env` | Generated, one per `db_users` entry, chmod 600, gitignored — never hand-edit, re-running setup regenerates them |
| `setup/src_setup/lib_setup/roles_grants.json` | Framework-level; GRANT SQL template per role — edit to change or add a role's privileges |
| `setup/src_setup/lib_setup/revoke_privileges.json` | Framework-level; matching REVOKE SQL template per role — keep in sync with the file above |
| `setup/git_hooks/pre-commit` | Blocks commits of plaintext passwords in JSON; install manually |

[framework_scheme_file]: {{ "/framework/scheme_file/" | relative_url }}
[manage_process]: {{ "/setup_db/manage_process/" | relative_url }}
[netrc]: {{ "/setup_db/netrc/" | relative_url }}
[building_initial_user]: {{ "/building/initial_user/" | relative_url }}
[building_add_users]: {{ "/building/add_users/" | relative_url }}
