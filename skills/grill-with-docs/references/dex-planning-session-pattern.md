# Dex planning session pattern

Use this when a grill-with-docs session is also meant to produce a Dex epic/task tree.

## Sequence

1. Inspect project docs first: `CONTEXT-MAP.md`, `CONTEXT.md`, `docs/adr/`, and obvious source files.
2. Verify Dex storage before creating tasks. In project repos, `dex dir` should resolve under the repo, usually `.dex`; if it points at global storage, fix project-local setup first.
3. Resolve one design branch at a time. For each answer:
   - patch `CONTEXT.md` only for domain language/glossary relationships;
   - create or update the relevant Dex task for execution detail;
   - avoid ADRs unless the decision is hard to reverse, surprising, and trade-off based.
4. Keep Dex task detail at the coordination level. Include expected steps/moves, acceptance, dependencies, verification, and implementation context. Do not turn Dex into the canonical content/spec source.
5. When the user corrects framing, rename the concept everywhere immediately. Example: placeholder replacement is an **Initial Content Build**, not a migration, when the old content was scaffold rather than canonical.

## Good Dex detail level

For a feature/content task, include:
- goal;
- techniques/areas covered;
- expected implementation moves;
- likely files/modules;
- verification gates;
- acceptance criteria.

Avoid exact final prose, exact canonical copy, or large data blobs in Dex. Those belong in project files.
