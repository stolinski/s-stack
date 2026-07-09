# Visual Capture

A design critique must be based on what the interface actually renders, not on reading code alone. Capture the real pixels across states and viewports before judging.

Capture is for **confirming and communicating** defects; **finding** structural breakage is the job of the deterministic detectors (`references/broken-ui-detectors.md`) — vision review of screenshots misses overflow, overlap, clipping, and small misalignment even when they are obvious to a human. Run the detectors; use screenshots as evidence.

## Contents
- Tool selection
- Find the right URL first
- Capture fidelity rules (non-negotiable)
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

## Capture Fidelity Rules (Non-Negotiable)

A defect that isn't in the pixels you look at cannot be found. Low-fidelity capture is the single biggest cause of missed "obviously broken" UI:

1. **Full resolution, always.** Never downscale review captures. On `argent`, pass `scale: 1.0` explicitly — the default is 0.3, which erases 2–8px defects entirely.
2. **Tile long pages; never rely on one full-page shot.** A full-page screenshot of a tall page compresses every component into a thumbnail. Capture viewport-height segments: screenshot, scroll one viewport (`window.scrollBy(0, innerHeight)`), repeat until the bottom. Judge each tile at full size.
3. **Resize the window to the target viewport before capturing** (`resize_page` / device emulation) — do not simulate mobile by squinting at a desktop shot.
4. **Crop-zoom anything suspicious.** If a region might be off, capture that element/region alone (element screenshot by `uid`, or scroll it to center and shoot the viewport) and inspect it at full size before deciding.
5. **Evidence screenshots per finding.** Every visual finding carries a capture of the specific element/region, not just the page it sits on.

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
- For the measured precision audit (dimension 11), sample **sets of like elements** (all cards, all rows, all primary buttons, all inputs) with `getBoundingClientRect` + `getComputedStyle` — see `references/precision-audit.md` for the exact snippets. Inconsistent spacing/sizing is only visible across a set.

## Working From Screenshots Only

When only static images are available:
- Evaluate what is visible; do not assume states you cannot see.
- Explicitly list unverifiable states/viewports in the report's **Verified** line and, where a missing state matters, record it as a Gap with the caveat that it could not be confirmed.
- Estimate values (color, spacing) from the image, and label spec-conformance rows as approximate.
