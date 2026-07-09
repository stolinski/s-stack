# Convergence: Matching, Delta Taxonomy, Stop Conditions

How to compare two critique rounds and decide whether to loop again.

## Contents
- Matching findings across rounds
- Delta taxonomy
- Stop conditions
- Thrash detection

## Matching Findings Across Rounds

To classify a current-round finding, match it against the prior finding set. A current finding is the **same** as a prior finding when it shares:

1. the same **dimension** (e.g. Visual Hierarchy, Error Recovery, Dark Pattern), AND
2. the same **location/element** (same component, step, or token), AND
3. a **substantively equivalent issue** (the same problem, even if worded differently).

Guidance:
- Match on meaning, not F-id — F-ids are per-report and do not carry across rounds. Keep a mapping (prior F-id → current F-id) in the report so a reader can trace a finding.
- A finding at the same location whose *severity or specifics changed* but whose root problem remains is **Persisting**, not Resolved — note the change.
- A finding whose location was refactored away counts as Resolved for that location even if a related problem appears elsewhere (that new one is New/Regression).
- When unsure whether two findings are the same, treat them as Persisting (conservative — avoids falsely claiming a fix worked).

## Delta Taxonomy

Classify every finding after matching:

| Class | Definition | How to treat |
|-------|------------|--------------|
| **Resolved** | A prior finding has no match in the current round | Count as progress; note the F-id mapping |
| **Persisting** | A prior finding still matches in the current round | The fix missed or was partial — **escalate**; do not blindly re-queue the same fix |
| **New** | A current finding with no prior match, not attributable to this round's changes | Normal backlog item next round |
| **Regression** | A New finding **caused by** this round's changes (touched the area, or a strength that was preserved is now broken) | **Highest priority**; leads the report; never bury |

Distinguishing New from Regression: a New finding is a Regression when it lands in code/area the round modified, or when a prior-round **Strength** is now a finding. When in doubt and the area was touched this round, classify as Regression (conservative — surfaces risk).

## Stop Conditions

Stop the loop when ANY holds. Report which one tripped.

| Condition | Test | Meaning |
|-----------|------|---------|
| **Converged** | No P0 or P1 findings remain | Clean enough to ship; P2/P3 can be deferred |
| **Steady state** | No Regressions and no actionable New/Persisting (only P3/suggestions left) | Further looping yields diminishing returns |
| **Thrash** | Regressions ≥ Resolved this round | Changes are net-negative; stop before making it worse |
| **Persisting wall** | A finding persisted across 2 consecutive fix attempts | The fix approach is wrong — needs a human decision, not another round |
| **Budget** | Round count reached the max (default 3) | Safety bound; report state and hand back |
| **User stop** | The user ends the loop at a round boundary | — |

Default "done" is **Converged** (no P0/P1). Confirm the target with the user if they want a stricter bar (e.g. zero findings) or a looser one (e.g. no P0 only).

## Thrash Detection

Track the P0/P1 count and the regression count per round. Healthy convergence shows P0/P1 trending down each round. Warning signs:

- P0/P1 count flat or rising across two rounds.
- Regressions appearing every round.
- The same finding oscillating (Resolved one round, back the next).

On any of these, stop and surface the trajectory rather than continuing — the loop is not converging and needs a human to change approach.
