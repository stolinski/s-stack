# Sources — critique-loop

## Classification
- **Class:** workflow-process (orchestration loop with convergence).
- **Path:** synthesis + authoring + description optimization + evaluation + registration/validation.
- **Structure pattern:** Domain Expert (SKILL.md + one reference).
- **Workflow pattern:** feedback loop / validate-fix-repeat (macro scale) — see `skill-writer/references/workflow-patterns.md`.
- **Output pattern:** strict per-round report + final report templates.

## Source Inventory
| # | Source | Path | Trust | Confidence | Contribution | Constraints |
|---|--------|------|-------|-----------|--------------|-------------|
| 1 | User request | conversation | canonical | high | The missing step: complete the epic, re-run the critic, report further changes, loop | Primary intent |
| 2 | workflow-patterns.md | `skills/skill-writer/references/workflow-patterns.md` | canonical | high | Feedback-loop pattern shape | Applied at macro scale |
| 3 | critique-to-dex | `skills/critique-to-dex/` | canonical | high | Backlog step each round; prior finding set = epic tasks | Orchestrate, don't restate |
| 4 | dex skill | `skills/dex/` | canonical | high | Epic/task completion + verification/result discipline | Orchestrate, don't restate |
| 5 | ui-design-critique / ux-critique | `skills/*/` | canonical | high | The re-run critique; shared report contract enables matching | Loop is critique-agnostic |
| 6 | Repo conventions | `README.md`, `AGENTS.md` | canonical | high | Registration; no destructive git; verify-before-claim | Register under Planning/Workflow |

## Coverage Matrix
| Dimension (workflow-process) | Where | Status |
|------------------------------|-------|--------|
| Preconditions | Invariants + Preconditions sections | complete |
| Ordered flow | 6-step per-round loop + checklist | complete |
| Failure handling | Persisting escalation; thrash detection; persisting-wall stop | complete |
| Safety boundaries | Loop budget; verify-before-recritique; never bury regressions; confirm at round boundaries | complete |
| Convergence logic | `convergence.md` matching rules + delta taxonomy + stop conditions | complete |
| Generalization | Critique-agnostic (any shared-contract critique) | complete |

## Key Decisions
| Decision | Rationale | Status |
|----------|-----------|--------|
| Hold critique/scope/anchors constant across rounds | A controlled comparison is the whole point; changing inputs makes deltas meaningless | adopted |
| Four-class delta taxonomy (Resolved/Persisting/New/Regression) | "Further changes?" is meaningless without distinguishing fixed vs failed-fix vs newly-broken | adopted |
| Regressions lead the report; conservative New→Regression call | New problems caused by fixes are the highest-value signal and easiest to miss | adopted |
| Escalate persisting findings instead of re-queueing | A finding surviving a dedicated fix means the approach is wrong — needs a human | adopted |
| Default stop = Converged (no P0/P1); max 3 rounds; thrash stop | Practical "clean enough" with safety bounds against infinite/worsening loops | adopted |
| Confirm at each round boundary by default | High-stakes autonomous change; keep a human checkpoint unless they opt into autonomous mode | adopted |
| Orchestrate existing skills, don't restate them | Avoid duplication drift with critique/critique-to-dex/dex | adopted |

## Gaps / Next
- Finding-matching is heuristic (dimension + location + issue equivalence); no machine key across reports. Acceptable — the report keeps a prior→current F-id mapping for traceability.
- Could later persist per-round P0/P1 trajectory to a file for long-running convergence tracking.

## Stopping Rationale
The loop composes fully-specified components (critiques, critique-to-dex, dex) and the only novel logic — matching, delta taxonomy, and stop conditions — is captured in convergence.md. Further retrieval is low-yield.

## Changelog
- Initial creation: SKILL.md + references/convergence.md + this file.
