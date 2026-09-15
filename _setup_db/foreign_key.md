---
title: "Foreign key"
layout: single
sidebar:
  nav: "setup_db"
excerpt: "How a process parameter like territory_id__territory_name resolves a foreign key by name instead of by id."
permalink: /setup_db/foreign_key/
author_profile: false
date: 2026-09-15
last_modified_at: 2026-09-15
---

Reference page — not required reading to complete the core database setup. Explains how the framework resolves a foreign key given by name, using `territory_id__territory_name` from the shipped `community.organisation` insert as the example.

## How foreign keys normally resolve

A process parameter named `xxx_id__yyy` (e.g. `territory_id__territory_name`) is read as: write the result into the `xxx_id` column, and find it by searching the table literally named `xxx` for the value given. This works as long as a table named `xxx` actually exists.

`json/community/organisation_records_v10_sql.json` inserts the default `community.organisation` row using the column `territory_id__territory_name` rather than a raw `territory_id`. The framework strips the `_id` suffix from the parameter's prefix to get the table name — `territory` — and looks for a table with that exact name. It finds `utility.territory`, created by `json/utility/territory_v10_sql.json` with a unique `name` column, and matches the given value against it. The matching row's `id` is substituted into `territory_id` before the insert runs.

This direct lookup only works because a table literally named `territory` exists and has a matching unique column. Not every foreign key can rely on that — for parameters where no table matches the prefix, Xspatula falls back to a more advanced resolution via the `utility.foreign_key` table. See [Foreign key explained](https://xspatula.github.io/xspatula_lucas_docs/lucas_2009/foreign_key_explained/) for that mechanism.
