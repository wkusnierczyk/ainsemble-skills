---
description: Decompose a target into a structured work plan with parallel waves
agent: planner
allowed-tools:
  - Bash
  - Grep
  - Glob
  - Read
  - Write
---

Decompose the following target into a structured work plan. Parse the target, gather context, break the work into parallel waves of agent tasks, and write the plan to `.dev/{id}/plan.md`.

Target: $ARGUMENTS
