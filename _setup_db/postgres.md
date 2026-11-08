---
title:  "PostgreSQL"
layout: single
sidebar:
  nav: "setup_db"
excerpt: "TPostgreSQL is a powerful, open-source relational database system used to store, manage, and query structured data reliably. It supports advanced features like complex queries, transactions, extensibility, and compliance with SQL standards. Known for its robustness and scalability, it’s widely used in applications ranging from small projects to large enterprise systems."
permalink: /setup_db/postgres/
author_profile: false
date:   2026-03-15 16:13:03 +0200
last_modified_at:   2026-11-08
---

[PostgreSQL][postgres] (or postgres for short) is a powerful, open-source relational database system used to store, manage, and query structured data reliably. It supports advanced features like complex queries, transactions, extensibility, and compliance with SQL standards. Known for its robustness and scalability, it’s widely used in applications ranging from small projects to large enterprise systems. Postgres forms the backbone of the framework system laid out on this site.

## Installing PostgreSQL

The installation of PostgreSQL depends on your operating system. You can download distributions from the [official download page][download_postgres]. For MacOS you can alternatively use [Homebrew][homebrew], also mentioned on the postgreSQL official download page for MacOS. For details you can consult Karttur's post on [Install postgreSQL and postGIS][karttur_postgres_macos]. Also for Linux (Ubuntu) you can use alternative installation, e.g. as outlined in Karttur's post [installations for Ubuntu][karttur_postgres_ubuntu].

[postgres]: https://www.postgresql.org

[download_postgres]: https://www.postgresql.org/download/

[karttur_postgres_macos]: https://karttur.github.io/setup-ide/setup-ide/install-postgres/

[karttur_postgres_ubuntu]: https://karttur.github.io/setup-ide/blog/ubuntu20-setup-spide/#postgresql

[homebrew]: https://brew.sh

## Securing authentication (pg_hba.conf)

`pg_hba.conf` decides how PostgreSQL authenticates each connection. If it says `trust`, the server accepts **any** password, and every credential in your `.netrc` is decoration. Whether you need to edit it depends on how PostgreSQL was installed:

| Install | Default for localhost TCP | Action |
|---|---|---|
| Homebrew (`brew install postgresql@17`) | `trust` | **Must edit** |
| Postgres.app (macOS) | `trust` | **Must edit** |
| EDB installer (Windows/macOS) | `scram-sha-256` | Check only |
| Debian/Ubuntu `apt` | `scram-sha-256` (TCP), `peer` (socket) | Check only |

### Procedure

1. **Check** (as the superuser):

   ```sql
   SELECT type, database, address, auth_method FROM pg_hba_file_rules;
   ```

   If any `host` row says `trust`, passwords are not checked at all.

2. **Set passwords first, then switch.** Set all role passwords before changing authentication, or you lock yourself out:
   - pg_users: add them to [`.netrc`][netrc] and run the setup cell of `setup_db.ipynb`;
   - superuser: `psql -U <superuser> -d postgres -c "\password <superuser>"`.

3. **Edit** `pg_hba.conf`. `SHOW hba_file;` gives the path (Homebrew: `/opt/homebrew/var/postgresql@17/pg_hba.conf`). Change **all four** `host` lines (`all` and `replication`, for both `127.0.0.1/32` and `::1/128`) to `scram-sha-256`:

   ```
   # TYPE  DATABASE        USER            ADDRESS                 METHOD
   local   all             all                                     trust
   host    all             all             127.0.0.1/32            scram-sha-256
   host    all             all             ::1/128                 scram-sha-256
   local   replication     all                                     trust
   host    replication     all             127.0.0.1/32            scram-sha-256
   host    replication     all             ::1/128                 scram-sha-256
   ```

   Leave the `local` lines as `trust`. They are your fallback over the Unix socket (`psql -U <superuser> -d postgres`, without `-h`) if something goes wrong.

4. **Reload**:

   ```bash
   psql -U <superuser> -d postgres -c "select pg_reload_conf()"
   ```

5. **Verify the reload happened.** This is easy to miss:

   ```sql
   select pg_conf_load_time();
   ```

   must show the current time. Then test that a *wrong* password is rejected over TCP:

   ```bash
   psql -h localhost -U user_cat_1 -d postgres   # type a wrong password: must fail
   ```

6. **Homebrew: put psql on PATH.** `postgresql@17` is keg-only, so `psql` is not on PATH. Add to `~/.zshrc`:

   ```bash
   export PATH="/opt/homebrew/opt/postgresql@17/bin:$PATH"
   ```

   The same is needed for `pg_dump`/`pg_restore` in [Dump and restore][dump_restore].

Also keep `listen_addresses = 'localhost'` in `postgresql.conf` unless you actually want remote access.

[netrc]: {{ "/setup_db/netrc/" | relative_url }}

[dump_restore]: {{ "/setup_db/dump_restore/" | relative_url }}
