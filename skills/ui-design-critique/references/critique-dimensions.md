# Critique Dimensions

Evaluate every dimension. For each issue: name the element/location, state what is wrong and why it matters, then classify (Gap / Inconsistency / Suggestion / Strength) and assign a priority (P0–P3). AI Slop is dimension 1 and lives in `ai-slop-detection.md`; the nine below complete the pass.

## Contents
1. AI Slop Detection → see `ai-slop-detection.md`
2. Visual Hierarchy
3. Information Architecture
4. Emotional Resonance
5. Discoverability & Affordance
6. Composition & Balance
7. Typography as Communication
8. Color with Purpose
9. States & Edge Cases
10. Microcopy & Voice

Use these probing questions to find issues. A "no" is usually a finding.

## 2. Visual Hierarchy

- Does the eye land on the most important element first?
- Is there a single clear primary action, spotted in under 2 seconds?
- Do size, color, weight, and position communicate importance correctly?
- Is anything competing for attention that should not?

Common findings: no discernible primary action (Gap, P0); two elements fighting for first read (Inconsistency/P1); everything the same weight (Gap/P1).

## 3. Information Architecture

- Is the structure intuitive to a first-time user?
- Is related content grouped logically?
- Are there too many choices presented at once (cognitive overload)?
- Is navigation clear and predictable?

Common findings: flat lists that should be grouped (P1); overloaded screens (P1); ambiguous nav labels (P2).

## 4. Emotional Resonance

- What emotion does the interface evoke — and is that intentional?
- Does it match the brand personality (from the spec, if present)?
- Does it feel trustworthy / approachable / premium / playful — whatever it should?
- Would the target user think "this is for me"?

Common findings: tone mismatch vs. brand intent (Inconsistency/P1); no discernible personality (Suggestion→Gap depending on product).

## 5. Discoverability & Affordance

- Are interactive elements obviously interactive?
- Would a user know what to do without instructions?
- Do hover/focus/active states give useful feedback?
- Are important features hidden that should be visible?

Common findings: buttons that look like text or vice versa (P1); missing hover/focus feedback (Gap/P2); actions buried behind hover on touch (P1).

## 6. Composition & Balance

- Does the layout feel balanced, or uncomfortably weighted?
- Is whitespace used intentionally, or is it just leftover?
- Is there visual rhythm in spacing and repetition?
- Does asymmetry feel designed or accidental?

Common findings: even spacing with no grouping rhythm (P2); accidental imbalance (P2); cramped or floating elements (P2).

## 7. Typography as Communication

- Does the type hierarchy clearly signal read order (first, second, third)?
- Is body text comfortable — line length (~45–75ch), line-height, size?
- Do font choices reinforce brand and tone?
- Is there enough contrast between heading levels?

Common findings: heading levels too close in size (Inconsistency/P1); body text too small or lines too long (Gap/P1); no type personality (see AI slop).

## 8. Color with Purpose

- Is color used to communicate, not just decorate?
- Does the palette feel cohesive?
- Do accent colors draw attention to the right things?
- Does meaning survive for colorblind users (not just technical contrast)?

Check WCAG contrast (AA: 4.5:1 body, 3:1 large text/UI). If a DESIGN.md exists, `npx @google/design.md lint` reports component contrast. Common findings: accent used decoratively so it no longer signals action (P1); low-contrast text (Gap/P0–P1); meaning carried by color alone (Gap/P1).

## 9. States & Edge Cases

Missing states are Gaps and often high priority — they are where real products break.

- **Empty:** does it guide toward action, or just say "nothing here"?
- **Loading:** does it reduce perceived wait (skeletons/optimistic UI) vs. a spinner on blank?
- **Error:** helpful and non-blaming, with a path to recovery?
- **Success:** confirms and guides the next step?
- Also: disabled, stale-while-refetching, no-permission, long content, zero/overflow data.

Common findings: unhandled empty/loading/error (Gap/P0–P1); dead-end error copy (P1); flat "No data" empty states (P2).

## 10. Microcopy & Voice

- Is the writing clear and concise?
- Does it sound like a human — the right human for this brand?
- Are labels and buttons unambiguous about what they do?
- Does error copy help the user fix the problem?

Common findings: vague button labels ("Submit", "OK") (P2); jargon or robotic tone (P2); blaming error copy (P1); generic boilerplate (see AI slop).

## Distinguishing Findings from Taste

- A violation of the DESIGN.md spec or a broken usability/accessibility principle is a real finding (Gap or Inconsistency).
- A defensible alternative that you happen to prefer is a **Suggestion**, and must be labeled as taste-based.
- Do not flag a deliberate, well-executed choice just because it is unconventional. Intent + execution beats convention.
