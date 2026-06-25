# SvelteKit workspace route UX

Session lesson from the OP-XY practice app.

## Problem
After making lesson pages bookmarkable (`/op-xy/{missionSlug}`), clicking between missions caused the page to jump back to the top and repeatedly exposed a large marketing/header area. The route shape was technically correct but hostile for practice use.

## Pattern
- Keep marketing/landing content on `/` or an entry route.
- Make app/device routes workspace-first.
- Use real anchors for route semantics, but add `data-sveltekit-noscroll` when adjacent route navigation should preserve scroll position.
- If a route param owns the selected primary item, use a derived value from the param rather than initialising `$state(param ?? fallback)`, which captures only the first value and triggers Svelte's `state_referenced_locally` warning.
- Let localStorage restore secondary state only when it does not conflict with route ownership.

## Verification probe
Use Chrome DevTools/MCP or browser automation:
1. Navigate to an app route.
2. Force `scrollY` to a meaningful value.
3. Click a real route link to an adjacent item.
4. Assert URL and content changed while `scrollY` stayed stable.
5. Confirm app routes do not render landing/hero content.

Example assertion shape:
```js
window.scrollTo(0, 420);
const before = scrollY;
document.querySelector('a[href="/device/next-item"]')?.click();
await new Promise((resolve) => setTimeout(resolve, 250));
return { before, after: scrollY, path: location.pathname };
```
