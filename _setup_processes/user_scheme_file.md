#### Scheme file for all other processing

Once you have setup your database you should avoid using the postgres superuser for managing access and other privileges in your new database. Instead you log in as an ordinary application user, typically the initial user created at setup. The scheme file for using the database does not need information on the postgres database cluster, only a `.netrc` reference for the user. The shipped template is `setup/zzz/scheme_template_use.json`:

```json
{
  "project_path": "./my_project",
  "user_project": {
    "user_netrc_id": "my_initial_user"
  },
  "process": [
    {
      "execute": true,
      "verbose": 1,
      "overwrite": false,
      "delete": false
    }
  ]
}
```

The corresponding entry in `~/.netrc` (the same one `initial_user` uses at setup):

```
machine my_initial_user   login first_user         password <long-random-9>
```

`user_name` is optional: it is filled from the logged-in `community.user` record. A legacy `"user_name"` + `"password"` pair in `user_project` still works, but never commit it.

To learn more, see the documents on [scheme_file][scheme_file] and [.netrc][netrc].
