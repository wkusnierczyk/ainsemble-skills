# Planning Conventions

This skill defines how to decompose a target into a structured work plan. The planner agent uses these conventions to produce `.dev/{id}/plan.md` files.

## Target Types

A target is the thing the user wants planned. It can be a milestone, a GitHub issue, or free-text.

| Input format             | Type       | Derived `{id}`       | Example input → id             |
|--------------------------|------------|-----------------------|--------------------------------|
| `M01` or `m01`           | milestone  | `m{nn}` (lowercase)  | `M01` → `m01`                 |
| `#42`                    | issue      | `i{number}`           | `#42` → `i42`                 |
| `"add dark mode"` / text | free-text  | kebab-case slug       | `"add dark mode"` → `add-dark-mode` |

### Deriving the ID

- **Milestone**: strip `M`/`m` prefix, zero-pad to two digits, prepend `m`. `M1` → `m01`, `M12` → `m12`.
- **Issue**: strip `#`, prepend `i`. `#42` → `i42`, `#7` → `i7`.
- **Free-text**: lowercase, replace spaces and special characters with hyphens, collapse consecutive hyphens, trim leading/trailing hyphens. Max 40 characters — truncate at the last whole word.

## Output Location

Plans are written to `.dev/{id}/plan.md`. The `.dev/` directory must exist and be gitignored.

## Plan Structure

Every plan file follows this structure:

```markdown
# Plan: {target title}

- **Target**: {type} — {reference} ({title or description})
- **Dev branch**: {branch name per tpm-hygiene conventions}
- **Date**: {YYYY-MM-DD}
- **Status**: draft

---

## Branching Strategy

{Required section — see below}

## Wave 1: {description}

{Tasks that can run in parallel}

## Wave 2: {description}

{Tasks that depend on Wave 1 being complete}

...

## Final: Push & PR

{Final step — push dev branch and create PR}
```

## Branching Strategy Section

This section is **required** in every plan. It must explicitly state:

1. The development branch name (following tpm-hygiene naming conventions)
2. That main is protected — no direct pushes or merges to main
3. That subagent branches are local-only and never pushed to remote
4. The merge flow: subagent branches → dev branch (local merge) → push dev branch → PR → main

Example:

```markdown
## Branching Strategy

- **Dev branch**: `dev/m01-auth-identity` (pushed to remote)
- **Subagent branches**: local-only, merged into dev branch after each wave
- **Main**: protected — never push or merge directly to main
- **Flow**: subagent branch → merge into dev → (repeat per wave) → push dev → PR → main
```

## Waves

A wave is a set of tasks that can run **in parallel** — one agent per task, each on its own branch/worktree.

### Wave Rules

- Waves run **sequentially**: Wave 2 starts only after all Wave 1 tasks are merged into the dev branch.
- Within a wave, all tasks run in **parallel** (independent of each other).
- After each wave completes, all subagent branches are merged into the dev branch before the next wave begins.
- A single-agent task is one wave. Larger targets produce multiple waves with dependencies.

### Task Details

Each task within a wave must specify:

| Field               | Description                                                    |
|---------------------|----------------------------------------------------------------|
| **Branch**          | Subagent branch name (local-only, arbitrary but descriptive)   |
| **Agent**           | Which agent or agent type handles this task                    |
| **Description**     | What the task accomplishes (1-2 sentences)                     |
| **Acceptance criteria** | Concrete, verifiable conditions for task completion        |
| **Files**           | Key files expected to be created or modified                   |

Example:

```markdown
## Wave 1: Foundation

### Task 1.1: Set up auth middleware
- **Branch**: `agent/m01-auth-middleware`
- **Agent**: coding agent
- **Description**: Create Express middleware for JWT validation and session management.
- **Acceptance criteria**:
  - Middleware validates JWT tokens from Authorization header
  - Invalid/expired tokens return 401
  - Valid tokens attach user object to request
  - Unit tests pass
- **Files**: `src/middleware/auth.ts`, `src/middleware/__tests__/auth.test.ts`

### Task 1.2: Set up user model
- **Branch**: `agent/m01-user-model`
- **Agent**: coding agent
- **Description**: Create user database model with basic CRUD operations.
- **Acceptance criteria**:
  - User model with email, passwordHash, createdAt fields
  - Create, read, update, delete operations
  - Email uniqueness constraint
  - Unit tests pass
- **Files**: `src/models/user.ts`, `src/models/__tests__/user.test.ts`
```

## Final PR Section

Every plan ends with a PR step:

```markdown
## Final: Push & PR

1. Ensure all wave branches are merged into `{dev branch}`
2. Run full test suite on dev branch
3. Push dev branch to remote: `git push -u origin {dev branch}`
4. Create PR against main: `gh pr create --base main --head {dev branch} --title "{title}" --body "{summary}"`
```

## Task Sizing Guidelines

- **Atomic task**: can be completed by one agent in one session. This is the unit of work.
- **Single-wave target**: all tasks are independent → one wave.
- **Multi-wave target**: some tasks depend on others → multiple waves, sequenced by dependency.
- If a task is too large for one agent, split it into subtasks across waves.
- Prefer more smaller tasks over fewer larger ones — parallelism is cheap.

## Planner Log

After writing a plan, the planner appends a log entry to `.dev/planner.md`:

```markdown
## YYYY-MM-DD HH:MM — Plan: {target title}
- **Target**: {type} — {reference}
- **ID**: {id}
- **Plan**: `.dev/{id}/plan.md`
- **Waves**: {count}
- **Tasks**: {total task count}
```

The log is append-only. Never overwrite previous entries.
