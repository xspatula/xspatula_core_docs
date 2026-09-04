---
title: "Editing the notebooks"
layout: single
sidebar:
  nav: "building"
excerpt: "Every notebook that calls Run_process imports it from the old project module by name — find each one and change the import line."
permalink: /building/notebooks/
author_profile: false
date: 2026-08-24
last_modified_at: 2026-08-24
---

Once your own `src/<project>/` module exists (see [Naming a new project][building_project]), every notebook that runs a process still imports `Run_process` from the *old* module name. Each notebook needs the same one-line fix.

## The edit

Open a notebook in Jupyter or VS Code, find the cell that reads:

```python
from src.example_project import Run_process
```

Click into the cell, edit the text to your own module name:

```python
from src.<project> import Run_process
```

then run the cell (Shift+Enter, or the ▶ button next to it) so the change actually takes effect for the rest of that notebook session. Repeat for every other cell in the notebook that has the same import — there's usually only one, near the top.

## Which notebooks need this

In the shipped `project_example`, exactly three notebooks import `Run_process` this way:

| Notebook | Covered by |
|---|---|
| `project_example/import_data/load_utility_data.ipynb` | [Add user data][user_data] (two-step route) |
| `project_example/import_data/insert_utility_data.ipynb` | [Single-step insert][single_step_insert] |
| `project_example/user_management/register_users.ipynb` | [Setup Community][setup_community] |

A real project scales up from here — every notebook you add that drives a process (loading a new kind of data, running a new analysis) needs the same import line pointed at your module. There's no shortcut that avoids editing each one individually; the import happens once per notebook, not once centrally, because `Initiate_process` (the framework function that reads your scheme/job files) is deliberately generic and has no idea which project module you're using — it just hands back data for whatever `Run_process` you import to consume.

## A word of caution on copied notebooks

If you start a new notebook by copying one from another project (including the shipped examples, whose own markdown cells and comments haven't all been fully generalized), check more than just the import line — leftover text elsewhere in the notebook (markdown explanations, code comments) may still reference the project it was copied from. Only the import line actually affects behavior, but stale prose is confusing to come back to later.

[building_project]: /building/
[user_data]: /user_data/
[single_step_insert]: /user_data/single_step_insert/
[setup_community]: /setup_community/
