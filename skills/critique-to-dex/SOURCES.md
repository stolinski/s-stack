# Sources — critique-to-dex

## Classification
- **Class:** workflow-process (report → artifact conversion).
- **Path:** synthesis + authoring + description optimization + evaluation + registration/validation.
- **Structure pattern:** Domain Expert (SKILL.md + one reference).
- **Output pattern:** decision-table mapping + strict preview/creation output.

## Source Inventory
| # | Source | Path | Trust | Confidence | Contribution | Constraints |
|---|--------|------|-------|-----------|--------------|-------------|
| 1 | User request | conversation | canonical | high | Generalized priority-list → dex epic; must consume ui-design-critique output cleanly | Primary intent |
| 2 | dex skill | `skills/dex/SKILL.md` | canonical | high | Commands (`create --priority/--parent/--blocked-by`, `plan`), Epic→Task→Subtask, repo-local `dex dir` guardrail, good-context/result shapes | Depend on dex skill; do not duplicate |
| 3 | ui-design-critique | `skills/ui-design-critique/` | canonical | high | Source report shape → the contract adapter | Sibling; keep decoupled via contract |
| 4 | Repo conventions | `README.md`, `AGENTS.md` | canonical | high | Registration; Dex workflow rules | Register under Planning/Workflow |

## Coverage Matrix
| Dimension (workflow-process) | Where | Status |
|------------------------------|-------|--------|
| Preconditions | Verify `dex dir` scope before create | complete |
| Ordered flow | 6-step process | complete |
| Failure handling | Missing-field handling; thin-recommendation flags; missing-priority default | complete |
| Safety boundaries | Preview + confirm before writing tasks; strengths never become tasks | complete |
| Generalization | Priority findings contract + two adapters (ui-design-critique, generic list) | complete |

## Key Decisions
| Decision | Rationale | Status |
|----------|-----------|--------|
| Normalize to a "priority findings contract" | Decouples from any single critique; future critiques target the contract | adopted |
| Preview-and-confirm gate before creating | Creating dex tasks writes files; avoid surprise backlogs | adopted |
| Strengths → epic "Preserve" note, never tasks | A strength is a thing to protect, not work to do | adopted |
| Suggestions opt-in | Avoid flooding the backlog with taste-based items | adopted |
| P0→1 … P3→4 priority mapping | dex uses lower=higher; align impact rank to dex priority | adopted |
| Depend on dex skill, don't restate commands | Avoid duplication drift | adopted |

## Gaps / Next
- dex `--priority` numeric range not documented; assumed 1=highest. Revisit if dex clarifies.
- `dex plan` could bulk-create from generated markdown; kept explicit `dex create` as primary for deterministic priority+hierarchy+strength handling.

## Stopping Rationale
The dex skill fully specifies the target system; the ui-design-critique report shape is known and adapted; the contract covers arbitrary future critiques. Further retrieval is low-yield.

## Changelog
- Initial creation: SKILL.md + references/priority-findings-contract.md + this file.
