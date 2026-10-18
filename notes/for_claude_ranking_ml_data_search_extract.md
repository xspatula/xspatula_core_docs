# For Claude: auto naming (campaign/sampling_log swap) and foreign key rules (xspatula_core_docs)

Written 2026-10-14 from a session in `xspatula_lucas`. Work through the tasks below from within this
repo. Everything goes into **existing pages**. Core is the generic framework, so this repo gets the
general rules only:

- the changed auto-naming rules
- the foreign key resolution rules

The domain parts are covered by two sibling notes with the same file name. Link to those repos rather
than repeating them:

- `xspatula_ai4sh_docs/notes/`: the setup database and setup processes pages, ranked land classes, the
  new select processes
- `xspatula_lucas_docs/notes/`: the machine learning selection pages

## Background

The changes were made in `xspatula_lucas` (branch `feature/campaign-place-sampling-time`, commits
`d6608fd` to `15dc587`) and ported to `xspatula_ai4sh` (`a24cee054`). Three generic modules were then
copied into **xspatula_core** (not committed yet):

- `src/lib/structure.py`: auto naming, the same as lucas
- `src/postgres/pg_common.py`: foreign key resolution
- `src/utils/datumtid.py`: accepts `YYYY-MM-DD`

Core's `.gitignore` also had a bare `lib/` that excluded `src/lib/`. It is now `/lib/`, and `src/lib` is
staged. That needs no documentation, unless a page tells users to copy `src/lib` by hand.

Core itself has no `auto_name` rules in its process JSON, so the worked examples stay LUCAS-based, as
they are now. All LUCAS examples below are from the rebuilt, live LUCAS database.

**Sources of truth** (the code wins over this note):

- `../xspatula_core/src/lib/structure.py`: `compose_auto_name`, `find_record`, `null_value`,
  `auto_name_null_D` (line ~740)
- `../xspatula_core/src/postgres/pg_common.py`: `_Convert_id_code`, `_Check_get_foreign_key`
- `../xspatula_lucas/setup/zzz/lucas/setup_processes/json/observation/campaign_v10_sql.json` and
  `sampling_log_v10_sql.json` (the real `auto_name` blocks)
- `../xspatula_lucas/setup/zzz/lucas/setup_db/json/observation/campaign_v10_sql.json` and
  `sampling_log_v10_sql.json` (columns)

## 1. `_setup_processes/define_process_auto_naming.md` (main task)

The time part moved from the campaign to the sampling_log. A campaign is now a place (dataset +
territory + optional site); a sampling_log is one sampling of it at a time.

**Tag columns** (section ~line 24-40):

- `datetime_tag` is now a column of `observation.sampling_log` (`TEXT NOT NULL`; `0` = no date), not of
  `observation.campaign`.
- sampling_log also gained `display_name` and `url`.
- Update the tag-column table and the sentence "`datetime_tag` is the time part of a campaign tag".

**Concat arguments** (section ~line 87-110):

- Remove `campaign_territory_id` (row at line ~103) and the subsection "The campaign territory"
  (~line 147). The territory now comes in as a normal foreign key argument,
  `territory_id__territory_name` / `territory_id__territory_iso_code_a2_ext`.
- **Optional arguments.** An argument ending in `?` (e.g. `site?`) is optional; if it is empty it is
  dropped together with the separator before it.
- **Strict nulls** (`auto_name_null_D`). An empty `territory` or `datetime_tag` is not an error. It gives
  the null text in a name (`no location`, `no date`) and `0` in a tag.
- Any other empty argument is still an error.
- When composing a tag, values given directly as process parameters may only hold `a-z`, `0-9`, `.`
  and `-`.

**Record lookup** (section ~line 115-145): a value is now searched as name, tag, alias **or any other
single-column unique text column** of the table, e.g. `utility.territory.iso_code_a2_ext`. Update the
"Name, tag or alias" subsection and its error message.

**Worked example** (section ~line 151 onward). Rewrite it with the real LUCAS rules and values:

| Level | `name` concat | `tag` concat | name | tag |
|---|---|---|---|---|
| campaign | `'%s %s %s' %(dataset_id__dataset_alias, territory_id__territory_name, site?)` | `'%s_%s_%s' %(dataset_id__dataset_tag, territory_id__territory_iso_code_a2_ext, site?)` | `lucas european union` | `lucas_eu` |
| sampling_log | `'%s %s' %(campaign_id__campaign_name, datetime_tag)` | `'%s_%s' %(campaign_id__campaign_tag, datetime_tag)` | `lucas european union 2009` | `lucas_eu_2009` |
| observation_log | unchanged | unchanged | `lucas european union 2009@foss xds rapid content analyzer` | `lucas_eu_2009@foss-xds-rca` |

- The `display_name` concats are campaign `'%s %s' %(dataset_id__dataset_alias, territory_id__territory_name)`
  and sampling_log `'%s %s' %(campaign_id__campaign_display_name, datetime_tag)`. Verify both in the
  JSON; the LUCAS campaign display name in the DB is `LUCAS EU`.
- The campaign is now registered with dataset, territory (`eu`) and an optional site, and no
  `datetime_tag`.
- The sampling_log is registered with the campaign and `datetime_tag: 2009`.
- LUCAS 2015 is a second sampling_log (`lucas_eu_2015`) of the same campaign.
- Replace the JSON blocks at lines ~202-270 with the real ones.

**Errors table** (section ~line 350):

- Remove the two `campaign_location` / `iso_code_a2_ext` territory messages.
- Update: `<value> not found as name, tag, alias or other unique column in table <table>`.
- Add:
  - `concat <concat> for <parameter> has N placeholders but M arguments`
  - `searching <value> in <schema.table.column> failed: <error>`
  - `<key> = <value> can not be part of a tag (only a-z, 0-9, "." and "-" allowed)`
- Copy the exact wording from `structure.py` (`grep -n "ERROR auto_name" src/lib/structure.py`).

**Requirements** (~line 367): existing databases need a rebuild for the moved `datetime_tag`, because
`CREATE TABLE` does not alter existing tables.

## 2. `_setup_db/foreign_key.md` (short page, extend it)

It currently explains only the table-name convention and links to the LUCAS page for
`utility.foreign_key`. Add a section "Empty and unresolved values" with the rules from `_Convert_id_code`:

- **Empty value** (`''`, `none`, `null`): resolves to id **0**, the "none" row that catalogues such as
  `observation_utility.setting_system` carry (`0`, `no setting`). This is deliberate. Don't describe
  empty as NULL in general.
- **Empty `parent_*` key**: the exception. A self reference to a parent (e.g. `parent_landuse_id` of a
  top rank class) is left out, so the column is NULL.
- **Value not found**: the record stops with
  `❌ ERROR Can not find foreign key value <value> for <parameter>`. Before this fix the unresolved key
  went on to the INSERT, and Postgres reported `column "parent_landuse_id__landuse_alias" ... does not
  exist`, which hid the real cause.
- **Self references** use `utility.foreign_key`, since no table is named `parent_<table>`. Example rows:
  `parent_landcover_id` → `landscape_utility.landcover` (search `alias`, alt `tag`). Link to the ai4sh
  utility page for the domain use.

## 3. `_user_data/single_step_insert.md`

In "Verifying the data" (~line 112), add the two foreign key messages a user may now see:

- the not-found message above
- that an insert-only rerun is safe: existing rows are skipped, and the linked tables of an existing
  record (e.g. `dataset_setting_system`) are still filled in

## 4. Optional: `_framework/` or `_setup_processes/introduction.md`

If a page lists what `src/lib` contains or how core is cloned, check it still matches now that
`src/lib` is tracked.

## Checklist

- [ ] No `campaign_territory_id`, `campaign_location` lookup or campaign `datetime_tag` left
      (`grep -rn "campaign_territory_id\|campaign_location\|lucas_2009_eu\|lucas 2009 eu" _setup_processes _setup_db _user_data`)
- [ ] Optional `?` arguments, strict nulls and "other unique column" lookup described, with examples
- [ ] Worked example rebuilt from the real JSON and the LUCAS DB values above
- [ ] Errors table matches `structure.py` exactly
- [ ] Foreign key page: id 0 for empty, NULL for `parent_*`, error for not found, self reference via
      `utility.foreign_key`
- [ ] Links to the ai4sh setup pages and the LUCAS ML page where the domain details live
- [ ] Build locally (`notes/jekyll_serve.txt`) and check the links

## Done 2026-10-18 (Claude)

- `_setup_processes/define_process_auto_naming.md`: tag table (`datetime_tag` on sampling_log, `0` = no date, + `display_name`, `url`); campaign = place, sampling_log = time; `campaign_territory_id` row and subsection removed; new "Empty arguments" subsection (`?` optional, strict nulls from `AUTO_NAME_NULL_D`, other empty = error); tag charset rule `a-z0-9.-`; lookup now "name, tag, alias or other unique column" + composed-name fallback; worked example rebuilt (`lucas_eu`, `lucas_eu_2009`, `lucas_eu_2015`, display `LUCAS EU`) with the real campaign/sampling_log JSON and the sampling_log call from `lucas_2009_to_xspatula.py`; errors table synced with `structure.py`; requirements: rebuild needed.
- Beyond the note (code is newer): `#` is also a reserved separator, `datetime_tag` is in the separator check and must be `0|YYYY|YYYYMM|YYYYMMDD|YYYYMMDDhhmmss` (error added to table). `_setup_processes/define_process_options.md`: separator sentence now includes `#`.
- `_setup_db/foreign_key.md`: self-reference via `utility.foreign_key` (`parent_landcover_id` row, link to ai4sh utility page) and new section "Empty and unresolved values" (id 0 / NULL for `parent_*` / not-found error).
- `_user_data/single_step_insert.md`: "Verifying the data" lists the not-found FK message and the safe insert-only rerun.
- Item 4: no page lists `src/lib` content or tells users to copy it; no change.
- Local build OK, relative links and anchors checked.
