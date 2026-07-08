# Authoring a DESIGN.md

How to write a valid `DESIGN.md` from the decided identity. The file has two layers: machine-readable tokens (YAML front matter) and human-readable rationale (markdown prose). Tokens are the normative values; prose explains intent.

## Contents
- File structure
- Token schema
- Section order
- Component tokens
- Full example
- Lint & export

## File Structure

```md
---
# YAML token front matter (normative values)
---

## Prose sections (the "why")
```

## Token Schema

```yaml
version: alpha             # optional
name: <string>            # required — the identity name
description: <string>     # optional
colors:
  <token-name>: <Color>   # any CSS color: #hex, rgb(), oklch(), named
typography:
  <token-name>:           # e.g. h1, body-md, label-caps
    fontFamily: <string>
    fontSize: <dimension>       # px | em | rem
    fontWeight: <number>?
    lineHeight: <number|dimension>?
    letterSpacing: <dimension>?
rounded:
  <level>: <dimension>    # sm, md, lg
spacing:
  <level>: <dimension|number>
components:
  <name>:
    <property>: <string | "{token.reference}">
```

- **Token reference:** `"{colors.primary}"` — must resolve to a defined token, or `lint` reports `broken-ref`.
- Define a `primary` color and at least one typography token; omitting them triggers `missing-primary`/`missing-typography` and invites generated defaults (slop).

## Section Order

Prose sections use `##` headings. Omit any, but those present must appear in this canonical order:

1. Overview (alias: Brand & Style) — personality/intent; the distinctive angle
2. Colors — the role of each color
3. Typography
4. Layout (alias: Layout & Spacing)
5. Elevation & Depth
6. Shapes
7. Components
8. Do's and Don'ts — explicit rules, including this brand's anti-slop lines

Duplicate headings are rejected. Unknown headings are preserved.

## Component Tokens

Map a component name to sub-token properties; express variants (hover/active/pressed) as related entries:

```yaml
components:
  button-primary:
    backgroundColor: "{colors.tertiary}"
    textColor: "{colors.neutral}"
    rounded: "{rounded.sm}"
    padding: 12px
  button-primary-hover:
    backgroundColor: "{colors.tertiary-container}"
```

Valid component properties: `backgroundColor`, `textColor`, `typography`, `rounded`, `padding`, `size`, `height`, `width`.

## Full Example

```md
---
name: Heritage
description: Architectural minimalism with journalistic gravitas.
colors:
  primary: "#1A1C1E"      # deep ink — headlines, core text
  secondary: "#6C7278"    # slate — borders, captions, metadata
  tertiary: "#B8422E"     # "Boston Clay" — the sole interaction driver
  neutral: "#F7F5F2"      # warm limestone foundation
typography:
  h1: { fontFamily: Public Sans, fontSize: 3rem, fontWeight: 700 }
  body-md: { fontFamily: Public Sans, fontSize: 1rem, lineHeight: 1.6 }
  label-caps: { fontFamily: Space Grotesk, fontSize: 0.75rem, letterSpacing: 0.08em }
rounded: { sm: 4px, md: 8px }
spacing: { sm: 8px, md: 16px, lg: 32px }
components:
  button-primary:
    backgroundColor: "{colors.tertiary}"
    textColor: "{colors.neutral}"
    rounded: "{rounded.sm}"
    padding: 12px
---

## Overview
Architectural minimalism meets journalistic gravitas — a premium matte finish,
like a contemporary gallery. The distinctive angle: a single warm clay accent
carries every interaction, so color *means* "act here" and nothing else.

## Colors
High-contrast neutrals with one accent.
- **Primary (#1A1C1E):** deep ink for headlines and body.
- **Secondary (#6C7278):** slate for borders, captions, metadata.
- **Tertiary (#B8422E):** Boston Clay — interaction only.
- **Neutral (#F7F5F2):** warm limestone, softer than pure white.

## Typography
Public Sans for text; Space Grotesk for small caps labels. Editorial size jumps.

## Layout
Generous vertical rhythm on an 8px scale; content-led, not centered-by-default.

## Do's and Don'ts
- Do reserve the clay accent for interactive elements only.
- Do lead with type and space for hierarchy.
- Don't introduce gradients, glow, or a second accent.
- Don't use pure white or the default indigo palette.
```

## Lint & Export

Validate before handoff:

```bash
npx @google/design.md lint DESIGN.md      # errors on broken refs; warns on contrast/missing/order
```

Fix `broken-ref` (errors) and drive `contrast-ratio` pairs to WCAG AA (4.5:1 body). To inject the current spec into context: `npx @google/design.md spec`.

Optionally export tokens once the file is settled:

```bash
npx @google/design.md export --format css-tailwind DESIGN.md > theme.css     # Tailwind v4 @theme
npx @google/design.md export --format json-tailwind DESIGN.md > theme.json    # Tailwind v3 config
npx @google/design.md export --format dtcg DESIGN.md > tokens.json            # W3C tokens
```
