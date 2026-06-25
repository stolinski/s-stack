# Event handler browser verification pitfall

Session lesson from a SvelteKit 5 content-model refactor:

- `svelte-check` and the Svelte autofixer can pass while an interaction is still stale or effectively unverified in the browser.
- Capture-phase event names like `onclickcapture` are valid-looking enough to survive review, but ordinary app UI should usually use bubble-phase `onclick` unless capture behavior is explicitly required.
- After changing event handlers, verify the actual browser interaction, not just DOM shape:
  1. Click the real control with the browser tool or evaluate `element.click()`.
  2. Wait a short tick/delay before reading derived UI state.
  3. Assert the visible result changed: selected nav item, heading text, progress value, Move Map contents, etc.
  4. Check browser console for JS errors.

Good probe pattern:

```js
document.querySelectorAll('nav button')[7].click();
new Promise((resolve) =>
	setTimeout(
		() =>
			resolve({
				title: document.querySelector('h2')?.textContent,
				active: Array.from(document.querySelectorAll('nav button')).findIndex((button) =>
					button.classList.contains('active')
				),
				moves: Array.from(document.querySelectorAll('.song-map button')).map((button) =>
					button.textContent?.trim()
				)
			}),
		150
	)
);
```

This catches the thing static checks cannot: whether the user-facing interaction actually moved state through the component.