---
name: ui-design-critique
description: Design critique for user interfaces that produces a prioritized, strict report of design issues. Use when asked to "critique this design", "review the UI", "design review", "roast my UI", "is this AI slop", "does this look AI-generated", "audit the visual design", or evaluate a screen, page, or screenshot against brand guidelines or a DESIGN.md spec. Classifies every finding as a gap, inconsistency, suggestion, or strength and ranks it by impact. Runs deterministic DOM detectors to catch broken layout (overflow, overlap, clipping) and evaluates visually via browser/devtools capture and against design tokens.
metadata:
  category: Frontend, UI & Design
---

# UI Design Critique

Produce a rigorous, prioritized design critique of a user interface. Output a single report that lists every issue ranked by impact, classifies each finding, and calls out what already works. Be strict. The goal is a report a designer can act on top-to-bottom, not vague praise.

This skill evaluates and reports. It does not implement fixes. For building or polishing UI, use `design-engineering`. For self-evidence/affordance depth, cross-reference `ux-interface-design`.

## Non-Negotiables

1. **Breakage is computed, not judged.** Run the deterministic broken-UI smoke test (`references/broken-ui-detectors.md`) before any judgment-based review. Vision review reliably misses overflow, overlap, clipping, and misalignment — never rely on "looking at" a screenshot to find them. A page that fails the smoke test is broken regardless of its aesthetics.
2. **AI Slop Detection is the most important judgment check.** Run it immediately after the smoke test. See `references/ai-slop-detection.md`.
3. **Critique against intent, not personal taste.** When a DESIGN.md spec exists, it is the source of truth. Distinguish a violation of intent (a real finding) from a stylistic preference (a suggestion, clearly labeled).
4. **Every finding needs evidence.** Point to the specific element, region, or token. No pattern-only claims like "typography feels off" without naming the element and the reason.
5. **Every finding needs a priority and a classification.** No unranked, unclassified findings.
6. **Report strengths too.** A critique that only lists problems is incomplete and untrustworthy.

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

- Live UI: follow `references/visual-capture.md` to screenshot key states (default, hover/focus, empty, loading, error) and viewports (mobile + desktop at minimum). Capture at **full resolution** and tile long pages — downscaled or squeezed captures hide exactly the defects being hunted.
- Screenshots only: work from what is provided; note any states/viewports you could not verify.

### 3. Run the broken-UI smoke test (deterministic — MANDATORY on live UI)

Load `references/broken-ui-detectors.md` and run **every detector at every viewport in the matrix** via `evaluate_script`: page overflow, element overlap, text clipping, out-of-viewport content, broken/distorted images, zero-size content, unreadable text. Detector hits are objective P0/P1 findings — confirm each with an element-level screenshot and report the measured values. Do not proceed to judgment-based review as if the page were fine when the smoke test failed. On screenshots-only input, note that the smoke test could not be run.

### 4. Run AI Slop Detection (CRITICAL)

Load `references/ai-slop-detection.md`. Score the interface against every fingerprint. Answer the core test: *If someone said "AI made this," would they be believed instantly?* Record the AI-slop risk level in the summary. Slop tells are high-priority findings.

### 5. Run the 11 critique dimensions

Load `references/critique-dimensions.md` and evaluate each dimension. For each issue found, capture: element/location, what is wrong, why it matters, classification, priority.

### 6. Run the precision audit (measured — do not eyeball)

Load `references/precision-audit.md` and **measure** spacing, alignment, sizing, rhythm, and containment with real pixel values (`getBoundingClientRect` / `getComputedStyle` across sets of like elements). Derive selectors from the actual DOM first — an empty selector result means the selector is wrong, not that the page is clean. This is dimension 11 and it is mandatory — most "it looks broken / sloppy / off" problems live here and are invisible to judgment-based review. Every precision finding's recommendation must state the exact current values and the target value.

### 7. Check spec conformance (if a spec exists)

Compare rendered UI to the tokens: colors, type scale, spacing, radius, component styles. Every deviation from a defined token is an **Inconsistency** unless the deviation is clearly intentional and justified.

### 8. Classify, prioritize, and write the report

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
| **P0 — Critical** | Breaks usability, trust, or accessibility; visibly broken layout; or screams AI slop | No discernible primary action; unreadable contrast; content overflow/clipping; elements overlapping or touching edges; interface indistinguishable from generic AI output; broken/absent core states |
| **P1 — High** | Meaningfully hurts comprehension, flow, or brand fit; or reads as sloppy | Weak/competing hierarchy; off-brand palette; confusing IA; illegible body text; visible misalignment (>3px); mismatched control heights on the primary action |
| **P2 — Medium** | Noticeable friction or polish gap | Inconsistent padding across like elements; off-scale spacing; uneven rhythm; weak affordances; inconsistent component styling; flat empty states |
| **P3 — Low** | Minor refinement or preference | 1–2px optical alignment nits; microcopy tightening; optional delight |

Treat visible layout breakage and measurable inconsistency as **objective** findings, not taste — broken spacing and misalignment are always in scope regardless of any brand spec.

Sort findings P0 → P3. Within a priority, order by classification severity: Gap, Inconsistency, then Suggestion.

## Report Template

ALWAYS output this exact structure.

```markdown
# Design Critique: [Screen / Page / Component]

## Verdict
[2–4 sentences: overall quality, biggest risk, and the single most important thing to fix.]

- **Smoke Test:** Clean / X defects / Not run (screenshots only) — [viewports checked, e.g. "clean at 1280, 3 hits at 375"]
- **AI-Slop Risk:** None / Low / Moderate / High / Severe — [one-line reason]
- **Execution:** Clean / Minor issues / Sloppy / Broken — [one-line reason from the measured precision audit]
- **Spec Conformance:** N/A (no spec) / Strong / Partial / Poor — [one-line reason]
- **Findings:** X total — P0: _ · P1: _ · P2: _ · P3: _
- **Verified:** [states/viewports checked; note whether the smoke test and spacing/alignment were measured or only eyeballed]

## Findings

### P0 — Critical
#### [F-01] [Short title] · [Gap | Inconsistency | Suggestion]
- **Dimension:** [e.g. AI Slop / Visual Hierarchy / Color]
- **Location:** [element, region, or token — be specific]
- **Issue:** [what is wrong]
- **Impact:** [why it matters to the user or brand]
- **Evidence:** [screenshot region, token value vs. spec, code ref, or measured values — e.g. "padding 14/16/13px"]
- **Recommendation:** [concrete, actionable fix — for precision findings, state the exact target value, e.g. "set all three to 16px (`spacing.md`)"]

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
| `references/broken-ui-detectors.md` | Always on live UI — step 3, the deterministic smoke test |
| `references/ai-slop-detection.md` | Always — step 4, the critical judgment check |
| `references/critique-dimensions.md` | Always — step 5, the 11 dimensions |
| `references/precision-audit.md` | Always — step 6, the measured spacing/alignment/sizing pass |
| `references/design-md-spec.md` | A DESIGN.md or brand doc exists (steps 1, 7) |
| `references/visual-capture.md` | Evaluating a live URL or running app |
