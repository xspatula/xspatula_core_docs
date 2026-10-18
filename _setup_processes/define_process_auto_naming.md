---
title: "Automatic naming and tags"
layout: single
sidebar:
  nav: "setup_processes"
excerpt: "Records carry a free-text name for humans and a strict tag for machines. Composed names are built from parent records with reserved separators, and the auto_name rule looks up parents by name, tag or alias."
permalink: /setup_processes/auto_naming/
author_profile: false
date: 2026-10-11
last_modified_at: 2026-10-18
---

The basic `auto_name` block is introduced on the page [Process options][process_options_auto_naming]. This page gives the full description: why records carry both a name and a tag, which characters are reserved, how the arguments of a `concat` rule are resolved, and what to do when automatic naming fails. The worked example is the LUCAS 2009 chain from the sibling project `xspatula_lucas`, where the observation tables are defined.

## Name and tag

Records that are used to build other names have two identifiers:

- `name` is free text for humans. It may contain blanks and should read naturally, e.g. `land use and coverage area frame survey (lucas) topsoil data`.
- `tag` is a short, strict key for machines. It is used to build composed names and, in the machine learning steps, file names (parquet, images, pickle), e.g. `lucas_eu_2009`.

Think of the name as the label on a jar and the tag as the barcode. The label can be rewritten for readability; the barcode must stay short, unique and machine readable.

## Tag columns

The following columns hold tags:

| Table | Column | Constraint |
|---|---|---|
| `observation.dataset` | `tag` | `TEXT UNIQUE NOT NULL` |
| `observation.campaign` | `tag` | `TEXT UNIQUE NOT NULL` |
| `observation.sampling_log` | `tag` | `TEXT UNIQUE NOT NULL` |
| `observation.sampling_log` | `datetime_tag` | `TEXT NOT NULL` (`0` = no date) |
| `observation_utility.provision` | `tag` | `TEXT UNIQUE NOT NULL` |
| `observation.observation_log` | `tag` | `TEXT`, `UNIQUE (sampling_log_id, tag)` |

The tags of dataset, campaign, sampling_log and provision are **globally** unique. Note that `sampling_log` names are still only unique per campaign (`UNIQUE (campaign_id, name)`), but its tag is unique across the whole database.

A campaign is a **place**: dataset + territory + optional site. A sampling_log is **one sampling of that place at a time**. `datetime_tag` is therefore the time part of a sampling_log tag (e.g. `2009`), given explicitly when the sampling_log is registered. It must be `0` (no date) or a datestamp `YYYY`, `YYYYMM`, `YYYYMMDD` or `YYYYMMDDhhmmss`; anything else is rejected. sampling_log also carries a `display_name` and a `url`.

Do not confuse these tags with the table `observation.dataset_tag`, which holds descriptive keywords and substances for a dataset.

### The format block (planned, not enforced)

Tag parameters in the process definitions carry a `format` block:

```json
{
  "parameter": "datetime_tag",
  "parameter_type": "text",
  "required": true,
  "default_value": "",
  "hint": "Unique tag used for auto naming production files and results, should be less than 24 chars not contain underscore, at (@) or blank",
  "sketch": "schema_table and permission blocks omitted",
  "format": {
    "underscore": false,
    "at": false,
    "capital": false,
    "blank": false
  }
}
```

The `format` block documents the intended rules for each tag (no underscore, no at-sign, no capitals, no blanks). **No code reads the `format` block yet.** Three other checks cover most of it:

- the reserved separator rule below (parameters `name`, `alias`, `tag` and `datetime_tag`),
- the `datetime_tag` format check above (`0` or digits only, so `2009_06` is rejected),
- when a tag is composed, plain values given as process parameters may only hold `a-z`, `0-9`, `.` and `-` (see [Concat arguments](#concat-arguments)).

## Reserved separators

The underscore `_`, the at-sign `@` and the hash `#` join the parts of composed names, e.g. `lucas_eu_2009@foss-xds-rca`. For a composed name to be split back into its parts, no part may itself contain a separator. The rule is:

- A process parameter `name`, `alias`, `tag` or `datetime_tag` that contains `_`, `@` or `#` is **rejected**,
- unless it exactly equals what the `auto_name` rule of that parameter would produce.

In practice: either leave the parameter as `auto` and let the rule compose it, or give your own value without `_`, `@` and `#`. Use a hyphen if you need a word break inside a tag (e.g. `foss-xds-rca`).

### Exemptions

Exemptions are listed per process in `separator_exempt_D` in `src/lib/structure.py`. Today it holds a single entry:

```python
separator_exempt_D = {'manage_geolocation': ['name']}
```

The geolocation name (e.g. `fr_lucas@38622598`) may contain separators because the geolocation is registered *before* the sample and acts as the unique key for a sample. When different data sources give the same sample different names, the geolocation ties them together; without it the same sample could be registered twice under two names.

The consequence: **never use `geolocation_id__geolocation_name` as an argument in an `auto_name` concat.** A name that already contains separators can not be split again.

## Concat arguments

The `auto_name` block holds a `concat` string with a Python-style format string and a list of arguments:

```json
"auto_name": {
  "concat": "'%s_%s' %(campaign_id__campaign_tag, datetime_tag)"
}
```

The format string **must be in single quotes**. Without them the process stops with `can not parse concat`. The arguments go inside the last pair of parentheses, separated by commas, and their number must equal the number of `%s` placeholders.

Each argument is resolved as follows:

| Argument | Resolved to |
|---|---|
| `<table>_id__<table>_<column>`, e.g. `dataset_id__dataset_tag`, `campaign_id__campaign_name`, `territory_id__territory_iso_code_a2_ext` | The `<column>` of the record referenced by the process parameter `<table>_id__<table>_name` (see [Record lookup](#record-lookup)) |
| Anything else, e.g. `datetime_tag`, `site`, `profile_min` | The value of that process parameter as given. When a **tag** is composed, such a value may only hold `a-z`, `0-9`, `.` and `-` |

The territory is an ordinary foreign key argument: `territory_id__territory_name` gives the territory name (`european union`), `territory_id__territory_iso_code_a2_ext` its code (`eu`), both located from the process parameter `territory_id__territory_name`.

### Empty arguments

An empty argument is handled in one of three ways:

- **Optional (`?`).** An argument ending in `?`, e.g. `site?`, is optional. If it is empty, `none` or `0` it is dropped together with the separator before it. `'%s_%s_%s' %(dataset_id__dataset_tag, territory_id__territory_iso_code_a2_ext, site?)` gives `lucas_eu` without a site and `lucas_eu_<site>` with one.
- **Strict null.** The arguments listed in `AUTO_NAME_NULL_D` (`src/postgres/pg_common.py`) may be empty (or `0`). They then get a fixed null value: `0` in a tag, the null text in a name.

  | Argument | Null in name | Null in tag |
  |---|---|---|
  | territory (`territory_id__territory_<column>`) | `no location` | `0` |
  | `datetime_tag` | `no date` | `0` |

  A sampling_log without date is thus named `lucas european union no date` with tag `lucas_eu_0`.
- **Anything else** that is empty is an error: `parameter <key> required for <parameter> is missing`.

Automatic naming is triggered when the parameter value is `auto` (or `auto_name`, used by the sample processes). Leaving out a non-required parameter whose `default_value` is `auto` has the same effect.

### Convention for name, tag and display_name

- `tag` concats use tags as input and `_` or `@` as separators.
- `name` concats follow the separator rule; they may use names or aliases joined by blanks, or `@` when the parts must be split later.
- `display_name` concats are for humans only and use names, aliases and blanks. A `display_name` given explicitly (e.g. `LUCAS 2009`) overrides the rule.

## Record lookup

For an argument `<table>_id__<table>_<column>`, the record is located from the value of the process parameter `<table>_id__<table>_name`.

### Name, tag, alias or other unique column

The value given for `<table>_id__<table>_name` may be the record's **name, tag, alias or any other single-column unique text column**. It is searched in the columns `name`, `tag` and `alias`, in that order, and then in the remaining text columns that carry their own `UNIQUE` constraint, e.g. `utility.territory.iso_code_a2_ext`. Columns the table does not have are skipped. A column that is only unique together with others (e.g. `dataset_id` in `UNIQUE (dataset_id, name)`) does not identify a record on its own and is not searched.

So `territory_id__territory_name: "eu"` finds the territory `european union` through its `iso_code_a2_ext`.

If nothing matches, a composed value (e.g. `lucas_eu_2009@foss xds rca`, with the alias where the name was expected) is split at the separators of the table's own `auto_name` rules and each part is looked up the same way.

The suffix of the concat argument, not the column the value matched, decides what is returned:

| Concat argument | Returns |
|---|---|
| `dataset_id__dataset_name` | `name` of the dataset |
| `dataset_id__dataset_tag` | `tag` of the dataset |
| `dataset_id__dataset_alias` | `alias` of the dataset |

So a user who types the tag where the name was expected still gets the correct name back. In the LUCAS campaign, `dataset_id__dataset_name` is given as `lucas`, which matches the dataset tag; `dataset_id__dataset_alias` still returns the alias and `dataset_id__dataset_tag` the tag.

### Ambiguity and parent scoping

A value can match more than one record. A campaign name, for instance, is only unique within its dataset. When this happens, the search is narrowed by the parent records that the process also gives:

- The parents of a table are its `<parent>_id` columns, e.g. `dataset_id` for `campaign`, or `sampling_log_id` and `provision_id` for `observation_log`.
- Each parent is used only if the process has the parameter `<parent>_id__<parent>_name`. That parent is itself resolved with the same lookup, so scoping works recursively.
- If the scoped search still does not give exactly one record, the process is rejected:

```
❌ ERROR auto_name: <value> matches N records in <schema.table> - give the tag or add the parent (e.g. <dataset_id__dataset_name>) to the process
```

A tag always resolves to a single record, because tags are globally unique. When in doubt, give the tag.

## Worked example

The LUCAS data is registered as a chain dataset → campaign → sampling_log → observation_log. The campaign is the place (LUCAS in the European Union), each sampling_log one survey of it in time. Each level builds its name and tag from the level above.

| Level | `name` concat | `tag` concat | name | tag |
|---|---|---|---|---|
| dataset | given | given | `land use and coverage area frame survey (lucas) topsoil data` (alias `lucas`) | `lucas` |
| campaign | `'%s %s %s' %(dataset_id__dataset_alias, territory_id__territory_name, site?)` | `'%s_%s_%s' %(dataset_id__dataset_tag, territory_id__territory_iso_code_a2_ext, site?)` | `lucas european union` | `lucas_eu` |
| sampling_log | `'%s %s' %(campaign_id__campaign_name, datetime_tag)` | `'%s_%s' %(campaign_id__campaign_tag, datetime_tag)` | `lucas european union 2009` | `lucas_eu_2009` |
| observation_log | `'%s@%s' %(sampling_log_id__sampling_log_name, provision_id__provision_name)` | `'%s@%s' %(sampling_log_id__sampling_log_tag, provision_id__provision_tag)` | `lucas european union 2009@foss xds rapid content analyzer` | `lucas_eu_2009@foss-xds-rca` |

The `display_name` rules are campaign `'%s %s' %(dataset_id__dataset_alias, territory_id__territory_name)` and sampling_log `'%s %s' %(campaign_id__campaign_display_name, datetime_tag)`. The LUCAS campaign gives its display name explicitly as `LUCAS EU`, so the sampling_log display name becomes `LUCAS EU 2009`.

LUCAS 2015 is a second sampling_log of the same campaign: `datetime_tag: 2015` gives `lucas european union 2015` / `lucas_eu_2015`. No new campaign is needed.

The provision is registered with name `foss xds rapid content analyzer`, alias `foss xds rca` and tag `foss-xds-rca`.

### Campaign

From `setup/zzz/lucas/setup_processes/json/observation/campaign_v10_sql.json` (shortened):

```json
{
  "process": [
    {
      "process": "add_process",
      "overwrite": true,
      "parameters": {
        "root_process": "manage_table_data",
        "process": "manage_campaign",
        "min_user_stratum": 3,
        "title": "Manage a campaign.",
        "label": "Insert update or delete a dataset campaign"
      },
      "nodes": [
        {
          "parent": "process",
          "element": "parameters",
          "parameter": [
            {
              "parameter": "dataset_id__dataset_name",
              "parameter_type": "text",
              "required": true,
              "default_value": "",
              "hint": "The name of the dataset to which this campaign belongs",
              "sketch": "schema_table and permission blocks omitted"
            },
            {
              "parameter": "name",
              "parameter_type": "text",
              "required": false,
              "default_value": "auto",
              "sketch": "hint, schema_table and permission blocks omitted",
              "auto_name": {
                "concat": "'%s %s %s' %(dataset_id__dataset_alias, territory_id__territory_name, site?)"
              }
            },
            {
              "parameter": "tag",
              "parameter_type": "text",
              "required": false,
              "default_value": "auto",
              "sketch": "hint, schema_table and permission blocks omitted",
              "auto_name": {
                "concat": "'%s_%s_%s' %(dataset_id__dataset_tag, territory_id__territory_iso_code_a2_ext, site?)"
              },
              "format": {
                "at": false,
                "capital": false,
                "blank": false
              }
            },
            {
              "parameter": "display_name",
              "parameter_type": "text",
              "required": false,
              "default_value": "auto",
              "sketch": "hint, schema_table and permission blocks omitted",
              "auto_name": {
                "concat": "'%s %s' %(dataset_id__dataset_alias, territory_id__territory_name)"
              }
            },
            {
              "parameter": "territory_id__territory_name",
              "parameter_type": "text",
              "required": false,
              "default_value": "",
              "sketch": "hint, schema_table and permission blocks omitted"
            },
            {
              "parameter": "site",
              "parameter_type": "text",
              "required": false,
              "default_value": "",
              "sketch": "hint, schema_table and permission blocks omitted"
            },
            {
              "sketch": "further parameters omitted"
            }
          ]
        }
      ]
    }
  ]
}
```

The campaign is registered with `dataset_id__dataset_name: "lucas"`, `territory_id__territory_name: "eu"`, no `site` and `display_name: "LUCAS EU"`. It has no `datetime_tag`. `eu` is found in `utility.territory.iso_code_a2_ext`; the name rule returns the territory name `european union`, the tag rule its code `eu`. The empty `site?` is dropped together with its separator.

### Sampling log

From `sampling_log_v10_sql.json` (shortened to the `auto_name` blocks):

```json
{
  "parameter": [
    {
      "parameter": "campaign_id__campaign_name",
      "sketch": "only outlining the parameter array object parameter"
    },
    {
      "parameter": "name",
      "default_value": "auto",
      "sketch": "other keys omitted",
      "auto_name": {
        "concat": "'%s %s' %(campaign_id__campaign_name, datetime_tag)"
      }
    },
    {
      "parameter": "tag",
      "default_value": "auto",
      "sketch": "other keys omitted",
      "auto_name": {
        "concat": "'%s_%s' %(campaign_id__campaign_tag, datetime_tag)"
      },
      "format": {
        "at": false,
        "capital": false,
        "blank": false
      }
    },
    {
      "parameter": "display_name",
      "default_value": "auto",
      "sketch": "other keys omitted",
      "auto_name": {
        "concat": "'%s %s' %(campaign_id__campaign_display_name, datetime_tag)"
      }
    },
    {
      "parameter": "datetime_tag",
      "default_value": "0",
      "sketch": "other keys omitted"
    }
  ]
}
```

The process call gives the campaign and the time:

```json
{
  "process": [
    {
      "process": "manage_sampling_log",
      "parameters": {
        "campaign_id__campaign_name": "lucas_eu",
        "datetime_tag": "2009",
        "begun_at": 20090501,
        "ended_at": 20091031,
        "url": "https://esdac.jrc.ec.europa.eu/content/lucas-2009-topsoil-data",
        "contact_name": "inherit",
        "contact_email": "inherit",
        "abstract": "Sampling log for lucas eu 2009"
      }
    }
  ]
}
```

The campaign is given by its tag `lucas_eu`; the name `lucas european union` gives the same result.

### Observation log

From `observation_log_v10_sql.json` (shortened to the `auto_name` blocks):

```json
{
  "parameter": [
    {
      "parameter": "name",
      "default_value": "auto",
      "sketch": "other keys omitted",
      "auto_name": {
        "concat": "'%s@%s' %(sampling_log_id__sampling_log_name, provision_id__provision_name)"
      }
    },
    {
      "parameter": "tag",
      "default_value": "auto",
      "sketch": "other keys omitted",
      "auto_name": {
        "concat": "'%s@%s' %(sampling_log_id__sampling_log_tag, provision_id__provision_tag)"
      }
    }
  ]
}
```

The process call:

```json
{
  "process": [
    {
      "process": "manage_observation_log",
      "parameters": {
        "sampling_log_id__sampling_log_name": "lucas_eu_2009",
        "provision_id__provision_name": "foss xds rapid content analyzer",
        "contact_name": "inherit",
        "contact_email": "inherit",
        "preparation_id__preparation_name": "ds2",
        "preservation_id__preservation_name": "ds2",
        "storage_id__storage_name": "amb",
        "laboratory": 1
      }
    }
  ]
}
```

Giving `"provision_id__provision_name": "foss-xds-rca"` (the tag) or `"foss xds rca"` (the alias) produces the same name and tag.

## Errors

Every failed lookup logs an error and stops the process; nothing is skipped silently.

| Message | Cause and fix |
|---|---|
| `❌ ERROR auto_name: can not parse concat <concat> for <parameter>` | The format string is not in single quotes, or there are no arguments in the final parentheses. Fix the `concat` in the process definition. |
| `❌ ERROR auto_name: concat <concat> for <parameter> has N placeholders but M arguments` | The number of `%s` does not match the number of arguments. Fix the `concat`. |
| `❌ ERROR auto_name: parameter <key> required for <parameter> is missing` | An argument that is neither optional (`?`) nor a strict null (territory, `datetime_tag`) has no value. Give it in the process call. |
| `❌ ERROR auto_name: parameter <table>_id__<table>_name is missing` | The parameter that identifies the referenced record is not in the call, or is empty. Add it. |
| `❌ ERROR auto_name: table <table> has no column <column> (from <key>)` | The concat asks for a column that does not exist, e.g. a `tag` on a table without tags. Fix the concat or add the column. |
| `❌ ERROR auto_name: searching <value> in <schema.table.column> failed: <error>` | The database query failed, e.g. a type mismatch. The first line of the database error is shown. |
| `❌ ERROR auto_name: <value> not found as name, tag, alias or other unique column in table <table>` | No record matches. Check spelling and that the parent was registered first. |
| `❌ ERROR auto_name: <value> matches N records in <schema.table> - give the tag or add the parent (e.g. <...>) to the process` | The value is ambiguous. Give the tag, or add the parent parameter named in the message. |
| `❌ ERROR auto_name: record id <id> in <schema.table> has no <column>` | The record exists but the requested column is empty, e.g. a record created before tags were introduced. Fill in the column. |
| `❌ ERROR auto_name: <key> = <value> can not be part of a tag (only a-z, 0-9, "." and "-" allowed)` | A plain value (e.g. `site`) used in a tag concat holds capitals, blanks, `_` or other characters. Change the value. |
| `❌ ERROR parameter <datetime_tag> = <value> must be 0 or YYYY, YYYYMM, YYYYMMDD or YYYYMMDDhhmmss for process <process>` | `datetime_tag` is not a datestamp. Use `0` for no date. |
| `❌ ERROR parameter <key> = <value> contains "_", "@" or "#" (reserved for composed names) for process <process>` | An explicit `name`, `alias`, `tag` or `datetime_tag` contains a separator and does not equal the auto name. Set it to `auto`, omit it, or remove the separator. |

## Requirements

- **Load order.** Parents must exist, with tags, before children are auto named: dataset → campaign (with `territory_id__territory_name`) → sampling_log (with `datetime_tag`) → observation_log. Territories and provisions must be registered before the records that use them.
- **Parameter presence.** The `<table>_id__<table>_name` parameter must be defined in the process for a lookup to work. A parent can only scope an ambiguous search if its parameter is also defined in, and given to, the process.
- **Existing databases need a rebuild.** `datetime_tag` moved from `observation.campaign` to `observation.sampling_log`, and sampling_log gained `display_name` and `url`. `CREATE TABLE` in the setup files does not alter tables that already exist, so a database built before 2026-10 must be rebuilt and the data loaded again (a `pg_dump` of the old database restores the old tables). Patching by hand with `ALTER TABLE` is possible but error prone, because the campaign and sampling_log tags change as well (`lucas_2009` + `lucas_2009_eu` become `lucas_eu` + `lucas_eu_2009`).
- **Explicit names.** Process calls that previously gave composed names explicitly (e.g. `lucas_2009_eu@...`) are now rejected unless the value equals the auto name. Set them to `auto` or leave them out.

[process_options_auto_naming]: ../process_options/#automatic-naming
[foreign_key]: ../../setup_db/foreign_key/
