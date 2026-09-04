---
title:  "Visual Studio Code"
layout: single
sidebar:
  nav: "framework"
excerpt: "Visual Studio Code (VSCode) is a free, lightweight, and powerful source code editor developed by Microsoft for Windows, macOS, and Linux. You can use it for running the Xspatula framework pre-built Jupyter notebooks for setting up a PotgreSQL database and preparing some basic framework processes."
permalink: /framework/vscode/
author_profile: false
date:   2026-03-15 16:13:03 +0200
last_modified_at:   2026-03-29 20:50:10 +0200
---

Visual Studio Code (VSCode) is a free, lightweight, and powerful source code editor developed by Microsoft for Windows, macOS, and Linux. You can use it for running the Xspatula framework pre-built Jupyter notebooks for setting up a PotgreSQL database and some basic framework processes.

To learn about and download VSCode, visit the [VSCode official page][vscode_home]. Download and install VSCode on you machine and then you can the [Jupyter notebooks][notebook] that are included in the framework repository. But before you can run any of them you first have to:

1. install or get access to a [postgreSQL][postgres] server,
2. edit the [scheme file][scheme_file] to define your postgres superuser credentials,
3. create an [Anaconda][anaconda] virtual python environment to use with VSCode for running the framework.

[notebook]: ../notebook/

[postgres]: ../setup_db/postgres/

[scheme_file]: ../scheme_file/

[anaconda]: ../setup_db/anaconda/

[vscode_home]: https://code.visualstudio.com
