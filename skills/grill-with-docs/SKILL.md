---
name: grill-with-docs
description: Grilling session that challenges a plan against the existing domain model, sharpens terminology, and updates documentation (CONTEXT.md, ADRs) inline as decisions crystallize. Use when the user wants to stress-test a plan against project language and documented decisions.
---

# Grill With Docs

Run a `grilling` session using the `domain-modeling` discipline.

Use `grilling` for the interview shape: one consequential question at a time, a recommended answer attached to each question, and immediate correction when the user rejects the recommendation.

Use `domain-modeling` for the documentation side effects: challenge fuzzy language, cross-check claims against existing code/docs, update `CONTEXT.md` as terms crystallize, and offer ADRs only when a durable decision is worth recording.

## Process

1. Read existing domain docs before asking questions: `CONTEXT.md`, `CONTEXT-MAP.md`, `docs/adr/`, and nearby context-specific docs if present.
2. Ask one question at a time, with a recommended answer.
3. If a question can be answered by exploring files, docs, or code, explore instead of asking.
4. Update `CONTEXT.md` immediately when domain language is resolved.
5. Offer an ADR sparingly, only when the decision is hard to reverse, surprising without context, and the result of a real trade-off.
6. If the session is explicitly creating a Dex backlog, update Dex tasks as planning decisions crystallize.

## Dex Backlog Output

For Dex-backed planning sessions, see [Dex planning session pattern](./references/dex-planning-session-pattern.md). Only use Dex when the user asks for a backlog, task list, execution plan, or explicitly mentions Dex.

When the grilling session is meant to produce a Dex epic/task list:

1. Load the `dex` skill and verify where Dex will store tasks before creating anything.
2. If `dex dir` points at global storage for a project session, make the project a git repo or otherwise configure repo-local storage before creating the backlog.
3. Create the top-level epic once the scope boundary is stable.
4. Add or patch child tasks immediately after decisions become stable.
5. Keep domain language in `CONTEXT.md`; keep execution details, acceptance criteria, dependencies, and implementation sequencing in Dex.
6. Do not wait until the end to batch the backlog. The backlog is part of the conversation artifact.
7. Keep asking one question at a time after each write.

## Domain Awareness

Most repos have a single context:

```
/
|-- CONTEXT.md
|-- docs/
|   `-- adr/
|       |-- 0001-event-sourced-orders.md
|       `-- 0002-postgres-for-write-model.md
`-- src/
```

If a `CONTEXT-MAP.md` exists at the root, the repo has multiple contexts. The map points to where each one lives:

```
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

## Documentation Rules

Use the format in [CONTEXT-FORMAT.md](./CONTEXT-FORMAT.md) when updating domain language.

`CONTEXT.md` should be totally devoid of implementation details. Do not treat `CONTEXT.md` as a spec, a scratch pad, or a repository for implementation decisions. It is a glossary and nothing else.

When the user corrects the framing of a term, patch the glossary immediately and propagate the correction into any planning artifact already created. Example: if you called placeholder replacement a "migration" and the user says the old content is just placeholder scaffold, replace the domain term with something like **Initial Content Build** and update the Dex/task descriptions that used the old term. The live decision beats stale recalled context or earlier task wording.

## Treat Placeholder Code As Placeholder

Do not over-dignify scaffold or seeded demo data as a "migration" if the user says it is placeholder. Update the domain language immediately: prefer terms like **Initial Content Build**, **First Real Authoring Pass**, or **Replace Placeholder Content** over migration/compatibility language. Ask whether existing code is canonical before creating migration ceremony, adapters, or parity gates around it.
