---
title: "Dump and restore"
layout: single
sidebar:
  nav: "setup_db"
excerpt: "Copy a complete Xspatula database to a single file and restore it again, on the same machine or another. Restoring a premade database replaces the database setup, the process setup and the data import."
permalink: /setup_db/dump_restore/
author_profile: false
date: 2026-10-18
last_modified_at: 2026-10-18
---

A complete Xspatula database can be written to a single dump file and restored from it. Think of the setup notebooks as the recipe and the dump as the finished, frozen dish: cooking from the recipe takes time, thawing takes minutes.

Use a dump to:

- **back up** a database before an update, or after a long data import,
- **move** a database to another machine,
- **share a premade database**, so that the receiver can skip [defining schemas and tables][schemas_tables], [setting up processes][setup_processes] and importing data. Only where the data licence allows it; for LUCAS it does [not](#sharing-a-premade-database), and AI4SH data may only be shared within the AI4SoilHealth partners.

For the LUCAS database in the sibling project `xspatula_lucas`, setup and data import take hours; a restore takes minutes. The same module ships with `xspatula_core`, `xspatula_ai4sh` and `xspatula_lucas`, and nothing in it is project specific.

## What a dump contains

A dump is a full copy of **one** database:

- all schemas and tables (`community`, `process`, `utility` and any project schemas) with all rows,
- every `SERIAL` id **and** its sequence position, so new inserts after a restore continue where the original left off,
- functions and triggers, including the audit triggers and `audit.logged_actions`,
- table privileges (GRANTs) to the database roles,
- the application users in `community.user`, including their bcrypt password hashes. Existing users can log in after a restore with their old passwords.

**Not** included are the PostgreSQL login roles (`community_admin`, `login_evaluation`, `user_cat_1` ... `user_cat_5`). Roles belong to the server, not to a database. On the same server they already exist; on a new server the restore creates them from `db_users` in the scheme file (see [Restore](#restore)).

**Limitation:** a dump can only be restored as a whole database. It can **not** be merged into another project's database (e.g. LUCAS data into an AI4SH database). Both use `SERIAL` ids starting at 1, so the ids of samples, units, indicators etc. collide. Combining projects needs a merge by natural keys (names, tags), which is not implemented.

## Files

| File | Role |
|---|---|
| `setup/dump_restore_db.ipynb` | Notebook with one dump cell and one restore cell |
| `setup/src_setup/lib_setup/dump_restore_db.py` | `Dump_database(notebook_FP, scheme_file)` and `Restore_database(notebook_FP, scheme_file)` |
| `setup/src_setup/lib_setup/__init__.py` | Exports both functions |

No job, pilot or process files are needed. Everything is read from the `postgresdb` object of the scheme file.

## Scheme file settings

Use the same [scheme file][scheme_file] as for `setup_db.ipynb` (e.g. `setup/zzz/scheme_local_setup.json`) and add `dump_path` under `postgresdb`:

```json
"postgresdb": {
  "host": "localhost",
  "port": 5432,
  "db": "xspatula",
  "user_name": "karttur",
  "password": "...",
  "dump_path": "~/xspatula/db_dump/xspatula_core.dump",
  "db_users": [ ... ]
}
```

| Key | Required | Meaning |
|---|---|---|
| `db`, `host`, `port` | yes | Database to dump, or to restore into |
| `host_netrc_id` **or** `user_name` + `password` | yes | Credentials, as for the database setup. Must be the database owner or superuser (the setup user) |
| `dump_path` | no | Dump file. **Default: `~/xspatula/db_dump/<db>.dump`** |
| `pg_bin_path` | no | Folder with `pg_dump` and `pg_restore`, if they are not found automatically |
| `db_users` | no | If present, the roles are created and granted after a restore, as in the database setup |

Keep dumps **outside** the project folder. They are large and contain user data. Suggested paths:

| Project | `dump_path` |
|---|---|
| xspatula_core | `~/xspatula/db_dump/xspatula_core.dump` |
| xspatula_ai4sh | `~/xspatula/db_dump/xspatula_ai4sh.dump` |
| xspatula_lucas | `~/xspatula/db_dump/xspatula_lucas.dump` |

Paths follow the standard rule: `~/` is the home directory, `/...` is absolute, and anything else is resolved against the project root (`project_path`). Write `~/xspatula/...`, not `~xspatula/...`; without the slash the path is read as relative to the project root.

`pg_dump` and `pg_restore` are searched for in this order: `pg_bin_path` → `PATH` → `/opt/homebrew/opt/postgresql@*/bin` → `/usr/local/opt/postgresql@*/bin` → `/Applications/Postgres.app/Contents/Versions/latest/bin` → `/usr/lib/postgresql/*/bin`. The major version of `pg_dump` must be equal to or higher than that of the server.

## Using the notebook

Open `setup/dump_restore_db.ipynb` and run:

1. **Imports and setup**.
2. **Scheme file**: set `scheme_file` (relative to the notebook, `~/` or absolute).
3. **Dump database** or **Restore database**. Both cells start with `%%script false --no-raise-error`; remove that line to run the cell.

```python
scheme_file = './zzz/scheme_local_setup.json'

Dump_database(notebook_path, scheme_file)
```

### Dump

- Writes a compressed `pg_dump --format=custom` file to `<dump_path>.tmp` and renames it to `dump_path` only when the dump succeeded. A failed dump never destroys the previous one.
- Creates the dump folder if it is missing.
- Prints the elapsed time and the file size.

Dump again after every import or change. A restore is only as current as its dump.

### Restore

⚠️ **Destructive.** An existing database with the same name is replaced.

1. If the database exists you are asked to confirm. It is then dropped with `DROP DATABASE ... WITH (FORCE)`, which also closes open connections from other notebooks (PostgreSQL 13 or later).
2. An empty database is created.
3. `pg_restore --no-owner --jobs=4` loads the dump. All objects become owned by the restoring user, so a dump from one machine restores on another with a different superuser name. Privileges are kept.
4. If `db_users` is set in the scheme file, the per-role `.env` files in `src/postgres/environment/` are rewritten, and any role missing on this server is created and granted.

`pg_restore` errors are printed but do not stop the restore. On a new server, expect `role ... does not exist` errors for the GRANTs; step 4 fixes them.

## Sharing a premade database

To give someone a ready database instead of letting them run the whole setup:

> ⚠️ **Not for LUCAS.** A database holding LUCAS soil data (`xspatula_lucas`) must **not** be shared outside your own organisation. The [LUCAS_SOIL licence agreement][lucas_license] is personal and non-transferable, with no sub-licence (art. 3), and access is limited to the licensee's organisation (art. 8). Every user must request the data from ESDAC and accept the licence themselves. Dump LUCAS for your own backup and restore only; see [LUCAS license][lucas_docs_license] in the LUCAS docs.

> ⚠️ **AI4SH: partners only.** The AI4SH data has no licence yet and has not been reviewed and verified. A database holding AI4SH data (`xspatula_ai4sh`) may only be shared within the [AI4SoilHealth][ai4sh] project partners; see [AI4SH data: partners only][ai4sh_docs_partners] in the AI4SH docs.

Check the licence of every dataset in a database before sharing it.

- **Sender:** dump the database and send the file. The dump holds every row, including the users in `community.user` with their e-mail addresses and password hashes. Remove or replace personal users before dumping a database you share outside your own group.
- **Receiver:**
  - install PostgreSQL ([PostgreSQL][postgres]) with a version at least as new as the sender's, and the Python environment ([Anaconda][anaconda]);
  - write a setup scheme file with their own superuser credentials, `db`, `dump_path` pointing at the received file, and `db_users` with their own role passwords;
  - run **Restore database**.

The database name and the role passwords are the receiver's own; only the content comes from the dump. The application users in `community.user`, and their passwords, come with the dump.

After the restore, continue with [adding user data][user_data]. The steps [Defining schemas & tables][schemas_tables], [Bootstrapping process management][manage_process] and [Setup processes][setup_processes] are already done.

## Command line alternatives

All commands run from the repository's `setup/` folder, the same working directory as the notebooks.

### Python, same functions as the notebook

Run inside the project's conda environment (`conda activate xspatula_py_3.12`):

```bash
cd setup
python -c "from os import getcwd; import sys; sys.path.append('..'); \
from src_setup.lib_setup import Dump_database; \
Dump_database(getcwd(), './zzz/scheme_local_setup.json')"

python -c "from os import getcwd; import sys; sys.path.append('..'); \
from src_setup.lib_setup import Restore_database; \
Restore_database(getcwd(), './zzz/scheme_local_setup.json')"
```

The restore still asks for confirmation when the database exists.

### Plain PostgreSQL tools

Set the connection once. The password is read from `~/.pgpass` or `PGPASSWORD`; never put it on the command line.

```bash
export PATH=/opt/homebrew/opt/postgresql@17/bin:$PATH   # if pg_dump is not on PATH
export PGHOST=localhost PGPORT=5432 PGUSER=karttur
mkdir -p ~/xspatula/db_dump
```

Dump:

```bash
pg_dump --format=custom --file=$HOME/xspatula/db_dump/xspatula_core.dump --dbname=xspatula
```

Restore (drops the existing database):

```bash
psql -d postgres -c 'DROP DATABASE IF EXISTS xspatula WITH (FORCE);'
psql -d postgres -c 'CREATE DATABASE xspatula;'
pg_restore --dbname=xspatula --no-owner --jobs=4 $HOME/xspatula/db_dump/xspatula_core.dump
```

On a new server, create the roles afterwards with `Restore_database` above or the setup cell of `setup_db.ipynb`, or create them before restoring.

Useful checks:

```bash
pg_restore --list $HOME/xspatula/db_dump/xspatula_core.dump | head   # dump contents
psql -d xspatula -c "SELECT schemaname, relname, n_live_tup FROM pg_stat_user_tables ORDER BY 1,2;"
```

## Input files

| File | Purpose |
|---|---|
| `scheme_local_setup.json` | Setup scheme file; `postgresdb` gives the database, the superuser credentials, `dump_path` and `db_users` |
| `dump_restore_db.ipynb` | Notebook running `Dump_database` and `Restore_database` |
| `<db>.dump` | The dump file, outside the project folder |

[schemas_tables]: ../schemas_tables/

[manage_process]: ../manage_process/

[postgres]: ../postgres/

[anaconda]: ../anaconda/

[scheme_file]: ../../framework/scheme_file/

[setup_processes]: ../../setup_processes/

[user_data]: ../../user_data/

[lucas_license]: https://esdac.jrc.ec.europa.eu/projects/LUCAS/Documents/LUCAS_SOIL_LIC_AGR_final_for_web.pdf

[ai4sh]: https://ai4soilhealth.eu

[ai4sh_docs_partners]: https://xspatula.github.io/xspatula_ai4sh_docs/all_data/#ai4sh-data-partners-only

[lucas_docs_license]: https://xspatula.github.io/xspatula_lucas_docs/lucas_2009/#lucas-license-the-data-may-not-be-shared
