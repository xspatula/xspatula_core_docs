---
title: "Auditing Queries"
layout: single
sidebar:
  nav: "auditing"
excerpt: "Copy-paste SQL for reading the audit log in any SQL GUI — no Python, no notebook required."
permalink: /auditing/queries/
author_profile: false
date: 2026-08-15 08:00:00 +0200
last_modified_at: 2026-08-15 08:00:00 +0200
---

Everything on this page is plain SQL you can paste directly into any SQL editor, connected as a role that can read the `audit` schema (see [Auditing][auditing_introduction] for the `login_evaluation` grant). No Python, no notebook required — the framework's own example database is named `xspatula`; your own project's database will have whatever name you gave it.

If you don't already have a SQL client, [DBeaver][dbeaver] (free) is a common choice: install it, create a new PostgreSQL connection using the `login_evaluation` credentials (or any other role that can read the `audit` schema), open a SQL editor against your database, and paste in any query below.

**Prerequisite**: these queries assume the "Apply audit triggers" notebook cell has already been run against this database — see [Auditing setup][auditing_setup]. Otherwise `audit.logged_actions` doesn't exist yet and every query below will error.

## 1. What just happened — most recent changes first

```sql
SELECT schema_name, table_name, op, changed_by, changed_by_user_id, changed_at
FROM audit.logged_actions
ORDER BY changed_at DESC
LIMIT 50;
```

## 2. Everything for one table

Filter by both `schema_name` and `table_name` — table names aren't unique across schemas:

```sql
SELECT id, op, old_data, new_data, changed_by, changed_by_user_id, changed_at
FROM audit.logged_actions
WHERE schema_name = 'utility' AND table_name = 'territory'
ORDER BY changed_at DESC;
```

## 3. Only real changes, not no-op updates

```sql
SELECT id, op, old_data, new_data, changed_by, changed_at
FROM audit.logged_actions
WHERE schema_name = 'community' AND table_name = 'user'
  AND old_data IS DISTINCT FROM new_data
ORDER BY changed_at DESC;
```

Why `IS DISTINCT FROM` and not `!=`: `!=` returns `NULL` (which excludes the row) whenever either side is `NULL`. That would silently drop every `INSERT` (`old_data IS NULL`) and every `DELETE` (`new_data IS NULL`) from the results. `IS DISTINCT FROM` treats `NULL` as a real, comparable value, so inserts and deletes correctly count as "changed."

## 4. What specifically changed, not the whole row

Only makes sense for `op = 'U'` — an insert or delete has one side entirely `NULL`, so there's no meaningful field-by-field diff:

```sql
SELECT id, op, changed_at,
       (SELECT jsonb_object_agg(key, new_data->key)
        FROM jsonb_each(new_data)
        WHERE new_data->key IS DISTINCT FROM old_data->key) AS changed_fields
FROM audit.logged_actions
WHERE schema_name = 'utility' AND table_name = 'territory' AND op = 'U'
ORDER BY changed_at DESC;
```

## 5. Who did it — the individual, not just the role

`changed_by` only identifies the shared login role (e.g. `user_cat_2`). To find the actual person, join `changed_by_user_id` back to `community.user`:

```sql
SELECT a.schema_name, a.table_name, a.op, a.changed_at,
       u.first_name, u.last_name, u.email
FROM audit.logged_actions a
LEFT JOIN community.user u ON u.id = a.changed_by_user_id
WHERE a.changed_at > now() - interval '7 days'
ORDER BY a.changed_at DESC;
```

`changed_by_user_id` — and so `u.*` here — will be `NULL` for anything run outside a real login (e.g. through `setup_db.ipynb`/`delete_db.ipynb`, which connect as the raw superuser). That's expected, not missing data.

## 6. Everything from one transaction

Useful after finding one interesting row, to see what else happened alongside it in the same commit:

```sql
SELECT schema_name, table_name, op, changed_at
FROM audit.logged_actions
WHERE txid = 123456
ORDER BY id;
```

## 7. What won't show up here

Three things are missing from the log **by design**, not by accident:

- `INSERT`s on any table you've deliberately configured as `UPDATE`/`DELETE`-only, see [Auditing][auditing_introduction] — relevant once your project adds bulk pipeline-written tables of its own; the framework's own default install has no table configured this way.
- Anything on `audit.logged_actions` itself except `UPDATE`/`DELETE` — it audits itself, but never its own `INSERT` (the self-audit recursion gotcha, also covered on the previous page).
- Anything at all on a table with no `"audit"` key — it simply isn't audited, see [Auditing setup][auditing_setup].

## 8. What's currently covered

Coverage is declared per table and changes as tables are added — don't trust the snapshot table on the introduction page for a database you're actually working with; check it directly:

```sql
SELECT event_object_schema, event_object_table, string_agg(event_manipulation, ', ')
FROM information_schema.triggers
WHERE trigger_name LIKE '%_audit'
GROUP BY 1, 2
ORDER BY 1, 2;
```

[auditing_introduction]: /auditing/
[auditing_setup]: /auditing/setup/
[dbeaver]: https://dbeaver.io
