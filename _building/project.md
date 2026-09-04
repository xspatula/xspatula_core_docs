---
title: "Naming a new project"
layout: single
sidebar:
  nav: "building"
excerpt: "Building a new Xspatula project means choosing two independent names — a project directory and a Python module — and creating the module every notebook will call into."
permalink: /building/
author_profile: false
date: 2026-08-24
last_modified_at: 2026-08-24
---

Everything documented so far on this site uses the framework's own shipped example, `project_example`. The Building pages walk through what changes when you build your own project instead — starting with naming it.

## Two independent names

Building a new project means choosing, and keeping straight, two separate names that have nothing to do with each other:

1. **The project directory** — wherever `project_path` in your [scheme file][scheme_file] points. This is purely a filesystem path. The framework never inspects it for anything except finding your job/process files.
2. **The Python module under `src/`** — `src/<project>/`, the package every notebook imports `Run_process` from.

The shipped example makes this distinction concrete on purpose: the directory is `project_example/`, but the Python module is `src/example_project/` — deliberately different words, so nothing about the framework's own behavior implies they have to match. It's easy to assume otherwise if the first real project you look at is one where they happen to coincide (a project named, say, `ai4sh` typically ends up with both a directory and a module called `ai4sh`) — but that's a coincidence of that project's choice, not a rule.

You can name your directory and your module the same thing if you like — most projects do, since it's less to keep track of — just don't treat it as required.

## What to create under `src/<project>/`

Mirror the shipped `src/example_project/` structure:

```
src/<project>/
├── __init__.py
├── version.py
├── process.py
└── import_data/
    ├── __init__.py
    ├── import_data.py
    ├── structure_json_for_db.py
    └── version.py
```

Copy `src/example_project/` wholesale to start, then rename the top-level directory. Almost nothing inside needs editing:

- **`__init__.py`** — needs no change. It's a generic relative-import re-export (`from .process import Run_process`), which is exactly what makes `from src.<project> import Run_process` work at the package level once you rename the directory.
- **`version.py`** — cosmetic metadata, harmless to leave as-is.
- **`import_data/__init__.py`, `import_data/import_data.py`, `import_data/structure_json_for_db.py`** — copy unchanged to start; you'll extend these as your project grows.
- **`process.py`** — the one file with a required edit, covered next.

## The one required edit

Inside `src/<project>/process.py`, find the import line that pulls in the generic import/insert handler:

```python
from src.example_project.import_data import Process_import_JSON
```

Change it to point at your own module:

```python
from src.<project>.import_data import Process_import_JSON
```

That's the entire renaming edit inside `process.py` itself. Nothing else in `Run_process`'s signature, docstring, or dispatch logic needs to change — you'll come back to this same file in [Adding a new process][building_processes] once you start registering processes beyond the generic translate/insert ones.

## Everything this phase touches, at a glance

| What | Why |
|---|---|
| `src/<project>/` (new directory) | Your project's Python entry point — see above |
| `src/<project>/process.py` | One import line to edit — see above |
| Every notebook calling `Run_process` | Each one imports from the old module name — see [Editing the notebooks][building_notebooks] |
| Scheme file `project_path` | Just a directory path — no change required unless you also want to rename the directory |

[scheme_file]: /framework/scheme_file/
[building_notebooks]: /building/notebooks/
[building_processes]: /building/processes/
