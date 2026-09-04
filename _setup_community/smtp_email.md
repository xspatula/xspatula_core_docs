---
title: "SMTP Email"
layout: single
sidebar:
  nav: "setup_community"
excerpt: "New users get their password by email, generated automatically — nobody types or is shown a password except the recipient. That requires real SMTP credentials, which don't ship with the repo."
permalink: /setup_community/smtp_email/
author_profile: false
date: 2026-08-15 08:00:00 +0200
last_modified_at: 2026-08-15 08:00:00 +0200
---

Every newly registered user (except the [bootstrap user][setup_community_bootstrap_user]) gets a password that's generated automatically and emailed to them once — nobody, including the admin running the notebook, ever types or sees it except the recipient. That requires a working outgoing-email connection, which needs real SMTP credentials. Credentials are a secret, not code, so they don't ship with the repository.

## Credential file: src/postgres/environment/.xspatula_email.env

This file **does not exist in a fresh clone** — it's gitignored, the same way the database credential files next to it are, and must be created by hand. Its shape:

```
SMTP_HOST=send.one.com
SMTP_PORT=587
SMTP_USER=you@yourdomain.example
SMTP_PASSWORD=your-mailbox-password
SMTP_USE_TLS=true
```

The values above (`send.one.com`, port 587 with STARTTLS) are one.com's published settings, used here as a working example. Any SMTP provider works — these are just what was tested.

This mirrors the existing `src/postgres/environment/.<db>.env` pattern already used for database credentials: same directory, same "never committed" convention.

## What reads this file

`src/community/email.py` is the code that loads it and sends mail:

- `Get_smtp_env_var()` — loads and validates the five settings above from the env file, returning `None` (with an error printed) if the file is missing or a required variable is absent.
- `Send_email(to_addr, subject, body_text)` — sends one plaintext email over SMTP, using STARTTLS or implicit SSL depending on `SMTP_USE_TLS`. Returns `True` if the message was handed off to the SMTP server, `False` on any failure (with the error printed).

Useful to read directly if you want to understand what's actually happening, not just copy-paste the credential file.

## How do I know it worked?

`Send_email` returns `True`/`False` and prints an error on failure, so the quickest check is a manual test send from a Python shell:

```python
from src.community.email import Send_email

Send_email('you@yourdomain.example', 'Test', 'This is a test email.')
```

If that returns `True` and the message arrives, the registration notebook's password emails (see [Welcome email][setup_community_welcome_email]) will work the same way.

[setup_community_bootstrap_user]: /setup_community/bootstrap_user/
[setup_community_welcome_email]: /setup_community/welcome_email/
