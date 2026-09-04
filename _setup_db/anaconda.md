---
title:  "Anaconda - virtual python environments"
layout: single
sidebar:
  nav: "setup_db"
excerpt: "Anaconda is an application that allows setting up virtual Python environments. You define the Python version and packages to include with each virtual environment and then Anaconda creates a self-contained Python environment for you."
permalink: /setup_db/anaconda/
author_profile: false
date:   2026-03-15 16:13:03 +0200
last_modified_at:   2026-03-29 20:13:03 +0200
---

Anaconda is an application that allows setting up virtual Python environments. You define the Python version and packages to include with each virtual environment and then Anaconda creates a self-contained Python environment for you.

## Prerequisits

To setup an Anaconda (conda for short) environment you need to [download][download_anaconda] the distribution for your operation system and then follow the [online installation guide][install_anaconda].

## Framework virtual python environment

To create a virtual python environment for the database functions of the framework you can use the file _xspatula_py_3.12.yml_. It is included in the Xspatula repo under the path ./setup/anaconda and shown below.

```
name: xspatula_py_3.12
channels:
  - conda-forge
  - defaults
dependencies:
  - python=3.12
  - numpy
  - pandas
  - openpyxl
  - matplotlib
  - pyserial
  - psycopg2
  - scipy
  - python-dotenv
  - tabulate
```

Open a Terminal window and navigate to the folder that contains _xspatula_py_3.12.yml_. Then execute the command _conda env create_ as shown below. When you start a notebook in the framework package for the first time you will be asked to select Python environment. Select the _xspatula_py_3.12.yml_ as the Python environment.

```
conda env create --file xspatula_py_3.12.yml
```

If you want to remove the virtual environment, run the command:

```
conda remove --name xspatula_py_3.12 --all
```

## Input files

| File | Purpose |
|---|---|
| `xspatula_py_3.12.yml` | Conda environment definition; specifies Python 3.12 with numpy, pandas, openpyxl, matplotlib, pyserial, psycopg2, scipy, python-dotenv and tabulate |

[download_anaconda]: https://www.anaconda.com/download

[install_anaconda]: https://www.anaconda.com/docs/getting-started/anaconda/install/overview
