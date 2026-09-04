---
title: "Register Notebook"
layout: single
sidebar:
  nav: "setup_community"
excerpt: "The admin's checkpoint: review the spreadsheets, then run register_users.ipynb cell by cell to translate and insert the accepted organisations and users."
permalink: /setup_community/register_notebook/
author_profile: false
date: 2026-08-15 08:00:00 +0200
last_modified_at: 2026-08-15 08:00:00 +0200
---

## The workflow

1. Someone — a workshop participant, a colleague — fills in a row in `organisation.xlsx` and/or `user.xlsx` (see [Excel intake][setup_community_excel_intake]) and gets it to the admin somehow: email, a shared drive, whatever's convenient. That handoff is outside the scope of the framework.
2. **The admin reviews the spreadsheet directly and deletes any rows they don't want to accept, before running anything.** There is no in-notebook accept/reject step — curation happens in Excel, not in code.
3. The admin runs `project_example/user_management/register_users.ipynb`, cell by cell, top to bottom.

## What the notebook does

The notebook mirrors the same translate-then-manage pattern documented in [Add user data][user_data], applied to organisations and users:

1. **Translate `organisation.xlsx`** — `translate_tabular_data` reads the spreadsheet and generates `user_management/organisation/manage_process/manage_organisation.json`, one process block per accepted row.
2. **Insert organisations** — runs that generated file through the registered `manage_organisation` process, inserting each row into `community.organisation`.
3. **Translate `user.xlsx`** — generates `user_management/user/manage_process/manage_user.json`. No password parameter yet — that's the next step.
4. **Generate, hash and email a password for each user** — `Provision_user_passwords` (from `src.community`) fills in the missing password: for each row, it generates a random password, stores its bcrypt hash back into the JSON file (so the next step can insert it like any other field), and emails the plaintext password once to that row's address (see [SMTP email][setup_community_smtp_email] and [Welcome email][setup_community_welcome_email]). It prints one summary line per row, e.g. `{'user_name': ..., 'email': ..., 'emailed': True}` — check for any `emailed: False` and follow up manually; that row's account is still created with a working password hash, it just didn't reach the user by email.
5. **Insert users** — runs the password-populated file through the registered `manage_user` process, inserting each row into `community.user`.

Because this runs through the normal login flow (not the raw-superuser `setup_db.ipynb`/`delete_db.ipynb` path), every insert it makes shows up in `audit.logged_actions` with `changed_by_user_id` populated as the admin's own `community.user.id` — see the auditing collection's [Auditing][auditing_introduction] page for what that column means and why it's `NULL` for setup-time changes but populated here.

## Troubleshooting: re-running a translate step

`translate_tabular_data` runs with `overwrite: false` by default, so re-running step 1 or 3 after the target JSON file already exists is a no-op — it won't pick up corrections made to the spreadsheet. If a spreadsheet was corrected and needs re-translating, delete the stale `manage_process/*.json` file first, then re-run that translate cell. 

[setup_community_excel_intake]: /setup_community/excel_intake/
[setup_community_smtp_email]: /setup_community/smtp_email/
[setup_community_welcome_email]: /setup_community/welcome_email/
[user_data]: /user_data/
[auditing_introduction]: /auditing/
