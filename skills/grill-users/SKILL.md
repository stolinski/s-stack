---
name: grill-users
description: Interview the user to define who an app's users actually are. Use when asked to "define our users", "identify the users", "who is this for", "create user personas", "make a USERS.md", "define the target audience", or "what user types do we have". Grills one question at a time to produce a USERS.md — multiple distinct user types with their goals, context, proficiency, motivations, frustrations, and the cues that make each feel "this is for me".
metadata:
  category: Frontend, UI & Design
---

# Grill Users

Interview the user to pin down who actually uses the app, then author a `USERS.md` that defines each distinct user type.

The job is not to record a demographic. It is to push past "everyone" to a small set of behaviorally distinct user types, each with a goal that must succeed, and to name who the app is explicitly *not* for.

## Non-Negotiables

1. **Grill, don't transcribe.** One question at a time with a recommended answer. Reject vague answers.
2. **Refuse "everyone."** "All users" / broad demographics describe no one and give the critiques nothing to test. Force a primary user who must succeed, and behavioral (not demographic) distinctions between types.
3. **Differentiate or collapse.** If two proposed user types would use the app the same way with the same goals, merge them. Distinct types must differ in goals, context, proficiency, or what they care about.
4. **Name the anti-users.** Who the app is *not* for is as clarifying as who it is for.
5. **Author a usable artifact.** USERS.md must be specific enough that someone could pick a scenario and weight priorities from it without asking follow-ups.

## Grilling Method

Apply the `grilling` skill's method:

- Ask ONE consequential question at a time, then wait.
- Attach a **recommended answer** with a brief reason to every question.
- Treat the user's correction as truth immediately; never drift back.
- **Explore before asking.** Inspect the repo/product first — analytics, existing copy, landing-page audience language, auth roles, pricing tiers, README, support docs. Do not ask what the material already answers.
- Stop when the primary user and any genuinely distinct secondary types are specific enough to author.

## Process

### 1. Recon

Scan for existing signals of who uses this: marketing copy and its "you", role/permission tiers in code, pricing tiers, onboarding, testimonials, support tickets, an existing USERS.md. Summarize what you found so questions build on it.

### 2. Grill through the persona tree

Work the question tree in `references/grill-questions.md`, one question per turn. It covers: what the app does, the primary user, distinct secondary types, each type's context/proficiency/goal/motivations/frustrations/success, the "this is for me" cue, anti-users, and relative priority.

### 3. Reflect the set back

Before authoring, restate the user types in a few lines each (label, the goal that must succeed, what differentiates them, priority). Confirm or correct. Merge any types that aren't genuinely distinct.

### 4. Author USERS.md

Write a valid `USERS.md` following `references/users-md-format.md`: structured YAML front matter (one entry per user type + anti-users) plus a prose section per type for nuance. Default location: repo root `USERS.md` — confirm the path.

### 5. Hand off

Report the file path, the primary user, and the distinct types with their priority.

## What a Good USERS.md Requires

Refuse to finish until it clears this bar:

- **A primary user** — the one whose failure means the product failed. Exactly one is primary.
- **Behaviorally distinct types** — each differs in goal, context, proficiency, or motivation, not just job title or age.
- **A concrete goal per type** — a job-to-be-done with an observable success state, usable directly as a test scenario.
- **A priority lens per type** — what they care about most, which says what to weight heavily.
- **A resonance cue per type** — what makes them feel the product is for them.
- **Named anti-users** — who it is deliberately not for.

If the product genuinely has one user type, that is fine — say so and make that one type sharp. Do not invent personas to fill a quota.

## Reference Files

| File | Load when |
|------|-----------|
| `references/grill-questions.md` | Always — the persona question tree + how to reject generic answers |
| `references/users-md-format.md` | Authoring the file (step 4) |
