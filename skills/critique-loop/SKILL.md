---
name: critique-loop
description: Drive a subject to convergence by repeatedly completing its critique-generated Dex epic, re-running the same critique, and reporting what changed — until it's clean or hits a stop condition. Use when asked to "loop the critique", "keep fixing until clean", "re-critique after the fixes", "iterate on the design/UX until it converges", "complete the epic then re-run the critic", or "run the critique loop". Works with any critique that emits the shared report contract (ui-design-critique, ux-critique, …).
metadata:
  category: Planning, Thinking & Workflow
---

# Critique Loop

Close the loop between critique and fix. One critique pass is rarely enough: fixes miss, land partially, or introduce new problems. This skill runs rounds — complete the epic, re-critique, compare, decide — until the subject converges or a stop condition trips.

It orchestrates existing skills; it does not re-specify them. Critiquing is done by a critique skill (`ui-design-critique`, `ux-critique`, or any that emits the shared report contract). Backlogging is done by `critique-to-dex`. Task tracking is done by the `dex` skill. Building fixes is delegated to the appropriate implementation skill.

## The Invariants (do not violate)

The comparison across rounds is only meaningful if the experiment is controlled. Hold these **constant for the entire loop**:

1. **Same critique skill** — you cannot compare a UX round to a UI round.
2. **Same scope** — the same screen/flow/component.
3. **Same anchors** — the same `DESIGN.md` / `USERS.md` (and same user type, for UX). If an anchor changes mid-loop, the loop restarts at round 1.

If any invariant would change, stop and tell the user; do not silently continue.

## Preconditions

- A subject that has been (or will be) critiqued with a chosen critique skill.
- The anchors that critique uses, held constant.
- A way to build the fixes (a dev environment + the relevant implementation skill).
- Either an existing Dex epic from `critique-to-dex`, or none (the loop bootstraps round 1).

## The Loop

Track progress with this checklist per round:

```
Round N
- [ ] 1. Ensure a current epic exists
- [ ] 2. Complete the epic (all tasks verified)
- [ ] 3. Re-run the SAME critique (same scope + anchors)
- [ ] 4. Compare to the prior round (delta taxonomy)
- [ ] 5. Report the round
- [ ] 6. Decide: continue / converged / stop
```

### 1. Ensure a current epic exists

If starting fresh: run the chosen critique, then `critique-to-dex` to create the round-1 epic. If an epic already exists from a prior step, use it as the current round's epic. Record which findings (F-IDs + their issue/location) this epic represents — that is the **prior finding set** for the next comparison.

### 2. Complete the epic

Work every actionable task to done, delegating the building to the appropriate implementation skill — for UI/visual fixes apply `design-engineering` craft rather than hacking CSS ad hoc (ad-hoc edits are the usual cause of spacing/alignment regressions each round). Use the exact target values from the finding's recommendation. For each task: implement the fix, then **verify it** the way that critique verifies (re-walk the flow for `ux-critique`; re-capture the screen for `ui-design-critique`) plus the project's own gates (build/check/tests). Mark each task complete in `dex` with concrete evidence. Do not mark the epic done on intent — only on verified completion.

### 3. Re-run the same critique

Run the identical critique skill against the identical scope with the identical anchors. Produce a fresh report (the **current finding set**). If the critique defines a deterministic check (e.g. `ui-design-critique`'s broken-UI smoke test), it must actually be re-run this round — a report whose smoke test was skipped or carried over from a prior round is invalid for comparison.

### 4. Compare to the prior round

Match current findings against the prior finding set to classify each — Resolved, Persisting, New, Regression. Load `references/convergence.md` for the matching rules and the delta taxonomy.

### 5. Report the round

Emit the round report (template below): counts, the delta lists, and the new priority profile. Regressions and Persisting findings are surfaced first — they are the signal that fixes are failing.

### 6. Decide

Apply the stop conditions in `references/convergence.md`. **Converged requires both** no P0/P1 findings **and** a passing objective gate (the critique's deterministic check, re-run fresh this round) — critic judgment alone never closes the loop when a deterministic check exists. If continuing, feed the current finding set into `critique-to-dex` to create the next round's epic (Persisting and Regression findings carry a note that a prior fix attempt failed), then loop to step 1 as round N+1.

## Autonomy

Default: **confirm at each round boundary** — present the round report and ask whether to continue. Offer an autonomous mode ("run up to N rounds, then report") when the user asks for it; even then, **always stop early and surface** regressions or a thrash condition rather than pressing on.

## Guardrails

- **Never bury a regression.** If the round introduced new problems, that leads the report.
- **Escalate persisting findings.** A finding that survives a dedicated fix means the approach is likely wrong — flag it for human judgment instead of retrying the same fix blindly.
- **Respect the loop budget.** Default max 3 rounds; stop and report rather than looping indefinitely.
- **Stop on thrash.** If a round's regressions ≥ its resolved findings, the changes are making it worse net — stop and surface.
- **Verify before re-critiquing.** An unverified epic invalidates the comparison.

## Round Report Template

```markdown
## Critique Loop — Round N — [subject]
- **Critique:** [skill] · **Anchors:** [DESIGN.md/USERS.md + user type] (constant) · **Scope:** [same]
- **Epic:** <epic_id> — [M/M tasks verified]
- **Current findings:** X — P0: _ · P1: _ · P2: _ · P3: _
- **Objective gate:** Clean at [viewports] / X detector hits / N/A (critique has no deterministic check)

### Delta vs Round N-1
- **⚠ Regressions (new, caused by this round):** [F-ids + titles] — or None
- **↺ Persisting (fix missed/partial — escalate):** [prior F-ids still present] — or None
- **✓ Resolved:** [prior F-ids now gone]
- **+ New (not regressions):** [new F-ids]
- **Net:** [e.g. −4 P0/P1, +1 regression]

### Decision
CONVERGED — [reason] · or · CONTINUE → round N+1 epic <id> · or · STOP — [stop condition]
```

## Final Report Template (when the loop ends)

```markdown
## Critique Loop Complete — [subject]
- **Rounds:** N · **Outcome:** Converged / Stopped ([reason]) · **Objective gate:** Passed / Failed / N/A
- **Trajectory:** Round 1 [P0/P1 count] → Round N [P0/P1 count]
- **Resolved overall:** [count] · **Still open:** [remaining findings by priority]
- **Unresolved escalations:** [persisting findings needing human judgment, if any]
- **Next:** [nothing — converged / a specific human decision needed]
```

## Reference Files

| File | Load when |
|------|-----------|
| `references/convergence.md` | Always — matching rules, delta taxonomy, and stop conditions |
