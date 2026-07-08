# Priority Findings Contract

The normalized shape this skill consumes. Any critique that emits findings in this shape converts to a Dex epic cleanly. This is the generalization boundary — future critique skills should target this contract rather than a bespoke format.

## Contents
- Contract schema
- Field semantics
- Adapter: ui-design-critique
- Adapter: generic prioritized list
- Handling missing fields

## Contract Schema

```yaml
subject: string            # what was critiqued — becomes the epic subject
source_report: string?     # file path or "conversation" — pointer for the epic
findings:
  - id: string             # stable handle, e.g. "F-01" (generate one if absent)
    title: string          # short imperative summary of the fix
    priority: string       # source priority token (P0/High/Critical/…) — normalized later
    classification: enum   # Gap | Inconsistency | Suggestion | Strength
    location: string?      # element / file / token / region
    issue: string          # what is wrong
    impact: string         # why it matters
    evidence: string?      # screenshot region, token diff, code ref, measurement
    recommendation: string # the concrete fix
```

Only `subject`, and per finding `title`, `priority`, `classification`, and `issue` + `recommendation` are strictly required to create a useful task. The rest enrich the task body.

## Field Semantics

- **classification** drives whether a finding becomes a task and how it's grouped (see SKILL.md mapping). `Strength` is never a task.
- **priority** is normalized to a rank (see SKILL.md priority table). Preserve the original token in the task for traceability.
- **id** must be stable so tasks trace back to the report. If the source has none, assign `F-01`, `F-02`, … in report order.
- **recommendation** becomes the "How"; **issue**+**location** the "What"; **impact** the "Why". Derive "Done when" from the recommendation.

## Adapter: critique report contract (ui-design-critique / ux-critique)

`ui-design-critique` and `ux-critique` share one report contract, so this single adapter covers both. The `ux-critique` report adds a `## Flow Walkthrough` section (context for the epic, not findings) and uses UX dimensions, but the finding block, `[F-01]` ids, `P0–P3` sections, `· classification` tags, and `What Works`/`What Looks Good` strengths are identical. Map 1:1:

| Report field | Contract field |
|--------------|----------------|
| `# Design Critique: [X]` heading | `subject` |
| `[F-01]` id in each finding | `id` |
| Finding short title | `title` |
| `P0/P1/P2/P3` section | `priority` |
| `· [Gap\|Inconsistency\|Suggestion]` tag | `classification` |
| **Location** | `location` |
| **Issue** | `issue` |
| **Impact** | `impact` |
| **Evidence** | `evidence` |
| **Recommendation** | `recommendation` |
| **What Looks Good** / **What Works** bullets | findings with `classification: Strength` |

Both reports are already priority-sorted (P0→P3), so preserve their order. For `ux-critique`, the `## Flow Walkthrough` section feeds the epic description, not individual tasks.

## Adapter: Generic Prioritized List

For a looser list (bullets, a table, a numbered list):

- Treat each item as a finding. Derive `title` from the item text.
- Look for a priority marker (P0–P3, Critical/High/Medium/Low, 🔴/🟠/🟡, "must/should/could"). If none, ask the user for a priority or default everything to rank 3 (Medium) and say so.
- Infer `classification`: missing thing → Gap; contradiction/off-spec → Inconsistency; "consider/maybe/could" → Suggestion; praise → Strength.
- If items have no fix, treat the item text as both `issue` and `recommendation` and flag that the recommendation is thin.

## Handling Missing Fields

- Missing `evidence` → omit the Evidence line; do not fabricate a screenshot or measurement.
- Missing `impact` → infer a brief impact from the issue, and mark it inferred.
- Missing `recommendation` → create the task but flag "needs remediation detail" in the body so it isn't started blind.
- Missing `priority` across the whole source → ask once for a scheme, or default all to Medium and state the assumption.
