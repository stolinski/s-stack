# Sources — grill-users

## Classification
- **Class:** workflow-process (interview → artifact), hybrid with the grilling method.
- **Path:** synthesis + authoring + description optimization + evaluation + registration/validation.
- **Structure pattern:** Domain Expert (SKILL.md + two references).
- **Output pattern:** grilling loop + strict authored-artifact template.

## Source Inventory
| # | Source | Path/URL | Trust | Confidence | Contribution | Constraints |
|---|--------|----------|-------|-----------|--------------|-------------|
| 1 | User request | conversation | canonical | high | Anchor critiques to WHO the users are, not a design spec; multiple user types serving UX and design | Primary intent |
| 2 | grilling skill | `skills/grilling/SKILL.md` | canonical | high | One-question-at-a-time method, recommended answers, explore-before-ask | Depend on the method |
| 3 | grill-design skill | `skills/grill-design/SKILL.md` | canonical | high | Sibling structure/format for a grill→artifact skill | Mirror shape; different artifact |
| 4 | ux-critique | `skills/ux-critique/` | canonical | high | Consumer: scenarios + priority lens; the universal-truths boundary | USERS.md sets scenario/weighting, never the standard |
| 5 | ui-design-critique | `skills/ui-design-critique/` | canonical | high | Consumer: Emotional Resonance ("this is for me") | Feeds dimension 4 |
| 6 | Persona / JTBD practice | established product-discovery practice (personas, jobs-to-be-done) | canonical (external, stable) | high | Behavioral (not demographic) persona shape; primary-user discipline | Stable practice |
| 7 | Repo conventions | `README.md`, `AGENTS.md` | canonical | high | Registration | Register under Frontend, UI & Design |

## Coverage Matrix
| Dimension (workflow-process) | Where | Status |
|------------------------------|-------|--------|
| Preconditions | Repo/product recon before grilling | complete |
| Ordered flow | 5-step process + 11-node question tree | complete |
| Failure handling | Reject "everyone"; merge non-distinct types; challenge aspirational users | complete |
| Safety boundaries | Reflect-and-confirm set before authoring; confirm file path | complete |
| Artifact validity | users-md-format.md schema + filled example + downstream-read notes | complete |
| Consistency with universal-truths | "Why This Is Not a DESIGN.md" section; personas set scenario/weighting only | complete |

## Key Decisions
| Decision | Rationale | Status |
|----------|-----------|--------|
| Anchor artifact is WHO (USERS.md), not a design spec | UX is brand-independent; the legitimate anchor is the audience, which also informs design resonance | adopted |
| Personas set scenario + priority lens, never the usability standard | Preserves the universal-truths stance of ux-critique | adopted |
| Exactly one primary user; behaviorally distinct types only | "Everyone" and interchangeable personas give the critiques nothing to test | adopted |
| Require anti-users | Edges clarify the identity as much as the center | adopted |
| YAML front matter + prose (mirrors design.md) | Machine-readable for critiques; prose for nuance | adopted |
| Feeds grill-design, ui-design-critique, ux-critique | Completes the discovery→define→critique→backlog chain | adopted |

## Gaps / Next
- No linter for USERS.md (unlike design.md); validity is enforced by the "what a good USERS.md requires" bar.
- Could later support quantitative persona weighting from analytics if a source exists.

## Stopping Rationale
The grilling method and sibling grill-design fully specify the interview→artifact shape; the two consumer skills define exactly what fields they need; persona/JTBD practice is stable. Further retrieval is low-yield.

## Changelog
- Initial creation: SKILL.md + references/{grill-questions,users-md-format}.md + this file.
