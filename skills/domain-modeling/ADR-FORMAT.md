# ADR Format

ADRs live in `docs/adr/` and use sequential numbering: `0001-slug.md`, `0002-slug.md`, and so on.

Create the `docs/adr/` directory lazily, only when the first ADR is needed.

## Template

```md
# {Short title of the decision}

{1-3 sentences: what is the context, what did we decide, and why.}
```

That is enough. The value is recording that a decision was made and why, not filling out sections.

## Optional Sections

Only include these when they add genuine value. Most ADRs do not need them.

- **Status** frontmatter (`proposed | accepted | deprecated | superseded by ADR-NNNN`) - useful when decisions are revisited.
- **Considered Options** - only when rejected alternatives are worth remembering.
- **Consequences** - only when non-obvious downstream effects need to be called out.

## Numbering

Scan `docs/adr/` for the highest existing number and increment by one.

## When To Offer An ADR

All three of these must be true:

1. Hard to reverse - the cost of changing your mind later is meaningful.
2. Surprising without context - a future reader will wonder why the project works this way.
3. The result of a real trade-off - there were genuine alternatives and one was picked for specific reasons.

If a decision is easy to reverse, skip it. If it is not surprising, skip it. If there was no real alternative, there is nothing to record.
