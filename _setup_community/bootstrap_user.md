---
title: "Bootstrap User"
layout: single
sidebar:
  nav: "setup_community"
excerpt: "The very first community.user can't be created through the Excel-intake workflow, because that workflow requires an already-logged-in user. It is seeded at database setup from one .netrc entry, with the bcrypt hash computed automatically."
permalink: /setup_community/bootstrap_user/
author_profile: false
date: 2026-08-15 08:00:00 +0200
last_modified_at: 2026-11-08
---

## The chicken-and-egg problem

Registering a new user through [Excel intake][setup_community_excel_intake] and [Register notebook][setup_community_register_notebook] runs through a registered process, `manage_user` — and running any process requires an already-logged-in `community.user`. That's fine for the second user onwards, but the *first* user can't log in before they exist. Someone has to be seeded directly.

That first user is seeded at database setup through the `initial_user` mechanism: a `.netrc` entry, the placeholder `"{initial_user}"` as the bootstrap row's `password` in `setup/zzz/<project>/setup_db/json/community/user_records_v10_sql.json`, and an `initial_user` block in the setup scheme. The full walkthrough is [Setting up the initial user][building_initial_user].

## Passwords are bcrypt hashes, computed for you

`community.user.password` is stored as a bcrypt hash, verified in Python after the row is fetched. You never type a password or a hash into the seed file: at setup the framework reads the password from `.netrc`, hashes it in memory and fills the placeholder row.

## Getting the password into the database

- **Fresh database** — `initial_user` + placeholder, then run `setup_db.ipynb`. No hashing step.
- **Existing database** — the seed only inserts rows that don't exist yet, so editing `.netrc` and re-running setup does **not** change an existing user's password. Generate a hash with `setup/hash_password.py` and update the row directly:
  ```sql
  UPDATE community.user SET password = '<hash>' WHERE user_name = '...';
  ```
  then update `.netrc` to match. See [If the database already exists][building_initial_user_live].

## Scope

This is a manual, admin-only escape hatch for bootstrapping — not something wired into the Excel intake, and not something an ordinary user ever touches. The Excel-based registration flow deliberately never sees or sets a plaintext password at all; every subsequent user gets one generated and emailed automatically, covered in [Welcome email][setup_community_welcome_email].

[setup_community_excel_intake]: {{ "/setup_community/excel_intake/" | relative_url }}
[setup_community_register_notebook]: {{ "/setup_community/register_notebook/" | relative_url }}
[setup_community_welcome_email]: {{ "/setup_community/welcome_email/" | relative_url }}
[building_initial_user]: {{ "/building/initial_user/" | relative_url }}
[building_initial_user_live]: {{ "/building/initial_user/#if-the-database-already-exists" | relative_url }}
