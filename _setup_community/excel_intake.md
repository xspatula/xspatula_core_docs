---
title: "Excel Intake"
layout: single
sidebar:
  nav: "setup_community"
excerpt: "Organisations and users are registered by filling in one of two Excel spreadsheets — organisation.xlsx and user.xlsx. Column headers must match exactly; the framework does not validate them."
permalink: /setup_community/excel_intake/
author_profile: false
date: 2026-08-15 08:00:00 +0200
last_modified_at: 2026-08-15 08:00:00 +0200
---

Two spreadsheets cover the whole intake: one row per organisation, one row per user. Both feed the generic `translate_tabular_data` → `manage_*` pipeline already used elsewhere on this site (see [Add user data][user_data] for another worked example of the same pattern).

**Column headers must match exactly, case-sensitive.** The framework does not validate headers against the target table — a wrong or misspelled header fails silently or errors late in the pipeline rather than up front, so get this right before filling in real data.

## organisation.xlsx

Path: `project_example/user_management/organisation/excel/organisation.xlsx`

Columns:

```
name, alias, url, address1, address2, postal_address, postal_zip_code, state,
territory_id__territory_name, telephone, contact_name, contact_email
```

## user.xlsx

Path: `project_example/user_management/user/excel/user.xlsx`

Columns:

```
organisation_id__organisation_name, user_name, first_name, middle_name, last_name,
email, email_alt, address1, address2, postal_address, postal_zip_code, state,
territory_id__territory_name, telephone, department, section, position
```

**There is no password column.** That's deliberate — see [Welcome email][setup_community_welcome_email]. A password is generated automatically for each accepted row and emailed once to that user; nobody chooses or is shown it in advance.

## The `..._id__..._name` columns

`organisation_id__organisation_name` and `territory_id__territory_name` use the framework's existing lookup-by-name convention: instead of requiring the spreadsheet author to know an internal database id, the translate step resolves the given name to the matching row's id at insert time. This isn't new to community registration — the same convention is used throughout this site wherever a foreign key is filled in by name rather than by id.

## Where the files live, and whether they're gitignored

```
project_example/user_management/organisation/excel/
project_example/user_management/user/excel/
```

These directories are **not** excluded by the framework's own `.gitignore` — unlike the database and SMTP credential files, which are. A filled-in sheet contains real people's personal data, which has no business being committed to a repository, so add `project_example/user_management/*/excel/` (or the equivalent path for your own project) to your project's own `.gitignore` before putting real data in these spreadsheets. A blank spreadsheet gets created by hand the first time, or copied from an existing project once one exists.

[user_data]: /user_data/
[setup_community_welcome_email]: /setup_community/welcome_email/
