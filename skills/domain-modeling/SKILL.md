---
name: domain-modeling
description: Build and sharpen a project's domain model. Use when the user wants to pin down domain terminology, record an architectural decision, or when another skill needs to maintain the domain model.
---

# Domain Modeling

Actively build and sharpen the project's domain model as you design. This is the active discipline: challenge terms, invent edge-case scenarios, and write the glossary and decisions down the moment they crystallize.

Merely reading `CONTEXT.md` for vocabulary is not this skill. This skill is for when you are changing the model, not just consuming it.

## File Structure

Most repos have a single context:

```text
/
|-- CONTEXT.md
|-- docs/
|   `-- adr/
|       |-- 0001-event-sourced-orders.md
|       `-- 0002-postgres-for-write-model.md
`-- src/
```

If a `CONTEXT-MAP.md` exists at the root, the repo has multiple contexts. The map points to where each one lives:

```text
/
|-- CONTEXT-MAP.md
|-- docs/
|   `-- adr/                          # system-wide decisions
`-- src/
    |-- ordering/
    |   |-- CONTEXT.md
    |   `-- docs/adr/                 # context-specific decisions
    `-- billing/
        |-- CONTEXT.md
        `-- docs/adr/
```

Create files lazily, only when you have something to write. If no `CONTEXT.md` exists, create one when the first term is resolved. If no `docs/adr/` exists, create it when the first ADR is needed.

## During The Session

### Challenge Against The Glossary

When the user uses a term that conflicts with the existing language in `CONTEXT.md`, call it out immediately.

Example: "Your glossary defines 'cancellation' as X, but you seem to mean Y. Which is it?"

### Sharpen Fuzzy Language

When the user uses vague or overloaded terms, propose a precise canonical term.

Example: "You're saying 'account'. Do you mean the Customer or the User? Those are different things."

### Discuss Concrete Scenarios

When domain relationships are being discussed, stress-test them with specific scenarios. Invent scenarios that probe edge cases and force the user to be precise about the boundaries between concepts.

### Cross-Reference With Code

When the user states how something works, check whether the code agrees. If you find a contradiction, surface it.

Example: "Your code cancels entire Orders, but you just said partial cancellation is possible. Which is right?"

### Update CONTEXT.md Inline

When a term is resolved, update `CONTEXT.md` immediately. Do not batch these up. Use the format in [CONTEXT-FORMAT.md](./CONTEXT-FORMAT.md).

`CONTEXT.md` should be totally devoid of implementation details. Do not treat `CONTEXT.md` as a spec, scratch pad, or repository for implementation decisions. It is a glossary and nothing else.

### Offer ADRs Sparingly

Only offer to create an ADR when all three are true:

1. Hard to reverse - the cost of changing your mind later is meaningful.
2. Surprising without context - a future reader will wonder why the project works this way.
3. The result of a real trade-off - there were genuine alternatives and one was picked for specific reasons.

If any of the three is missing, skip the ADR. Use the format in [ADR-FORMAT.md](./ADR-FORMAT.md).
