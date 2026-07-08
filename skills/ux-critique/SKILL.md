---
name: ux-critique
description: UX critique for interfaces and flows that produces a prioritized, strict report of experience issues. Use when asked to "critique the UX", "UX review", "usability review", "heuristic evaluation", "why is this flow confusing", "is this hard to use", "find friction", or evaluate whether a user can complete a task. Walks the actual flow, classifies every finding as a gap, inconsistency, suggestion, or strength, and ranks by impact on task success. Flags dark patterns. Shares the critique report contract with ui-design-critique so critique-to-dex ingests it directly.
metadata:
  category: Frontend, UI & Design
---

# UX Critique

Evaluate how well a user can understand and complete their goal. Output a single report that ranks experience issues by impact on task success, classifies each finding, documents the actual walkthrough, and calls out what works.

This is the experience counterpart to `ui-design-critique` (visual design). Where that skill judges how it *looks*, this judges how it *works*: task success, friction, feedback, recovery, cognitive load, navigation, forms, and manipulative patterns. The two are complementary — run both for a full picture.

**The fundamental difference:** aesthetics are brand-relative, so `ui-design-critique` needs a stated intent (a DESIGN.md) to judge against. UX is not brand-relative. It is judged against **universal usability truths** that hold regardless of brand, aesthetic, or taste — a keyboard trap, a dead-end flow, a silent failure, and a deceptive cancel path are wrong everywhere. So this skill needs **no design spec, DESIGN.md, or equivalent**. The principles in `references/ux-heuristics.md` and `references/dark-patterns.md` are the standard. The only thing you establish per-critique is the *scenario* — the user and their goal — not a source-of-truth document.

Scope: this skill evaluates and reports. It does not implement fixes. For building/polishing interaction and motion feel, use `design-engineering`; for self-evidence/affordance authoring, `ux-interface-design`.

## Non-Negotiables

1. **Judge against universal usability truths and the user's goal, not aesthetics or a spec.** The heuristics and dark-pattern checks are objective standards that apply to any product. Establish the task the flow is meant to support first; a finding matters to the degree it blocks or slows that goal. Never treat a provided design/journey doc as the standard — it describes intent, and intent can itself be wrong.
2. **Walk the real flow.** Do not critique UX from a single static screenshot when the flow is runnable. Step through it. See `references/task-walkthrough.md`.
3. **Dark patterns are a first-class, P0-capable check.** Manipulation, forced friction, and deception harm the user even when "usable." See `references/dark-patterns.md`.
4. **Every finding needs evidence, a priority, and a classification.** Point to the step/screen/element. No unranked, unclassified, or evidence-free claims.
5. **Report strengths too.** Name what works so it is preserved.

## Inputs

| Input | How to evaluate |
|-------|-----------------|
| Live URL / running app | Walk the flow — see `references/task-walkthrough.md` |
| A described task or user goal | Use it as the success criterion for the walkthrough |
| `USERS.md` (personas) | Draw the scenarios (real user goals) and the priority lens (which truths matter most) — see note below |
| Screenshots / a recording | Reconstruct the flow from them; note steps you could not verify |
| Component / page code | Read it; render + walk if a dev server is available |

If a `USERS.md` exists (from `grill-users`), use it to choose realistic scenarios and to weight priority — but it sets the *scenario and emphasis*, never the usability standard. A product spec or user-journey doc is likewise only useful for **inferring the intended task/goal**, never as the standard the flow is judged against. The standard is always the universal principles.

If no task is stated, infer the primary task from context and confirm it once, or ask. A critique with no defined goal is unfocused.

## Process

### 1. Establish the task and success criteria

State the primary user, their goal, and what "done" looks like (the observable success state) — the *scenario*, not a standard. Note secondary tasks worth checking. This defines what the walkthrough tries to accomplish; the heuristics define whether it does so well.

If a `USERS.md` exists, take the primary user, the `goals`/`success` scenario, and the `cares_about`/`stakes` from it rather than guessing — and record which user type you are critiquing for. Different types weight priority differently (an expert who cares about speed makes accelerators high-priority; a high-stakes novice makes error-prevention high-priority).

### 2. Walk the flow

Follow `references/task-walkthrough.md`: step through the task end to end, recording each step, decision point, click, wait, and dead end. Capture friction where the user would hesitate, backtrack, or fail. Check the unhappy paths too (bad input, back button, interruption, empty/slow/error states in the flow).

### 3. Run the heuristic pass

Load `references/ux-heuristics.md` and evaluate each dimension against what the walkthrough revealed. For each issue: step/location, what's wrong, why it harms the goal, classification, priority.

### 4. Run the dark-pattern check

Load `references/dark-patterns.md`. Flag any manipulation, forced friction, or deception. These are high-priority regardless of surface polish.

### 5. Classify, prioritize, and write the report

Use the taxonomy and priority scale below and the shared report template.

## Classification Taxonomy

One classification per finding (identical to `ui-design-critique`, so downstream tooling is uniform).

| Classification | Meaning (UX) |
|----------------|--------------|
| **Gap** | Something required is missing — no feedback on a key action, no error recovery, no undo/cancel, an unhandled state in the flow, no keyboard path, no confirmation before a destructive action. |
| **Inconsistency** | Behavior contradicts platform convention or itself — a label that doesn't match what the action does, two flows for the same goal that differ, back/escape behaving unexpectedly. |
| **Suggestion** | The flow works but could be smoother — an accelerator, a smarter default, progressive disclosure, a defensible alternative. Label taste-based ones as such. |
| **Strength** | Something done well. Name it so it is preserved. |

## Priority Scale (impact on the user's goal)

| Priority | Bar | Examples |
|----------|-----|----------|
| **P0 — Critical** | Blocks task completion, risks data loss, deceives/harms the user, or blocks a whole input mode | Dead-end with no path forward; destructive action with no confirm/undo; dark pattern; keyboard trap or unlabeled control on the core path |
| **P1 — High** | Major friction; likely abandonment or error | No feedback after a key action; unrecoverable error message; hidden required step; heavy cognitive load at a decision point |
| **P2 — Medium** | Noticeable friction or confusion | Extra unnecessary steps; inconsistent behavior; poor validation timing; weak wayfinding |
| **P3 — Low** | Minor friction or refinement | Missing accelerator; small recall burden; micro-hesitation |

Sort findings P0 → P3; within a priority, order Gap, then Inconsistency, then Suggestion. Weight impact by the user type being critiqued for: the same issue can be P1 for one user type and P3 for another depending on their `cares_about` and `stakes` in `USERS.md`. State the user type in the verdict.

## Report Template

ALWAYS use this structure. It shares the critique report contract with `ui-design-critique` — same finding block and taxonomy — so `critique-to-dex` maps it 1:1.

```markdown
# UX Critique: [Flow / Task / Screen]

## Verdict
[2–4 sentences: can the user succeed, where it breaks down, and the single most important fix.]

- **Task:** [the goal being evaluated] — for [primary user]
- **Task Completion:** Yes / Partial / No / N/A — [one-line reason]
- **Friction Level:** Low / Moderate / High / Severe — [one-line reason]
- **Dark Patterns:** None / Present — [what, if any]
- **Findings:** X total — P0: _ · P1: _ · P2: _ · P3: _
- **Verified:** [paths/states/input modes walked; note anything unverified]

## Flow Walkthrough
Numbered steps actually taken, with friction noted inline:
1. [step] → [what happened] [· friction: ...]
2. ...
[Mark the point of failure or the success state reached.]

## Findings

### P0 — Critical
#### [F-01] [Short title] · [Gap | Inconsistency | Suggestion]
- **Dimension:** [e.g. Dark Pattern / System Status / Error Recovery / Task Flow]
- **Location:** [step, screen, or element]
- **Issue:** [what is wrong]
- **Impact:** [how it harms the user's goal]
- **Evidence:** [step observed, message text, click count, screenshot region]
- **Recommendation:** [concrete fix]

[continue F-02, F-03, ... across all priority sections]

### P1 — High
...
### P2 — Medium
...
### P3 — Low
...

## What Works (Strengths)
- **[Dimension]:** [specific thing done well and why — preserve this]

## Top 3 Actions
1. [highest-leverage fix]
2. ...
3. ...
```

If no findings exist at a priority level, note "None." Do not invent friction to fill sections; if the flow is genuinely smooth, say so and keep the list honest.

## Reference Files

| File | Load when |
|------|-----------|
| `references/task-walkthrough.md` | Always — how to define and walk the flow |
| `references/ux-heuristics.md` | Always — the evaluation dimensions |
| `references/dark-patterns.md` | Always — the manipulation/friction check |
