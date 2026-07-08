# Sources — ui-design-critique

Provenance, decisions, and coverage for the `ui-design-critique` skill.

## Classification

- **Class:** generic (design critique), authored against the `security-review` structural profile — a review skill that finds issues, prioritizes by impact, controls false positives, and prescribes remediation.
- **Path:** synthesis + authoring + description optimization + evaluation + registration/validation (new skill from scratch).
- **Structure pattern:** Domain Expert (SKILL.md + conditional references).
- **Output pattern:** strict report template (high fragility — the user requires a specific report shape).

## Source Inventory

| # | Source | Path/URL | Trust tier | Confidence | Contribution | Usage constraints |
|---|--------|----------|-----------|-----------|--------------|-------------------|
| 1 | User critique framework | conversation (this request) | canonical | high | The 10 critique dimensions + "AI Slop Detection is CRITICAL"; report must classify findings and rank by impact | Primary intent; drives dimensions + taxonomy |
| 2 | DESIGN.md format + spec + CLI | github.com/google-labs-code/design.md (README fetched) | canonical | high | Design-doc source of truth; token schema; `lint`/`spec` CLI; lint-rule → finding mapping | Format is `alpha`; expect schema changes over time |
| 3 | design-engineering skill | `skills/design-engineering/SKILL.md` | canonical | high | Craft bar, AI-slop antidotes, states/affordance/motion/a11y depth; sibling to cross-reference for fixes | Implementation skill — critique delegates building to it |
| 4 | ux-interface-design skill | `skills/ux-interface-design/SKILL.md` | canonical | medium | Self-evidence/affordance framing for dimension 5 | Cross-reference, avoid duplicating |
| 5 | security-review skill | `skills/security-review/SKILL.md` | canonical | high | Structural template: confidence/false-positive controls, severity table, evidence-required findings, report format | Structural model only (domain differs) |
| 6 | agent-browser / chrome-devtools MCP / argent | AGENTS.md guidance + available MCP tools | canonical | high | Visual capture workflow and value sampling | agent-browser preferred; devtools MCP fallback |
| 7 | Repo conventions | `README.md`, `AGENTS.md` | canonical | high | Registration table, UI rules, port-detection workflow | Register under "Frontend, UI & Design" |

## Coverage Matrix

| Required dimension (review profile) | Where covered | Status |
|-------------------------------------|---------------|--------|
| Issue-class definitions + prerequisites | `critique-dimensions.md` (10 dims) + `ai-slop-detection.md` | complete |
| Impact / severity calibration | SKILL.md Priority Scale (P0–P3) | complete |
| Classification taxonomy (gap/inconsistency/suggestion/strength) | SKILL.md Classification Taxonomy | complete |
| Evidence requirement (no pattern-only claims) | SKILL.md Non-Negotiables + report template Evidence field | complete |
| False-positive controls (taste vs. defect) | `critique-dimensions.md` "Distinguishing Findings from Taste"; DESIGN.md intent-as-truth | complete |
| Remediation / concrete fixes | report template Recommendation field; ai-slop antidote list | complete |
| Spec conformance (intent source of truth) | `design-md-spec.md` + report Spec Conformance table | complete |
| Visual evaluation across states/viewports | `visual-capture.md` | complete |
| Strengths ("what looks good") | report template + taxonomy Strength | complete |

## Key Decisions

| Decision | Rationale | Status |
|----------|-----------|--------|
| AI Slop Detection is dimension 1 and P0-capable | User named it the most important check | adopted |
| Four-way classification (Gap/Inconsistency/Suggestion/Strength) | User explicitly requested these categories in the report | adopted |
| P0–P3 impact scale + sort order | User asked for priority based on impact | adopted |
| DESIGN.md as the intent source of truth, with `lint` automation | User asked whether design.md is the way to lock in brand intent — yes; adds an automated, objective layer | adopted |
| Strict report template | High-fragility output; user wants a consistent, actionable report | adopted |
| Critique-only; delegate building to design-engineering | Keeps scope tight; avoids overlap with sibling skills | adopted |
| Tool-agnostic capture (agent-browser → devtools MCP → argent) | agent-browser symlink is currently unresolved in this repo; must not hard-depend on it | adopted |

## Gaps / Next Retrieval Actions

- The `agent-browser` skill is referenced by AGENTS.md but its symlink is unresolved in this checkout; `visual-capture.md` degrades gracefully to `chrome-devtools` MCP. Revisit if agent-browser lands.
- DESIGN.md is `alpha`; revisit token schema/CLI flags if the format changes.

## Stopping Rationale

The user-provided framework is the canonical intent and is fully encoded. The DESIGN.md README/spec, the sibling design skills, and the security-review structural template together cover all required review dimensions with concrete, actionable material. Additional retrieval (more AI-slop listicles, more design-critique blog posts) would be low-yield and risk diluting the user's explicit framework.

## Changelog

- Initial creation: SKILL.md + `references/{ai-slop-detection,critique-dimensions,design-md-spec,visual-capture}.md` + this file.
