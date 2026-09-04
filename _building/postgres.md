---
title: "Postgres — summary of edits"
layout: single
sidebar:
  nav: "building"
excerpt: "Every file and module touched when a project adds its own postgres functionality — a consolidated reference for the pattern covered in Setting up the database and Adding a new process."
permalink: /building/postgres/
author_profile: false
date: 2026-08-24
last_modified_at: 2026-08-24
---

[Setting up the database][building_database] and [Adding a new process][building_processes] each touch several files as part of a bigger pattern. This page pulls all of it into one table — a checklist for what to create or edit when a project adds postgres functionality of its own.

| File / module | What to do | Required? |
|---|---|---|
| `setup/zzz/<project>/setup_db/json_<project>/<schema>/` | Table definitions (`create_table` blocks, each with an optional `"audit"` key) for your project's own schemas | Required for any project-specific data |
| `setup/zzz/<project>/setup_db` pilot file | Add each new schema/table JSON file's path, in dependency order (schemas before the tables that reference them) | Required whenever a new file is added |
| `src/postgres/pg_<project>.py` | Define your own `PG_manage_<Project>` class — a bundle of custom SQL helper methods, each taking the session object as an explicit parameter | Optional — only if generic CRUD isn't enough |
| `setup/zzz/<project>/setup_processes/json_<project>/<domain>/` | Process registration JSON (`add_root_process` + `add_process` blocks) for each new root process | Required per new root process |
| `setup/zzz/<project>/setup_processes` pilot file | Add each new process registration file's path, root process before its sub-processes | Required whenever a new file is added |
| `src/<project>/process.py` | Add an `elif root_process == '<yours>':` branch dispatching to your handler class; if using a `PG_manage_<Project>` class, construct it (`self.pg_<project>_C = PG_manage_<Project>(pg_session_C)`) wherever it's needed | Required per new root process |
| `src/<project>/<handler>.py` | One new module per root process, defining a handler class with a `_Sub_process` method dispatching by exact process name | Required per new root process |

Two patterns run through this whole table, both covered in detail on the pages linked above: nothing here is automatic beyond one database lookup (a process name resolves to its `root_process` at runtime) — every dispatch step past that is a plain, hand-written `if`/`elif` chain; and the framework's own core (`src/postgres/`, `src/lib/`) is never edited by any of this — every addition lives in your project's own files, alongside the framework rather than inside it.

[building_database]: /building/database/
[building_processes]: /building/processes/
