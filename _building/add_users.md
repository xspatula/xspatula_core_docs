---
title: "Adding organisations and users via Excel"
layout: single
sidebar:
  nav: "building"
excerpt: "Once your project's initial user exists and SMTP is configured, ordinary organisations and users can register by filling in a spreadsheet — no login, no JSON, no direct database access required."
permalink: /building/add_users/
author_profile: false
date: 2026-08-24
last_modified_at: 2026-08-24
---

With your [initial user][building_initial_user] seeded and [SMTP][building_smtp] configured, you can open up registration to anyone else without hand-editing JSON on their behalf or handing out superuser access. This is the same Excel-intake workflow documented in full in [Setup Community][setup_community] — this page is the "doing it for your own project" version.

## The workflow

1. **Someone fills in a spreadsheet.** An organisation, or a user, fills in one row of `organisation.xlsx` or `user.xlsx` under your project's `user_management/` directory and gets it to you however's convenient — email, a shared drive. That handoff is outside the framework's scope.
2. **You review it before running anything.** Open the spreadsheet directly and delete any rows you don't want to accept. There's no in-notebook accept/reject step — curation happens in the spreadsheet, not in code.
3. **You run the registration notebook**, cell by cell, top to bottom. It translates both spreadsheets, generates and hashes a password for each accepted user, emails it to them once, and inserts everything.

There's no password column in `user.xlsx` — that's deliberate. A password is generated automatically per accepted row and emailed once; nobody chooses or sees it in advance, including you.

## Setting it up for your own project

The generic pipeline (`translate_tabular_data` → `manage_organisation`/`manage_user`) is already registered for the shipped example project; for your own project you register the same two processes the same way any process is registered — see [Define and register a process][setup_processes_define] — pointing `schema_table` at your own `community.organisation`/`community.user` tables. Then build your own `user_management/organisation/` and `user_management/user/` directory trees (spreadsheet, job file, pilot file, process definition) mirroring the shipped example's shape, and your own copy of `register_users.ipynb` (copied from the shipped one, with the `Run_process` import fixed per [Editing the notebooks][building_notebooks]).

**Before putting real people's data in these spreadsheets**, add your project's `user_management/*/excel/` paths to your own `.gitignore` — the framework doesn't exclude them by default, and a filled-in sheet contains real personal data that has no business being committed to a repository.

## More detail

Every column both spreadsheets need, the exact notebook steps, the `..._id__..._name` lookup-by-name convention, and troubleshooting a stale translate step are all documented in full in [Excel intake][setup_community_excel_intake] and [Register notebook][setup_community_register_notebook].

[building_initial_user]: /building/initial_user/
[building_smtp]: /building/smtp/
[building_notebooks]: /building/notebooks/
[setup_community]: /setup_community/
[setup_community_excel_intake]: /setup_community/excel_intake/
[setup_community_register_notebook]: /setup_community/register_notebook/
[setup_processes_define]: /setup_processes/define_process/
