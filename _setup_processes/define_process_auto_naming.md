---
title: "Automatic naming and tags"
layout: single
sidebar:
  nav: "setup_processes"
excerpt: "Records carry a free-text name for humans and a strict tag for machines. Composed names are built from parent records with reserved separators, and the auto_name rule looks up parents by name, tag or alias."
permalink: /setup_processes/auto_naming/
author_profile: false
date: 2026-10-11
last_modified_at: 2026-10-11
---

The basic `auto_name` block is introduced on the page [Process options][process_options_auto_naming]. This page gives the full description: why records carry both a name and a tag, which characters are reserved, how the arguments of a `concat` rule are resolved, and what to do when automatic naming fails. The worked example is the LUCAS 2009 chain from the sibling project `xspatula_lucas`, where the observation tables are defined.

## Name and tag

Records that are used to build other names have two identifiers:

- `name` is free text for humans. It may contain blanks and should read naturally, e.g. `land use and coverage area frame survey (lucas) topsoil data`.
- `tag` is a short, strict key for machines. It is used to build composed names and, in the machine learning steps, file names (parquet, images, pickle), e.g. `lucas_2009_eu`.

Think of the name as the label on a jar and the tag as the barcode. The label can be rewritten for readability; the barcode must stay short, unique and machine readable.

## Tag columns

The following columns hold tags:

| Table | Column | Constraint |
|---|---|---|
| `observation.dataset` | `tag` | `TEXT UNIQUE NOT NULL` |
| `observation.campaign` | `tag` | `TEXT UNIQUE NOT NULL` |
| `observation.campaign` | `datetime_tag` | `TEXT NOT NULL` |
| `observation.sampling_log` | `tag` | `TEXT UNIQUE NOT NULL` |
| `observation_utility.provision` | `tag` | `TEXT UNIQUE NOT NULL` |
| `observation.observation_log` | `tag` | `TEXT`, `UNIQUE (sampling_log_id, tag)` |

The tags of dataset, campaign, sampling_log and provision are **globally** unique. Note that `sampling_log` names are still only unique per campaign (`UNIQUE (campaign_id, name)`), but its tag is unique across the whole database.

`datetime_tag` is the time part of a campaign tag (e.g. `2009`). It is given explicitly when the campaign is registered.

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

The `format` block documents the intended rules for each tag (no underscore, no at-sign, no capitals, no blanks). **No code enforces it yet.** The only check that is active is the reserved separator rule below, and that rule only covers the parameters `name`, `alias` and `tag`. A `datetime_tag` such as `2009_06` is therefore accepted today, but it would break the splitting of composed tags. Keep to the `format` rules by hand until they are enforced.

## Reserved separators

The underscore `_` and the at-sign `@` join the parts of composed names, e.g. `lucas_2009_eu@foss-xds-rca`. For a composed name to be split back into its parts, no part may itself contain a separator. The rule is:

- A process parameter `name`, `alias` or `tag` that contains `_` or `@` is **rejected**,
- unless it exactly equals what the `auto_name` rule of that parameter would produce.

In practice: either leave the parameter as `auto` and let the rule compose it, or give your own value without `_` and `@`. Use a hyphen if you need a word break inside a tag (e.g. `foss-xds-rca`).

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
  "concat": "'%s_%s' %(dataset_id__dataset_tag, datetime_tag)"
}
```

The format string **must be in single quotes**. Without them the process stops with `can not parse concat`. The arguments go inside the last pair of parentheses, separated by commas.

Each argument is resolved as follows:

| Argument | Resolved to |
|---|---|
| `campaign_territory_id` | Territory code of the campaign: `observation.campaign_location.territory_id` → `utility.territory.iso_code_a2_ext` (e.g. `eu`) |
| `<table>_id__<table>_<column>`, e.g. `dataset_id__dataset_tag`, `campaign_id__campaign_name`, `dataset_id__dataset_alias` | The `<column>` of the record referenced by the process parameter `<table>_id__<table>_name` (see [Record lookup](#record-lookup)) |
| Anything else, e.g. `datetime_tag`, `tag`, `profile_min` | The value of that process parameter as given |

Automatic naming is triggered when the parameter value is `auto` (or `auto_name`, used by the sample processes). Leaving out a non-required parameter whose `default_value` is `auto` has the same effect.

### Convention for name, tag and display_name

- `tag` concats use tags as input and `_` or `@` as separators.
- `name` concats follow the separator rule; they may use names or aliases joined by blanks, or `@` when the parts must be split later.
- `display_name` concats are for humans only and use names, aliases and blanks. A `display_name` given explicitly (e.g. `LUCAS 2009`) overrides the rule.

## Record lookup

For an argument `<table>_id__<table>_<column>`, the record is located from the value of the process parameter `<table>_id__<table>_name`.

### Name, tag or alias

The value given for `<table>_id__<table>_name` may be the record's **name, tag or alias**. It is searched in the columns `name`, `tag` and `alias`, in that order. Columns the table does not have are skipped.

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

### The campaign territory

`campaign_territory_id` is a special argument. It locates the campaign through the process parameter `campaign_id__campaign_name` (same name → tag → alias lookup), reads its `territory_id` from `observation.campaign_location` and returns the `iso_code_a2_ext` of that territory from `utility.territory`. The campaign location is written when the campaign is registered with the parameter `territory_id__territory_name` (see [Foreign key][foreign_key]).

## Worked example

The LUCAS 2009 data is registered as a chain dataset → campaign → sampling_log → observation_log. Each level builds its name and tag from the level above.

| Level | Concat (name / tag) | name | tag |
|---|---|---|---|
| dataset | given | `land use and coverage area frame survey (lucas) topsoil data` (alias `lucas`) | `lucas` |
| campaign | `'%s %s' %(dataset_id__dataset_alias, datetime_tag)` / `'%s_%s' %(dataset_id__dataset_tag, datetime_tag)` | `lucas 2009` | `lucas_2009` |
| sampling_log | `'%s %s' %(campaign_id__campaign_name, campaign_territory_id)` / `'%s_%s' %(campaign_id__campaign_tag, campaign_territory_id)` | `lucas 2009 eu` | `lucas_2009_eu` |
| observation_log | `'%s@%s' %(sampling_log_id__sampling_log_name, provision_id__provision_name)` / `'%s@%s' %(sampling_log_id__sampling_log_tag, provision_id__provision_tag)` | `lucas 2009 eu@foss xds rapid content analyzer` | `lucas_2009_eu@foss-xds-rca` |

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
              "hint": "Name of the campaign",
              "sketch": "schema_table and permission blocks omitted",
              "auto_name": {
                "concat": "'%s %s' %(dataset_id__dataset_alias, datetime_tag)"
              }
            },
            {
              "parameter": "tag",
              "parameter_type": "text",
              "required": false,
              "default_value": "auto",
              "hint": "Unique tag used for auto naming production files and results, auto = dataset tag + _ + datetime_tag. A tag given by hand must not contain underscore, at (@), capitals or blank.",
              "sketch": "schema_table and permission blocks omitted",
              "auto_name": {
                "concat": "'%s_%s' %(dataset_id__dataset_tag, datetime_tag)"
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
              "hint": "Display name of the campaign",
              "sketch": "schema_table and permission blocks omitted",
              "auto_name": {
                "concat": "'%s %s' %(dataset_id__dataset_alias, datetime_tag)"
              }
            },
            {
              "sketch": "datetime_tag, territory_id__territory_name and further parameters omitted"
            }
          ]
        }
      ]
    }
  ]
}
```

The campaign is registered with `dataset_id__dataset_name: "lucas"`, `datetime_tag: 2009`, `display_name: "LUCAS 2009"` and `territory_id__territory_name: "eu"`. Name and tag are left to the rule.

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
        "concat": "'%s %s' %(campaign_id__campaign_name, campaign_territory_id)"
      }
    },
    {
      "parameter": "tag",
      "default_value": "auto",
      "sketch": "other keys omitted",
      "auto_name": {
        "concat": "'%s_%s' %(campaign_id__campaign_tag, campaign_territory_id)"
      },
      "format": {
        "at": false,
        "capital": false,
        "blank": false
      }
    }
  ]
}
```

The process call only needs the campaign:

```json
{
  "process": [
    {
      "process": "manage_sampling_log",
      "parameters": {
        "campaign_id__campaign_name": "lucas 2009",
        "contact_name": "inherit",
        "contact_email": "inherit",
        "abstract": "Sampling log for lucas eu 2009"
      }
    }
  ]
}
```

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
        "sampling_log_id__sampling_log_name": "lucas 2009 eu",
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
| `❌ ERROR auto_name: parameter <key> required for <parameter> is missing` | A plain argument (e.g. `datetime_tag`) has no value. Give it in the process call. |
| `❌ ERROR auto_name: parameter <table>_id__<table>_name is missing` | The parameter that identifies the referenced record is not in the call, or is empty. Add it. |
| `❌ ERROR auto_name: table <table> has no column <column> (from <key>)` | The concat asks for a column that does not exist, e.g. a `tag` on a table without tags. Fix the concat or add the column. |
| `❌ ERROR auto_name: <value> not found as name, tag or alias in table <table>` | No record matches. Check spelling and that the parent was registered first. |
| `❌ ERROR auto_name: <value> matches N records in <schema.table> - give the tag or add the parent (e.g. <...>) to the process` | The value is ambiguous. Give the tag, or add the parent parameter named in the message. |
| `❌ ERROR auto_name: record id <id> in <schema.table> has no <column>` | The record exists but the requested column is empty, e.g. a record created before tags were introduced. Fill in the column. |
| `❌ ERROR auto_name: campaign id <id> has no territory in observation.campaign_location` | The campaign was registered without `territory_id__territory_name`. Update the campaign. |
| `❌ ERROR auto_name: territory id <id> has no iso_code_a2_ext` | The territory record lacks the code. Fill in `utility.territory.iso_code_a2_ext`. |
| `❌ ERROR parameter <key> = <value> contains "_" or "@" (reserved for composed names) for process <process>` | An explicit `name`, `alias` or `tag` contains a separator and does not equal the auto name. Set it to `auto`, omit it, or remove the separator. |

## Requirements

- **Load order.** Parents must exist, with tags, before children are auto named: dataset → campaign (including `territory_id__territory_name`, which writes `observation.campaign_location`) → sampling_log → observation_log. Provisions must be registered before the observation logs that use them.
- **Parameter presence.** The `<table>_id__<table>_name` parameter must be defined in the process for a lookup to work. A parent can only scope an ambiguous search if its parameter is also defined in, and given to, the process.
- **Existing databases.** The `tag` and `datetime_tag` columns need a rebuild of the database, or an `ALTER TABLE` for each table. `CREATE TABLE` in the setup files does not alter tables that already exist. Existing rows must be given tags before `NOT NULL` can be set:

```sql
ALTER TABLE observation.campaign ADD COLUMN tag TEXT UNIQUE;
ALTER TABLE observation.campaign ADD COLUMN datetime_tag TEXT;
-- fill in tag and datetime_tag for all existing rows, then
ALTER TABLE observation.campaign ALTER COLUMN tag SET NOT NULL;
ALTER TABLE observation.campaign ALTER COLUMN datetime_tag SET NOT NULL;
```

- **Explicit names.** Process calls that previously gave composed names explicitly (e.g. `lucas_eu_2009@...`) are now rejected unless the value equals the auto name. Set them to `auto` or leave them out.

[process_options_auto_naming]: ../process_options/#automatic-naming
[foreign_key]: ../../setup_db/foreign_key/
