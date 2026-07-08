---
name: critique-to-dex
description: Convert a prioritized critique or findings report into a Dex epic with tasks. Use when asked to "turn this critique into tasks", "make a dex epic from these findings", "create tasks from the design review", "add these to dex", or "backlog this critique". Works with any prioritized findings list — including the ui-design-critique report — by normalizing to a common contract, then creating a Dex epic with priority-ranked, well-contextualized tasks.
metadata:
  category: Planning, Thinking & Workflow
---

# Critique → Dex

Turn a prioritized critique into an actionable Dex epic. Read a findings report, normalize it to the priority findings contract, then create a single epic with one task per actionable finding — ranked by priority, each carrying enough context (what/why/how/done-when) to be worked without the original report.

This skill is source-agnostic. It works cleanly with `ui-design-critique` output and with any future critique that emits the contract in `references/priority-findings-contract.md`.

For dex command semantics and the repo-local storage guardrail, this skill relies on the `dex` skill — do not duplicate that knowledge here; apply it.

## Inputs

Accept the findings from any of:

| Input | Handling |
|-------|----------|
| A report already in this conversation (e.g. a critique just produced) | Use it directly |
| A file path (`.md`) | Read it |
| Pasted text / loose list | Parse what is present; fill gaps by asking or inferring |

## Process

### 1. Locate and normalize the findings

Extract every finding into the contract (`references/priority-findings-contract.md`): `subject`, and per finding `id`, `title`, `priority`, `classification`, `location`, `issue`, `impact`, `evidence`, `recommendation`.

- Normalize priority to a rank using the table below, whatever scheme the source used.
- If a required field is missing and cannot be inferred from context, note it; do not invent evidence.

### 2. Verify Dex storage scope

Before creating anything, confirm Dex is writing to repo-local storage (run `dex dir`; if it resolves to a global path, fix the workspace first). Follow the `dex` skill's repo-local storage guardrail exactly. Never create a project epic in global Dex by accident.

### 3. Build the epic + task plan

Apply the mapping rules below to produce:
- one **epic** for the whole critique,
- one **task** per actionable finding,
- optional **subtasks** when a single finding has clearly separable parts,
- optional **blockers** when one fix must precede another (only if the source states or clearly implies a dependency).

### 4. Preview and confirm

Creating Dex tasks writes files. ALWAYS show the plan first and get confirmation before creating:

```markdown
## Dex plan preview — [subject]
Epic: [epic title]  (strengths preserved in epic description; N strengths)
Tasks (M):
| # | Priority → dex | Class | Title |
|---|----------------|-------|-------|
| 1 | P0 → 1 | Gap | ... |
| 2 | P1 → 2 | Inconsistency | ... |
Excluded: [e.g. 3 Suggestions (P3) — include? / Strengths → not tasks]
```

Ask once: include Suggestions as tasks, or list-only? Default: include P0–P2 findings of any class; include P3/Suggestions only on confirmation.

### 5. Create in Dex

Create the epic first, then tasks in priority order. Build each task description with the "good context" shape (see mapping). Attach `--priority` and `--parent`/`--blocked-by` as planned. Capture the returned IDs.

### 6. Report

Return the epic ID, the ordered task IDs with their priorities, what was excluded, and the strengths preserved in the epic description.

## Mapping Rules

### Priority normalization

| Source priority | Rank | dex `--priority` |
|-----------------|------|------------------|
| P0 / Critical / Blocker / Sev0 | 1 | 1 |
| P1 / High / Major | 2 | 2 |
| P2 / Medium / Moderate | 3 | 3 |
| P3 / Low / Minor / Nit | 4 | 4 |

(dex priority: lower number = higher priority. If a source uses a 1–5 or reversed scheme, map by ordinal position, highest-impact → 1.)

### Classification handling

| Classification | Becomes | Notes |
|----------------|---------|-------|
| **Gap** | Task | Usually the highest-value work |
| **Inconsistency** | Task | |
| **Suggestion** | Task (optional) | Include only if the user opts in; keep at its source priority |
| **Strength** | NOT a task | Record in the epic description under "Preserve — do not regress" |

### Epic

- **Title:** `[Critique type]: [subject]` — e.g. `Design critique: Checkout page`.
- **Description:** the report's verdict/summary, the finding counts by priority, a "Preserve — do not regress" list built from Strengths, and a pointer to the full report (file path if one exists).

### Task description (per finding)

Build the "what / why / how / done-when" shape Dex expects:

```
## What
[issue] — at [location]

## Why
[impact]

## How
[recommendation]
Evidence: [evidence]

## Done when
[acceptance derived from the recommendation — the observable state that means it's fixed]
```

Keep the source finding `id` (e.g. `F-01`) in the task title suffix so tasks trace back to the report: `Fix unreadable CTA contrast (F-03)`.

## Output Format

```markdown
## Created Dex epic — [subject]
- **Epic:** <epic_id> — [title]
- **Tasks:** N created
  - <id> · P0→1 · Gap · [title]
  - <id> · P1→2 · Inconsistency · [title]
  - ...
- **Excluded:** [suggestions/strengths and why]
- **Preserved in epic:** [count] strengths
- **Next:** `dex start <first_id>`
```

## Reference Files

| File | Load when |
|------|-----------|
| `references/priority-findings-contract.md` | Always — the normalization schema and source adapters |
