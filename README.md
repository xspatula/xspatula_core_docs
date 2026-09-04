# xspatula_core_docs

Core documentation site for the Xspatula framework — the generic framework and database-setup docs shared by every Xspatula project, so project-specific doc sites (e.g. `xspatula_ai4sh_docs`, `xspatula_lucas_docs`) can link back here instead of repeating them. Originally copied from [setup_core_db_docs](https://github.com/xspatula/setup_core_db_docs), the docs for the [setup_core_db](https://github.com/xspatula/setup_core_db) Python package, and extended with generic content pulled from `xspatula_ai4sh_docs`.

**Live site**: [xspatula.github.io/xspatula_core_docs](https://xspatula.github.io/xspatula_core_docs)

---

## What is Xspatula?

Xspatula is a Python framework for building database-integrated modelling workflows. All execution logic lives in JSON files, not in code. You define schemes, jobs, pilots, and processes in JSON; a Jupyter notebook calls the framework; the framework reads the JSON and runs the pipeline.

This separation means you can:

- swap datasets without touching Python
- version-control experimental configurations separately from the codebase
- hand workflows to non-programmers who edit JSON rather than code

The framework uses **PostgreSQL** as its backbone. Every result, parameter set, and process definition is stored in the database, giving you full audit trails and making it easy to query what ran, when, and with what inputs.

---

## The Python package: `setup_core_db`

The sibling repository [xspatula/setup_core_db](https://github.com/xspatula/setup_core_db) contains the framework source and the Jupyter notebooks used to drive it.

### Key components

| Path | Purpose |
|---|---|
| `src/lib/` | Core library: database initiation, login, pilot execution, structure definition |
| `src/utils/` | Utilities: JSON I/O, logging, date/time helpers, pretty-printing |
| `setup/setup_db.ipynb` | Notebook for creating a new PostgreSQL database |
| `setup/delete_db.ipynb` | Notebook for deleting a database or parts of it |
| `setup/setup_processes.ipynb` | Notebook for registering processes in the database |

### How it works

Every run starts with a **scheme file** (JSON) that defines:

- the PostgreSQL host, port, and database name
- superuser credentials (inline or via `.netrc`)
- the PostgreSQL users (`pg_users`) and their access categories
- a pointer to the **job file** for the current run

The **job file** links to a **pilot file** (a plain-text list of process files to run). Each **process file** is a JSON array of operations — for example, creating a schema, inserting a table, or populating seed records.

### Security model

The framework generates `.env` files for eight built-in PostgreSQL user categories:

| Role | Purpose |
|---|---|
| `community_admin` | Manage users and organisations |
| `login_evaluation` | Validate login attempts (minimal rights) |
| `user_cat_0` – `user_cat_5` | Data access, from most restricted (0) to most permissive (5) |

Credentials can be stored in a `.netrc` file rather than in plain text in the scheme file.

### Default database schemas

A freshly seeded Xspatula database contains four schemas:

- **utility** — territory codes and shared lookup tables
- **community** — organisations, users, and user categories
- **process** — root processes, processes, parameters, defaults, and permissions
- **observation_utility** / **observation** — domain-specific data (populated by the user's own processes)

---

## Documentation site

This repository is the source for the documentation site, built with [Jekyll](https://jekyllrb.com) and the [Minimal Mistakes](https://mmistakes.github.io/minimal-mistakes/) theme (v4.27.3, local install).

### Content sections

| Section | URL | Covers |
|---|---|---|
| Framework | `/framework/` | Scheme files, job files, pilot files, process files, notebook interface, VS Code setup |
| Database setup | `/setup_db/` | PostgreSQL installation, Anaconda environment, `.netrc` credentials, schemas and tables |
| Setup processes | `/setup_processes/` | Defining and registering processes in the database |
| Add user data | `/user_data/` | Translating and inserting user tabular (Excel) data via JSON conversion |
| Auditing | `/auditing/` | The native PostgreSQL trigger-based audit log: setup and queries |
| Community | `/setup_community/` | Registering organisations and users via Excel intake, SMTP email |
| Building | `/building/` | Building a new Xspatula project from scratch |

### Serving locally

```bash
bundle exec jekyll serve --config _config.yml,_config_local.yml
```

### Repository structure

```
xspatula_core_docs/
├── _config.yml          # production config
├── _config_local.yml    # local dev override (empty URL/baseurl)
├── _framework/          # Framework documentation pages
├── _setup_db/           # Database setup documentation pages
├── _setup_processes/    # Process setup documentation pages
├── _user_data/          # User data loading documentation pages
├── _auditing/           # Auditing documentation pages
├── _setup_community/    # Community (organisations/users) documentation pages
├── _building/           # Building-a-new-project documentation pages
├── _data/
│   └── navigation.yml   # site navigation
├── assets/media/        # logos and schema diagrams
└── _site/               # generated output (do not edit)
```

---

## Licenses

- **Code**: [MIT License](https://github.com/xspatula/setup_core_db/blob/main/LICENSE)
- **Data**: [Creative Commons Attribution (CC-BY)](https://creativecommons.org/licenses/by/4.0/)
