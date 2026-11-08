# Docs update: credentials via .netrc, initial_user, role attributes, pg_hba.conf

Written 2026-11-08 from a session in `xspatula_lucas` (log: `xspatula_lucas/notes/for_claude_pg_user_credentials_20261108.md`). The code changes are in `xspatula_core` (reference copy for these docs) and in the lucas/ai4sh/xspectre repos. Update the pages listed below. Keep the existing page structure and front matter, and bump `last_modified_at`.

## Why (the story the docs must carry)

Plaintext passwords in scheme files were committed and pushed to two **public** repos: the postgres superuser, all 7 pg_users and the bootstrap user's login. On top of that, the Homebrew PostgreSQL default `pg_hba.conf` was `trust`, so the server accepted *any* password. The new system has three rules, and the docs should state them plainly and early:

1. **No password ever goes in a scheme file or a seed JSON.** Every credential lives in `~/.netrc` and is referenced by a `*_netrc_id`.
2. **The database must actually check passwords**: `pg_hba.conf` set to `scram-sha-256` for TCP connections, and the config reloaded.
3. **Hashing is automatic.** The bootstrap user's bcrypt hash is computed at setup time from `.netrc`. No manual `hash_password.py` step and no hash in the repo.

## What changed in the code (facts to document)

### 1. pg_users from .netrc
- `db_users` entries can now be `{"user_netrc_id": "<machine>", "role": "<role>"}`. The netrc `login` becomes the PostgreSQL role name; an explicit `"user_id"` overrides it.
- The old `{"user_id", "password", "role"}` still works, but prints `⚠️ plaintext password in scheme file`.
- Resolved in memory by `Resolve_db_users()` (`setup_db_initiate.py`), only on setup (not delete), and fail-fast: if any entry can't be resolved, nothing is set up.
- **Deleting a database never needs pg_user passwords.** Roles are server-wide (shared by every database on the server) and are not dropped with a database. The delete scheme only needs `host_netrc_id`.
- Generated `.env` files under `src/postgres/environment/` are now chmod 600. They are still plaintext and gitignored.

### 2. Role passwords and attributes are re-applied on every setup run
- `Create_db_roles` (`setup_db.py`) now runs, for every role that already exists: `ALTER ROLE <user> WITH <attributes> PASSWORD <pw>`.
  - Password: from `.netrc`. **To change a pg_user password: edit `.netrc`, re-run the setup cell.** That's all.
  - Attributes: new `Role_attributes()` reads the `CREATE USER … WITH` clause in `roles_grants.json` and sets all of SUPERUSER / CREATEDB / CREATEROLE / LOGIN / REPLICATION / BYPASSRLS explicitly (listed → on, not listed → `NO…`). `roles_grants.json` is now authoritative; manual `ALTER ROLE … NOLOGIN` etc. is undone on the next setup run.
- The password is sent as a bound parameter, never formatted into SQL, and is no longer part of the `grants_hash` stored as the role comment. (Consequence: the first setup run after upgrading re-applies all grants once. Harmless.)
- Correct the existing text in `_building/pg_users.md` that says roles/passwords are only set on creation.

### 3. initial_user: automatic bootstrap user from .netrc
- New optional top-level scheme key (sibling of `postgresdb`):
  ```json
  "initial_user": {"user_netrc_id": "my_initial_user", "email": "...", "first_name": "...", ...}
  ```
- In the seed file `setup/zzz/<project>/setup_db/json/community/user_records_v10_sql.json`, the bootstrap user's `password` value is the placeholder **`"{initial_user}"`**.
- At setup, `Resolve_initial_user()` (`setup_db_initiate.py`) reads `.netrc`, bcrypt-hashes the password **in memory**, and `Apply_initial_user()` (`setup_db.py`) fills the placeholder row:
  - netrc `login` → `user_name` (→ `email` if the login contains `@`)
  - any other keys in `initial_user` override the seed row's columns (email, names, organisation, territory, stratum_code)
- No `initial_user` in the scheme → the placeholder row is **skipped** with a warning (the placeholder is never stored as a password).
- Other seed rows with real bcrypt hashes are untouched. In core: `ini_cat5` uses the placeholder (netrc `xspatula_ini_cat5`); `user_manager` and `jane_doe` keep their hashes.
- Limitation to state: every placeholder row gets the same identity, so use one placeholder row per seed file.
- **The same netrc entry is used to log in afterwards**: the use scheme's `user_project.user_netrc_id` points to it. One entry, nothing to remember.
- `setup/hash_password.py` still exists. Re-scope it to: resetting a password on a **live** database (`UPDATE community.user SET password = '<hash>' …`) and seeding additional non-bootstrap users. It's no longer the main route.

### 4. Use schemes: user_netrc_id alone is enough
- `Project_login` (`src/lib/login.py`) now fills `user_project.user_name` from the logged-in `community.user` record. A use scheme only needs `"user_project": {"user_netrc_id": "<machine>"}`. Previously `user_name` was also required (it's used as `creator` when registering processes).

### 5. .netrc parse errors no longer leak passwords
- One malformed line breaks *every* lookup in the file (Python's `netrc` parses the whole file). The old error message quoted the offending token, which often is a password, and it ended up in notebook output. `User_netrc_credentials` now prints only the file and line number.
- Common mistake to document: a line missing the `password` keyword (`machine x login y secret`).
- Also document: an editor that has `.netrc` open can save over entries added by setup tooling in the meantime. Close it, or reload it before saving.

### 6. Guard rails
- `*_pswd.json` is gitignored (the files that exist only to hold a plaintext password).
- `setup/git_hooks/pre-commit`: blocks commits of JSON files with a plaintext `"password"`/`"pswd"` value (empty, `{initial_user}` and `$2…` bcrypt hashes are allowed). It's not automatic, so document the install:
  ```bash
  cp setup/git_hooks/pre-commit .git/hooks/ && chmod +x .git/hooks/pre-commit
  ```

### 7. Templates (new, tracked, no secrets)
- `setup/zzz/scheme_template_setup.json`: superuser, 7 pg_users and initial_user, all via netrc ids
- `setup/zzz/scheme_template_use.json`: `user_project.user_netrc_id` only

Show the setup template in full on the pages below, together with the matching `.netrc` block:

```
machine my_pg_superuser   login postgres_superuser password <long-random-1>
machine community_admin   login community_admin    password <long-random-2>
machine login_evaluation  login login_evaluation   password <long-random-3>
machine user_cat_1        login user_cat_1         password <long-random-4>
machine user_cat_2        login user_cat_2         password <long-random-5>
machine user_cat_3        login user_cat_3         password <long-random-6>
machine user_cat_4        login user_cat_4         password <long-random-7>
machine user_cat_5        login user_cat_5         password <long-random-8>
machine my_initial_user   login first_user         password <long-random-9>
```
Rules: one entry per line; all three keywords `machine`/`login`/`password` on every line; no spaces or `#` inside a password; `chmod 600 ~/.netrc`. Generate passwords with `python3 -c "import secrets; print(secrets.token_urlsafe(18))"`. Machine names are global for the whole server (the roles are shared by every database on it), so all projects on one server share the pg_user entries.

## pg_hba.conf: required or not?

**Not always, but you must check it.** It depends on how PostgreSQL was installed:

| Install | Default for localhost TCP | Action |
|---|---|---|
| Homebrew (`brew install postgresql@17`) | `trust` | **Must edit** |
| Postgres.app (macOS) | `trust` | **Must edit** |
| EDB installer (Windows/macOS) | `scram-sha-256` | Check only |
| Debian/Ubuntu `apt` | `scram-sha-256` (TCP), `peer` (socket) | Check only |

Document the procedure:
1. **Check** (as the superuser): `SELECT type, database, address, auth_method FROM pg_hba_file_rules;` If any `host` row says `trust`, passwords are not checked at all.
2. **Order matters.** Set all role passwords first (`.netrc` + setup run; superuser: `psql … -c "\password <superuser>"`), *then* switch authentication. Otherwise you lock yourself out.
3. **Edit** `pg_hba.conf` (`SHOW hba_file;` gives the path; Homebrew: `/opt/homebrew/var/postgresql@17/pg_hba.conf`). Change **all four** `host` lines (`all` and `replication`, for both `127.0.0.1/32` and `::1/128`) to `scram-sha-256`. Leave the `local` lines as `trust` as a socket fallback (`psql -U <superuser> -d postgres` without `-h`).
4. **Reload**: `psql -U <superuser> -d postgres -c "select pg_reload_conf()"`
5. **Verify the reload happened** (this was missed in practice): `select pg_conf_load_time();` must show the current time. Then test that a *wrong* password is rejected over TCP.
6. Homebrew note: `postgresql@17` is keg-only, so `psql` isn't on PATH. Add `export PATH="/opt/homebrew/opt/postgresql@17/bin:$PATH"` to `~/.zshrc` (also needed for `pg_dump`/`pg_restore` in dump/restore).

Also tell readers to keep `listen_addresses = 'localhost'` unless remote access is intended.

## Pages to update

| Page | Change |
|---|---|
| `_setup_db/netrc.md` | Expand from 23 lines into the main credentials page: why (leak story, kept generic), entry format and rules, the full example block above, parse-error pitfall, chmod 600, password generation, editor-overwrite pitfall, list of every `*_netrc_id` key the framework reads (`host_netrc_id`, `db_users[].user_netrc_id`, `initial_user.user_netrc_id`, `user_project.user_netrc_id`) |
| `_setup_db/introduction.md` | "Hidden files and security": lead with the rule "no passwords in scheme files". Replace the setup scheme example (lines ~84–135, plaintext `password` for superuser and pg_users) with the template. Replace "Hashing user passwords" (~148–175) with a short pointer to initial_user; keep `hash_password.py` only for live resets. Delete scheme (~177–200): drop the `password` line, `host_netrc_id` only |
| `_setup_db/postgres.md` | New section "Securing authentication (pg_hba.conf)": the table and the 6 steps above, plus the PATH note |
| `_framework/scheme_file.md` | "Setup database credentials" (~77–120): netrc first, plaintext as discouraged legacy. "Define pg_users" (~121–196): replace the 7 `"password": "...-difficult-password"` entries with `user_netrc_id`. New section "initial_user". "Login credentials" (~213–260): `user_netrc_id` alone is enough |
| `_building/initial_user.md` | Rewrite: Step 1 add a netrc entry; Step 2 put the `{initial_user}` placeholder in the seed row; Step 3 add `initial_user` to the setup scheme; Step 4 run setup; Step 5 log in with the same netrc id. Move the hash CLI to "If the database already exists" |
| `_building/pg_users.md` | `db_users` with `user_netrc_id`; `.env` files are chmod 600; the "Editing and revoking role grants" section: passwords and attributes are re-applied on every run, `roles_grants.json` is authoritative, manual `ALTER ROLE` gets reverted; to change a password, edit `.netrc` and re-run |
| `_setup_community/bootstrap_user.md` | "Fresh rebuild" now = `initial_user` + placeholder (no delete/rebuild needed to change the password: edit `.netrc`, but note the seed only inserts if the row doesn't exist, so for an existing user use the live `UPDATE` route) |
| `_setup_processes/scheme_file.md` and `user_scheme_file.md` | Remove the plaintext `"password": "armchair…"` examples or mark them legacy; show `scheme_template_use.json`; state that `user_name` is optional with `user_netrc_id` |
| `_setup_db/dump_restore.md` | Restore resolves `db_users` from `.netrc` before re-creating roles; needs `psql`/`pg_dump` on PATH (Homebrew note) |
| `_data/navigation.yml` | Optional: rename the ".netrc" entry to ".netrc & credentials", and add "Securing PostgreSQL" under Setup DB |

Input-file tables on each page: add `scheme_template_setup.json`, `scheme_template_use.json`, `setup/git_hooks/pre-commit`, `~/.netrc`; mark `hash_password.py` as optional.

## Things to NOT put in the docs
- No real machine names, logins or passwords from Thomas's setup (`karttur`, `thomasg_*`, `xspatula_ini_cat5`, etc.). Use the generic `my_*` names from the templates.
- Don't name the repos that leaked or explain how. "Passwords committed to a public repository are compromised, even after deletion; change them" is enough.

## Verify after editing
- `bundle exec jekyll build` (see `notes/jekyll_serve.txt`) without broken links
- `grep -rn '"password"' _*/ ai4sh/` shows only legacy-marked examples or the `{initial_user}` placeholder

## Work log

### 2026-11-08 (done)
- `_setup_db/netrc.md`: rewritten as main credentials page (3 rules, format, full block, rules, netrc-id table, changing passwords, pitfalls, guard rails, legacy, input files).
- `_setup_db/postgres.md`: new "Securing authentication (pg_hba.conf)" section (install table, 6 steps, listen_addresses).
- `_setup_db/introduction.md`: security section leads with no-passwords rule; setup scheme = template; hashing section → "Passwords for application users" (pointer to initial_user, hash_password.py live-only); delete scheme host_netrc_id only.
- `_framework/scheme_file.md`: netrc-first credentials, db_users via user_netrc_id, new initial_user section, login with user_netrc_id only, input table.
- `_building/initial_user.md`: rewritten (5 steps + live-DB route).
- `_building/pg_users.md`: user_netrc_id, fail-fast, .env chmod 600, passwords/attributes re-applied every run, roles_grants.json authoritative.
- `_setup_community/bootstrap_user.md`, `_setup_processes/scheme_file.md`, `_setup_processes/user_scheme_file.md`: plaintext examples removed, template_use shown.
- `_setup_db/dump_restore.md`: real superuser name removed from example → host_netrc_id; db_users resolved from .netrc after pg_restore (verified in dump_restore_db.py); Homebrew PATH note.
- Extra (not in list, found by grep): `ai4sh/setup_ai4sh_db.md` had 7 passphrase-style pg_user passwords → netrc ids; `_user_data/introduction.md` plaintext use scheme → netrc.
- `_data/navigation.yml`: ".netrc & credentials" + "Securing PostgreSQL".
- Verified: jekyll build OK; all new links/anchors resolve (pre-existing broken: /about/, user_data ../single_step_insert/).

### 2026-11-08, follow-up: links and stale file names (done)
- Root-relative markdown links (`/building/...` etc.) were 404 on the live site (baseurl `/xspatula_core_docs` is not added to markdown links). 81 links in 30 files converted to `{{ "/path/" | relative_url }}` (code blocks untouched).
- Relative-depth fixes: `_user_data/introduction.md` (`../../` → `../`, single_step_insert → `./`), `_framework/vscode.md` (`../setup_db/` → `../../setup_db/`).
- Anchors: `_setup_db/schemas_tables.md` → `#passwords-for-application-users` (text updated to `{initial_user}` route); `ai4sh/edit_ai4sh_db.md` → `/setup_db/#default-human-users`.
- Stale names `scheme_xspatula_local_setup/use.json` → `scheme_local_setup/use.json` in `_auditing/setup.md`, `_framework/notebook.md`, `_setup_db/introduction.md`, `_setup_processes/introduction.md`; table descriptions say netrc ids.
- Nav "About" (`../about/`) resolves to https://xspatula.github.io/about/ (200) in production; left as is.
- Verified: production and local builds, 0 broken content links/anchors (checker: scratchpad check.py).
- For code repo: `setup/scripts/verify_audit_system.py` default scheme is `scheme_ai4sh_local_setup.json` (absent in core); `project_example/scheme_xspatula.json` carries `user_name` + a personal netrc machine name.

## Update instructions 2026-11-08 (part 3): restore order changed in the code

Written from a session in `xspatula_lucas`. Your previous note was correct: `Restore_database` used to resolve `db_users` from `.netrc` *after* `pg_restore`. That order was a bug (a missing `.netrc` entry left a freshly restored database without roles), so the **code was changed**, not the docs' description. Now pushed to `xspatula_lucas`, `xspatula_core` and `xspatula_ai4sh` (`setup/src_setup/lib_setup/dump_restore_db.py`).

### New order in `Restore_database`
1. Load scheme, check the dump file exists.
2. **If `db_users` is set: resolve every entry from `.netrc` (in memory). If any entry fails → abort with `❌ ERROR - db_users not resolved - nothing dropped or restored`. The existing database is untouched.**
3. Connect; if the database exists, ask to confirm, then `DROP DATABASE ... WITH (FORCE)`.
4. `CREATE DATABASE` (empty).
5. `pg_restore --no-owner --jobs=4`.
6. If `db_users` is set: rewrite the per-role `.env` files in `src/postgres/environment/`, then create/grant any role missing on this server (`Setup_prod_roles`).

### Edit `_setup_db/dump_restore.md`
- **"Restore" numbered list (~lines 106–115):** add a new step 1 before the confirm/drop step: "If `db_users` is set in the scheme file, its entries are resolved from `.netrc` first. If one fails, the restore stops before anything is dropped." Renumber the rest. Rewrite the old step 4 (now step 5) to: "If `db_users` is set, the per-role `.env` files in `src/postgres/environment/` are rewritten and any role missing on this server is created and granted." — remove the parenthesis "(if one fails, roles are not created, but the restored database stays)".
- The sentence after the list ("expect `role ... does not exist` errors for the GRANTs; step 4 fixes them") → "step 5 fixes them".
- **Scheme settings table, `db_users` row (~line 68):** "resolved from `.netrc` before the roles are re-created" → "resolved from `.netrc` before anything is dropped; a missing entry aborts the restore with the database left as it was".
- Optional tip under Restore: test the `.netrc` entries are complete before a real restore, e.g. by running the setup cell of `setup_db.ipynb`, which resolves the same `db_users`.

### Also update the earlier sections of this note
- Section "Pages to update", row `_setup_db/dump_restore.md`, and the 2026-11-08 work log entry ("db_users resolved from .netrc after pg_restore (verified in dump_restore_db.py)") describe the old order. Leave the log as history, but add a line to the work log when this is done.

### Verify
- `grep -n "after pg_restore\|restored database stays\|step 4 fixes" _setup_db/dump_restore.md` → no hits.
- `grep -rn "Restore_database\|pg_restore" _* ai4sh --include=*.md` → check no other page states the old order (`_setup_db/postgres.md` only has the PATH note; no change expected).
- `bundle exec jekyll build` without broken links.

### 2026-11-08, part 3 done (docs side)
- `_setup_db/dump_restore.md`: Restore list renumbered; new step 1 = resolve `db_users` from `.netrc`, abort before drop (verified: `Resolve_db_users` line 334 before `DROP DATABASE` line 361 in `dump_restore_db.py`); step 5 = .env + roles; "step 5 fixes them"; `db_users` table row updated; receiver bullet in "Sharing" notes the early check.
- Suggested tip NOT used as written: the setup cell of `setup_db.ipynb` asks "Set up database?" *before* `Resolve_db_users` (setup_db_initiate.py ~266/275), and if all entries resolve it proceeds with a full setup. Replaced with a read-only python snippet that prints OK/MISSING per netrc id (tested against scheme_template_setup.json).
- Verify greps clean; only `_setup_db/postgres.md` mentions pg_restore (PATH note, no change).
- For code repo (optional): a "check credentials only" mode (resolve host/db_users/initial_user, then exit) would make a safe built-in pre-flight check for both setup and restore.
