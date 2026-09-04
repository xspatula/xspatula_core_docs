---
title:  "Edit AI4SH DB"
layout: single
sidebar:
  nav: "ai4sh"
excerpt: "The AI4SH database is defined using a set of JSON structured process files. The syntax for creating schemas and tables and also inserting data in selected table is based on SQL."
permalink: /ai4sh/edit_ai4sh_db
author_profile: false
date:   2026-03-22 06:13:03 +0200
last_modified_at:   2026-03-22 06:13:03 +0200
---

The AI4SH database is defined using a set of JSON structured process files. The syntax for creating schemas and tables and also inserting data in selected table is based on SQL.

## Prerequisites

You need to clone or download (or fork) the [GitHub][github] framework_project_ai4sh_database for getting access to the JSON files defining the database. To access the project you need to send a request to join that repo and then click [here][github_framework_project_ai4sh_database]. [TO BE COMPLETED]

## Edit the AI4SH database structure

In the AI4SH project, the definition of the database is under the path

```
./ai4sh_db/setup_db
```

with following content:

```
.
├── db_setup.txt
└── jsonsql
    ├── community_create_grant_db_user_new_v10_sql.json
    ├── community_create_grant_db_user_v10_sql.json
    ├── community_user_organisation_records_v10_sql.json
    ├── community_user_organisation_v10_sql.json
    ├── observation_campaign_v10_sql.json
    ├── observation_dataset_v10_sql.json
    ├── observation_geolocation_v10_sql.json
    ├── observation_measurement_v10_sql.json
    ├── observation_observation_log_v10_sql.json
    ├── observation_observation_v10_sql.json
    ├── observation_organisation_person_v10_sql.json
    ├── observation_sample_v10_sql.json
    ├── observation_sampling_log_v10_sql.json
    ├── observation_utility_apparatus_v10_sql.json
    ├── observation_utility_classification_v10_sql.json
    ├── observation_utility_indicator_default_unit_v10_sql.json
    ├── observation_utility_indicator_v10_sql.json
    ├── observation_utility_juxtaposition_v10_sql.json
    ├── observation_utility_license_v10_sql.json
    ├── observation_utility_location_method_v10_sql.json
    ├── observation_utility_method_translate_v10_sql.json
    ├── observation_utility_method_v10_sql.json
    ├── observation_utility_monitor_v10_sql.json
    ├── observation_utility_preparation_v10_sql.json
    ├── observation_utility_preservation_v10_sql.json
    ├── observation_utility_profiling_v10_sql.json
    ├── observation_utility_provider_v10_sql.json
    ├── observation_utility_provision_indicator_v10_sql.json
    ├── observation_utility_provision_v10_sql.json
    ├── observation_utility_quantity_default_unit_v10_sql.json
    ├── observation_utility_quantity_v10_sql.json
    ├── observation_utility_setting_v10_sql.json
    ├── observation_utility_spatial_reference_v10_sql.json
    ├── observation_utility_storage_v10_sql.json
    ├── observation_utility_territory_v10_sql.json
    ├── observation_utility_transportation_v10_sql.json
    ├── observation_utility_unit_translate_v10_sql.json
    ├── observation_utility_unit_v10_sql.json
    ├── processes_records_v10_sql.json
    ├── processes_v10_sql.json
    ├── schema_v10_sql.json
    └── utility_v10_sql.json
```

## pilot file db_setup.txt

The single file in the root of the directory, db_setup.txt, is a [pilot][pilot_file] that lists JSON files to execute. If you want to only run a selection of JSON [process][process_file] files, you simply put a hashtag (#) in front of the the JSON process files to exclude.

Illustration for the first lines in the pilot file db_setup.txt.

```
#################################################
##### DEFINE AI4SH DB SCHEMAS AND TABELS #####
#################################################
#!!!!! The schemas must be defined before the tables
#!!!!!
### SCHEMAS ###

schema_v10_sql.json

### PROCESSES ###

processes_v10_sql.json

processes_records_v10_sql.json

### COMMUNITY - REQUIRED FOR SETUP AND LOGIN ###

community_user_organisation_v10_sql.json

community_user_organisation_records_v10_sql.json

### GENERAL UTILITIES ###

utility_v10_sql.json

### OBSERVATION UTILITIES ###

observation_utility_apparatus_v10_sql.json
...
...
```

## Generic process files

The first JSON process file to call must be the one that does the setup of the schemas you require (schema_v10_sql.json). The order of the remaining process files also has some importance and if you change the order some processes will crash. Note that for setting up the database there is no internal check of the SQL syntax prior to executing any SQL command.

The second process file in the pilot file is processes_v10_sql.json, that defines the [default tables required for defining any process][schemas_tables_default_processes]. Editing this table will cause subsequent changes in the whole framework construction.

The third process, processes_records_v10_sql.json, populates the tables defining processes with the information required for setting up any other process. Editing the content of that JSON process file will also require rewriting large chunks of the framework code.

The 4th and 5th process files in the pilot sets up and populates the tables for organisations and users in the schema community. As explained in the [framework synopsis][framework_synopsis_user], for logging in to the framework a user must be registered in the table user. It is thus recommended that you edit the JSON process file community_user_organisation_records_v10_sql.json, and add at least a single user with your own definitions, as detailed [here][framework_table_insert].

The 6th process file, utility_v10_sql.json also defines and populates generic tables that are similar across all databases built by the framework. Changing them will require rewriting the framework code.

## AI4SH database related process files

The specific AI4SH database structure is defined in two schemas:
- observation_utility, and
- observation.

### Observation utilities

The table below is a summary of the tables in the schema observation_utility.

| table | objective |
| :====== | :====== |
| apparatus | definition of generic data acquisition systems (instrument, service, tool, sensor, sensory organ) |
| order | The highest, most generic, classification level used for defining datasets, campaigns and samples |
| family | Mid classification level used for defining datasets, campaigns and sample |
| genus | Lower classification level used for defining datasets, campaigns and samples |
| species | Lowest classification level used for defining samples |
| specimen | individual sample labeling |
| indicator_default_unit | Default units for all indicators |
| indicator | soil status indicator with defined quantitative meaning |
| juxtaposition | attributes used for defining spatial neighbourhood setting |
| license | licences under which data is shared or accessed |
| location_method | methods used for spatial geolocation |
| method_translate | formulas for translating recorded values between different methods |
| method | description of methods, including international standard codes if applicable |
| monitor | the level of skills applied for analysis |
| preparation | sample preparation before analysis, if not part of a method standard |
| preservation | sample preservation between acquisition and analysis |
| profiling | profiling method and units recorded for 3D samples |
| provider | instrument, organisation, service or tool providing data |
| provision | data from provider supplied for a particular dataset or campaign |
| provision_serial_nr | if the serial number of an applied individual provision is known |
| provision_indicator | the indicator(s) that a particular provision is reporting including method and unit |
| quantity_default_unit | default units of all quantities |
| quantity | definition of quantities |
| setting | systems for defining juxtapositioning |
| spatial_reference | Spatial reference system applied for geospatial reference |
| storage | sample storage method between acquisition and analysis |
| territory | extended ISO codes for territories |
| transportation | sample transportation method between acquisition and analysis |
| unit |  units used for reporting values, distance, proximity etc |
| unit_translate | formulas for translation between defined units |

### Observation

The table below is a summary of the tables in the schema observation.

| table | objective |
| :====== | :====== |
| organisation | organisations related to datasets, campaigns and sampling/observation logs |
| person | individuals responsible for different parts; e.g. sampling, observation and data clearing |
| dataset | datasets available in the database |
| campaign | assembly of data collected and analysed as part of a dataset |
| sampling_log | log describing location and methods for soil sampling events|
| sample | individual samples acquired as part of soil sampling events |
| geolocation | geospatial coordinates for individual samples, if applicable |
| observation_log | log with identified analysis provision applied |
| observation | connects individual samples with a provision, quantity and method |
| measurement | the value(s) for a defined observation and indicator |

## Process structure for create table

The framework process for defining a table is <span class='sub_process'>create_table</span>. The process requires three parameters (arguments):

- schema (the name of the schema where to create the table),
- table (the name of the table)
- command (array of the columns to create in SQL conformed syntax).

The example below shows the command for creating the table <span class='db_table'>unit</span>:

```
{
  "process": [
    {
      "process_id": "create_table",
      "overwrite": false,
      "delete": false,
      "parameters": {
        "schema": "observation_utility",
        "table": "unit",
        "command": [
          "id SERIAL",
          "name TEXT UNIQUE",
          "alias TEXT UNIQUE",
          "abstract TEXT",
          "system TEXT",
          "PRIMARY KEY (id)"
        ]
      }
    }
  ]
}
```

The individual elements in the array can include any syntax that is compatible with postgreSQL.

If you want to change the definition on a table in the framework database, open the process file defining the table you want to change and edit the schema, table, or the elements in the command array. Then set overwrite to true and rerun the notebook setup_db, this will create, or drop and recreate, the table. Remember to reset overwrite to false once your table has the structure you wanted.

## Acknowledgments and Funding

This work was partly done as part of the AI4SoilHealth project, funded by the European Union's Horizon Europe Research and Innovation Programme under Grant Agreement No. 101086179.

_Funded by the European Union. The views expressed are those of the authors and do not necessarily reflect those of the European Union or the European Research Executive Agency._


[framework]: ../setup_db/

[vscode]: ../framework/vscode/

[postgres]: ../setup_db/postgres/

[anaconda]: ../setup_db/anaconda/

[netrc]: ../setup_db/netrc/

[notebook]: ../framework/notebook/

[scheme_file]: ../framework/scheme_file/

[scheme_pg_user]: ../framework/scheme_file/#define-pg_users

[job_file]: ../framework/job_file/

[pilot_file]: ../framework/pilot_file/

[process_file]: ../framework/process_file/

[schemas_tables]: ../setup_db/schemas_tables/

[schemas_tables_default_processes]: ../setup_db/schemas_tables/#default-processes

[framework_synopsis_user]: ../setup_db/schemas_tables/#default-human-users

[framework_table_insert]: ../setup_db/schemas_tables/#table-insert

[github]: https://github.com/

[github_framework_project_ai4sh_database]: https://github.com/AI4SH/framework_project_ai4sh_database
