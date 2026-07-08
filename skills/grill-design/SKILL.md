---
name: grill-design
description: Interview the user to create a unique DESIGN.md that defines an app's look. Use when asked to "grill me for a design.md", "create a design spec", "define the app's look", "make a DESIGN.md", "build a brand/design system doc", or "interview me for a visual identity". Grills one question at a time toward a committed, non-generic identity, then authors a valid DESIGN.md (tokens + prose) and lints it. Actively steers away from AI-slop defaults.
metadata:
  category: Frontend, UI & Design
---

# DESIGN.md Grill

Interview the user to pin down a distinctive visual identity, then author a valid `DESIGN.md` (the [design.md](https://github.com/google-labs-code/design.md) format) that defines the app's look. The output is a committed source of truth for how the app looks.

If a `USERS.md` exists, it is the most important input: the identity must be built *for* those user types. Read it before grilling and let who the product is for drive every recommendation.

The job is not to record whatever the user says. It is to push past generic answers ("modern, clean, minimal") to a specific, defensible identity with a point of view, and to keep the result off the AI-slop treadmill.

## Non-Negotiables

1. **Grill, don't transcribe.** One question at a time. Reject vague answers and push for specificity. Follow the grilling method below.
2. **Force uniqueness.** The identity must include at least one committed, non-obvious choice (a distinctive accent, a characterful typeface, a clear compositional stance) and captured anti-references. If every answer is a default, the identity is slop — say so and dig.
3. **Steer away from defaults.** Do not let the palette drift to indigo/violet Tailwind defaults or the type to Inter-by-reflex unless the user makes that choice deliberately and for a reason. See the anti-slop guardrails in `references/grill-questions.md`.
4. **Ground in the users when defined.** If a `USERS.md` exists, the identity must resonate with each user type's `resonance_cue`. Use it to shape recommended answers, skip questions it already answers, and flag any identity choice that would alienate the primary user.
5. **Author a valid, linted spec.** The output must parse against the design.md schema and pass `lint` (or have every remaining finding explained).

## Grilling Method

Apply the `grilling` skill's method:

- Ask ONE consequential question at a time, then wait.
- Attach a **recommended answer** with a brief reason to every question.
- Treat the user's correction as truth immediately; never drift back to your earlier recommendation.
- **Explore before asking.** Inspect the repo first — existing logo/brand assets, `tailwind.config`, CSS custom properties, current components, product copy, README. Do not ask what the code already answers.
- **Anchor to the users.** When a `USERS.md` exists, ground every recommended answer in the primary user's `resonance_cue`, context, and stakes, and skip questions it already answers (audience, the feeling it should evoke).
- Stop when the identity has enough load-bearing structure to author a coherent DESIGN.md.

## Process

### 1. Recon

Scan the repo for anything that already implies an identity: brand assets, current colors/fonts, existing DESIGN.md, product/marketing copy, target-audience hints. If a `USERS.md` exists, read it first and treat it as the primary brief — the identity must resonate with those user types (their `resonance_cue`), so let who the product is for shape the look. Summarize what you found so questions build on it instead of restarting.

### 2. Grill through the decision tree

Work the question tree in `references/grill-questions.md`, in order, one question per turn. The tree covers: product & feeling, references & anti-references, color & roles, typography, layout & density, depth & shape, motion posture, and explicit do's/don'ts. Reject generic answers using the guardrails in that file.

### 3. Draft the identity back

Before authoring, reflect the decided identity in 4–6 sentences (personality, palette roles, type, spatial stance, the one distinctive choice). If a `USERS.md` exists, state how the identity resonates with each user type and flag any tension with the primary user. Confirm or correct with the user. This is the last checkpoint before tokens.

### 4. Author DESIGN.md

Write a valid `DESIGN.md` following `references/design-md-format.md`: YAML token front matter + prose sections in canonical order, with a real **Do's and Don'ts** section that encodes this brand's anti-slop rules. Default location: repo root `DESIGN.md` — confirm the path.

### 5. Lint and fix

Run:

```bash
npx @google/design.md lint DESIGN.md
```

Fix `broken-ref` and `contrast-ratio` findings (adjust tokens until WCAG AA passes). Resolve `missing-primary`/`missing-typography` (never ship an undefined foundation — that invites slop). Explain any finding left unresolved.

### 6. Hand off

Report the file path, the distinctive choice(s) that make it unique, and lint status.

## What "Unique" Requires

Refuse to finish until the identity clears this bar:

- **A committed palette**, not the framework default. High-contrast neutrals plus an intentional accent are a strong baseline; the accent should not be reflexive indigo/violet.
- **Type with a point of view** — at least one family chosen for character, and a real hierarchy.
- **A spatial stance** — stated density and radius language (sharp vs. soft, airy vs. dense), not "whatever's default".
- **Depth from layering/contrast**, not glow and blur.
- **Anti-references** — named things the identity deliberately rejects. Uniqueness is defined as much by what it refuses as what it embraces.
- **Resonance with the users** — if a `USERS.md` exists, the identity must fit each type's `resonance_cue`; a look that alienates the primary user is not "unique", it is wrong.

If the user genuinely wants a safe, conventional look, that is allowed — but make it a deliberate, stated choice, and still commit specific values.

## Reference Files

| File | Load when |
|------|-----------|
| `references/grill-questions.md` | Always — the question tree + anti-slop guardrails |
| `references/design-md-format.md` | Authoring the file (steps 4–5) |
