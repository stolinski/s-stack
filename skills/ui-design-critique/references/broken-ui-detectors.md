# Broken-UI Detectors (Deterministic Smoke Test)

Basic breakage — overflow, overlap, clipping, elements off the viewport, broken images — is **computed, never judged from pixels**. Vision review reliably misses these defects; DOM math never does. Run this script pack before any judgment-based critique. A page that fails the smoke test is broken regardless of how good its typography is.

Rules:

- **Run every detector, every time.** No skipping because "the page looks fine" — looking fine in a screenshot is exactly the failure mode this pack exists to catch.
- **Run at every viewport in the matrix** (below). Most breakage is width-dependent.
- **Every hit is a finding.** Detector hits are objective; they enter the report with their measured values and are never downgraded to taste.
- **Confirm each hit visually** with an element-level screenshot (see "Confirming hits") so the report carries image evidence — but a hit without a screenshot is still a finding.

## Contents
- Viewport matrix
- Detector 1: Horizontal page overflow
- Detector 2: Element overlap
- Detector 3: Text clipping & truncation
- Detector 4: Out-of-viewport / edge-touching content
- Detector 5: Broken & distorted images
- Detector 6: Zero-size and invisible-but-present content
- Detector 7: Unreadable text (size & contrast)
- Confirming hits with screenshots
- Priorities
- Native mobile fallback (argent)

## Viewport Matrix

Run the full pack at each width, resizing first (`chrome-devtools` `resize_page` or the agent-browser equivalent), letting layout settle, then executing the detectors:

| Viewport | Size |
|----------|------|
| Mobile | 375 × 812 |
| Tablet (optional but recommended) | 768 × 1024 |
| Desktop | 1280 × 800 (or the project's target) |

Record per-viewport results — "clean at 1280, 3 defects at 375" is the normal shape of real breakage.

## Shared helper

Every detector below uses this visibility test. Prepend it inside each `evaluate_script` call (or combine detectors into one script):

```js
const isVisible = (el) => {
  const s = getComputedStyle(el);
  if (s.display === 'none' || s.visibility === 'hidden' || parseFloat(s.opacity) === 0) return false;
  const r = el.getBoundingClientRect();
  return r.width > 0 && r.height > 0;
};
const describe = (el) => {
  const r = el.getBoundingClientRect();
  return {
    sel: el.tagName.toLowerCase()
      + (el.id ? '#' + el.id : '')
      + (el.classList.length ? '.' + [...el.classList].slice(0, 3).join('.') : ''),
    text: (el.textContent || '').trim().slice(0, 40),
    x: Math.round(r.left), y: Math.round(r.top),
    w: Math.round(r.width), h: Math.round(r.height),
  };
};
```

## Detector 1: Horizontal Page Overflow

The page must never scroll horizontally at any tested viewport. Finds the offenders, not just the fact.

```js
() => {
  const vw = document.documentElement.clientWidth;
  const overflowing = document.documentElement.scrollWidth > vw + 1;
  const offenders = [...document.querySelectorAll('body *')]
    .filter(isVisible)
    .filter(el => {
      const r = el.getBoundingClientRect();
      return r.right > vw + 1 || r.left < -1;
    })
    // keep outermost offenders only — drop elements whose parent already offends
    .filter((el, _, arr) => !arr.includes(el.parentElement))
    .map(describe);
  return { viewport: vw, pageOverflows: overflowing, offenders: offenders.slice(0, 20) };
}
```

**Flag:** `pageOverflows: true`, or any offender → P0. Report the offender selector, its `right` value vs the viewport width.

## Detector 2: Element Overlap

Visible content elements that intersect when they shouldn't. Compares leaf-ish content nodes (text blocks, controls, media), skips ancestor/descendant pairs, and requires a meaningful intersection so intentional stacking (badges, dropdowns) can be triaged rather than drowning the result.

```js
() => {
  const candidates = [...document.querySelectorAll(
    'p, h1, h2, h3, h4, h5, h6, a, button, input, select, textarea, label, img, svg, li, td, th, [role="button"]'
  )].filter(isVisible)
   .filter(el => (el.textContent || '').trim() || ['IMG','SVG','INPUT','SELECT','TEXTAREA'].includes(el.tagName));

  const hits = [];
  for (let i = 0; i < candidates.length; i++) {
    for (let j = i + 1; j < candidates.length; j++) {
      const a = candidates[i], b = candidates[j];
      if (a.contains(b) || b.contains(a)) continue;
      const ra = a.getBoundingClientRect(), rb = b.getBoundingClientRect();
      const ox = Math.min(ra.right, rb.right) - Math.max(ra.left, rb.left);
      const oy = Math.min(ra.bottom, rb.bottom) - Math.max(ra.top, rb.top);
      if (ox > 8 && oy > 8) hits.push({ a: describe(a), b: describe(b), overlapPx: [Math.round(ox), Math.round(oy)] });
      if (hits.length >= 30) return { truncated: true, hits };
    }
  }
  return { truncated: false, hits };
}
```

**Triage before reporting:** exclude pairs where one element is an intentional overlay (positioned tooltip/dropdown/modal/badge with a stacking context). Everything else — text over text, controls colliding, image over copy — is a **P0**.

## Detector 3: Text Clipping & Truncation

Text cut off mid-glyph or overflowing its box without an intentional ellipsis.

```js
() => [...document.querySelectorAll('body *')]
  .filter(isVisible)
  .filter(el => [...el.childNodes].some(n => n.nodeType === 3 && n.textContent.trim()))
  .map(el => {
    const s = getComputedStyle(el);
    const clipX = el.scrollWidth > el.clientWidth + 2;
    const clipY = el.scrollHeight > el.clientHeight + 2;
    const intentional = s.textOverflow === 'ellipsis' || s.webkitLineClamp !== 'none';
    const hiddenX = ['hidden', 'clip'].includes(s.overflowX);
    const hiddenY = ['hidden', 'clip'].includes(s.overflowY);
    if ((clipX && hiddenX && !intentional) || (clipY && hiddenY && !intentional)) {
      return { ...describe(el),
        scroll: [el.scrollWidth, el.scrollHeight],
        client: [el.clientWidth, el.clientHeight] };
    }
    return null;
  })
  .filter(Boolean)
  .slice(0, 20)
```

**Flag:** each hit is P0–P1 (cut-off words/labels are literal breakage; a slightly clipped decorative line may be P2). Report scroll vs client dimensions.

## Detector 4: Out-of-Viewport / Edge-Touching Content

Content poking past the viewport (Detector 1 catches most) plus visible text/controls flush against the viewport edge with no breathing room.

```js
() => {
  const vw = document.documentElement.clientWidth;
  return [...document.querySelectorAll('p, h1, h2, h3, h4, h5, h6, button, a, input, img, li')]
    .filter(isVisible)
    .map(el => {
      const r = el.getBoundingClientRect();
      const flushLeft = r.left >= 0 && r.left < 4;
      const flushRight = r.right <= vw && vw - r.right < 4;
      if (flushLeft || flushRight) return { ...describe(el), flushLeft, flushRight };
      return null;
    })
    .filter(Boolean)
    .slice(0, 20);
}
```

**Triage:** full-bleed elements (hero images, edge-to-edge dividers) are intentional; body text or controls touching the edge are P1.

## Detector 5: Broken & Distorted Images

```js
() => [...document.images].map(img => {
  const r = img.getBoundingClientRect();
  const broken = img.complete && img.naturalWidth === 0;
  const natural = img.naturalWidth / (img.naturalHeight || 1);
  const rendered = r.width / (r.height || 1);
  const distorted = img.naturalWidth > 0 && r.width > 20 &&
    Math.abs(natural - rendered) / natural > 0.05 &&
    getComputedStyle(img).objectFit === 'fill';
  if (broken || distorted) return { src: (img.currentSrc || img.src).slice(-60), broken, distorted,
    natural: [img.naturalWidth, img.naturalHeight], rendered: [Math.round(r.width), Math.round(r.height)] };
  return null;
}).filter(Boolean)
```

**Flag:** broken → P0. Aspect-distorted → P1.

## Detector 6: Zero-Size and Invisible-but-Present Content

Elements that have real content but render at zero size, or opacity-0 elements left covering interactive areas (a common cause of "the button doesn't click").

```js
() => [...document.querySelectorAll('body *')]
  .filter(el => (el.textContent || '').trim().length > 0 || el.tagName === 'IMG')
  .map(el => {
    const r = el.getBoundingClientRect();
    const s = getComputedStyle(el);
    const zeroButPresent = s.display !== 'none' && s.visibility !== 'hidden'
      && (r.width === 0 || r.height === 0)
      && el.children.length === 0;
    const ghostBlocker = parseFloat(s.opacity) === 0 && s.pointerEvents !== 'none'
      && r.width > 40 && r.height > 40;
    if (zeroButPresent || ghostBlocker) return { ...describe(el), zeroButPresent, ghostBlocker };
    return null;
  })
  .filter(Boolean)
  .slice(0, 20)
```

**Flag:** zero-size content → P1 (something failed to lay out). Ghost blockers → P0 if over interactive content.

## Detector 7: Unreadable Text (Size & Contrast)

```js
() => {
  const parse = (c) => c.match(/[\d.]+/g)?.map(Number) ?? [0, 0, 0];
  const lum = ([r, g, b]) => {
    const f = (v) => { v /= 255; return v <= 0.03928 ? v / 12.92 : ((v + 0.055) / 1.055) ** 2.4; };
    return 0.2126 * f(r) + 0.7152 * f(g) + 0.0722 * f(b);
  };
  const bgOf = (el) => {
    let n = el;
    while (n && n !== document.documentElement) {
      const bg = getComputedStyle(n).backgroundColor;
      const p = parse(bg);
      if (p.length < 4 || p[3] > 0.9) return p.slice(0, 3);
      n = n.parentElement;
    }
    return [255, 255, 255];
  };
  return [...document.querySelectorAll('body *')]
    .filter(isVisible)
    .filter(el => [...el.childNodes].some(n => n.nodeType === 3 && n.textContent.trim().length > 2))
    .map(el => {
      const s = getComputedStyle(el);
      const size = parseFloat(s.fontSize);
      const fg = parse(s.color).slice(0, 3);
      const bg = bgOf(el);
      const [l1, l2] = [lum(fg), lum(bg)].sort((a, b) => b - a);
      const ratio = (l1 + 0.05) / (l2 + 0.05);
      const large = size >= 24 || (size >= 18.66 && parseInt(s.fontWeight) >= 700);
      const tooSmall = size < 11;
      const lowContrast = ratio < (large ? 3 : 4.5);
      if (tooSmall || lowContrast) return { ...describe(el), fontSize: size, contrast: +ratio.toFixed(2), tooSmall, lowContrast };
      return null;
    })
    .filter(Boolean)
    .slice(0, 25);
}
```

**Triage:** the background walk is approximate (images/gradients behind text won't be caught) — confirm low-contrast hits visually. Confirmed body-text contrast failures → P0–P1; sub-11px meaningful text → P1.

## Confirming Hits with Screenshots

For each detector hit, capture visual evidence:

1. Scroll the element into view: `evaluate_script` → `el.scrollIntoView({block: 'center'})`.
2. Take an **element or region screenshot at full resolution** (chrome-devtools `take_screenshot` with the element `uid`, or a viewport shot after scrolling).
3. Attach it to the finding's Evidence line alongside the measured values.

If a hit cannot be visually confirmed (detector artifact, intentional design), keep it in the report as a note with the reason it was dismissed — silent dismissal is how breakage gets re-missed.

## Priorities

| Hit | Priority |
|-----|----------|
| Page overflow, content overlap, broken image, ghost blocker over controls | P0 |
| Clipped text/labels, content past viewport, edge-touching controls/copy, distorted images, confirmed contrast failure | P0–P1 |
| Zero-size content, sub-11px text, edge-touching decorative content | P1–P2 |

## Native Mobile Fallback (argent)

No DOM on native apps, but `describe` returns normalized frames — run the same math on them:

- Overlap: any two interactive/text elements whose frames intersect by more than ~0.01 in both axes.
- Off-screen: `frame.x < 0`, `frame.x + frame.width > 1`, same for y.
- Edge-touching: text/control frames with `x < 0.01` or `x + width > 0.99`.
- Zero-size: visible elements with width or height of 0.

Screenshot at **`scale: 1.0`** (the default 0.3 hides exactly the defects this pack exists to find) to confirm hits visually.
