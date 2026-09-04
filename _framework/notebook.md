---
title:  "Xspatula notebooks"
layout: single
sidebar:
  nav: "framework"
excerpt: "To manage the Xspatula framework database it comes with three Jupyter notebooks: setup_db, delete_db and setup_processes. Before you can run the notebooks you need to create a python environment. To test the core functionality, the only thing you need to edit in the framework itself are your credentials for the postgreSQL database cluster you want to use."
permalink: /framework/notebook/
author_profile: false
date:   2026-03-15 16:13:03 +0200
last_modified_at:   2026-03-29 21:15:03 +0200
---

To manage the Xspatula framework database it comes with three Jupyter notebooks: setup_db, delete_db and setup_processes. Before you can run the notebooks you need to create a python environment. To test the core functionality, the only thing you need to edit in the framework itself are your credentials for the postgreSQL database cluster you have access to.

## Prerequisits

To build your own database, whether on your local machine or a remote server, you might need to install the following applications/files:

- [Visual Studion code (VScode)][vscode] (or other environment for running jupyter notebooks),
- [postgreSQL][postgres] (unless you have access to a server with postgres installed),
- [Anaconda][anaconda] or any other application for defining virtual python environments, and/or
- A [.netrc][netrc] file that defines your credentials for setting up new databases (to protect your login credentials).

## Python environment

To run a Jupyter notebook you need to define the Python environment that must contain all Python packages used by the notebook and its underlying code. In the Xspatula package path `./setup/anaconda` there is a prepared .yml file and a short instruction for how to setup the virtual environment. The same information is available in the [Anaconda][anaconda] document that also links to the Anaconda [download][download_anaconda] and [installation][install_anaconda] guides.

## Notebook structure

All Xspatula framework notebooks have the same basic structure. A first code block loads the underlying python packages, the second defines the [scheme][scheme_file] file, and the following points to either a [job][job_file] or [process][process_file] file and the actual processes to run.

### Code block 1 - imports and setup

The first code block in all notebooks defines the local path to the notebook itself and loads the necessary python packages. Only the last item, the package application import, differs between the notebooks.

```
# Standard library imports
from os import path, getcwd

import sys

# Get the working directory of the notebook
notebook_path = getcwd()

# Set the path to the root directory of this jupyter notebook (python) project
module_path = path.abspath(path.join('..'))

if module_path not in sys.path:

    sys.path.append(module_path)

# Package application imports
from src_setup.lib_setup import Initiate_database
```

### Code block 2 - scheme file

The second code block loads the [scheme][scheme_file] file, defining project path, login credentials and  default settings for execute, delete, overwrite and verbosity.

```
scheme_file = './zzz/scheme_xspatula_local_setup.json'
```

The default path for the _scheme_ file for setting up the database is in the (sleepy) directory zzz under the directory where the notebooks themselves are available. You can move the _scheme_ file to any other path you want, and either give a relative (vis-a-vis the notebook itself) or absolute path that you have access to.

See the document on [scheme file][scheme_file] for more information.

### Code block 3+ - job or process file

The third and following code block(s) points to either a _job_ file or a _process_ file (the framework will recognise which type it is and act accordingly).

```
job_file = 'job_setup_db.json'

Initiate_database(notebook_path, scheme_file, job_file)
```

The code in the block(s) calling the job/process file is different when setting up or deleting a database compared to all other notebooks, illustrated below by code block in the notebook _setup_processes_

```
#%%script false --no-raise-error
job_file = 'job_setup_processes.json'

structured_process_D, scheme_params_D = Initiate_process(notebook_path, scheme_file, job_file)

if structured_process_D is not None:

    Run_process(structured_process_D, scheme_params_D)
```
The first line is commented out (starts with a hashtag, #), if you remove the comment (#), the notebook will skip this cell while executing.

See the documents on [job file][job_file], [pilot file][pilot_file] and [process file][process_file] for more information.

## Input files

| File | Purpose |
|---|---|
| `scheme_xspatula_local_setup.json` | Scheme file; defines project path, postgres superuser credentials, pg_users and default process settings |
| `job_setup_processes.json` | Job file; connects to the processes to run |

[vscode]: https://code.visualstudio.com

[postgres]: ../../setup_db/postgres/

[anaconda]: ../../setup_db/anaconda/

[netrc]: ../../setup_db/netrc/

[download_anaconda]: https://www.anaconda.com/download

[install_anaconda]: https://www.anaconda.com/docs/getting-started/anaconda/install/overview

[scheme_file]: ../scheme_file

[job_file]: ../job_file

[pilot_file]: ../pilot_file

[process_file]: ../process_file
