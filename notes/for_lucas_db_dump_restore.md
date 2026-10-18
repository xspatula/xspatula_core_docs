# Database dump and restore

Added 2026-10-18. Shared by `xspatula_core`, `xspatula_ai4sh` and `xspatula_lucas` (identical module in all three).

Dumps a complete xspatula database to a single file and restores it again. Built for LUCAS, where re-running setup + data import takes hours while a restore takes minutes, but nothing in it is project specific.

## What is (and is not) included

A dump is a full copy of one database:

- all schemas and tables (`community`, `process`, `utility`, `observation*`, `landscape*`, `audit`, ...) with all rows;
- every `SERIAL` id **and** the sequence position, so new inserts after a restore continue where the original left off;
- functions and triggers (including the audit triggers and `audit.logged_actions`);
- table privileges (GRANTs) to the database roles;
- application user passwords (bcrypt hashes in `community.user`) — they survive unchanged, so existing users can log in after a restore.

**Not** included: PostgreSQL login roles (`community_admin`, `login_evaluation`, `user_cat_1`...`user_cat_5`). Roles belong to the server (cluster), not to a database. On the same server they already exist; on a new server the restore re-creates them from `db_users` in the scheme file (see below).

**Limitation:** a dump can only be restored as a whole database. It can **not** be merged on top of another project's database (e.g. LUCAS data into an AI4SH database): both use `SERIAL` ids starting at 1, so ids of samples, units, indicators etc. collide. Combining projects requires a merge by natural keys (names/codes), which is not implemented.

## Files

| File | Role |
|---|---|
| `setup/src_setup/lib_setup/dump_restore_db.py` | `Dump_database(notebook_FP, scheme_file)`, `Restore_database(notebook_FP, scheme_file)` |
| `setup/src_setup/lib_setup/__init__.py` | must export both functions: `from .dump_restore_db import Dump_database, Restore_database` |
| `setup/dump_restore_db.ipynb` | notebook with one dump cell and one restore cell |

No job, pilot or process files are needed — everything is read from the `postgresdb` object of the scheme file.

## Scheme file settings

Use the same scheme file as for `setup_db.ipynb` (e.g. `setup/zzz/scheme_local_setup.json`). Add `dump_path` under `postgresdb`:

```json
"postgresdb": {
  "host": "localhost",
  "port": 5432,
  "db": "lucas",
  "user_name": "karttur",
  "password": "...",
  "dump_path": "~/xspatula/db_dump/xspatula_lucas.dump",
  "db_users": [ ... ]
}
```

| Key | Required | Meaning |
|---|---|---|
| `db`, `host`, `port` | yes | database to dump / restore into |
| `host_netrc_id` **or** `user_name` + `password` | yes | credentials, same as for database setup. Must be the database owner/superuser (the setup user) |
| `dump_path` | no | dump file. **Default if missing: `~/xspatula/db_dump/<db>.dump`** |
| `pg_bin_path` | no | folder with `pg_dump`/`pg_restore` if they are not found automatically |
| `db_users` | no | if present, roles are created/granted after a restore (as in database setup) |

Suggested `dump_path` values per project — keep dumps **outside** the project folder (they are large and contain user data):

| Project | `dump_path` |
|---|---|
| xspatula_core | `~/xspatula/db_dump/xspatula_core.dump` |
| xspatula_ai4sh | `~/xspatula/db_dump/xspatula_ai4sh.dump` |
| xspatula_lucas | `~/xspatula/db_dump/xspatula_lucas.dump` |

Path notation follows the standard rule: `~/` = home directory, `/...` = absolute, anything relative is resolved against the **project root** (`project_path`). Note: write `~/xspatula/...`, not `~xspatula/...` — without the slash the path is treated as relative to the project root.

`pg_dump`/`pg_restore` are searched for in: `pg_bin_path` → `PATH` → `/opt/homebrew/opt/postgresql@*/bin` → `/usr/local/opt/postgresql@*/bin` → `/Applications/Postgres.app/Contents/Versions/latest/bin` → `/usr/lib/postgresql/*/bin`. The `pg_dump` major version must be equal to or higher than the server's.

## Using the notebook

Open `setup/dump_restore_db.ipynb` and run:

1. **Imports and setup** cell.
2. **Scheme file** cell — set `scheme_file` (relative to the notebook, `~/` or absolute).
3. **Dump database** — or — **Restore database**.

### Dump

- Writes `pg_dump --format=custom` (compressed) to `<dump_path>.tmp` and renames it to `dump_path` only when the dump succeeded. A failed dump never destroys the previous dump.
- Creates the dump folder if missing.
- Prints elapsed time and file size.

### Restore

⚠️ **Destructive.**

1. If the database exists you are asked to confirm; it is then dropped with `DROP DATABASE ... WITH (FORCE)` (closes open connections from other notebooks; requires PostgreSQL 13+).
2. An empty database is created.
3. `pg_restore --no-owner --jobs=4` loads the dump. All objects become owned by the restoring user, so a dump from one machine restores on another with a different superuser name. Privileges are kept.
4. If `db_users` is set in the scheme: the per-role `.env` files (`src/postgres/environment/`) are rewritten and `Setup_prod_roles` creates any role missing on this server and grants it.

`pg_restore` errors are printed but do not stop the restore. On a new server, expect `role ... does not exist` errors for GRANTs; step 4 fixes these.

Re-dump after every import change — a restore is only as current as its dump.

## Command line alternatives

All commands run from the repository's `setup/` folder (same working directory as the notebooks).

### Python, same functions as the notebook

```bash
cd setup
python -c "from os import getcwd; import sys; sys.path.append('..'); \
from src_setup.lib_setup import Dump_database; \
Dump_database(getcwd(), './zzz/scheme_local_setup.json')"

python -c "from os import getcwd; import sys; sys.path.append('..'); \
from src_setup.lib_setup import Restore_database; \
Restore_database(getcwd(), './zzz/scheme_local_setup.json')"
```

Run inside the project's conda environment (e.g. `conda activate xspatula_py_3.12`). The restore still asks for confirmation interactively when the database exists.

### Plain PostgreSQL tools (no Python)

Set the connection once; the password is read from `~/.pgpass` or `PGPASSWORD` (never put it on the command line):

```bash
export PATH=/opt/homebrew/opt/postgresql@17/bin:$PATH   # if pg_dump is not on PATH
export PGHOST=localhost PGPORT=5432 PGUSER=karttur
mkdir -p ~/xspatula/db_dump
```

Dump:

```bash
pg_dump --format=custom --file=$HOME/xspatula/db_dump/xspatula_lucas.dump --dbname=lucas
```

Restore (drops the existing database):

```bash
psql -d postgres -c 'DROP DATABASE IF EXISTS lucas WITH (FORCE);'
psql -d postgres -c 'CREATE DATABASE lucas;'
pg_restore --dbname=lucas --no-owner --jobs=4 $HOME/xspatula/db_dump/xspatula_lucas.dump
```

On a new server, then run the **Setup database** cell of `setup_db.ipynb` (or `Restore_database` above) to create the roles — or create them before restoring.

Useful checks:

```bash
pg_restore --list $HOME/xspatula/db_dump/xspatula_lucas.dump | head   # inspect dump contents
psql -d lucas -c "SELECT schemaname, relname, n_live_tup FROM pg_stat_user_tables ORDER BY 1,2;"
```

## Verified

2026-10-18, `xspatula_lucas`: dump of `lucas` (9.2 MB, 3 s) restored into a scratch database (1 s) — 110 tables with identical row counts per table, 110 triggers.

## Done 2026-10-18 (Claude, xspatula_core_docs)

- New page `_setup_db/dump_restore.md` (`/setup_db/dump_restore/`), last in the setup_db sidebar (`_data/navigation.yml`) and in `_data/story_order.yml` after `manage_process`. Content from this note with core values (`db: xspatula`, `xspatula_py_3.12`, `xspatula_core.dump`), plus a section "Sharing a premade database" (sender: personal users/hashes travel with the dump; receiver: own PG version >= sender, own scheme file with `dump_path` and `db_users`, then restore and go to user data).
- Cross-references: "Shortcut with a premade database" in `_setup_db/introduction.md` (Prerequisits), "Alternative: restore a premade database" in `_setup_db/schemas_tables.md` and `_setup_processes/introduction.md` (Prerequisites).
- Verified against `xspatula_core/setup/src_setup/lib_setup/dump_restore_db.py` and `setup/dump_restore_db.ipynb`. Local build OK, new links and previous/next checked.

## LUCAS licence caveat (2026-10-18)

`_setup_db/dump_restore.md`: the "share a premade database" bullet now notes that sharing requires the data licence to allow it, and LUCAS does not. "Sharing a premade database" has a ⚠️ "Not for LUCAS" box citing the ESDAC LUCAS_SOIL licence (art. 3, 8), with links to the licence PDF and to the LUCAS docs section `xspatula_lucas_docs/lucas_2009/#lucas-license-the-data-may-not-be-shared`.
