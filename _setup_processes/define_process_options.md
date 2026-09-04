---
title: "Define process options"
layout: single
sidebar:
  nav: "setup_processes"
excerpt: "Process parameters can be set to default values, defined to only accept a set of predefined values or restricted to numerical ranges, inherit values from existing database records or assigned automatic values"
permalink: /setup_processes/process_options/
author_profile: false
date: 2026-06-09
last_modified_at: 2026-06-09
---

The process setup_processes has additional options for facilitating the settings of parameter values when called, including accepting only a predefined set of values, restricting of numerical ranges, inherit values from existing database records or assigning automatic name values.

## Set value

When defining a string or integer type parameter in an `add_process` process, the definition can be accompanied with the block `set_value` that define the only accepted value entries.

```json
{
  "process": [
    {
      "process": "add_process",
      "overwrite": false,
      "parameters": {
        "root_process": "components",
        "process": "add_electronic_components",
        "min_user_stratum": 5,
        "title": "Add electronic components",
        "label": "Insert, update or delete an electronic component."
      },
      "nodes": [
        {
          "parent": "process",
          "element": "parameters",
          "parameter": [
            {
              "parameter": "name",
              "parameter_type": "text",
              "required": true,
              "default_value": "",
              "hint": "Name of electronic component",
              "schema_table": {
                "schema": "component",
                "table": "electronic",
                "write": true
              },
              "permission": {
                "update": true,
                "delete": false
              },
              "set_value": [
                {
                  "value": 10,
                  "label": "pcb"
                },
                {
                  "value": 20,
                  "label": "mcu"
                },
                {
                  "value": 30,
                  "label": "led"
                }                
              ]
            }
          ]
        }
      ]
    }
  ]
}
```

## Minmax ranges

Numerical parameters can in a similar manner be confined to only accept values within a range defined by a `minmax` block.

```json
{
  "process": [
    {
      "process": "add_process",
      "overwrite": false,
      "parameters": {
        "root_process": "compostion",
        "process": "add_reference_components",
        "min_user_stratum": 5,
        "title": "Add reference composition",
        "label": "Insert, update or delete a reference composition."
      },
      "nodes": [
        {
          "parent": "process",
          "element": "parameters",
          "parameter": [
            {
              "sketch": "only outlining the parameter array object parameter",
              "stuff": "value"
            },
            {
              "parameter": "fraction",
              "parameter_type": "float",
              "required": true,
              "default_value": "",
              "hint": "Fraction of stuff in reference sample",
              "schema_table": {
                "schema": "stuff",
                "table": "standard_reference",
                "write": true
              },
              "permission": {
                "update": true,
                "delete": false
              },
              "minmax": [
                {
                  "min": 0,
                  "max": 1
                }                   
              ]
            }
          ]
        }
      ]
    }
  ]
}
```

## Inherit

When defining a process you can set a parameter to be inherited from an existing record in another table. For example when defining a contact person for a particular sub task you can set the parameter to by default inherit the contact name for the main task. This is done in an `inherit` block:

```json
{
  "process": [
    {
      "process": "add_process",
      "overwrite": false,
      "parameters": {
        "root_process": "contact",
        "process": "add_sub_task_contact",
        "min_user_stratum": 5,
        "title": "Add sub task contact",
        "label": "Insert, update or delete a sub task contact."
      },
      "nodes": [
        {
          "parent": "process",
          "element": "parameters",
          "parameter": [
            {
              "parameter": "contact_name",
              "parameter_type": "text",
              "required": false,
              "default_value": "inherit",
              "hint": "Name of contact person for sub task",
              "schema_table": {
                "schema": "contact",
                "table": "sub_task",
                "write": true
              },
              "permission": {
                "update": true,
                "delete": false
              },
              "inherit": {
                "process_parameter": "contact_name",
                "src_schema": "contact",
                "src_table": "task",
                "src_column": "contact_name",
                "search_column": "id",
                "search_object": "contact_id__contact_name"
              }
            }
          ]
        }
      ]
    }
  ]
}
```

For the inherit function to kick in, the parameter must be set to its default value `inherit`. As the parameter is not required (`required: false`), omitting the parameter from the JSON process will automatically use the inherit function. Setting another value than `inherit` overrides the inherit function and the set value will be used.

## Automatic naming

Parameters that are composed of other parameters in the same process definition can be set to automatic naming by adding an `autonaming` block. This is useful for creating human readable records when a foreign key (FK) is used for defining links to other tables.

```json
{
  "process": [
    {
      "process": "add_process",
      "overwrite": false,
      "parameters": {
        "root_process": "manage_events",
        "process": "add_campaign",
        "min_user_stratum": 5,
        "title": "Add event campaign",
        "label": "Insert, update or delete an event campaign."
      },
      "nodes": [
        {
          "sketch": "only outlining the parameter array object parameter",
          "project_id__project_name": "value"
        },
        {
          "sketch": "only outlining the parameter array object parameter",
          "event_name": "value"
        },
        {
          "sketch": "only outlining the parameter array object parameter",
          "campaign_date": "value"
        },
        {
          "parent": "process",
          "element": "parameters",
          "parameter": [
            {
              "parameter": "name",
              "parameter_type": "text",
              "required": false,
              "default_value": "auto",
              "hint": "Name of the campaign",
              "schema_table": {
                "schema": "event",
                "table": "campaign",
                "write": true
              },
              "permission": {
                "update": false,
                "delete": false
              },
              "auto_name": {
                "concat": "'%s_%s_%s'  %(project_id__project_name, event_name, campaign_date)"
              }
            }
          ]
        }
      ]
    }
  ]
}
```
For generating an automatic name, the parameter must be set to its default value (`auto`). As the parameter is not required (`required: false`), omitting the parameter from the JSON process will use automatic naming. Setting another value than `auto` overrides the automatic naming and the set value will be used.

In addition, the parameters `project_id__project_name`, `event_name` and `campaign_date` must be other parameters defined for the same process.

The actual automatic naming is defined by the string
```
#%s_%s_%s#  %(project_id__project_name, event_name, campaign_date)
```
that is translated to python code when running the process.
