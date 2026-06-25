---
name: dex
description: Use when tracking complex multi-step tasks, creating task hierarchies, maintaining persistent task state across sessions, building backlogs, or when the user explicitly asks to "use dex" for task management. Dex provides persistent memory for AI agents with GitHub/Shortcut sync capabilities.
---

# Dex - Persistent Task Tracking for AI Agents

Dex is a task tracking system that acts as a **master coordinator** for complex work—breaking down tasks, tracking progress, and preserving context across sessions. Unlike ephemeral session tracking, dex tasks are stored as files in the repository.

## Installation

```bash
npm install -g @zeeg/dex
# or
pnpm add -g @zeeg/dex
# or
bun add -g @zeeg/dex
```

## Core Concepts

- **Persistence**: Tasks survive beyond the current session as `.dex/` files in the repo
- **Repository-local storage**: Before planning inside a project, run `dex dir`. If it points at global storage such as `~/.config/dex/local`, the current directory is not being treated as a repo-local Dex workspace. Initialize or enter the project repo, ensure `.dex/` exists if needed, then re-run `dex dir` before creating backlog items.
- **Context Preservation**: Decisions, progress, and rationale are captured
- **Hierarchy**: 3-level structure (Epic → Task → Subtask)
- **Configuration**: Tasks can block other tasks
- **GitHub/Shortcut Sync**: Optional sync to Issues/Stories for permanent records

Related reference: `references/content-audit-and-devtools-gates.md` covers content-completeness audit scripts plus Chrome DevTools MCP verification gates for content-model epics.

## Core Commands

### Dashboard & Listing

```bash
dex                    # Show dashboard (default command)
dex status             # Same as above - shows stats, ready tasks, blocked, completed
dex status --json      # Output as JSON

dex list               # List pending tasks in tree view
dex list --all         # Include completed tasks
dex list --ready       # Only unblocked tasks
dex list --blocked     # Only blocked tasks
dex list --in-progress # Only in-progress tasks
dex list --query "auth" # Search in name/description
dex list --flat        # Plain list instead of tree
```

### Creating Tasks

```bash
dex create "Task title"

dex create "Task title" --description "Full implementation details..."
dex create "Task title" --description "..." --priority 2
dex create "Task title" --blocked-by <other_task_id>
dex create "Subtask title" --parent <parent_task_id>
```

### Viewing & Editing

```bash
dex show <task_id>              # View full task details
dex show abc123 --expand        # Show ancestor descriptions
dex show abc123 --full          # No truncation
dex show abc123 --json          # JSON output

dex edit <task_id> --name "New name"
dex edit <task_id> --description "Updated context"
dex edit <task_id> --add-blocker <other_id>
dex edit <task_id> --remove-blocker <other_id>
```

### Status Management

```bash
dex start <task_id>             # Mark as in progress
dex start <task_id> --force     # Re-claim already in-progress task
dex complete <task_id> --result "What was accomplished"
dex complete <task_id> --result "..." --commit <sha>
dex complete <task_id> --result "..." --no-commit
dex delete <task_id>            # Delete task (and children)
dex archive <task_id>           # Archive completed task
```

### Planning

```bash
dex plan path/to/plan.md        # Create tasks from markdown plan
dex plan docs/SPEC.md --priority 2
dex plan feature.md --parent <id>
```

## GitHub Integration

```bash
dex sync                        # Sync all tasks to GitHub Issues
dex sync <task_id>              # Sync specific task
dex sync --dry-run              # Preview sync
dex sync --github               # GitHub only
dex sync --shortcut             # Shortcut only

dex import #42                  # Import GitHub issue
dex import --all                # Import all dex-labeled issues
dex import sc#123               # Import Shortcut story

dex export <task_id>            # Export to GitHub without sync
```

## When to Use Dex

**Use dex when:**
- Work spans multiple sessions or days
- Complex features need breakdown into steps
- Context and decisions must be preserved
- Work might be handed off or revisited
- Building a backlog before execution

**Skip dex when:**
- Quick, single-session task
- No context needs preserving
- Overhead isn't worth it

## Workflow for Agents

### Repository-local storage check

Before creating a project backlog, verify Dex is using repository-local storage:

```bash
dex dir
```

Expected for a project backlog: `<repo>/.dex`. If it resolves to a global path (for example `~/.config/dex/local`), make the project a git repo first:

```bash
git init
mkdir -p .dex
dex dir
```

Dex may have a global config already, so `dex init -y` can fail with "Config file already exists"; that is not the fix. The durable fix is ensuring the project is a git repo so Dex scopes storage locally. Keep `.dex/` project-local for epics/task lists that belong to the repo.

### Starting New Work

0. Verify storage scope before creating anything:
```bash
dex dir
```
If it prints a global path (for example `~/.config/dex/local`) while you are planning project work, stop and fix the workspace first: enter the correct repo or initialize git for the project, create `.dex/` if needed, then re-run `dex dir` until it points inside the project. Do not create project epics in global Dex by accident.

1. Create task with full context:
```bash
dex create "Add JWT authentication" --description "
## Goal
Implement JWT-based auth for the API.

## Requirements
- Token generation on login
- Verification middleware for protected routes
- Refresh token flow

## Files
- src/middleware/auth.ts
- src/routes/auth.ts

## Acceptance Criteria
- POST /api/auth/login returns JWT
- Protected routes return 401 without valid token
- All 24 tests passing
"
```

2. For complex work, break into subtasks:
```bash
dex create "Create auth middleware" --parent <parent_id>
dex create "Add login endpoint" --parent <parent_id>
dex create "Add token refresh flow" --parent <parent_id>
```

3. Mark as in progress:
```bash
dex start <task_id>
```

### During Implementation

```bash
dex list                        # Check current tasks
dex show <task_id>              # Review context
dex edit <task_id> --description "Updated: also need refresh tokens"
```

### Completing Tasks

```bash
dex complete <task_id> --result "
Added POST /api/auth/login endpoint.
Verifies credentials with bcrypt, returns JWT access token (15m)
and refresh token (7d). All 24 tests passing.
" --commit <sha>
```

## Writing Good Task Context

Include:
- **What** needs to be done (specific, not vague)
- **Why** it's needed (background, motivation)
- **How** to approach it (files to modify, patterns to follow)
- **Done when** (acceptance criteria)
- **Verification gates** for risky/big-bang work: content audits, build/check commands, and browser verification steps. If Chrome DevTools MCP is available, name the exact flows it must verify rather than leaving “browser tested” vague.

**Good:**
> "Add rate limiting to /api/auth endpoints. Use express-rate-limit, 5 requests per minute per IP for /login, 20/min for /refresh. Return 429 with Retry-After header. Add to src/middleware/rate-limit.ts, apply in src/routes/auth.ts."

**Bad:**
> "Add rate limiting"

## Writing Good Results

Include:
- **What changed** (implementation summary)
- **Key decisions** (and why)
- **Verification** (tests passing, manual testing done)

For browser-app implementation tasks, especially Scott's SvelteKit projects, record concrete Chrome DevTools MCP evidence in the Dex completion result. Do not write generic "browser tested". Name the route/page, the clicked flow, observed state changes, console result, and screenshot path if layout/highlighting was visually checked. See `references/devtools-mcp-verification.md`.

**Good:**
> "Added rate limiting with express-rate-limit. Login: 5/min, refresh: 20/min. Returns 429 with Retry-After header. Added 6 tests for rate limit scenarios. All 30 tests passing."

**Good browser-task result:**
> "npm run check passed; npm run build passed; Chrome DevTools MCP verified /op-xy loads, selecting Move 5 updates Touch Now, completing all Moves reaches 100%, console has no runtime errors, screenshot saved at /tmp/flow.png."

**Bad:**
> "Added rate limiting"

## Task Hierarchy

- **Epic**: Large initiatives (5+ tasks). Example: "Add user authentication system"
- **Task**: Significant work items. Example: "Implement JWT middleware"
- **Subtask**: Atomic steps. Example: "Add token verification function"

Maximum depth is 3 levels.

## Behavior Notes

- Subtasks must complete before parent can be completed
- Blocked tasks CAN be completed (warning shown) for flexibility
- Deleting a parent deletes all children
- Use GitHub issue numbers (permanent) in commits, not task IDs (ephemeral)

## Configuration

```bash
dex init                        # Interactive setup for global config (~/.config/dex/dex.toml)
dex init -y                     # Accept defaults; fails harmlessly if global config already exists
dex dir                         # Print task storage path; ALWAYS verify this before creating project tasks
dex dir --global                # Print global config path
dex config --list               # Show all config values
dex config sync.github.enabled=true  # Enable GitHub sync
dex doctor                      # Check for issues
dex doctor --fix                # Check and fix issues
```

### Repo-local storage guardrail

For project planning, prefer repo-local Dex storage so tasks live with the project instead of the global store.

Before creating an epic/task list:

```bash
# from the project root
git rev-parse --is-inside-work-tree || git init
mkdir -p .dex
dex dir
```

Proceed only after `dex dir` prints `<project>/.dex`. If it prints something under `~/.config/dex/local`, the current directory is not being treated as the project store yet — initialize/enter the git repo and re-check. This matters for Scott: project Dex planning should be local, not global.

## Session Handoff

When ending a session with incomplete work:

```bash
dex edit <task_id> --description "
## Progress
- Completed: JWT middleware added
- In Progress: Login endpoint (70% done)
- Blocked: Need refresh token strategy

## Next Steps
- Finish login endpoint validation
- Add refresh token rotation
"
```

Keep task in_progress so next session knows to continue.
