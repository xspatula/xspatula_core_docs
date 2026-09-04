---
title:  "Xspatula process file"
layout: single
sidebar:
  nav: "framework"
excerpt: "A process file is a JSON file defining the processes to run and the required arguments for that process. You can also define verbosity level, if the processes shall be executed or not, if the process should overwrite (update) existing data or delete existing data."
permalink: /framework/process_file/
author_profile: false
date:   2026-03-15 16:13:03 +0200
last_modified_at:   2026-03-15 16:13:03 +0200
---

A process file is a JSON file defining the processes to run and the required arguments for that process. You can also define verbosity level, if the processes shall be executed or not, if the process should overwrite (update) existing data or delete existing data.

If you do not define:
- verbosity,
- execute,
- overwrite, or
- delete

The values for those arguments will be inherited from the [scheme file][scheme_file].

The process file(s) to run are defined in the [job file][job_file].

## Process file for creating database schemas

The process create_schema requires a single parameter (_schema_) and the process file for setting up the default schemas of the framework database looks like this:

```
{
  "process": [
    {
      "process_id": "create_schema",
      "parameters": {
        "schema": "utility"
      }
    },
    {
      "process_id": "create_schema",
      "parameters": {
        "schema": "process"
      }
    },
    {
      "process_id": "create_schema",
      "parameters": {
        "schema": "community"
      }
    }
  ]
}
```

Note how the actual processes to run are constructed as an array and you can stack any number of processes in a single process file.

If you want to change verbosity, execute, overwrite, or delete for a particular process, you need to expand the JSON:

```
{
  "process": [
    {
      "process_id": "create_schema",
      "verbosity": 2,
      "overwrite": true,
      "parameters": {
        "schema": "utility"
      }
    },
    ...
    ...
  ]
}
```

## Input files

| File | Purpose |
|---|---|
| `schema_v10_sql.json` | Representative example of a process file; defines three `create_schema` processes for the `utility`, `community` and `process` schemas |

[scheme_file]: ../scheme_file

[job_file]: ../job_file
