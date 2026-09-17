# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository. This repo is the new documentation for the sibling repo that used to be under the path `setup_core_db`, but I changed the path to `xspatula_core` (to reflect the more advanced code repos `xspatula_lucas` and `xspatula_ai4sh` and their documentation paths `xspatula_lucas_docs` and `xspatula_ai4sh_docs`). The initial building of this repo is in the file `.claude/CLAUDE_version1_20260902.md`, the second in `.claude/CLAUDE_version2_20260907.md`

## Session task 1

In this session I want to implement the story order solution that is already in the sibling repo `xspatula_lucas_docs` under `_data/story_order.yml`. This solution defines the links in the page bottom buttons "Previous" and "Next" to follow the story line set from the collection rather than being arbitrary.

## Session task 2

For the simpler setup of the core database, I still think we need a short page of Foreing keys. Take the page  `_lucas_2009/foreign_key_explained.md` as a starting page but only retain the section on "How foreign keys normally resolve" and use the Foregin key "territory_id__territory_name" from the table definition in `setup/zzz/xspatula/setup_db/json/community/organisation_records_v10_sql.json`, with the key set in  to `setup/zzz/xspatula/setup_db/json/utility/territory_v10_sql.json` to explain the default Foreign key solutions. Then just mention that a more advanced function exists and link to `https://xspatula.github.io/xspatula_lucas_docs/lucas_2009/foreign_key_explained/`.

Put the new Foreign_key page as a REFERENCE, the way it is done for `xspatula_lucas_docs`. Do not forget the new page in the `_data/story_order.yml`.
