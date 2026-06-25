### Agents Rules

- NEVER RUN GIT REVERT OR GIT RESTORE WITHOUT ASKING. NO DESTRUCTIVE GIT COMMANDS EVER WITHOUT EXPLICIT PERMISSION

### UI & Frontend Code & HTML

- Use the simplest, flattest HTML structure that communicates the content — don't add wrapper elements or semantic tags unless they provide real value.
- keep html semantic, minimal and modern
- less ui is always better than too much
- no unessary labels or boxes within boxes
- don't add gradients to design unless it's specified as part of a project's design spec
- Don't give the user "refresh" buttons, data should always be fresh
- Don't give the user "save" buttons, forms should auto save on data change (never via a setInterval)
- Don't add text to explain the ui, ie don't put text explaining to the user what to do, the correct path should be obvious by the ui itself.

### Codebase Rules

- Never leave TODOs, placeholder implementations, or no-op stubs. If something needs to work, wire it now. If you can't wire it yet, the task isn't done — don't close it.

## Code style guidelines

### Formatting

- Follow the existing style in each touched file (some files differ).
- Avoid reformatting unrelated lines.

### Imports

- Group imports as:
  1. Node built-ins (`node:*`),
  2. external packages,
  3. internal relative modules.
- Keep type imports explicit with `import type`.
- Avoid wildcard imports.
- Never re-export code or types; always import directly from the source module.

### TypeScript and typing

- Preserve `strict` typing (`tsconfig.json` has `strict: true`).
- Forbid `any`; prefer `unknown` at trust boundaries.
- Narrow `unknown` before property access.
- Use literal unions for finite statuses/events.
- Use `type` for unions/aliases; use `interface` for object contracts when useful.
- Add explicit return types for exported functions/APIs.

### Naming

- `PascalCase`: classes, interfaces, Svelte components.
- `camelCase`: variables, functions, methods.
- `SCREAMING_SNAKE_CASE`: top-level constants.
- Boolean names should be intentful (`is*`, `has*`, `should*`).
- Prefer descriptive cross-process naming over abbreviations.

### Abstractions and utilities

- Do not add pass-through wrapper helpers that only call another function with the same arguments.
- If logic is generic/pure and not component-state specific, place it in the correct utility layer instead of defining it inside a component.
- Utility location is strict and must not drift:
  - All shared/pure utilities go in `src/lib/utils/*`
  - Do not create new utility folders. One pool: `src/lib/utils/`.
  - Before adding a new helper, check `src/lib/utils/` first and extend existing helpers when appropriate.
  - Never write a utility function in a context of where it's used. Always put it in the appropriate utils folder and import it.
- Prefer inline expressions for trivial one-off formatting; only extract helpers when there is meaningful reuse or domain logic.
- When extracting a helper, use a name that communicates domain meaning. Avoid aliases that add indirection without behavior.

### Feature colocation

- Keep feature-specific stores, types, components, and tests together in the same feature folder under `src/lib/components/<feature>/`.
- Do not place feature-specific files in global buckets like `src/lib/stores/*` or `src/lib/types/*` when they only serve one feature.
- Reserve shared/global locations for truly cross-feature modules only (for example app-wide stores like status/projects/tasks).

### Error handling

- Fail fast for bad input (`TypeError`/`Error` with clear messages).
- For best-effort persistence, catch and continue intentionally.
- Log with context (`console.error("message", error)`).
- Do not silently swallow errors unless clearly intentional.

### Async and state

- Prefer `async/await` over long promise chains.
- If a promise is intentionally not awaited, handle rejection with `.catch`.

## Svelte

- Do not use $effect unless you absolutely have to, most things don't need side effects.
- Prefer nested SvelteKit layouts for shared route-level UI composition; do not introduce wrapper components when a layout hierarchy solves it.
- When using Svelte don't fallback to react practices. If a global manager/store is the source of truth, read it directly where it is used.
- Components own their own data. If data is in a global store or available from the route (`page.params`, `page.route`), read it directly where it's used. Do not compute it in a parent and pass it down as props unless the parent truly owns that data or behavior.
- Use props at real generic boundaries. Use manager imports at feature/state boundaries.
- Do not create thin wrapper components or prop-forwarding layers that only relay manager state, manager methods, or route-derived values.
- In Svelte runes/state logic, keep derived values deterministic and side-effect free.
- Do not create rename-only `$derived` or `$derived.by` aliases for existing manager/store values. Only use local `$derived` when it computes something new.
- Bad:
  ```ts
  const state = $derived(epicRunManager.current);
  const currentTask = $derived(epicRunManager.currentTask);
  ```
  when those values are only being forwarded or lightly renamed.
- Good:
  ```svelte
  {#if epicRunManager.currentTask}
    {epicRunManager.currentTask.taskName}
  {/if}
  ```
- Good:
  ```ts
  const completedCount = $derived.by(() => {
    const current = epicRunManager.current;
    return current ? current.queue.filter((t) => t.status === "completed").length : 0;
  });
  ```
- NEVER use $derived or $derived.by for running side effects. $derived and $derived.by are strictly for deriving state.
- Always program to avoid side effects or mysterious behavior.
- Always prefer small components and refactor when they are getting too large.
- Do not write `onclick={() => { void handleSavePreWork(); }}`. Prefer `onclick={handleSavePreWork}` or, if the handler name matches, shorthand like `{onclick}`.

## Testing conventions

- Unit tests live near code (`*.test.ts` / `*.spec.ts`).
- Svelte component tests should use `@testing-library/svelte`.
- Prefer role/text-based assertions over brittle selectors.
- Keep tests deterministic and independent.
- During iteration, run one file or one test title first.

## Scope control

- Do not introduce new frameworks/tooling unless requested.
- Avoid incidental refactors.
- Do not change build/packaging config without a clear task need.
- Keep preload and renderer typing changes in sync.

## CSS

- CSS should be systemic avoiding one off styles in routes unless absolutely needed.
- Always prefer existing CSS vars, tokens and systems before creating new ones.
- CSS customization should be done at a global level via an imported css file OR scoped to a component.
- Never add a "button" class to a `<button>` element.
