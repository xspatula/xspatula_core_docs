---
title:  ".netrc - keeping your logins hidden"
layout: single
sidebar:
  nav: "setup_db"
excerpt: "A .netrc file is a simple way to store login details (like usernames and passwords) so applications can automatically authenticate to servers without asking you every time. The .netrc file is hidden in your home directory on you local machine."
permalink: /setup_db/netrc/
author_profile: false
date:   2026-03-15 16:13:03 +0200
last_modified_at:   2026-03-29 21:13:20 +0200
---

A .netrc file is a simple way to store login details (like user names and passwords) so applications can automatically authenticate to servers without asking you every time. The .netrc file is hidden in your home directory on you local machine.

## Creating a .netrc file

If you want to use a .netrc file to access postgreSQL, create a file named .netrc in the home directory of your machine and enter the credentials by stating _machine_, _login_ and _password_:

```
machine postgresql login <superuser> password <passphrase>
```

Where _machine_ is the code word you send to .netrc for the resource you want to access, and _login_ and _password_ are the credentials on the resource you are trying to enter.
