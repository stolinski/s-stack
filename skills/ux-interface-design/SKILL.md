---
name: ux-interface-design
description: Designs or reviews user interfaces that are self-evident, low-friction, and easy to understand with minimal explanation. Use when simplifying UI text, removing helper copy, improving affordances, tightening hierarchy, making forms more obvious, or evaluating whether an interface works without instructions.
metadata:
  category: Design & UX
  tags:
    - ux
    - ui
    - interaction-design
    - forms
    - usability
---

# UX Interface Design

Design interfaces that are self-evident, low-friction, and understandable with minimal explanatory text. Prefer structure, hierarchy, constraints, and interaction over labels and descriptions.

## Core Principle

If the UI needs instructions, fix the UI—not the copy.

## When to Use This Skill

Use this skill when:
- simplifying an interface that feels too wordy
- reducing helper text, labels, or onboarding copy
- reviewing forms, flows, or settings for clarity
- making the next action more obvious
- improving defaults, constraints, and inline feedback
- evaluating whether a UI works when users scan instead of read

Do not use this skill for:
- marketing copy or landing page messaging
- pure visual styling critiques unrelated to usability
- long-form product documentation or tutorials
- information-heavy interfaces where detailed explanation is the product itself

## Design Rules

### 1. Clarity Over Explanation
- Make layout, grouping, and affordances carry the meaning
- Use text to confirm meaning, not create it

### 2. Labels Are a Last Resort
- Avoid labels or helper text unless they are truly necessary
- Before adding text, first try:
  - improving layout
  - making controls more specific
  - choosing better defaults

### 3. Remove Redundant Text
- Do not repeat what context already makes obvious
- Avoid waste like:
  - “Submit Form” when “Submit” works
  - section headers that only restate the visible content
- Every word must justify its existence

### 4. Show, Don’t Tell
- Prefer:
  - real previews
  - inline examples
  - visible state changes
- Avoid instructional paragraphs when the interface can demonstrate the answer

### 5. Inline Feedback Only
- Put feedback at the point of interaction
- Do not rely on:
  - top-of-page error summaries
  - disconnected instructions

### 6. Default-Driven Design
- Use sensible defaults to remove decisions
- Preselect the most common option
- Reduce explanation by starting in the most useful state

### 7. Progressive Explanation
- Do not front-load instructions
- Let users act first
- Explain only when:
  - an error occurs
  - the system truly needs clarification

### 8. Constraints Over Instructions
- Prevent invalid input instead of explaining rules in advance
- Use:
  - input constraints
  - live validation
- Let the interface teach through interaction

### 9. Reduce AI Verbosity Bias
- Avoid over-labeling, over-describing, and naming every section
- Prefer implicit understanding through structure and visual priority

### 10. One-Sentence Limit
- If explanation is necessary, keep it to one short sentence
- If more is needed, redesign the interface

### 11. Trust User Intuition
- Do not narrate obvious actions
- Avoid lines like:
  - “Click below to continue”
- Assume baseline user competence

### 12. Hierarchy Over Whitespace
- Do not rely on empty space alone to create clarity
- Use:
  - strong grouping
  - clear primary actions
  - visible priority
- Dense is acceptable if it remains understandable

## Review Process

When designing or evaluating a UI:
1. Remove descriptive text and see whether the interface still works
2. Identify the single obvious next action
3. Fix layout, grouping, defaults, and constraints before adding copy
4. Keep any necessary explanation inline and brief
5. Delete anything that does not directly enable action

## Anti-Patterns

- helper text under every input
- repeated labels and descriptions
- instructions explaining obvious actions
- long onboarding tooltips for simple flows
- pages that explain before allowing interaction
- naming every section whether it needs a name or not

## Heuristics

Ask these questions:
- If all descriptive text disappeared, would the UI still work?
- Can a user complete the task without reading much?
- Is there exactly one obvious next action?
- Are errors prevented instead of explained?
- Does any text repeat what the UI already shows?

## Output Guidelines

When using this skill:
- minimize text aggressively
- prefer structure over explanation
- remove anything that does not directly enable action
- assume users scan, not read
- bias toward fewer words and stronger signals
