---
title:  "PostgreSQL"
layout: single
sidebar:
  nav: "setup_db"
excerpt: "TPostgreSQL is a powerful, open-source relational database system used to store, manage, and query structured data reliably. It supports advanced features like complex queries, transactions, extensibility, and compliance with SQL standards. Known for its robustness and scalability, it’s widely used in applications ranging from small projects to large enterprise systems."
permalink: /setup_db/postgres/
author_profile: false
date:   2026-03-15 16:13:03 +0200
last_modified_at:   2026-03-29 21:01:15 +0200
---

[PostgreSQL][postgres] (or postgres for short) is a powerful, open-source relational database system used to store, manage, and query structured data reliably. It supports advanced features like complex queries, transactions, extensibility, and compliance with SQL standards. Known for its robustness and scalability, it’s widely used in applications ranging from small projects to large enterprise systems. Postgres forms the backbone of the framework system laid out on this site.

## Installing PostgreSQL

The installation of PostgreSQL depends on your operating system. You can download distributions from the [official download page][download_postgres]. For MacOS you can alternatively use [Homebrew][homebrew], also mentioned on the postgreSQL official download page for MacOS. For details you can consult Karttur's post on [Install postgreSQL and postGIS][karttur_postgres_macos]. Also for Linux (Ubuntu) you can use alternative installation, e.g. as outlined in Karttur's post [installations for Ubuntu][karttur_postgres_ubuntu].

[postgres]: https://www.postgresql.org

[download_postgres]: https://www.postgresql.org/download/

[karttur_postgres_macos]: https://karttur.github.io/setup-ide/setup-ide/install-postgres/

[karttur_postgres_ubuntu]: https://karttur.github.io/setup-ide/blog/ubuntu20-setup-spide/#postgresql

[homebrew]: https://brew.sh
