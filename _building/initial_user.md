---
title: "Setting up the initial user"
layout: single
sidebar:
  nav: "building"
excerpt: "Before you can register any process for your new project, you need at least one working login — and community.user.password stores a bcrypt hash, so it has to be generated with a small CLI before it goes anywhere near the JSON file."
permalink: /building/initial_user/
author_profile: false
date: 2026-08-24
last_modified_at: 2026-08-24
---

Registering processes for your new project — the JSON registration in [Adding a new process][building_processes], and inserting through any registered process at all — requires an already-logged-in `community.user`. There's no user until you seed one, and `community.user.password` stores a bcrypt hash rather than plain text, so that hash has to be generated *before* you write it into the seed file. This page walks through doing that from scratch.

## Step 1: open a terminal

Open a terminal window and navigate to your local copy of the framework — the directory containing the `setup/` folder. If you're using an Anaconda environment, activate it first:

```bash
cd /path/to/setup_core_db
conda activate <your-environment-name>
```

## Step 2: run the hashing CLI

Run:

```bash
python3 setup/hash_password.py
```

You'll be prompted twice, with hidden input (nothing appears on screen as you type — that's expected):

```
Password to hash:
Repeat password:
```

Type the plain-text password you want your first user to log in with, and repeat it. If the two don't match, the script exits with an error and prints nothing — just run it again.

## Step 3: copy the printed hash

On success, the tool prints a single line — a bcrypt hash that looks like this (yours will be different every time, even for the same password):

```
$2b$12$KIXQ7z3nJ8mR5vT1wYbP9uL6oE2cN4sD0hG7fA3xZ8qW1yV6tR9mS
```

Select and copy that entire string. This is what goes into the JSON seed file — never the plain-text password itself.

You can also pass the password directly as a command-line argument (`python3 setup/hash_password.py 'some-password'`), which skips the prompts, but avoid this on a shared machine — the plain-text password then briefly appears in your shell history and process list.

## Step 4: paste the hash into your seed file

Open your project's copy of the user-seed file — for the shipped example this is `setup/zzz/xspatula/setup_db/json_core/community/user_records_v10_sql.json`; for your own project it's the equivalent file under your own `json_<project>/community/` — and paste the hash in as the `password` value for your first user:

```json
"password": "$2b$12$KIXQ7z3nJ8mR5vT1wYbP9uL6oE2cN4sD0hG7fA3xZ8qW1yV6tR9mS"
```

Repeat steps 2-4 once per default user you're seeding — each needs its own freshly-generated hash.

## Step 5: run setup_db.ipynb

With the hash (or hashes) in place, run `setup_db.ipynb`'s main "Setup database" cell as normal — this is the same run that creates your schemas and tables from [Setting up the database][building_database]. Your seeded user now exists with a working, hashed password, and can log in to register the processes covered in [Adding a new process][building_processes].

## If the database already exists

If you're adding a first user to a database you've already set up (rather than seeding it at creation), skip the JSON file and run the hash straight against the live database instead:

```sql
UPDATE community.user SET password = '<hash>' WHERE user_name = '...';
```

## More detail

This is the same CLI and mechanism documented in full on [Hashing user passwords][setup_db_hash_passwords] (Setup DB) and [Bootstrap user][setup_community_bootstrap_user] (Setup Community) — the latter covers the same "chicken-and-egg" problem in the specific context of registering later users through the Excel-intake workflow once this first one exists.

[building_processes]: /building/processes/
[building_database]: /building/database/
[setup_db_hash_passwords]: /setup_db/#hashing-user-passwords
[setup_community_bootstrap_user]: /setup_community/bootstrap_user/
