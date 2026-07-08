# Using DESIGN.md as the Source of Truth

`DESIGN.md` (the [google-labs-code/design.md](https://github.com/google-labs-code/design.md) format) is a structured description of a visual identity: machine-readable design tokens in YAML front matter plus human-readable rationale in markdown prose. When one exists, it is the source of truth for the critique — the tokens define correct values, and the prose defines intent.

## Contents
- File shape
- What to extract
- Automated linting
- Comparing UI to tokens
- Turning deviations into findings
- If no DESIGN.md exists

## File Shape

```md
---
name: Heritage
colors:
  primary: "#1A1C1E"
  secondary: "#6C7278"
  tertiary: "#B8422E"      # the single accent / interaction driver
  neutral: "#F7F5F2"
typography:
  h1: { fontFamily: Public Sans, fontSize: 3rem }
  body-md: { fontFamily: Public Sans, fontSize: 1rem }
rounded: { sm: 4px, md: 8px }
spacing: { sm: 8px, md: 16px }
components:
  button-primary:
    backgroundColor: "{colors.tertiary}"
    textColor: "{colors.neutral}"
    rounded: "{rounded.sm}"
---

## Overview        # brand intent / personality — the "why"
## Colors          # role of each color
## Typography
## Layout
## Components
## Do's and Don'ts  # explicit constraints — treat violations as findings
```

Token types: colors (any CSS color), dimensions (`px`/`em`/`rem`), typography objects, and token references like `{colors.primary}`. Section order is canonical: Overview → Colors → Typography → Layout → Elevation → Shapes → Components → Do's and Don'ts.

## What to Extract

1. **Palette + roles** — which color is primary, which is the accent/interaction driver, which are neutrals. Note the intended emotional read from the Overview.
2. **Type system** — families, sizes, the intended hierarchy.
3. **Spacing + radius scales** — the allowed values.
4. **Component tokens** — exact styling per component and its variants (hover/active/pressed).
5. **Do's and Don'ts** — explicit rules; any violation is a direct finding.
6. **Intent prose** — the personality the UI must evoke (feeds dimension 4, Emotional Resonance).

## Automated Linting

Run the linter to catch structural and contrast problems in the spec itself:

```bash
npx @google/design.md lint DESIGN.md
```

Findings map directly into the report:

| Lint rule | Severity | Report as |
|-----------|----------|-----------|
| `broken-ref` | error | Gap — token reference resolves to nothing |
| `contrast-ratio` | warning | Gap/Inconsistency — component pair below WCAG AA (feeds dimension 8) |
| `missing-primary` / `missing-typography` | warning | Gap — undefined foundation, agents will auto-generate (slop risk) |
| `orphaned-tokens` | warning | Suggestion — defined but unused |
| `section-order` / `unknown-key` | warning | Inconsistency — spec hygiene |

To inject the full format spec into context when needed: `npx @google/design.md spec`.

## Comparing UI to Tokens

For the rendered interface, sample the actual values (from screenshots, computed styles, or code) and compare to the spec:

- **Color:** every meaningful color should resolve to a defined token. Off-token colors are Inconsistencies. The accent should appear only where the spec says interaction happens.
- **Type:** heading/body sizes and families must match the type tokens. Drift (e.g. `0.875rem` where `body-md` is `1rem`) is an Inconsistency.
- **Spacing/radius:** values should come from the scales. One-off values are Inconsistencies unless justified.
- **Components:** each component should match its token block, including variant states.

Populate the **Spec Conformance** table in the report with `Token/Rule · Spec · Observed · Status`.

## Turning Deviations into Findings

- Deviation from a defined token → **Inconsistency**. Priority by impact (off-brand primary color = P1; a single off-scale margin = P3).
- Violation of a "Don't" rule → **Gap or Inconsistency**, priority by impact.
- Missing foundational token (no primary, no type) → **Gap**, and a slop risk.
- A deviation that is clearly intentional and better than the spec → **Suggestion** to update the spec, not a defect.

## If No DESIGN.md Exists

- Ask once whether a brand/style guide exists; if so, extract the same intent (colors, type, tone).
- If nothing exists, record "No design spec — intent cannot be verified" as a **Gap** (typically P2), critique against general craft standards, and offer to scaffold a DESIGN.md so future critiques have a source of truth.
