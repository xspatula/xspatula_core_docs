#### Scheme file for all other processing

Once you have setup your database you should avoid using the postgres superuser for managing access and other privileges in your new database. Instead you should login as one of the pg_users defined in the _scheme_ file for setting up the database. The scheme file for using the database once it is setup does not need information on the postgres database cluster, it only needs a user_name and password for the new database (xspatula in the examples throughout this site):

```
{
  "project_path": "./xspatula",
  "user_project": {
    "user_name": "ini_cat_5_user",
    "password": "armchair-scouting-windowframe-bicycling"
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

You can replace the user_name and password with a user_netrc_id, and hide your user login credentials in a .netrc file.

```
{
  "project_path": "./xspatula",
  "user_project": {
    "user_netrc_id": "ini_cat_5_user"
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

The corresponding entry in the .netrc for the user ini_cat_5_user as given above becomes:

```
machine xspatula login ini_cat_5_user password armchair-scouting-windowframe-bicycling
```

To learn more, see the documents on [scheme_file][scheme_file] and [.netrc][netrc].
