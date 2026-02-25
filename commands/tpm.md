---
description: Run a TPM project health audit
agent: tpm
allowed-tools:
  - Bash
  - Grep
  - Glob
  - Read
---

Run a full project health audit on this repository. Check all 6 areas: untriaged issues, milestone health, stale items, open PRs, branch hygiene, and code TODOs. Produce a structured health report and propose fixes.

$ARGUMENTS
