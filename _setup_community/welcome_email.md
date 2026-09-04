---
title: "Welcome Email"
layout: single
sidebar:
  nav: "setup_community"
excerpt: "What a newly registered user actually receives: their user name and a randomly generated password, with no self-service way to change or recover it."
permalink: /setup_community/welcome_email/
author_profile: false
date: 2026-08-15 08:00:00 +0200
last_modified_at: 2026-08-15 08:00:00 +0200
---

Once [Register notebook][setup_community_register_notebook]'s step 4 runs, every accepted row in `user.xlsx` gets exactly one email, generated from the template in `src/community/registration.py`:

**Subject:**

```
Your xspatula account
```

**Body:**

```
Hello %s,

An account has been created for you.

  user name: %s
  password:  %s

Please keep this password somewhere safe - there is currently no self-service way to
change or recover it, so contact the administrator if you need it reset.
```

The three `%s` placeholders are filled with the row's `first_name` (falling back to `user_name` if `first_name` was left blank), `user_name`, and the freshly generated plaintext password — the only place that plaintext password is ever shown to anyone. It is not logged, not written anywhere else in cleartext, and not shown back to the admin running the notebook — only the bcrypt hash is stored in `community.user.password`.

## No self-service password reset

This is stated plainly in the email itself because it's a real limitation: there is currently no way for a user to change or recover their own password. It's a deliberate scope decision for workshop-scale use, not an oversight — if a user loses their password, the admin has to reset it by hand (see [Bootstrap user][setup_community_bootstrap_user] for the same `hash_password.py` + `UPDATE community.user` mechanism used to seed the first account, which also works here). Otherwise, the generated password is permanent.

[setup_community_register_notebook]: /setup_community/register_notebook/
[setup_community_bootstrap_user]: /setup_community/bootstrap_user/
