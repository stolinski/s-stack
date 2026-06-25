---
name: grilling
description: Interview the user relentlessly about a plan or design. Use when the user wants to stress-test a plan before building, or uses any grill trigger phrases.
---

# Grilling

Interview the user relentlessly about every aspect of a plan until you reach shared understanding. Walk down each branch of the design tree, resolving dependencies between decisions one by one.

## Core Rules

- Ask one question at a time, then wait for feedback before continuing.
- Attach a recommended answer to each question so the user can accept, reject, or correct it quickly.
- Treat the user's correction as the new truth immediately. Do not defend the earlier recommendation or drift back to it later.
- If a question can be answered by exploring files, docs, or code, explore instead of asking.
- Stop when the decision tree has enough load-bearing structure to produce the next artifact.

## Question Shape

Use this shape for each turn:

```markdown
Question: [one consequential decision]

Recommended answer: [your best answer, with a brief reason]
```

## Avoid

- Asking multiple questions at once.
- Turning incidental detail into ceremony.
- Continuing after the remaining questions no longer change the artifact.
- Treating your recommendation as authoritative after the user corrects it.
