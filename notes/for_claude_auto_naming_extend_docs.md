# For Claude: document tag-based automatic naming (xspatula_core_docs)

Written 2026-10-11 from a session in `xspatula_lucas`. Work through the tasks below from within this
repo. This repo holds the **full** description; `xspatula_lucas_docs` and `xspatula_ai4sh_docs` get
short summary pages that link back here, so keep the anchors listed under Task 1 stable.

## Background

The automatic naming (`auto_name`) of process parameters was extended in the code projects
`xspatula_lucas` and `xspatula_ai4sh`:

- branch `feature/tag-auto-naming`
- PRs: https://github.com/xspatula/xspatula_lucas/pull/1 and https://github.com/xspatula/xspatula_ai4sh/pull/9
- all LUCAS processes run clean on the live DB; AI4SH not yet tested live

**Sources of truth.** Read these and do not document from this note alone; if the code and this
note disagree, the code wins:

- `../xspatula_lucas/src/lib/structure.py`, method `_Assemble_parameters`, nested functions
  `extract_concat_parts`, `column_schema`, `find_record`, `parent_keys`, `parent_scope`,
  `lookup_column`, `lookup_campaign_territory` and `compose_auto_name`, plus the
  `separator_exempt_D` name check after the auto_name block
- `../xspatula_lucas/setup/zzz/lucas/setup_processes/json/observation/campaign_v10_sql.json`,
  `sampling_log_v10_sql.json` and `observation_log_v10_sql.json` (the real `auto_name` concats)
- `../xspatula_lucas/setup/zzz/lucas/setup_db/json/observation/*.json` and
  `observation_utility/provision_v10_sql.json` (the new `tag` columns and constraints)
- `../xspatula_lucas/.claude/CLAUDE.md`, section "No underscore "_" or at-sign "@" allowed in name
  or alias" (the user's own statement of the naming rule and the geolocation exception)

## What changed (summary to expand into the docs)

1. **Name versus tag.** `name` is free text for humans. `tag` is a short, strict key for machines,
   used to build composed names and, later, file names in the machine learning steps (parquet,
   images, pickle). Analogy: the name is the label on the jar, the tag is the barcode.
2. **New columns:**
   - `tag TEXT UNIQUE NOT NULL` on `observation.dataset`, `observation.campaign`,
     `observation.sampling_log` and `observation_utility.provision`. All four are **globally**
     unique; sampling_log used to be unique only per campaign.
   - `tag` on `observation.observation_log`
   - `datetime_tag TEXT NOT NULL` on `observation.campaign`
3. **Separators are reserved.** `_` and `@` join parts of composed names so that the names can
   later be split again.
   - Any parameter `name`, `alias` or `tag` containing `_` or `@` is **rejected**, unless it
     exactly equals what its `auto_name` rule would produce.
   - Exemptions are listed in `separator_exempt_D` in `structure.py`. Today it holds only
     `manage_geolocation` → `name` (e.g. `fr_lucas@38622598`).
   - Why geolocation is exempt: the geolocation is registered *before* the sample and is the
     unique key for a sample when different data sources (AI4SH) give the same sample different
     names. Without it, the same sample could be registered under two names.
   - Consequence to state in the docs: never use `geolocation_id__geolocation_name` in an
     `auto_name` concat, because it can't be split.
4. **How concat arguments are resolved** (`compose_auto_name`):

   | Argument | Resolved to |
   |---|---|
   | `campaign_territory_id` | territory code of the campaign: `observation.campaign_location.territory_id` → `utility.territory.iso_code_a2_ext` (e.g. `eu`) |
   | `<table>_id__<table>_<column>` (e.g. `dataset_id__dataset_tag`, `campaign_id__campaign_name`, `dataset_id__dataset_alias`) | `<column>` of the record referenced by the process parameter `<table>_id__<table>_name` |
   | anything else (`datetime_tag`, `tag`, `profile_min`, …) | the process parameter value as given |

5. **Finding the record** (`find_record`):
   - The value of `<table>_id__<table>_name` may be the record's **name, tag or alias**. It is
     searched in that order, and columns the table lacks are skipped.
   - The concat suffix decides what is returned (`_name` → name, `_tag` → tag, `_alias` → alias).
     So a user who types the tag where the name was expected still gets the correct name back.
6. **Ambiguity:**
   - If a value matches more than one record, the search is narrowed by the parent records that
     the process also gives. Parents are the table's `<parent>_id` columns, resolved recursively.
     Example: a campaign name is unique only within its dataset, so `dataset_id__dataset_name`
     scopes it.
   - If the value is still ambiguous, the process is rejected with
     `<value> matches N records in <schema.table> - give the tag or add the parent (e.g. <…>) to the process`.
   - A tag always works, because tags are globally unique.
7. **Errors.** Every failed lookup logs `❌ ERROR auto_name: ...` and stops the process; nothing is
   silently skipped any more. List the messages; they are in `structure.py`.
8. **Concat syntax.** The format string must be in single quotes:
   `"concat": "'%s_%s' %(dataset_id__dataset_tag, datetime_tag)"`. An unquoted string fails with
   `can not parse concat`. The old docs example omits this point.
9. **Convention:** `display_name` concats use names and blanks (human readable); `name` and `tag`
   concats follow the separator rules.
10. **The `format` block** on tag parameters (`underscore`, `at`, `capital`, `blank`) is present in
    the JSON definitions but **not yet enforced by any code**. Document it as planned, not as
    working.

### Worked example (LUCAS 2009; verify against the JSON files)

| Level | Concat (name / tag) | name | tag |
|---|---|---|---|
| dataset | (given) | `land use and coverage area frame survey (lucas) topsoil data`, alias `lucas` | `lucas` |
| campaign | `'%s %s' %(dataset_id__dataset_alias, datetime_tag)` / `'%s_%s' %(dataset_id__dataset_tag, datetime_tag)` | `lucas 2009` | `lucas_2009` |
| sampling_log | `'%s %s' %(campaign_id__campaign_name, campaign_territory_id)` / `'%s_%s' %(campaign_id__campaign_tag, campaign_territory_id)` | `lucas 2009 eu` | `lucas_2009_eu` |
| observation_log | `'%s@%s' %(sampling_log_id__sampling_log_name, provision_id__provision_name)` / `'%s@%s' %(sampling_log_id__sampling_log_tag, provision_id__provision_tag)` | `lucas 2009 eu@foss xds rca` | `lucas_2009_eu@xds` |

(`xds` is an illustrative provision tag; take the real one from the LUCAS provision data.)

### Practical requirements to document

- **Load order.** Parents must exist, with tags, before children are auto named: dataset → campaign
  (including its `territory_id__territory_name`, which writes `campaign_location`) → sampling_log →
  observation_log.
- **Parameter presence.** The `<table>_id__<table>_name` parameter must be defined in the process
  for the lookup to work. A parent can only scope an ambiguous search if its parameter is in the
  process.
- **Existing databases.** New columns need a DB rebuild or `ALTER TABLE`; `CREATE TABLE` does not
  alter existing tables.
- **Explicit names.** Records that previously gave composed names explicitly (e.g.
  `lucas_eu_2009@...`) are now rejected unless they match the auto name. Set them to `auto` or omit
  them.

## Task 1: new page after "Process options"

- Create `_setup_processes/define_process_auto_naming.md`, following the front matter pattern of
  `define_process_options.md`:
  - `title: "Automatic naming and tags"`
  - `permalink: /setup_processes/auto_naming/`
  - `sidebar: nav: "setup_processes"`
  - today's `date` and `last_modified_at`
- Add it to `_data/navigation.yml` under `setup_processes`, directly after "Process options".
  Check whether `nav_order` → `setup_processes` in `_config.yml` should list it too (it currently
  doesn't list `process_options` either; follow whatever the site actually uses).
- Suggested sections. Keep these anchors stable, because the lucas and ai4sh docs will link to them:
  - `#name-and-tag`: why there are two identifiers, and future use for ML file naming
  - `#tag-columns`: tables, constraints, `datetime_tag`, the `format` block (not yet enforced)
  - `#reserved-separators`: the `_`/`@` rule, the exact-match exception, `separator_exempt_D`, the
    geolocation rationale
  - `#concat-arguments`: resolution table, quoting, display_name convention
  - `#record-lookup`: name → tag → alias, returned column, ambiguity and parent scoping, the
    `campaign_territory_id` special case
  - `#worked-example`: the LUCAS chain above with the real JSON `auto_name` blocks
  - `#errors`: the error messages and what to do about each
  - `#requirements`: load order, parameter presence, DB rebuild, explicit names
- Use real JSON snippets copied from the LUCAS `setup_processes` files (shorten with a `"sketch"`
  node as the existing page does).

## Task 2: small update to `define_process_options.md` (Process options)

In the section `## Automatic naming`:

- The prose says "an `autonaming` block"; the key is `auto_name`. Fix it.
- State that the format string must be quoted, and fix the trailing display block
  (`#%s_%s_%s#  %(...)`) to show the quoted form.
- The example concat `'%s_%s_%s' %(project_id__project_name, event_name, campaign_date)` joins
  free-text names with `_`. Under the new rules, `project_id__project_name` returns the stored
  project *name* (possibly containing blanks). Either switch the example to
  `project_id__project_tag`, or keep it and note that tags are the recommended input for `name` and
  `tag` concats.
- Add a short paragraph and link: "For tags, reserved separators, lookups via name/tag/alias and a
  full worked example, see [Automatic naming and tags](/setup_processes/auto_naming/)". Use the
  site's link style.
- Update `excerpt` if needed and `last_modified_at`.

## Task 3: check other pages

`grep -rn -i 'auto_name\|"auto"' _setup_processes _setup_db` and check:

- `_setup_processes/define_process.md`
- `_setup_db/schemas_tables.md`
- `_setup_db/introduction.md`

Update where they describe the observation tables without the new `tag`/`datetime_tag` columns, or
describe auto naming in the old way. Ignore `.claude/CLAUDE_version2_20260907.md`.

## Done when

- The new page renders locally (`jekyll_serve.txt` in `notes/`) and appears in the sidebar after
  Process options.
- All anchors listed in Task 1 exist.
- Process options links to the new page.
- Nothing in the docs claims the `format` block is enforced.
