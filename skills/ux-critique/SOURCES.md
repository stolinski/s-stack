# Sources — ux-critique

## Classification
- **Class:** generic (experience critique), authored against the `security-review` structural profile — a review skill that finds issues, prioritizes by impact, controls false positives (taste vs. defect), and prescribes remediation.
- **Path:** synthesis + authoring + description optimization + evaluation + registration/validation.
- **Structure pattern:** Domain Expert (SKILL.md + conditional references).
- **Output pattern:** strict report template, deliberately identical in contract to `ui-design-critique`.

## Source Inventory
| # | Source | Path/URL | Trust | Confidence | Contribution | Constraints |
|---|--------|----------|-------|-----------|--------------|-------------|
| 1 | User request | conversation | canonical | high | Build a UX critique sibling to ui-design-critique | Primary intent |
| 2 | ui-design-critique | `skills/ui-design-critique/` | canonical | high | Shared report contract, taxonomy, priority scale, capture pattern | Match contract exactly so critique-to-dex ingests both |
| 3 | Nielsen usability heuristics | Nielsen Norman Group (10 heuristics) | canonical (external, stable) | high | Dimensions 2–8, 12 in ux-heuristics.md | Well-established; no version drift |
| 4 | Deceptive/dark-pattern taxonomy | Brignull "Deceptive Patterns" body of work | canonical (external, stable) | high | The dark-patterns.md check | Terminology stable |
| 5 | dex skill (via critique-to-dex) | `skills/dex/`, `skills/critique-to-dex/` | canonical | high | Downstream consumer; drives contract compatibility | Keep finding block identical |
| 6 | design-engineering / ux-interface-design | `skills/*/SKILL.md` | canonical | medium | Scope boundaries — delegate building/feel | Cross-reference, don't duplicate |
| 7 | Capture tooling | AGENTS.md + agent-browser/chrome-devtools MCP/argent | canonical | high | Walkthrough tooling and objective signals (console/network/trace) | agent-browser preferred; devtools MCP fallback |
| 8 | Repo conventions | `README.md`, `AGENTS.md` | canonical | high | Registration; UI/UX rules | Register under Frontend, UI & Design |

## Coverage Matrix
| Dimension (review profile) | Where | Status |
|----------------------------|-------|--------|
| Issue-class definitions | `ux-heuristics.md` (12 dims) + `dark-patterns.md` | complete |
| Impact/severity calibration | SKILL.md Priority Scale (impact on task success) | complete |
| Classification taxonomy | SKILL.md — Gap/Inconsistency/Suggestion/Strength (shared) | complete |
| Evidence requirement | Non-Negotiables + report Evidence field + Flow Walkthrough | complete |
| False-positive controls | `ux-heuristics.md` "Findings vs. Taste"; protective-friction vs. dark-pattern table | complete |
| Remediation | report Recommendation field; dark-pattern fix example | complete |
| Live evaluation (walk the flow) | `task-walkthrough.md` incl. unhappy paths + input modes | complete |
| Universal standard (no spec anchor) | heuristics + dark patterns are the objective standard; no DESIGN.md equivalent | complete |
| Strengths | report + taxonomy Strength | complete |

## Key Decisions
| Decision | Rationale | Status |
|----------|-----------|--------|
| Judge against universal usability truths, NOT a spec | UX is brand-independent; unlike aesthetics it has no legitimate DESIGN.md equivalent. A provided journey doc only supplies the scenario/goal, never the standard — intent itself can be wrong | adopted |
| Identical report contract to ui-design-critique | `critique-to-dex` ingests both with the same adapter; consistent output across the critique series | adopted |
| Task success is the headline metric | UX is judged by goal completion, not looks | adopted |
| Walk the flow, incl. unhappy paths | Flow/feedback/recovery problems are invisible in a static screenshot | adopted |
| Dark patterns as a first-class, P0-capable check | The UX equivalent of ui-design-critique's AI-slop check; high user harm | adopted |
| Interaction a11y here; visual contrast stays in ui-design-critique | Avoid overlap; keep each skill's lane clear | adopted |
| Critique-only; delegate building to design-engineering/ux-interface-design | Tight scope, no overlap with sibling design skills | adopted |

## Gaps / Next
- `agent-browser` symlink currently unresolved in this checkout; `task-walkthrough.md` degrades to chrome-devtools MCP.
- No automated linter equivalent to design.md for UX; evaluation is walkthrough-driven by design.

## Stopping Rationale
The heuristic and dark-pattern canons are stable and fully encoded; the shared contract is fixed by the sibling skill; capture tooling is known. Additional retrieval (more heuristic listicles) would be low-yield and risk diluting the shared contract.

## Changelog
- Initial creation: SKILL.md + references/{ux-heuristics,task-walkthrough,dark-patterns}.md + this file.
