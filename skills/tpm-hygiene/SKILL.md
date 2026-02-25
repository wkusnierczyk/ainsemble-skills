# TPM Hygiene Conventions

This skill defines the project hygiene standards used by the TPM agent to audit and triage GitHub repositories.

## GitHub Project

Every repo should have an associated GitHub Project (Projects V2). The project name should reflect the repo in title case — e.g., repo `aigent-skills` gets project **Aigent Skills**.

### Tabs

The project should have three views: **Board**, **Roadmap**, **Table**. These cannot be created via `gh` — flag for manual setup if missing.

### Project Fields

The project needs these fields/settings:

#### Issue types (org-level, not a project field)

GitHub issue types are a **built-in org-level feature**, not a project single-select field. They appear as the "Type" column in projects automatically. They are managed under **org Settings > Issue types**, not via project field APIs.

Expected issue types:

| Type      | Maps from labels                                    |
|-----------|-----------------------------------------------------|
| Feature   | `type:feature`                                      |
| Bug       | `type:bug`                                          |
| Task      | `type:task`, `type:cleanup`, `type:documentation`   |

To set an issue's type, use the `updateIssue(issueTypeId:)` GraphQL mutation — not `item-edit --field-id`. The CLI's `field-list` and `field-create` commands do not see or manage issue types.

#### Priority field (project single-select)

A custom single-select field on the project.

| Option | Color                | Maps from label     |
|--------|----------------------|---------------------|
| P0     | `#D93F0B` (red)      | `priority:high`     |
| P1     | `#FFA500` (orange)   | `priority:medium`   |
| P2     | `#FBCA04` (yellow)   | `priority:low`      |

#### Status field (project single-select)

Replaces the default project Status options (Todo, In Progress, Done) with:

| Option      | Color                | Replaces        |
|-------------|----------------------|-----------------|
| Pending     | `#1D76DB` (blue)     | Todo            |
| Progressing | `#FBCA04` (yellow)   | In Progress     |
| Completed   | `#0E8A16` (green)    | Done            |

**Warning**: Renaming single-select options via the `updateProjectV2Field` mutation clears existing values on all items. After renaming options, you must re-set the Status value on every item. Always flag this for user confirmation before executing — it is destructive.

Every issue, PR, and milestone in the project must have a Status value set.

### Agent Behavior

The agent should:
1. Check if a project exists for the repo (via `gh project list --owner {owner}`)
2. If not, create one with `gh project create --owner {owner} --title "{Title Case Repo Name}"`
3. Verify the **Priority** field exists as a project single-select. If missing, flag for manual creation.
4. Verify **Status** field options are Pending/Progressing/Completed (not the defaults Todo/In Progress/Done). If wrong, flag for manual rename **and warn that renaming clears existing values**.
5. Verify **issue types** (Feature, Bug, Task) exist at the org level. If missing or wrong, flag for manual setup under org Settings > Issue types. Do not attempt to create them via project field APIs.
6. Ensure every open issue, every open PR, and every milestone is added to the project
7. When triaging issues, set their issue type (via GraphQL `updateIssue`), Priority, and Status
8. When triaging PRs, set their Status to Progressing and ensure they have an assignee

## Milestone Structure

Every issue should be assigned to a milestone. Milestones represent workstreams, not sprints.

### Milestone titles

Every milestone must have a meaningful title starting with a `M{NN}: ` prefix, where `NN` is a zero-padded two-digit number starting at `01`.

Examples: `M01: Authentication & Identity`, `M02: Connections`, `M03: Monetization`

When creating a new milestone, use the next available number. When renaming existing milestones that lack the prefix, preserve the original title and prepend the prefix. Flag milestones with missing or malformed prefixes.

### Milestone due dates

Every milestone must have a due date. When creating a new milestone:
1. Check existing milestones and find the one with the latest due date
2. If no milestones exist, use today's date
3. Add 2 days to that date
4. Use the resulting date as the new milestone's due date

## Issue Specs

### Label Taxonomy

Every issue should have exactly **one type label**, **one priority label**, and **one or more focus labels**.

#### Type Labels

| Label                | Color                    | Description            |
|----------------------|--------------------------|------------------------|
| `type:bug`           | `#D93F0B` (red)          | Bug fix                |
| `type:feature`       | `#0E8A16` (green)        | New feature            |
| `type:task`          | `#1D76DB` (blue)         | Task or chore          |
| `type:cleanup`       | `#BFD4F2` (light blue)   | Refactoring or cleanup |
| `type:documentation` | `#0075CA` (dark blue)    | Documentation          |

#### Priority Labels

| Label              | Color                 | Description               |
|--------------------|-----------------------|---------------------------|
| `priority:high`    | `#D93F0B` (red)       | Must be addressed soon    |
| `priority:medium`  | `#FFA500` (orange)    | Important but not urgent  |
| `priority:low`     | `#FBCA04` (yellow)    | Nice to have              |

#### Focus Labels

Focus labels describe **what area** an issue relates to. They use the `focus:` prefix and `#C5DEF5` (light periwinkle) color.

Examples: `focus:ux`, `focus:reliability`, `focus:security`, `focus:performance`, `focus:accessibility`, `focus:api`, `focus:data-model`

This list is **not exhaustive**. The agent should read the issue content and create new `focus:*` labels as needed. Reuse existing focus labels when appropriate — check `gh label list` before creating a new one. Keep names lowercase, hyphenated, and concise.

#### Optional Labels
- `duplicate` — already tracked elsewhere
- `wontfix` — intentionally not addressing

### Issue Title Qualifiers

Every issue title must start with a **qualifier prefix** in square brackets. The qualifier identifies what the issue is about at a glance.

#### Standard qualifiers (match type labels)

| Qualifier | Use when               |
|-----------|------------------------|
| `[FEAT]`  | Feature work           |
| `[BUG]`   | Bug fix                |
| `[TASK]`  | Task, cleanup, or docs |

#### Domain qualifiers (more specific)

When an issue relates to a specific cross-cutting concern, use a domain qualifier instead. Domain qualifiers may include a numeric suffix to group related issues around the same problem — the number is **not** the issue ID.

Examples:
- `[SEC-01]` — security problem 1 (multiple issues can share `[SEC-01]` if they address the same vulnerability)
- `[SEC-02]` — a different security problem
- `[PERF]` — performance improvement (no number needed if standalone)
- `[PERF-03]` — performance problem 3 (grouped with other `[PERF-03]` issues)
- `[A11Y]` — accessibility
- `[INFRA]` — infrastructure / DevOps
- `[DATA]` — data model or migration
- `[I18N]` - internationalization

This list is **not exhaustive**. The agent should choose or create qualifiers that best describe the issue. Keep them short (3-5 chars), uppercase, optionally with a `-NN` suffix for grouping.

#### Agent behavior

- Flag issues missing a qualifier prefix
- When proposing label fixes, also propose a title rename adding the appropriate qualifier
- Use `gh issue edit {number} --title "[QUAL] Original title"` to fix

### Issue due dates

Every issue within a milestone must have a due date.
- If an issue has no due date, set it to the milestone's due date
- If an issue has a due date **later** than the milestone's due date, set it to the milestone's due date
- If an issue has a due date **earlier** than the milestone's due date, leave it as is

### Triage Rules

An issue is considered **triaged** when it has ALL of:
1. Title starts with a `[QUALIFIER]` prefix
2. At least one `type:*` label
3. At least one `priority:*` label
4. At least one `focus:*` label
5. A milestone assignment
6. A due date (at or before the milestone's due date)
7. An assignee (if unassigned, assign to current user via `gh api user --jq .login`)
8. Added to the GitHub Project with Type, Priority, and Status fields set

An issue is **untriaged** if it is missing any of the above.

## Pull Request Specs

### PR Project Membership

Every open PR must be added to the GitHub Project, just like issues.

### PR Status

New PRs must have their project Status set to **Progressing**. When a PR is merged, its status should move to **Completed**.

### PR Assignee

- New PRs must be assigned to the current user (`gh api user --jq .login`).
- When auditing existing PRs, any PR without an assignee must be assigned to the current user.

### PR Labels

A PR's labels must match the labels of the issues it addresses. To determine which issues a PR addresses:

1. Parse the PR body for `Closes #N`, `Fixes #N`, or `Resolves #N` references.
2. Fetch labels from each referenced issue.
3. Collect the union of all `type:*`, `priority:*`, and `focus:*` labels from referenced issues.
4. The PR should carry all of these labels. If a referenced issue's milestone is known, the PR should also carry the corresponding labels.

If a PR has no issue references, flag it — PRs should always reference the issues they close.

### PR Body Format

Every PR body must follow this structure:

````markdown
# Summary

{optional paragraph stating the overall goal}

* {Work item done}
  Closes #{issue}
  Closes #{issue}

* {Another item done}
  Closes #{issue}

...

# Testing strategy

## Automated

- [ ] {whatever test run is valid}
...

## Manual

- [ ] {short description of the test}
```
code block showing what to do if needed
```
{manual testing instructions}
````

#### Body rules

- The **Summary** section lists each discrete work item as a bullet, with `Closes #N` directives indented under the relevant bullet.
- Every addressed issue must appear as a `Closes #N` line. Do not use `Fixes` or `Resolves` — use `Closes` consistently.
- The **Testing strategy** section has two subsections: **Automated** (CI/test commands) and **Manual** (human verification steps). Both are checklists.
- Manual test items may include fenced code blocks showing commands or steps to reproduce.

### PR Triage Rules

A PR is considered **triaged** when it has ALL of:
1. Added to the GitHub Project
2. Project Status set to **Progressing**
3. An assignee (if unassigned, assign to current user)
4. Labels matching the union of its referenced issues' labels
5. Body follows the required format (Summary + Testing strategy sections)

A PR is **untriaged** if it is missing any of the above. Flag untriaged PRs in the health report.

## Branch & Worktree Hygiene

- Merged branches should be deleted (both local and remote)
- Feature branches should reference an issue number (e.g., `feat/42-auth-flow`)
- Bug branches should reference an issue number (e.g., `bug/2-fix-typo`)
- All other branches should reference an issue number (e.g., `dev/13-move-harness`)
- Long-lived branches diverged 30+ days from main should be flagged

### Branch naming

| Scope         | Pattern               | Example                                             |
|---------------|-----------------------|-----------------------------------------------------|
| Milestone     | `dev/mNN-{name}`      | `dev/m01-user-interface` (for M01: User Interface) |
| Feature issue | `feat/{id}-{name}`    | `feat/32-cancel-button`                             |
| Bug issue     | `bug/{id}-{name}`     | `bug/110-authentication`                            |
| Fix issue     | `fix/{id}-{name}`     | `fix/55-typo-in-header`                             |
| Task issue    | `task/{id}-{name}`    | `task/78-update-deps`                               |
| Other/unclear | `dev/{id}-{name}`     | `dev/13-move-harness`                               |
| Subagent work | arbitrary (local only)| `agent/m01-sidebar-layout`                          |

- `{id}` is the GitHub issue number
- `{name}` is a short kebab-case description
- For milestone branches, `NN` is the zero-padded milestone number

### Merge flow

```
subagent branches (local only)
  └──merge──▶ development branch (pushed to remote)
                └──PR──▶ main
```

- Subagent branches are merged into the development branch locally
- Subagent branches are never pushed to remote
- Only the development branch is pushed and used for a PR against main
- Chunks of work that don't capture whole issues (e.g., work split across multiple subagents) use arbitrary local branch names

### Worktree cleanup

Subagents and agent teams may work in git worktrees (typically under `.claude/worktrees/`). When pruning merged branches:
- Run `git worktree list` to find active worktrees
- If a worktree's branch has been merged or deleted, it is stale
- Propose `git worktree remove {path}` for stale worktrees, then `git worktree prune` to clean up
- Never remove a worktree without user confirmation

## Work Plan Review

The TPM may be asked to review a work plan. When reviewing, enforce the following branching and workflow rules.

### Branching rules

1. **Never push or merge directly to main.** The plan must explicitly state this. All new development happens on a development branch.
2. Only the development branch may be pushed to remote for a PR. Subagent branches stay local.

### Review checklist

A work plan **must explicitly state its branching strategy**. This is not optional — a plan without a clear branching section is incomplete. The plan should name:
- The development branch (with correct naming convention)
- Which branches subagents will use
- That main is protected (no direct pushes or merges)
- The merge flow (subagent → dev branch → PR → main)

**Reject or request revision** if:
- The plan has no branching strategy section
- The plan does not explicitly forbid pushing/merging to main
- Branch names don't follow the naming conventions
- Subagent branches are planned to be pushed to remote
- Multiple agents plan to push to the same remote branch
- The merge flow is unclear or missing

## Staleness Thresholds

| Item                       | Stale After | Action                       |
|----------------------------|-------------|------------------------------|
| Issue with no activity     | 7 days      | Flag for review              |
| Open PR                    | 2 days      | Flag for review or close     |
| Merged branch not deleted  | 0 days      | Should be deleted immediately |

## Code TODOs

`TODO`, `FIXME`, and `HACK` comments in source code should be tracked as GitHub issues. The TPM agent scans for these and flags any that don't have a corresponding open issue.

## Work Log

The TPM agent must log all its work to `.dev/tpm.md` in the repo root. This file is for the user's review and should never be committed.

- Create `.dev/` directory if it doesn't exist
- Ensure `.dev/` is in `.gitignore` (add it if missing)
- Append each run as a timestamped section:
  ```
  ## YYYY-MM-DD HH:MM — {action summary}
  - what was audited
  - what was flagged
  - what fixes were proposed
  - what fixes were executed (with gh command and result)
  ```
- The log is append-only — never overwrite previous entries

## Health Report Format

The TPM agent produces a structured health report covering these areas:
1. **Project Setup** — project exists, has correct views and fields
2. **Untriaged Issues** — missing labels, milestones, or project field values
3. **Milestone Health** — milestones without due dates, empty milestones
4. **Stale Items** — issues and PRs past staleness thresholds
5. **PR Triage** — PRs missing project membership, assignee, labels, or body format
6. **Branch Hygiene** — merged branches not deleted, stale feature branches
7. **Code TODOs** — untracked TODO/FIXME/HACK comments
