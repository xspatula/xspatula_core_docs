---
title:  "AI4SH project synopsis"
layout: single
sidebar:
  nav: "ai4sh"
excerpt: "The EU funded project AI4SoilHealth (AI4SH) database is built for holding soil data derived both from field sampling and satellite images, with a large number of different instruments, laboratories, sensors and methods being used for capturing the data. To accommodate this data with FAIR principles the framework was used to build a comprehensive postgreSQL database."
permalink: /ai4sh/
author_profile: false
date:   2026-03-22 16:13:03 +0200
last_modified_at:   2026-03-22 16:13:03 +0200
---

The EU funded project AI4SoilHealth (AI4SH) database is built for holding soil data derived both from field sampling and satellite images, with a large number of different instruments, laboratories, sensors and methods being used for capturing the data. To accommodate this data with [FAIR (Findability, Accessibility, Interoperability, and Reuse)][fair] principles the Xspatula framework was used to build a comprehensive postgreSQL database.

## Prerequisits

To create your own database using the framework, you must first setup a dedicated postgreSQL database as described in the [framework][framework] section of this site. This section will assume that you have already done that, and that setup adhered to the default settings of the framework.

## Outline of the AI4SH postgreSQL database

The AI4SH postgres database incudes 6 schemas:
- utility,
- community,
- process,
- observation_utility, and
- observation.

NOTE THAT IS NOT FINAL, THERE MIGHT BE MORE OR LESS

### Utility schema

The utility schema contains support tables for general information used across several tables and processes. This is a default framework schema.

### Community schema

The community schema contains tables for organisations and users. All users logging into the system must be registered in the table user. This is a default framework schema.

### Process schema

The process schema contains all the processes defined for the AI4SH database. This is a default framework schema.

### Observation_utility schema

The observation_utility schema contains generic data required for fulfilling the FAIR principles of data access, including data on quantities, units, methods, positioning, profiles and much more. The data in the observation utility schema pertain to most types of soil data and is not particular for the data collected within the AI4SH project.

### Observation schema

The observation schema is where data on actual soil properties are stored, but it also contains tables that store information on campaigns, and logs for sampling events and observations.

## Schematic flow chart for data entry

For entering data on soil properties the flow chart below illustrates what is required to fulfil the data requirement of FAIR principles and how they are reflected in the database. If the chain of data is broken at a single step, the data can not be entered into the database.

TO BE COMPLETED

## Building the AI4SH database

To build the AI4SH database, follow the sequential instructions in the following documents:

- [setup the AI4SH database][setup_db]

## Acknowledgments and Funding

This work was partly done as part of the AI4SoilHealth project, funded by the European Union's Horizon Europe Research and Innovation Programme under Grant Agreement No. 101086179.

_Funded by the European Union. The views expressed are those of the authors and do not necessarily reflect those of the European Union or the European Research Executive Agency._

[fair]: https://www.go-fair.org/fair-principles/

[framework]: ../setup_db/

[vscode]: ./vscode/

[postgres]: ./postgres/

[anaconda]: ./anaconda/

[netrc]: ./netrc/

[notebook]: ./notebook/

[scheme_file]: ./scheme_file/

[job_file]: ./job_file/

[pilot_file]: ./pilot_file/

[process_file]: ./process_file/

[schemas_tables]: ./schemas_tables/
