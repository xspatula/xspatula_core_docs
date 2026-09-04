---
title: "SMTP setup"
layout: single
sidebar:
  nav: "building"
excerpt: "Once you're ready to let people register themselves for your project, the framework needs real SMTP credentials to actually send the generated-password emails — here's how to set that up from a blank slate."
permalink: /building/smtp/
author_profile: false
date: 2026-08-24
last_modified_at: 2026-08-24
---

[Adding organisations and users via Excel][building_add_users] emails each newly-registered user a generated password automatically — nobody, including the admin running the notebook, ever sees it except the recipient. That requires a working outgoing-email connection. Credentials are a secret, not code, so there's nothing to configure until you set it up yourself.

## Create the credential file

The framework looks for SMTP credentials in a specific, gitignored location: `src/postgres/environment/.xspatula_email.env`. This file doesn't exist in a fresh clone — create it by hand, in the same directory your database credential `.env` files already live in.

Its contents, five lines:

```
SMTP_HOST=send.one.com
SMTP_PORT=587
SMTP_USER=you@yourdomain.example
SMTP_PASSWORD=your-mailbox-password
SMTP_USE_TLS=true
```

The values shown (`send.one.com`, port 587, STARTTLS) are one.com's published settings, used here as a working example — any SMTP provider works, these just happen to be what was tested. Fill in your own `SMTP_USER`/`SMTP_PASSWORD` for whatever mailbox you're sending from, and adjust `SMTP_HOST`/`SMTP_PORT`/`SMTP_USE_TLS` if you're using a different provider.

## Test it

Before relying on it inside a notebook, send a one-off test message directly from a Python shell:

```python
from src.community.email import Send_email

Send_email('you@yourdomain.example', 'Test', 'This is a test email.')
```

This returns `True` and the message arrives if everything's configured correctly; it returns `False` and prints an error if something's wrong (missing/misspelled env var, wrong host/port, bad credentials) — fix that before running the registration notebook for real, since a failed send there just means that one user's password never reached them (the account is still created correctly; see [Welcome email][setup_community_welcome_email] for what to do about it).

## More detail

This is the same credential file and the same two functions (`Get_smtp_env_var`, `Send_email`) documented in full on [SMTP email][setup_community_smtp_email], including what actually reads the file and how failures are reported — worth a read if you want to understand what's happening under the hood rather than just copy-paste the credential file.

[building_add_users]: /building/add_users/
[setup_community_smtp_email]: /setup_community/smtp_email/
[setup_community_welcome_email]: /setup_community/welcome_email/
