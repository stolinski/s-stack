# Sources — grill-design

## Classification
- **Class:** workflow-process (interview → artifact), hybrid with grilling method.
- **Path:** synthesis + authoring + description optimization + evaluation + registration/validation.
- **Structure pattern:** Domain Expert (SKILL.md + two references).
- **Output pattern:** grilling loop + strict authored-artifact template.

## Source Inventory
| # | Source | Path/URL | Trust | Confidence | Contribution | Constraints |
|---|--------|----------|-------|-----------|--------------|-------------|
| 1 | User request | conversation | canonical | high | Grill the user to produce a UNIQUE design.md defining the app's look | Primary intent; uniqueness is the deliverable |
| 2 | design.md format + CLI | github.com/google-labs-code/design.md | canonical | high | Token schema, section order, component tokens, `lint`/`spec`/`export` | Format is `alpha`; expect change |
| 3 | grilling skill | `skills/grilling/SKILL.md` | canonical | high | One-question-at-a-time method, recommended answers, explore-before-ask, corrections-as-truth | Depend on the grilling method |
| 4 | ui-design-critique (ai-slop-detection) | `skills/ui-design-critique/references/ai-slop-detection.md` | canonical | high | Anti-slop fingerprints → grill guardrails and Do's/Don'ts | Sibling; the produced doc feeds the critique |
| 5 | design-engineering | `skills/design-engineering/SKILL.md` | canonical | medium | Motion posture; defer craft depth to it | Cross-reference, don't duplicate |
| 6 | Repo conventions | `README.md`, `AGENTS.md` | canonical | high | Registration; UI rules (no reflexive gradients/defaults) | Register under Frontend, UI & Design |

## Coverage Matrix
| Dimension (workflow-process) | Where | Status |
|------------------------------|-------|--------|
| Preconditions | Repo recon before grilling | complete |
| Ordered flow | 6-step process + 14-node question tree | complete |
| Failure handling | Reject-generic-answer tactics; re-open branches if slop persists | complete |
| Safety boundaries | Confirm identity + file path before/at authoring; lint gate | complete |
| Artifact validity | design-md-format.md schema + full example + lint/fix | complete |
| Uniqueness enforcement | "What unique requires" bar + anti-slop guardrails table | complete |

## Key Decisions
| Decision | Rationale | Status |
|----------|-----------|--------|
| Grill, don't transcribe; reject vague adjectives | A recorded "modern/clean" produces slop, not identity | adopted |
| Force one distinctive choice + anti-references | Uniqueness is defined by commitment and refusal | adopted |
| Anti-slop guardrails steer palette/type away from defaults | Directly counters the tells ui-design-critique flags | adopted |
| Self-contained design.md format ref (duplicated from critique) | Skills should stand alone; cross-skill duplication is acceptable | adopted |
| Lint gate before handoff; fix contrast/broken-ref | Ship a valid, accessible source of truth | adopted |
| Output feeds ui-design-critique | Closes the loop: define look → critique against it → backlog via critique-to-dex | adopted |

## Gaps / Next
- design.md is `alpha`; revisit schema/CLI if it changes.
- Motion is intentionally light here; deep interaction craft lives in `design-engineering`.

## Stopping Rationale
The design.md format is fully specified from its README/spec; the grilling method is defined by the local skill; anti-slop material is already synthesized in the sibling critique skill. Further retrieval is low-yield.

## Changelog
- Initial creation: SKILL.md + references/{grill-questions,design-md-format}.md + this file.
