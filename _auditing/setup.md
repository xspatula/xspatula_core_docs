---
title: "Auditing Setup"
layout: single
sidebar:
  nav: "auditing"
excerpt: "Auditing is opt-in per table, declared with an inline \"audit\" key, and turned on with a second, optional notebook cell — separate from creating the database itself."
permalink: /auditing/setup/
author_profile: false
date: 2026-08-21 08:00:00 +0200
last_modified_at: 2026-08-21 08:00:00 +0200
---

A table declares its own audit coverage inline, and turning auditing on for a database is a
separate, optional step from creating the database at all.

## Declaring coverage on a table

Every table's `create_table` definition (under `setup/zzz/xspatula/setup_db/json_core/...`) may
carry a sibling `"audit"` key:

```json
"audit": {
  "INSERT": true,
  "UPDATE": true,
  "DELETE": true
}
```

No `"audit"` key at all — or one where every value is `false` — means that table simply isn't
audited. There's no implicit fallback: if you want a table audited, its own file has to say
so. See [Auditing introduction][auditing_introduction] for the two-tier convention (full
coverage for admin/config/catalogue tables, `UPDATE`/`DELETE`-only for bulk pipeline data) that
decides which values to set.

This is the **one and only thing you ever hand-edit**. Everything described below is generated
from it automatically.

## Turning auditing on: the two-cell workflow

`setup_db.ipynb` has two relevant cells, and they do genuinely different things:

1. **"Setup database"** — the cell you already run to create/update every schema and table.
   As a side effect, it scans every table's `"audit"` key across the whole pilot list
   and (re)writes the per-schema `audit_triggers_<schema>_v10_sql.json` files, plus a
   generated pilot file (its own header says "do not hand-edit" — treat it the same way
   you'd treat any other build artifact). This step is **pure file assembly** — it does not
   touch the database, create any audit objects, or need a live Postgres connection. After
   running only this cell, no audit triggers exist yet.

   The generated pilot file's name is an opaque, generated string, not something to rely on
   or hand-edit — and worth calling out explicitly: as of today it's literally
   `db_xspatula_ai4sh_audit.txt` regardless of what project you're running this in. That's a
   naming quirk in an internal constant in the framework code, not a per-project setting —
   don't expect it to match your own project's name, and don't be surprised to see "ai4sh" in
   it on an unrelated project. The file works identically either way.
2. **"Apply audit triggers"** (`job_setup_audit.json`) — a second, separate, **optional** cell.
   This is what actually creates `audit.logged_actions`, `audit.if_modified_func()`, and every
   table's trigger in the database, by calling `Initiate_audit()`. Skip it and the database
   simply has zero audit triggers — there's no longer an implicit "auditing is just always
   there" guarantee.

Run cell 1, then cell 2, in that order. Cell 2 is safe to re-run any time — immediately,
later, or repeatedly — because it always re-applies the *complete* current config rather than
only whatever changed. It can't be broken by a table having been added out of order in the
pilot list, which is the exact class of bug the old hand-maintained design was vulnerable to.

**Adding auditing to a new table later**: set its `"audit"` key, then re-run cell 1 (to
regenerate the trigger config files) followed by cell 2 (to apply it). This is a
one-time-per-database-refresh action — you don't need to think about it again until you add
another audited table.

The very first time cell 2 runs against a database with no audit system yet, it bootstraps
`audit.logged_actions` and `audit.if_modified_func()` — you'll be prompted (a plain y/n
question) to confirm setting up an audit system for that database. If those objects already
exist, cell 2 skips straight to applying the current trigger config.

## Config files: shipped vs. generated

Everything lives under `setup/zzz/xspatula/setup_db/json_core/audit/`, but the files there fall
into two very different categories.

**Shipped with the framework** — these 3 files come with xspatula itself and don't change per
project or per table:

| File | Defines |
|---|---|
| `audit_table_v10_sql.json` | The `audit.logged_actions` table |
| `audit_function_v10_sql.json` | The `audit.if_modified_func()` trigger function |
| `audit_triggers_audit_v10_sql.json` | The `audit` schema's own self-audit trigger (see the self-audit gotcha above) |

**Auto-generated on every "Setup database" run** — one file per audited schema, written fresh
each time from every table's `"audit"` key. For the framework's own default install today
that's 3 files, one per default schema:

| File | Schema |
|---|---|
| `audit_triggers_community_v10_sql.json` | `community` |
| `audit_triggers_process_v10_sql.json` | `process` |
| `audit_triggers_utility_v10_sql.json` | `utility` |

A project that adds its own schemas gets one more generated file per audited schema it adds.
Plus, at the pilot-file level: the generated pilot file cell 2 runs (see the naming note
above). All the generated files (and the pilot file) are build artifacts. If you edit one by
hand, the next "Setup database" run will overwrite your edit — change the source table's
`"audit"` key instead.

## Checking what's actually covered

Since coverage is now declared per table rather than fixed in a checked-in file, don't trust a
number written into documentation (including the snapshot table on the [introduction
page][auditing_introduction]) — check the database you're actually looking at:

```sql
SELECT event_object_schema, event_object_table, string_agg(event_manipulation, ', ')
FROM information_schema.triggers
WHERE trigger_name LIKE '%_audit'
GROUP BY 1, 2
ORDER BY 1, 2;
```

A dry-run script, `setup/scripts/verify_audit_system.py`, does the equivalent check without
touching the database — useful if you want to confirm the assembled trigger-config files match
every table's declared `"audit"` key before applying anything. Run it from the `setup/`
directory, passing your own scheme file explicitly (its own built-in default points at a file
that won't exist in a fresh project, so don't rely on running it with no arguments):

```
python scripts/verify_audit_system.py zzz/scheme_xspatula_local_setup.json
```

## Input files

| File | Purpose |
|---|---|
| `audit_table_v10_sql.json` | Shipped; defines the `audit.logged_actions` table |
| `audit_function_v10_sql.json` | Shipped; defines the `audit.if_modified_func()` trigger function |
| `audit_triggers_audit_v10_sql.json` | Shipped; the `audit` schema's own self-audit trigger |
| `audit_triggers_<schema>_v10_sql.json` | Generated; one per audited schema — do not hand-edit |
| generated pilot file (cell 2's manifest) | Generated; lists every audit config file cell 2 should apply — do not hand-edit |
| `setup/scripts/verify_audit_system.py` | Optional dry-run check; confirms the assembled config matches every table's `"audit"` key without touching the database |

[auditing_introduction]: /auditing/
