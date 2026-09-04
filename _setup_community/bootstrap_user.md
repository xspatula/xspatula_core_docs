---
title: "Bootstrap User"
layout: single
sidebar:
  nav: "setup_community"
excerpt: "The very first community.user can't be created through the Excel-intake workflow, because that workflow requires an already-logged-in user. It still has to be hand-seeded — here's how, now that passwords are stored as bcrypt hashes."
permalink: /setup_community/bootstrap_user/
author_profile: false
date: 2026-08-15 08:00:00 +0200
last_modified_at: 2026-08-15 08:00:00 +0200
---

## The chicken-and-egg problem

Registering a new user through [Excel intake][setup_community_excel_intake] and [Register notebook][setup_community_register_notebook] runs through a registered process, `manage_user` — and running any process requires an already-logged-in `community.user`. That's fine for the second user onwards, but the *first* user can't log in before they exist. Someone has to be seeded directly.

That first-user seed still requires: hand-editing
`setup/zzz/xspatula/setup_db/json_core/community/user_records_v10_sql.json` and running `setup_db.ipynb`.

## Passwords are bcrypt hashes now, not plaintext

`community.user.password` are stored in the database as bcrypt hash, verified in Python after the row is fetched. That's safer, but it also means there's no way to just type a password directly into `user_records_v10_sql.json` any more — only a hash belongs in that column.

## Hashing a password by hand

Use the same `setup/hash_password.py` CLI the rest of the site's default-user setup uses — see [Hashing user passwords][setup_db_hash_passwords] on the Setup DB page for the full walkthrough. It prints a bcrypt hash you paste into the `password` value below, never the plaintext itself.

## Applying the hash

Two ways to get the hash into the database, depending on where you are in the setup lifecycle:

- **Fresh rebuild** — paste the hash into the `password` value inside `user_records_v10_sql.json`, then run `delete_db.ipynb` followed by `setup_db.ipynb`.
- **Live database, no rebuild** — run directly against the database:
  ```sql
  UPDATE community.user SET password = '<hash>' WHERE user_name = '...';
  ```

## Scope

This is a manual, admin-only escape hatch for bootstrapping — not something wired into the Excel intake, and not something an ordinary user ever touches. The Excel-based registration flow deliberately never sees or sets a plaintext password at all; every subsequent user gets one generated and emailed automatically, covered in [Welcome email][setup_community_welcome_email].

[setup_community_excel_intake]: /setup_community/excel_intake/
[setup_community_register_notebook]: /setup_community/register_notebook/
[setup_community_welcome_email]: /setup_community/welcome_email/
[setup_db_hash_passwords]: /setup_db/#hashing-user-passwords
