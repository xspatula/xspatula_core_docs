# Notes for updating `xspatula_ai4sh_docs`

Written after merging the `auditing` and `setup_community` collections into `xspatula_core_docs`
(2026-09-02). Nothing in `xspatula_ai4sh_docs` was touched by this session.

## Key finding: no content actually needed transferring

The task (`.claude/CLAUDE.md`) called for transferring the `auditing` and `setup_community`
collections *from* `xspatula_ai4sh_docs` into `xspatula_core_docs`. But `setup_core_db_docs`
already had its own `_auditing/` and `_setup_community/` collections — file-for-file the same six
pages `xspatula_ai4sh_docs` has — and diffing them showed `setup_core_db_docs`'s versions are
**newer and already generalized** (mtimes 2026-08-24/25/26 vs. `xspatula_ai4sh_docs`'s
2026-08-15/21) — a prior session evidently did this exact generalization already, using the
AI4SH-specific pages as a starting point. Examples of what changed:

- `ai4sh/user_management/organisation/excel/organisation.xlsx` → generic
  `project_example/user_management/organisation/excel/organisation.xlsx`
- `setup/zzz/ai4sh/setup_db/json_ai4sh/...` paths → generic
  `setup/zzz/xspatula/setup_db/json_core/...`
- Cross-links to `[dataset_meta]: /dataset_meta/` → `[user_data]: /user_data/` (the collection
  name in the generic/core site)
- AI4SH-specific audit-coverage numbers (7 schemas incl. `landscape`, `observation`,
  `observation_utility`, `landscape_utility`) → generic guidance ("full coverage" vs.
  "`UPDATE`/`DELETE`-only" rule, with the framework's own default install as the only concrete
  example)
- `_setup_community/introduction.md` and others reworded to talk about "an xspatula project"
  rather than the AI4SH workshop specifically

So the copy simply **kept `setup_core_db_docs`'s versions as-is** — pulling the older,
AI4SH-specific `xspatula_ai4sh_docs` wording back in would have been a regression. No file
content from `xspatula_ai4sh_docs`'s `_auditing/`/`_setup_community/` ended up in
`xspatula_core_docs`.

## What this means for `xspatula_ai4sh_docs` going forward

Per the parent goal in `.claude/CLAUDE.md` ("rewrite `xspatula_ai4sh_docs` and replace the full
documentation in that repo with links back to the new core docs repo"), the natural next step
for *this* sibling — not done in this session, needs your go-ahead — is:

1. Delete (or stub with a redirect/pointer page) `xspatula_ai4sh_docs/_auditing/` and
   `_setup_community/`, since their content now lives, generalized, at `xspatula_core_docs`'s
   `/auditing/` and `/setup_community/`.
2. Update `xspatula_ai4sh_docs/_data/navigation.yml` to link those top-navbar entries at the
   `xspatula_core_docs` site instead of local collection pages.
3. **Don't lose the AI4SH-specific facts** that were dropped when generalizing — if still wanted,
   they belong as a short AI4SH-specific addendum/page, not folded back into the generic pages:
   - The 7 audited schemas and their tier (`community`, `process`, `utility`,
     `observation_utility`, `landscape_utility` = full coverage; `observation`, `landscape` =
     `UPDATE`/`DELETE`-only)
   - The `ai4sh` database name used directly in the copy-paste SQL on the queries page
   - `setup/zzz/ai4sh/setup_db/json_ai4sh/...` as the concrete AI4SH path, and
     `db_xspatula_ai4sh_audit.txt` as the concrete generated-pilot-file name
   - `ai4sh/user_management/...` as the concrete Excel/notebook paths for community setup

## Also check before deleting anything there

`xspatula_ai4sh_docs/for_claude_changes_20260821.md`,
`for_claude_setup_db_updates_20260821.md`, and `notes/claude_plan_setup_db_updates_20260821.md`
(dated the same week as its `_auditing`/`_setup_community` pages) may record in-progress or
planned work on those exact collections — read them first so this consolidation doesn't clobber
something already mid-flight.
