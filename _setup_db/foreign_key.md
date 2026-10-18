---
title: "Foreign key"
layout: single
sidebar:
  nav: "setup_db"
excerpt: "How a process parameter like territory_id__territory_name resolves a foreign key by name instead of by id."
permalink: /setup_db/foreign_key/
author_profile: false
date: 2026-09-15
last_modified_at: 2026-10-18
---

Reference page — not required reading to complete the core database setup. Explains how the framework resolves a foreign key given by name, using `territory_id__territory_name` from the shipped `community.organisation` insert as the example.

## How foreign keys normally resolve

A process parameter named `xxx_id__yyy` (e.g. `territory_id__territory_name`) is read as: write the result into the `xxx_id` column, and find it by searching the table literally named `xxx` for the value given. This works as long as a table named `xxx` actually exists.

`json/community/organisation_records_v10_sql.json` inserts the default `community.organisation` row using the column `territory_id__territory_name` rather than a raw `territory_id`. The framework strips the `_id` suffix from the parameter's prefix to get the table name — `territory` — and looks for a table with that exact name. It finds `utility.territory`, created by `json/utility/territory_v10_sql.json` with a unique `name` column, and matches the given value against it. The matching row's `id` is substituted into `territory_id` before the insert runs.

This direct lookup only works because a table literally named `territory` exists and has a matching unique column. Not every foreign key can rely on that — for parameters where no table matches the prefix, Xspatula falls back to a more advanced resolution via the `utility.foreign_key` table. See [Foreign key explained](https://xspatula.github.io/xspatula_lucas_docs/lucas_2009/foreign_key_explained/) for that mechanism.

The fallback is used for **self references**: no table is named `parent_<table>`, so a parameter such as `parent_landcover_id__landcover_alias` can only be resolved through a row in `utility.foreign_key`:

| foreign_key | dst_schema | dst_table | dst_search_column | dst_alt_search_column |
|---|---|---|---|---|
| `parent_landcover_id` | `landscape_utility` | `landcover` | `alias` | `tag` |

The value is searched in `dst_search_column`, then in `dst_alt_search_column` (each also in lower case). The domain use, the ranked land classes, is described on the ai4sh [utility page](https://xspatula.github.io/xspatula_ai4sh_docs/setup_db/utility/#foreign_key).

## Empty and unresolved values

The rules below are applied to every `xxx_id__yyy` parameter before the insert (`_Convert_id_code` in `src/postgres/pg_common.py`).

| Value | Result |
|---|---|
| Empty (`''`, `none`, `null`) | id **0** |
| Empty in a `parent_*` key | the key is left out, the column is **NULL** |
| Not found | the record **stops** with an error |

- **Empty resolves to id 0, not NULL.** Catalogue tables carry a "none" row with id 0, e.g. `observation_utility.setting_system` has `0`, `no setting`. An empty value points to that row. This is deliberate: the foreign key is always filled, and "nothing" is an explicit record.
- **`parent_*` is the exception.** A self reference to a parent, e.g. `parent_landuse_id` of a top rank land use class, has no parent. The key is left out of the insert and the column stays NULL.
- **Not found stops the record** with

```
❌ ERROR Can not find foreign key value <value> for <parameter>
```

Check the spelling, or that the referenced record was inserted first. Before this check the unresolved key was passed on to the INSERT as a column name, and Postgres reported e.g. `column "parent_landuse_id__landuse_alias" ... does not exist`, which hid the real cause.

Auto naming uses the same convention but searches wider (name, tag, alias and other unique columns); see [Automatic naming and tags][auto_naming].

[auto_naming]: ../../setup_processes/auto_naming/
