# UX Heuristics (Evaluation Dimensions)

These are universal usability truths, not brand preferences. They hold for any product regardless of aesthetic, industry, or design intent — which is why UX critique needs no DESIGN.md or spec to judge against. Evaluate each dimension against what the walkthrough revealed. A "no" is usually a finding. Built on Nielsen's usability heuristics plus task-flow, forms, and interaction-accessibility dimensions. For each issue: step/location, what's wrong, why it harms the goal, classification, priority.

## Contents
1. Task Flow & Efficiency
2. Visibility of System Status
3. Match to the User's Mental Model
4. User Control & Freedom
5. Consistency & Standards
6. Error Prevention & Recovery
7. Recognition over Recall (Cognitive Load)
8. Flexibility & Accelerators
9. Navigation & Information Architecture
10. Forms & Input
11. Interaction Accessibility
12. Content & Comprehension

## 1. Task Flow & Efficiency

- Can the user complete the core task at all? Where does it break?
- How many steps/clicks/screens to done — and how many are truly necessary?
- Are there dead ends, forced detours, backtracks, or loops?
- Is the shortest reasonable path also the obvious one?

Findings: dead-end with no path forward (Gap/P0); hidden required step (Gap/P1); unnecessary steps (Suggestion or Inconsistency/P2).

## 2. Visibility of System Status

- After every meaningful action, does the system confirm what happened?
- Are loading, progress, and background work visible (not a frozen screen)?
- Is current location/state always clear (selected, saved, submitted)?
- Are slow operations given progress or optimistic feedback?

Findings: no feedback after a key action (Gap/P1); silent failure (Gap/P0–P1); spinner-on-blank with no context (P2).

## 3. Match to the User's Mental Model

- Does language use the user's words, not internal jargon?
- Do steps follow the order the user expects for this task?
- Do metaphors and labels mean what the user assumes?

Findings: jargon labels (Inconsistency/P2); flow ordered around the system, not the user (P1).

## 4. User Control & Freedom

- Can the user undo, cancel, go back, or exit at any point?
- Are destructive/irreversible actions confirmed or reversible (undo > confirm where possible)?
- Does the back button / escape behave as expected without losing work?
- Can the user leave and return without starting over?

Findings: destructive action with no confirm/undo (Gap/P0); back button loses entered data (Gap/P1); no exit from a step (P1).

## 5. Consistency & Standards

- Do the same actions look and behave the same across the flow?
- Does it follow platform/web conventions (link vs button, form submit, modal dismissal)?
- Are terms, icons, and patterns used consistently?

Findings: same goal reachable two ways that behave differently (Inconsistency/P2); non-standard control behavior (P1–P2).

## 6. Error Prevention & Recovery

- Are errors prevented by design (constraints, sensible defaults, format tolerance, confirmations)?
- When an error occurs, does the message say what happened, why, and how to fix it — without blame or codes?
- Is the error shown where the user can act on it, at the right time?
- Can the user recover without losing their work?

Findings: blaming/opaque error (P1); validation that clears the form (Gap/P1); no prevention of a common mistake (P2).

## 7. Recognition over Recall (Cognitive Load)

- Does the interface show options rather than make users remember them?
- Are there too many choices/fields presented at once?
- Must the user hold information in their head across steps?
- Are instructions available in context, not memorized?

Findings: recall burden across steps (P2); overloaded decision point (P1); required info not carried forward (P2).

## 8. Flexibility & Accelerators

- Are there shortcuts/accelerators for frequent or expert users?
- Are defaults smart enough that most users change little?
- Is complexity progressively disclosed rather than dumped up front?

Findings: no defaults where obvious ones exist (Suggestion/P2–P3); no keyboard shortcuts for a power flow (Suggestion/P3).

## 9. Navigation & Information Architecture

- Can the user tell where they are, where they can go, and how to get back?
- Is content grouped and labeled the way users would look for it?
- Is search/findability adequate for the content volume?
- Are entry points to the core task discoverable?

Findings: no wayfinding/orientation (Gap/P1); buried primary entry point (P1); confusing grouping (P2).

## 10. Forms & Input

- Are labels persistent (not placeholder-only) and fields clearly required/optional?
- Is validation timed well (on blur / on submit, not aggressive per-keystroke)?
- Are errors adjacent to their field, with focus moved to the first error?
- Does it support paste, autofill, password managers, and forgiving formats (spaces in card/phone)?
- Is the input type/keyboard appropriate on mobile?

Findings: placeholder-as-label (Gap/P2); error summary with no field association (P1); rejects valid formatted input (P1).

## 11. Interaction Accessibility

Interaction operability, distinct from visual contrast (that lives in `ui-design-critique`).

- Is the whole task operable by keyboard, in a logical focus order, with no traps?
- Is focus visible and moved sensibly on route/modal changes?
- Are controls labeled for assistive tech (name/role/state)?
- Are touch targets large enough and not hover-dependent?
- Do time limits / auto-dismissing messages give enough time or a way to extend?

Findings: keyboard trap or unreachable control on core path (Gap/P0); unlabeled interactive control (Gap/P1); action hidden behind hover on touch (P1).

## 12. Content & Comprehension

- Is microcopy clear, specific, and action-oriented (buttons say what they do)?
- Does empty/first-run content teach the next action?
- Is help available where confusion happens, without leaving the flow?
- Is tone appropriate and non-blaming, especially in errors and confirmations?

Findings: vague CTA ("Submit"/"OK") (P2); dead-end empty state (P2); help only in a separate manual (P3).

## Findings vs. Taste

- A broken usability principle, a blocked task, or a dark pattern is a real finding (Gap/Inconsistency, priority by impact).
- A defensible alternative you merely prefer is a **Suggestion**, labeled as taste-based.
- Do not flag a deliberate, well-executed choice just because it is unconventional — unless it measurably harms the goal.
