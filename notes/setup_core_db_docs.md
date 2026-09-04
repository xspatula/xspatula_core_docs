# Notes for updating `setup_core_db_docs`

Written after copying `setup_core_db_docs` → `xspatula_core_docs` (2026-09-02). Nothing in
`setup_core_db_docs` was touched by this session — everything below is a recommendation for a
separate pass on that repo, not a record of changes already made there.

## What was copied

The full repo, minus `.git/`, `_site/`, `.jekyll-cache/`, `.DS_Store` (build artifacts, not
content). Two files were then edited **only in the copy** to reflect the new repo identity:

- `_config.yml` — `baseurl` and `repository` changed from `setup_core_db_docs` to
  `xspatula_core_docs`.
- `README.md` — title, live-site link, content-section table (added the six collections beyond
  `framework`/`setup_db` that `setup_core_db_docs` already ships but its old README didn't
  mention), and repository-structure tree.

Also added to the copy only, not present in `setup_core_db_docs`:

- `.gitignore` (`_site/`, `.jekyll-cache/`, `.sass-cache/`, `.bundle/`, `vendor/`,
  `Gemfile.lock`, `.DS_Store`) — modeled on `xspatula_ai4sh_docs`'s.

## LICENSE mismatch — do not copy this file forward

`setup_core_db_docs`'s root `LICENSE` file is a BSD 3-Clause license with copyright holder
"suelolum" — not the MIT/xspatula license its own `README.md` claims ("Code: MIT License"). This
looks like leftover cruft (possibly from the Minimal Mistakes theme template) rather than the
license actually intended for this repo. `xspatula_core_docs` kept its own correct MIT/xspatula
`LICENSE` and did **not** copy this one over. Worth fixing at the source — either restore the MIT
license there too, or confirm BSD/suelolum is deliberate and update the README to match.

## Two things worth fixing in `setup_core_db_docs` itself

1. **No `.gitignore`.** `_site/` and `.jekyll-cache/` (thousands of generated cache files) are
   currently tracked in git. `xspatula_ai4sh_docs` already has a sane `.gitignore` — worth
   copying that pattern back.

2. **`_config_local.yml` has stale, unrelated cruft.** It defines a `collections:` /
   `nav_order:` block for `in-situ_methods` (`chruby-installation`, `ruby-installation`, etc.) —
   leftover from an unrelated template, not part of the Xspatula docs. Because Jekyll deep-merges
   `--config _config.yml,_config_local.yml`, this silently registers a bogus `in-situ_methods`
   collection on every local `jekyll serve`. Removed it in the `xspatula_core_docs` copy; worth
   removing at the source too.

## Orphaned content: `ai4sh/` top-level pages

`setup_core_db_docs` (and now the copy) ships a top-level `ai4sh/` directory — `synopsis.md`,
`setup_ai4sh_db.md`, `insert_ai4sh_db.md`, `edit_ai4sh_db.md` — with `sidebar: nav: "ai4sh"`
front matter, but `_data/navigation.yml` has no `ai4sh:` nav list and no top-navbar entry links
to it. These pages build (Jekyll treats any non-underscore directory with front-matter files as
plain pages) but are unreachable through site navigation, and their content is AI4SH-specific —
out of place in what's meant to be the generic "core" framework documentation. Left untouched in
the copy pending your call on whether to delete them, fold them into `xspatula_ai4sh_docs`
instead, or wire them into `_data/navigation.yml`.

## Ongoing relationship

`xspatula_core_docs` is now a fork/copy of `setup_core_db_docs`'s generic content
(`framework`, `setup_db`, `setup_processes`, `user_data`, `auditing`, `setup_community`,
`building`), not a symlink or submodule — the two repos will drift unless kept in sync by hand.
Until you decide `setup_core_db_docs` should itself become a thin pointer at
`xspatula_core_docs` (mirroring the plan for `xspatula_ai4sh_docs` — see the parent
`.claude/CLAUDE.md`), any future generic-content edit made in one should be ported to the other.
