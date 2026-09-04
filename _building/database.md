---
title: "Setting up the database"
layout: single
sidebar:
  nav: "building"
excerpt: "Define your own schemas and tables the same way the example project's are, then optionally extend the postgres layer with project-specific SQL helpers, kept separate from the reusable core."
permalink: /building/database/
author_profile: false
date: 2026-08-24
last_modified_at: 2026-08-24
---

A real project needs its own schemas and tables beyond what the framework ships by default. This page covers both defining them, and — for anything beyond simple CRUD (Create, Read, Update, and Delete) — extending the postgres layer with your own project-specific SQL.

## Defining your own schemas and tables

Follow exactly the pattern already documented in [Setup DB][setup_db] / [Defining schemas & tables][schemas_tables] — `create_schema` and `create_table` process files, run through `setup_db.ipynb`. The only thing that changes for your own project is where these files live: create your own tree mirroring the framework's own `setup/zzz/xspatula/setup_db/json_core/`:

```
setup/zzz/<project>/setup_db/json_<project>/
├── schema/          # create_schema definitions
├── community/        # (if you extend the default community tables)
├── process/           # (if you extend the default process tables)
├── utility/            # (if you extend the default utility tables)
├── audit/               # generated — see Auditing below, don't hand-edit
└── <your_domain>/         # your own project-specific schemas and tables
```

While you're defining each table, decide whether it should be audited — see [Auditing][building_auditing] on this collection for the one decision you need to make at this stage.

## Extending the postgres layer (optional)

Most projects eventually need SQL queries that don't fit the framework's generic insert/update/select machinery — a lookup that spans several tables, a custom aggregation, something specific to your data model. The framework's convention for this keeps that code **out of the reusable core** entirely, in its own small class.

A real project doing exactly this exists as a worked example: `xspatula_ai4sh` adds `src/postgres/pg_ai4sh.py`, defining a class of custom SQL helper methods used only by that project's own application modules. The core `src/postgres/` files (`pg_session.py`, `pg_common.py`, `pg_processes.py`, `pg_get_schema_table.py`) are untouched — that one new file is the entire footprint on the postgres layer.

### The shape of the class

```python
class PG_manage_<Project>:
    '''
    Project-specific SQL helpers for <project>.
    '''

    def _Retrieve_something(self, query_D, pg_session_C):

        sql = "SELECT ... FROM <schema>.<table> WHERE ... = '%(some_key)s';" % query_D

        rec = pg_session_C._Execute_search_single_sql(sql)

        return rec
```

Worth calling out plainly, since it looks unusual at first glance: each method takes the database session (`pg_session_C`) as an explicit parameter, rather than the class storing it once in `__init__` and reusing it internally. That's the real, working shape of this pattern in practice — not a mistake to "fix." Think of the class as a bundle of related SQL helper functions grouped under one name, not a stateful connection wrapper.

### Hooking it into your project module

In whichever of your `src/<project>/` handler classes needs it (often `process.py`, or a handler module like the one covered in [Adding a new process][building_processes]), import your class and construct it right next to where you already store the session:

```python
from src.postgres.pg_<project> import PG_manage_<Project>

class Process_something(Get_schema_table):

    def __init__(self, process_S, pg_session_C):

        self.process_S = process_S
        self.pg_session_C = pg_session_C
        self.pg_<project>_C = PG_manage_<Project>(pg_session_C)
```
where the  `__init__` parameter `process_S`is a compiled struct (_S) of the JSON defined process and `pg_session_C` is the activated postgres session class (_C).

You don't have to guess where this goes — the framework's own shipped `src/example_project/import_data/import_data.py` ships a commented-out fossil line marking exactly this spot:

```python
#from src.postgres.pg_ai4sh import PG_manage_AI4SH
```

That's a leftover from how the shipped example itself was derived from a real project — replace it with your own import and instantiation line, following the shape above.

From then on, call your helpers through that attribute, passing the session explicitly, the same way the class's methods expect it: `self.pg_<project>_C._Retrieve_something(query_D, self.pg_session_C)`.

See [Postgres — summary of edits][building_postgres] for the complete list of files this pattern touches, alongside the process-registration pattern in [Adding a new process][building_processes].

[setup_db]: /setup_db/
[schemas_tables]: /setup_db/schemas_tables/
[building_auditing]: /building/auditing/
[building_processes]: /building/processes/
[building_postgres]: /building/postgres/
