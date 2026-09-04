# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Purpose

`xspatula_core_docs` is a documentation site written in Markdown using the Jekyll theme Minimal Mistakes (https://mmistakes.github.io/minimal-mistakes/) for the database-integrated Xspatula framework written in python in the sibling repos `setup_core_db` and `xspatula_ai4sh`. Both of the these repos have their own documentation repos `setup_core_db_docs` and `xspatula_ai4sh_docs`. Building yet another xspatula python repo (`xspatula_lucas`) I realised  that it must repeat too much from `xspatula_ai4sh_docs`. I thus want to make an attempt to extend the introductory documentation `setup_core_db_docs` with parts taken from `xspatula_ai4sh_docs` in order to not repeat large parts in every new documentation. The ain of this repo is thus to take the content of `xspatula_core_docs` and extend with as mush generic documentation from `xspatula_ai4sh_docs`. Once that is done I can rewrite `xspatula_ai4sh_docs` and replacing the full documentation in that repo with links back to the new, extended "core" documentation repo.

## Starting point

Take the full repo `setup_core_db_docs` and make copy in this repo `xspatula_core_docs` while checking that internal and external links and references remain intact.

Then continue by transferring the following collections from `xspatula_ai4sh_docs` to this this repo `xspatula_core_docs`:

- auditing
- setup_community

while updating the internal and external references and links.

## Current state (2026-09-02)

This repo is not yet populated — it currently contains only `LICENSE`, `.claude/`, and an empty `notes/` directory. The copy-and-merge described above has not been done yet. Everything below documents the *source* repos (`setup_core_db_docs`, `xspatula_ai4sh_docs`) so the copy/merge can be done correctly; once content lands here, re-derive paths/collections from this repo directly rather than trusting this file.

Sibling repos live alongside this one under `~/GitHub_xspatula/`:

| Repo | Role |
|---|---|
| `setup_core_db_docs` | Copy source for this repo — the generic Xspatula framework docs |
| `setup_core_db` | Python framework documented by `setup_core_db_docs` |
| `xspatula_ai4sh_docs` | Source of the `auditing` and `setup_community` collections to merge in |
| `xspatula_ai4sh` | Python package (AI4SH) documented by `xspatula_ai4sh_docs` |
| `xspatula_lucas_docs` / `xspatula_lucas` | The project that motivated this consolidation (not itself a source to copy from) |

**Do not edit the sibling repos.** Per the Important notes below, capture any change that *should* eventually be back-ported to a sibling as a note under `notes/`, not as a direct edit.

## Site architecture (inherited from `setup_core_db_docs`)

- **Engine**: Jekyll with the Minimal Mistakes theme, pinned via `Gemfile` (`gem "minimal-mistakes-jekyll", "4.27.3"`, local install — not `remote_theme`).
- **Serve locally**: `bundle exec jekyll serve --config _config.yml,_config_local.yml` (the local config overrides `url`/`baseurl` to empty so links resolve at `localhost:4000`).
- **Build**: `bundle exec jekyll build` (CI passes `--baseurl "$base_path"`; see `.github/workflows/jekyll.yml`, which deploys to GitHub Pages on push to `main`).
- **Content = Jekyll collections**, one directory per top-level nav section, each with `output: true` in `_config.yml` and a page order list under `nav_order:`. Every page needs Jekyll front matter (`layout`, `title`, `categories`, `tags`) and `sidebar: { nav: "<key>" }` to appear in the right side nav.
- **Navigation** is hand-maintained in `_data/navigation.yml` (`main_navigation` = top navbar, plus one list per collection for its sidebar) — a copied/merged collection is invisible until it's added here too.
- **Search**: Lunr, client-side, full content (`search_full_content: true`).
- Internal links use Jekyll reference-style links (e.g. `[user_data]`) resolved via link labels embedded in each collection's pages — grep existing pages for the label style before adding new cross-links.
- `_site/` and `.jekyll-cache/` are build output — never hand-edit. **Note**: `setup_core_db_docs` has no `.gitignore` and currently commits these; `xspatula_ai4sh_docs` does ignore them (`_site/` commented out, but `.jekyll-cache/`, `.sass-cache/`, `.bundle/`, `vendor/`, `Gemfile.lock`, `.DS_Store`, `notes/` are ignored). Add a `.gitignore` to this repo modeled on `xspatula_ai4sh_docs`'s rather than perpetuating the untracked-artifacts problem — but don't ignore `notes/` here, since the Important notes section below requires it to be written and kept.

### Collections expected once the copy is done

From `setup_core_db_docs` (`_config.yml` collections + `nav_order`): `framework`, `setup_db`, `setup_processes`, `user_data`, `auditing`, `setup_community`, `building`. Note `setup_core_db_docs` *already has* its own `_auditing/` and `_setup_community/` directories — they are not empty stubs.

From `xspatula_ai4sh_docs`, the two collections named in the task instructions: `_auditing/` (`introduction.md`, `setup.md`, `queries.md`) and `_setup_community/` (`introduction.md`, `bootstrap_user.md`, `excel_intake.md`, `register_notebook.md`, `smtp_email.md`, `welcome_email.md`).

**Important discrepancy to resolve during the merge**: `setup_core_db_docs`'s existing `_auditing/` and `_setup_community/` content is *not* identical to `xspatula_ai4sh_docs`'s — spot-checking `introduction.md` in both collections shows `setup_core_db_docs`'s copy is already more generic/updated (e.g. its auditing intro talks about "an audited database" rather than "the AI4SH database", and its `setup_community` intro already reuses generic "adding user data" language rather than AI4SH-specific framing). Don't blindly overwrite the `setup_core_db_docs` versions with the `xspatula_ai4sh_docs` versions — diff them page by page and merge, since `setup_core_db_docs`'s copy may be the more current/generic one despite the task's phrasing of "transferring" from `xspatula_ai4sh_docs`.

## Important notes

Leave the sibling repos untouched but write instructions for your own (claude) session under the `notes` directory, one for each sibling, that I can use if for updating the content of the siblings.
