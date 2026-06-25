---
name: svelte-core-bestpractices
description: Guidance on writing fast, robust, modern Svelte code. Load this skill whenever in a Svelte project and asked to write/edit or analyze a Svelte component or module. Covers reactivity, event handling, styling, integration with libraries and more.
---

## `$state`

Only use the `$state` rune for variables that should be _reactive_ — in other words, variables that cause an `$effect`, `$derived` or template expression to update. Everything else can be a normal variable.

Objects and arrays (`$state({...})` or `$state([...])`) are made deeply reactive, meaning mutation will trigger updates. This has a trade-off: in exchange for fine-grained reactivity, the objects must be proxied, which has performance overhead. In cases where you're dealing with large objects that are only ever reassigned (rather than mutated), use `$state.raw` instead. This is often the case with API responses, for example.

## `$derived`

To compute something from state, use `$derived` rather than `$effect`:

```js
// do this
let square = $derived(num * num);

// don't do this
let square;

$effect(() => {
	square = num * num;
});
```

> [!NOTE] `$derived` is given an expression, _not_ a function. If you need to use a function (because the expression is complex, for example) use `$derived.by`.

Deriveds are writable — you can assign to them, just like `$state`, except that they will re-evaluate when their expression changes.

If the derived expression is an object or array, it will be returned as-is — it is _not_ made deeply reactive. You can, however, use `$state` inside `$derived.by` in the rare cases that you need this.

## `$effect`

Effects are an escape hatch and should mostly be avoided. In particular, avoid updating state inside effects.

- If you need to sync state to an external library such as D3, it is often neater to use [`{@attach ...}`](references/@attach.md)
- If you need to run some code in response to user interaction, put the code directly in an event handler or use a [function binding](references/bind.md) as appropriate
- If you need to log values for debugging purposes, use [`$inspect`](references/$inspect.md)
- If you need to observe something external to Svelte, use [`createSubscriber`](references/svelte-reactivity.md)

Never wrap the contents of an effect in `if (browser) {...}` or similar — effects do not run on the server.

## `$props`

Treat props as though they will change. For example, values that depend on props should usually use `$derived`:

```js
// @errors: 2451
let { type } = $props();

// do this
let color = $derived(type === 'danger' ? 'red' : 'green');

// don't do this — `color` will not update if `type` changes
let color = type === 'danger' ? 'red' : 'green';
```

## `$inspect.trace`

`$inspect.trace` is a debugging tool for reactivity. If something is not updating properly or running more than it should you can add `$inspect.trace(label)` as the first line of an `$effect` or `$derived.by` (or any function they call) to trace their dependencies and discover which one triggered an update.

## SvelteKit route UX

When adding bookmarkable SvelteKit routes to an app-like surface, preserve the distinction between landing/marketing pages and workspace/application routes. A hero that is appropriate on `/` can become a usability bug on `/app/[item]` if every in-app navigation scrolls the user back through the marketing header.

Before browser QA in an existing local project, use the already-running dev server when the project convention or prior context says one exists. Do not reflexively start a second `npm run dev`; first navigate/curl the known local URL. If it is stale or down, restart through the project’s managed dev command/service rather than raw Vite unless the repo has no manager. This avoids noisy port-in-use failures and respects Scott’s live dev setup.

For workspace-style route links:

- Use real `<a href="...">` links for bookmarkability and accessibility, not button-only local state.
- Add `data-sveltekit-noscroll` when route changes should preserve the user's working position instead of jumping to the top.
- Let explicit route params own selected entity state. Local persistence can restore sub-state (for example selected move/step), but the route slug should own the selected mission/item when present.
- Verify in a real browser by setting `document.documentElement.scrollTop`/`window.scrollTo`, clicking the route link, waiting for navigation, and comparing `scrollY` before/after. Also verify the route does not re-render landing-only content on workspace pages.

## Events

Any element attribute starting with `on` is treated as an event listener:

```svelte
<button onclick={() => {...}}>click me</button>

<!-- attribute shorthand also works -->
<button {onclick}>...</button>

<!-- so do spread attributes -->
<button {...props}>...</button>
```

Prefer plain bubble-phase handlers like `onclick` for ordinary UI interactions. Do not reach for capture-phase names such as `onclickcapture` unless the component specifically needs capture semantics; they are easy to miss in review because `svelte-check` can pass while the app behavior is wrong or stale in the browser. After changing event handlers, verify the actual interaction in a browser and, when reading DOM state immediately after a click, allow a short tick/delay before asserting derived UI changed. See `references/event-handler-browser-verification.md` for a concrete verification probe pattern.

For SvelteKit app-workspace navigation, do not let route links destroy the user's working position unless the route is genuinely a new document/landing context. Use real anchors for bookmarkability, and add `data-sveltekit-noscroll` on in-app navigation where preserving scroll matters. Verify with a browser probe that sets `document.documentElement.style.scrollBehavior = 'auto'`, forces `scrollTop`/`scrollY` to a meaningful value, clicks the link, waits for navigation, and asserts the scroll position remained stable.

If you need to attach listeners to `window` or `document` you can use `<svelte:window>` and `<svelte:document>`:

```svelte
<svelte:window onkeydown={...} />
<svelte:document onvisibilitychange={...} />
```

Avoid using `onMount` or `$effect` for this.

## SvelteKit route UX

When converting local-state navigation to real SvelteKit routes, verify scroll behavior in the browser. App/workspace routes often should not jump back to the top when switching adjacent items. Use `data-sveltekit-noscroll` on route links when preserving the current workspace position is the intended interaction. See `references/sveltekit-workspace-route-ux.md` for the full pattern and verification probe.

Keep route-owned state explicit: if a route parameter selects an item, derive selected state from the param instead of capturing it once into `$state`. Let local persistence restore secondary state like the selected step/move, but let the route own the primary item.

## Snippets

[Snippets](references/snippet.md) are a way to define reusable chunks of markup that can be instantiated with the [`{@render ...}`](references/@render.md) tag, or passed to components as props. They must be declared within the template.

```svelte
{#snippet greeting(name)}
	<p>hello {name}!</p>
{/snippet}

{@render greeting('world')}
```

> [!NOTE] Snippets declared at the top level of a component (i.e. not inside elements or blocks) can be referenced inside `<script>`. A snippet that doesn't reference component state is also available in a `<script module>`, in which case it can be exported for use by other components.

## Each blocks

Prefer to use [keyed each blocks](references/each.md) — this improves performance by allowing Svelte to surgically insert or remove items rather than updating the DOM belonging to existing items.

> [!NOTE] The key _must_ uniquely identify the object. Do not use the index as a key.

Avoid destructuring if you need to mutate the item (with something like `bind:value={item.count}`, for example).

## Using JavaScript variables in CSS

If you have a JS variable that you want to use inside CSS you can set a CSS custom property with the `style:` directive.

```svelte
<div style:--columns={columns}>...</div>
```

You can then reference `var(--columns)` inside the component's `<style>`.

## Styling child components

The CSS in a component's `<style>` is scoped to that component. If a parent component needs to control the child's styles, the preferred way is to use CSS custom properties:

```svelte
<!-- Parent.svelte -->
<Child --color="red" />

<!-- Child.svelte -->
<h1>Hello</h1>

<style>
	h1 {
		color: var(--color);
	}
</style>
```

If this is impossible (for example, the child component comes from a library) you can use `:global` to override styles:

```svelte
<div>
	<Child />
</div>

<style>
	div :global {
		h1 {
			color: red;
		}
	}
</style>
```

## Context

Consider using context instead of declaring state in a shared module. This will scope the state to the part of the app that needs it, and eliminate the possibility of it leaking between users when server-side rendering.

Use `createContext` rather than `setContext` and `getContext`, as it provides type safety.

## Async Svelte

If using version 5.36 or higher, you can use [await expressions](references/await-expressions.md) and [hydratable](references/hydratable.md) to use promises directly inside components. Note that these require the `experimental.async` option to be enabled in `svelte.config.js` as they are not yet considered fully stable.

## Avoid legacy features

Always use runes mode for new code, and avoid features that have more modern replacements:

- use `$state` instead of implicit reactivity (e.g. `let count = 0; count += 1`)
- use `$derived` and `$effect` instead of `$:` assignments and statements (but only use effects when there is no better solution)
- use `$props` instead of `export let`, `$$props` and `$$restProps`
- use `onclick={...}` instead of `on:click={...}`
- use `{#snippet ...}` and `{@render ...}` instead of `<slot>` and `$$slots` and `<svelte:fragment>`
- use `<DynamicComponent>` instead of `<svelte:component this={DynamicComponent}>`
- use `import Self from './ThisComponent.svelte'` and `<Self>` instead of `<svelte:self>`
- use classes with `$state` fields to share reactivity between components, instead of using stores
- use `{@attach ...}` instead of `use:action`
- use clsx-style arrays and objects in `class` attributes, instead of the `class:` directive
