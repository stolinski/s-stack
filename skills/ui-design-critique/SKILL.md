---
name: ui-design-critique
description: Design critique for user interfaces that produces a prioritized, strict report of design issues. Use when asked to "critique this design", "review the UI", "design review", "roast my UI", "is this AI slop", "does this look AI-generated", "audit the visual design", or evaluate a screen, page, or screenshot against brand guidelines or a DESIGN.md spec. Classifies every finding as a gap, inconsistency, suggestion, or strength and ranks it by impact. Evaluates visually via browser/devtools capture and against design tokens.
metadata:
  category: Frontend, UI & Design
---

# UI Design Critique

Produce a rigorous, prioritized design critique of a user interface. Output a single report that lists every issue ranked by impact, classifies each finding, and calls out what already works. Be strict. The goal is a report a designer can act on top-to-bottom, not vague praise.

This skill evaluates and reports. It does not implement fixes. For building or polishing UI, use `design-engineering`. For self-evidence/affordance depth, cross-reference `ux-interface-design`.

## Non-Negotiables

1. **AI Slop Detection is the first and most important check.** Run it before anything else. See `references/ai-slop-detection.md`.
2. **Critique against intent, not personal taste.** When a DESIGN.md spec exists, it is the source of truth. Distinguish a violation of intent (a real finding) from a stylistic preference (a suggestion, clearly labeled).
3. **Every finding needs evidence.** Point to the specific element, region, or token. No pattern-only claims like "typography feels off" without naming the element and the reason.
4. **Every finding needs a priority and a classification.** No unranked, unclassified findings.
5. **Report strengths too.** A critique that only lists problems is incomplete and untrustworthy.

## Inputs

Accept any of these; ask only if none are available:

| Input | How to evaluate |
|-------|-----------------|
| Live URL / running dev server | Capture visually — see `references/visual-capture.md` |
| Screenshot(s) | Evaluate the image directly |
| Component / page code | Read the code, and render + capture if a dev server is available |
| `DESIGN.md` (or brand doc) | Load as source of truth for *how it looks* — see `references/design-md-spec.md` |
| `USERS.md` (personas) | Load for *who it's for* — the `resonance_cue` per user type drives the Emotional Resonance check |

If no design spec is provided, ask once whether a `DESIGN.md` or brand guideline exists. If none exists, critique against general craft standards and note the absence of a spec as a gap (intent cannot be verified).

## Process

### 1. Establish intent (source of truth)

- If a `DESIGN.md` exists, load it and its tokens. Run `references/design-md-spec.md`. Optionally lint it: `npx @google/design.md lint DESIGN.md`.
- If a `USERS.md` exists (from `grill-users`), load the target users and their `resonance_cue` — the aesthetic is judged partly on whether each user type would feel "this is for me" (dimension 4, Emotional Resonance).
- If only a brand/style doc exists, extract stated colors, type, tone, and personality.
- If nothing exists, state the intended audience and product category from context, and flag "no design spec" as a finding.

### 2. Capture the interface

- Live UI: follow `references/visual-capture.md` to screenshot key states (default, hover/focus, empty, loading, error) and viewports (mobile + desktop at minimum).
- Screenshots only: work from what is provided; note any states/viewports you could not verify.

### 3. Run AI Slop Detection (CRITICAL)

Load `references/ai-slop-detection.md`. Score the interface against every fingerprint. Answer the core test: *If someone said "AI made this," would they be believed instantly?* Record the AI-slop risk level in the summary. Slop tells are high-priority findings.

### 4. Run the 10 critique dimensions

Load `references/critique-dimensions.md` and evaluate each dimension. For each issue found, capture: element/location, what is wrong, why it matters, classification, priority.

### 5. Check spec conformance (if a spec exists)

Compare rendered UI to the tokens: colors, type scale, spacing, radius, component styles. Every deviation from a defined token is an **Inconsistency** unless the deviation is clearly intentional and justified.

### 6. Classify, prioritize, and write the report

Use the taxonomy and priority scale below. Produce the report in the exact template.

## Classification Taxonomy

Every finding gets exactly one classification.

| Classification | Meaning |
|----------------|---------|
| **Gap** | Something required is missing — an unhandled state (empty/loading/error), a missing focus style, no clear primary action, no design spec, absent hierarchy. |
| **Inconsistency** | Something contradicts the spec or itself — off-token color, mismatched spacing, two buttons styled differently for the same role, drifting type scale. |
| **Suggestion** | The design works but could be stronger — an opportunity, an optional refinement, or a taste-based preference. Label taste-based ones as such. |
| **Strength** | Something done well. Name it specifically so it is preserved. |

## Priority Scale (by impact)

Rank by how much the issue harms the user's ability to understand and use the interface, and how much it damages perceived quality.

| Priority | Bar | Examples |
|----------|-----|----------|
| **P0 — Critical** | Breaks usability, trust, or accessibility; or screams AI slop | No discernible primary action; unreadable contrast; interface indistinguishable from generic AI output; broken/absent core states |
| **P1 — High** | Meaningfully hurts comprehension, flow, or brand fit | Weak/competing hierarchy; off-brand palette; confusing IA; illegible body text |
| **P2 — Medium** | Noticeable friction or polish gap | Uneven spacing rhythm; weak affordances; inconsistent component styling; flat empty states |
| **P3 — Low** | Minor refinement or preference | Optical alignment nits; microcopy tightening; optional delight |

Sort findings P0 → P3. Within a priority, order by classification severity: Gap, Inconsistency, then Suggestion.

## Report Template

ALWAYS output this exact structure.

```markdown
# Design Critique: [Screen / Page / Component]

## Verdict
[2–4 sentences: overall quality, biggest risk, and the single most important thing to fix.]

- **AI-Slop Risk:** None / Low / Moderate / High / Severe — [one-line reason]
- **Spec Conformance:** N/A (no spec) / Strong / Partial / Poor — [one-line reason]
- **Findings:** X total — P0: _ · P1: _ · P2: _ · P3: _
- **Verified:** [states/viewports checked; note anything unverifiable]

## Findings

### P0 — Critical
#### [F-01] [Short title] · [Gap | Inconsistency | Suggestion]
- **Dimension:** [e.g. AI Slop / Visual Hierarchy / Color]
- **Location:** [element, region, or token — be specific]
- **Issue:** [what is wrong]
- **Impact:** [why it matters to the user or brand]
- **Evidence:** [screenshot region, token value vs. spec, or code ref]
- **Recommendation:** [concrete, actionable fix]

[repeat for each finding, continuing F-02, F-03, ... across all priority sections]

### P1 — High
...

### P2 — Medium
...

### P3 — Low
...

## What Looks Good (Strengths)
- **[Dimension]:** [specific thing done well and why it works — preserve this]
- ...

## Spec Conformance (if a DESIGN.md/brand doc was provided)
| Token / Rule | Spec | Observed | Status |
|--------------|------|----------|--------|
| colors.primary | #1A1C1E | #6366F1 | ✗ off-token |
| body-md size | 1rem | 0.875rem | ✗ drift |
| ... | | | ✓ / ✗ |

## Top 3 Actions
1. [highest-leverage fix]
2. [next]
3. [next]
```

If the interface is genuinely strong, say so plainly and keep the findings list honest — do not invent problems to fill sections. If there are no findings at a priority level, omit that section's body and note "None."

## Reference Files

| File | Load when |
|------|-----------|
| `references/ai-slop-detection.md` | Always — step 3, the critical check |
| `references/critique-dimensions.md` | Always — step 4, the 10 dimensions |
| `references/design-md-spec.md` | A DESIGN.md or brand doc exists (steps 1, 5) |
| `references/visual-capture.md` | Evaluating a live URL or running app |
