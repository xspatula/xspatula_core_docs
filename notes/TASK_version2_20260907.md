# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository. This repo is the new documentation for the sibling repo `setup_core_db`. The initial building of this repo is in the file `.claude/CLAUDE_version1_20260902.md`.

In this session I want to add one section in the markdown file `_setup_processes/define_process_options.md`.

## Collection _setup_processes

### define_process_options.md

Add a section on how the object "default_value" works - i.e. it kicks in when the object "required" is true and the user do not enter a custom value. Put this section between the sections on "Minmax ranges" and "inherit".

In that new section also explain the 2 special cases when "default_value" is set to either "auto" or "inherit" and the process includes the associated objects of "auto_name" / "inherit". Refer to the following 2 sections that details how "inherit" and "auto_name" function.
