---
title: "Turning on auditing"
layout: single
sidebar:
  nav: "building"
excerpt: "Decide which of your new tables to audit while you define them, then run setup_db.ipynb's two cells as usual — the mechanism itself is already fully documented in the Auditing collection."
permalink: /building/auditing/
author_profile: false
date: 2026-08-24
last_modified_at: 2026-08-24
---

Auditing is opt-in, declared per table, and the full mechanism is documented in depth in the [Auditing][auditing_introduction] collection — this page is just the "where does this fit in while I'm building my project" version.

## The one decision: while defining a table

Back in [Setting up the database][building_database], as you write each table's `create_table` definition, add an `audit` key alongside its `parameters`:

```json
"audit": {
  "INSERT": true,
  "UPDATE": true,
  "DELETE": true
}
```

No key at all (or one with every value `false`) means that table simply isn't audited — there's no separate registration step. The rule of thumb, covered in full on the introduction page: full coverage for admin/config/catalogue tables you add one row at a time; `UPDATE`/`DELETE` for bulk pipeline-written data, where auditing every `INSERT` would double your write volume for little audit value.

## Turning it on

Nothing extra to run — `setup_db.ipynb`'s existing two cells handle it:

1. The main "Setup database" cell you're already running to create your schemas and tables also, as a side effect, assembles the audit-trigger configuration for every table that declared an `audit` key. This is pure file writing — no audit objects exist in the database yet.
2. The second, optional "Apply audit triggers" cell actually creates `audit.logged_actions`, the trigger function, and every table's trigger. Skip it and your database simply has no auditing; run it (safe to re-run any time) to switch it on.

Full detail — every column of `audit.logged_actions`, the self-audit gotcha, read-access grants, and copy-paste SQL for actually querying the log — is in [Auditing][auditing_introduction] and [Auditing queries][auditing_queries].

[auditing_introduction]: /auditing/
[auditing_queries]: /auditing/queries/
[building_database]: /building/database/
