# Precision Audit (Measured — Do Not Eyeball)

The execution failures that make a UI look broken — inconsistent padding, off-scale spacing, misaligned edges, mismatched control heights, overflow, ragged wrapping — are invisible to judgment-based review. They are caught by **measuring actual pixel values and comparing them**. Never report "spacing feels off"; report "card padding is 14 / 16 / 13px — should be one value." Numbers or it didn't happen.

## Contents
- Find the real selectors first
- How to measure
- Check 1: Spacing scale & consistency
- Check 2: Alignment
- Check 3: Sizing consistency
- Check 4: Rhythm & density
- Check 5: Containment (overflow / clipping / wrapping)
- Check 6: Optical alignment
- Priority & reporting
- Screenshots-only fallback

## Find the Real Selectors First

Do not guess selectors like `.card` — most apps won't match them, and **an empty selector result means the selector is wrong, not that the page is clean**. Never move on from a query that returned `[]`; fix the selector.

Enumerate the page's actual repeated structures first, then measure those:

```js
// Discover repeated structures: group visible elements by tag + class signature
() => {
  const groups = {};
  for (const el of document.querySelectorAll('body *')) {
    const r = el.getBoundingClientRect();
    if (r.width < 40 || r.height < 16) continue;
    const key = el.tagName.toLowerCase() + '.' + [...el.classList].sort().slice(0, 3).join('.');
    (groups[key] ??= []).push(el);
  }
  return Object.entries(groups)
    .filter(([, els]) => els.length >= 2)
    .sort((a, b) => b[1].length - a[1].length)
    .slice(0, 25)
    .map(([key, els]) => ({ selector: key, count: els.length }));
}
```

Take the top groups (they are the page's cards, rows, buttons, inputs — whatever they're actually named), then run the measurement snippets below against those real selectors. Cross-check counts against what the screenshot shows: if you can see 6 cards but your selector matched 2, the selector is wrong.

## How to Measure

On a live page, pull real values with `chrome-devtools` MCP `evaluate_script` (or the agent-browser equivalent). Two primitives do most of the work: `getBoundingClientRect()` for position/size and `getComputedStyle()` for box values.

Sample **sets of like elements** (all cards, all list rows, all primary buttons, all inputs) — inconsistency is only visible across a set.

```js
// Box + spacing for a set of like elements — replace the selector with one
// discovered above; an empty result = wrong selector, retry
() => [...document.querySelectorAll('<discovered-selector>')].map(el => {
  const r = el.getBoundingClientRect();
  const s = getComputedStyle(el);
  return {
    x: Math.round(r.left), y: Math.round(r.top),
    w: Math.round(r.width), h: Math.round(r.height),
    pad: s.padding, margin: s.margin, gap: s.gap,
    radius: s.borderRadius,
  };
})
```

```js
// Control heights + hit areas across a role
() => [...document.querySelectorAll('button, .btn, input, select')].map(el => {
  const r = el.getBoundingClientRect();
  return { tag: el.tagName, label: el.textContent.trim().slice(0,20),
           h: Math.round(r.height), w: Math.round(r.width) };
})
```

## Check 1: Spacing Scale & Consistency

- Collect every padding, margin, and gap in the sampled region.
- Do they come from a small, consistent scale? If a `DESIGN.md` `spacing` scale exists, values must be on it. Otherwise expect a base unit (commonly 4 or 8px).
- **Flag:** off-scale one-offs (13px, 17px, 22px), and **inconsistency across like elements** (cards padded 14/16/13px; sections gapped 24px then 30px).
- Report the actual set of values and the single value they should collapse to.

## Check 2: Alignment

- For stacked/grouped elements, compare `left` (and `right`) edges — they should match within ~1px. For elements in a row, compare `top` / baseline.
- Check content aligns to consistent gutters/grid; labels align to their inputs; icons align to text.
- **Flag** each misalignment with the pixel delta: "form label left=24px, input left=26px — 2px misalignment." A 2px drift reads as "broken" even when nothing else is wrong.

## Check 3: Sizing Consistency

- Same-role controls should share dimensions: all primary buttons the same height; buttons and adjacent inputs the same height; icons a consistent size.
- **Flag:** "primary button 38px tall, inputs 40px — mismatched"; inconsistent icon sizes; touch targets under the platform minimum (~44px) on the core path.

## Check 4: Rhythm & Density

- Gaps between repeated items (list rows, grid cells, nav items) should be equal — measure them.
- Space around a group should relate to space within it (outer ≥ inner) so grouping reads correctly.
- **Flag:** uneven item gaps; inner padding larger than the gap separating groups (breaks grouping); cramped elements with sub-scale spacing.

## Check 5: Containment (Overflow / Clipping / Wrapping)

The page-wide versions of these are computed by the smoke test (`references/broken-ui-detectors.md`) — this check re-verifies containment within the specific region being audited.

- Look for content overflowing its container, text clipped or truncated unintentionally, elements touching a container edge with no padding, and ragged/awkward wrapping at common widths (test the captured viewports).
- **Flag** these as high priority — they are literal visible breakage.

## Check 6: Optical Alignment

- Metric-centered is not always optically centered: icons with visual weight to one side, triangular/play glyphs, single characters in a circle, text with differing ascenders.
- **Flag** where optical adjustment is needed (usually P2–P3), but only after the measured checks above.

## Priority & Reporting

- **P0/P1:** visible breakage — overlap, overflow, clipping, obvious misalignment (>3px on aligned elements), cramped/touching elements, mismatched control heights on the primary action.
- **P2:** off-scale spacing, inconsistent padding across like elements, uneven rhythm.
- **P3:** subtle optical adjustments, 1–2px refinements.

Every precision finding's **Recommendation must carry the number and the target**: "Set all three card paddings to 16px (`spacing.md`); currently 14/16/13px." Vague recommendations are why fixes break spacing — give the exact value so the fix is mechanical, not interpretive.

## Screenshots-Only Fallback

Without a live page you cannot get exact values. Measure proportionally from the image, flag clearly visible issues (obvious misalignment, overflow, uneven gaps), and label every value as approximate. Note that precise spacing/sizing consistency could not be verified.
