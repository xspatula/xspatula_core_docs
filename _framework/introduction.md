---
title: "Xspatula framework"
layout: home
sidebar:
  nav: "framework"
excerpt: "The Xspatula framework is a skeleton allowing users to build a postgreSQL database and define the arguments (parameters) for processes that utilise the database."
permalink: /framework/
author_profile: false
date:   2026-03-15 16:13:03 +0200
last_modified_at:   2026-03-30 09:34:58 +0200
---

The Xspatula framework is a skeleton for building python programs with arguments, input data and results stored in a postgreSQL database. The framework was primarily built for handling large, complex datasets, and for predictive modeling using this data.  

When setting up an Xspatula framework you first have to setup a postgreSQL database and then  define the parameters (arguments) for processes that utilise the database. Once the framework is set up you have to add the processes you want to run by either importing and linking existing python packages, create python bindings or write your own processes in python code. Each python operation that you want to include must then be defined as a process in the postgreSQL database.

## Framework interface

At its core the framework is a set of python scripts that create, update, populate and delete postgreSQL databases, schemas and tables. There is no graphical user interface, all commands are run via JSON (JavaScript Object Notation) commands. At first this might seem cumbersome but it comes with great advantages: you can safely update and delete trials and rerun the whole seeding procedure in a matter of seconds. And once you start using the framework for your own production you will always have a full documentation of all the process steps.

### Hierarchical scheme and project structure

Everything in the framework is defined as a project, with a hierarchical structure defining [scheme][scheme_file], [project (job)][job_file] and [process(es)][process_file] to run. The scheme, project and processes are all defined using JSON files, where:

- scheme defines user, database and the project to run,
- project (job) defines the processes to run, and
- process defines the python operation to run and all required arguments for that particular process.

### Notebook interface

The framework is built for running with Jupyter notebooks. The core framework includes three notebooks for seeding the postgreSQL database:

- ./setup/setup_db.ipynb
- ./setup/delete_db.ipynb
- ./setup/setup_processes.ipynb

In the [notebooks][notebook] you define the scheme and the project (job) that you want to run.

All python operations specific to your data and modeling must be developed as part of building up your own framework.

## Xspatula access and license

The Xspatula framework is provided under the following licenses:

- Data License: Creative Commons Attribution license (**CC-BY**)
- Code License: Massachusetts Institute of Technology License (**MIT License**)

[scheme_file]: ./scheme_file/

[job_file]: ./job_file/

[pilot_file]: ./pilot_file/

[process_file]: ./process_file/

[notebook]: ./notebook/
