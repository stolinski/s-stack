# Visual Capture

A design critique must be based on what the interface actually renders, not on reading code alone. Capture the real pixels across states and viewports before judging.

## Contents
- Tool selection
- Find the right URL first
- What to capture
- Sampling real values
- Working from screenshots only

## Tool Selection

| Situation | Tool |
|-----------|------|
| Web UI (preferred) | The `agent-browser` skill — navigation, screenshots, extraction |
| Web UI (DevTools features) | `chrome-devtools` MCP — screenshots, computed styles, contrast, snapshots, performance |
| Mobile app (iOS/Android) | `argent` — `screenshot` and `describe` on a booted simulator/emulator |
| Nothing runnable | Ask for screenshots, or render the component in a dev server first |

Prefer `agent-browser` for general capture; fall back to `chrome-devtools` MCP for computed-style/contrast/DevTools-only needs.

## Find the Right URL First

Do not guess the port. Verify before navigating:

1. Check the project's dev config for a custom port (e.g. `vite.config.ts` → `server.port`).
2. Check running dev-server output.
3. Confirm with `lsof -i :<port>` (common defaults: `5173` Vite, `3000` Next/React, `4173` Vite preview, `8080`).

## What to Capture

Capture enough to judge every dimension. At minimum:

**States** (a design lives or dies here — see dimension 9):
- Default / populated
- Hover and focus (for interactive elements)
- Empty
- Loading
- Error
- Success (if applicable)

**Viewports:**
- Mobile (~375–390px wide)
- Desktop (~1280px+)
- Optionally a cramped/narrow container to test responsive behavior

For each capture, note which state and viewport it represents so the report's **Verified** line is accurate. Record any state you could not reach.

## Sampling Real Values

To compare against a DESIGN.md spec, get actual values, not guesses:

- `chrome-devtools` MCP `take_snapshot` for the a11y/DOM tree, and `evaluate_script` to read computed styles:
  ```js
  () => {
    const el = document.querySelector('[data-cta]');
    const s = getComputedStyle(el);
    return { color: s.color, bg: s.backgroundColor, font: s.fontFamily, size: s.fontSize, radius: s.borderRadius };
  }
  ```
- Use `chrome-devtools` MCP `lighthouse_audit` (accessibility) or the snapshot for contrast signals.
- Sample the primary action, headings, body text, and a representative card/surface.

## Working From Screenshots Only

When only static images are available:
- Evaluate what is visible; do not assume states you cannot see.
- Explicitly list unverifiable states/viewports in the report's **Verified** line and, where a missing state matters, record it as a Gap with the caveat that it could not be confirmed.
- Estimate values (color, spacing) from the image, and label spec-conformance rows as approximate.
