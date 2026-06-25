---
name: design-engineering
description: Use when designing, building, reviewing, or polishing user interfaces, motion, component behavior, interaction details, and product-feel across any frontend stack. Generalized from Emil Kowalski-style design engineering principles; framework-neutral, implementation-first.
---

# Design Engineering

## Purpose

Use this skill to make interfaces feel intentional, fast, and physically believable — not merely functional. It is framework-neutral: apply it to HTML/CSS, Svelte, React, Vue, native apps, Web Components, canvas/WebGL, or design specs.

The goal is not decoration. The goal is product feel: the compounding effect of layout, motion, responsiveness, accessibility, and tiny interaction details that users rarely notice consciously but absolutely feel.

## When to Use

Use for:

- UI polish passes
- Interaction design and component behavior
- Motion design, transitions, gestures, and microinteractions
- Reviewing frontend code for feel, responsiveness, and accessibility
- Turning a rough UI into something crisp and shippable
- Designing component APIs where behavior and styling must stay coherent
- Diagnosing why an interface feels slow, awkward, cheap, or janky

Do not use for:

- Pure backend work with no user-facing behavior
- Brand strategy without UI execution
- Large design-system governance unless the task includes concrete UI behavior

## Core Philosophy

### Taste is trained

Good taste is not vibes. It is pattern recognition built by studying great interfaces, reverse-engineering why they feel good, inspecting tiny details, and practicing until the defaults become sharp.

Do not stop at “it works.” Ask what the user feels in the first 100ms after they act.

### Invisible details compound

The best UI details disappear. A popover opens from the trigger. A button visibly responds to pressure. A list reorders without losing spatial continuity. A destructive action feels serious without feeling dramatic.

Users may not name these details, but their nervous system does.

### Beauty is leverage

Most software is good enough. Taste is a differentiator. Strong defaults, precise motion, and coherent interaction details make tools feel more trustworthy and more desirable.

## Operating Loop

1. Identify the user action.
2. Identify the expected physical response.
3. Decide whether motion helps or hurts.
4. Make the first frame responsive.
5. Preserve spatial continuity.
6. Implement with the least fragile primitive.
7. Test slowly, quickly, interrupted, repeated, keyboard-only, touch, and reduced-motion.
8. Remove anything that feels ornamental under repetition.

## Review Output Contract

When reviewing UI or code, prefer a compact table:

| Before | After | Why |
| --- | --- | --- |
| `transition: all 300ms` | `transition: transform 180ms var(--ease-out), opacity 180ms var(--ease-out)` | Exact properties avoid accidental slow/janky animations |
| Element appears from `scale(0)` | Start at `scale(.96)` + `opacity: 0` | Real objects do not appear from mathematical nothing |
| Popover scales from center | Popover scales from trigger edge/origin | Motion should explain where the surface came from |
| Keyboard command opens animated palette | Keyboard command opens instantly | Frequent keyboard actions must feel immediate |

If making changes, also report:

- Changed: files/areas touched
- Verified: exact command/browser/device checks run
- Remaining: only if something material is still unresolved

## Animation Decision Framework

### 1. Should this animate?

Ask how often the user sees it.

| Frequency | Decision |
| --- | --- |
| Hundreds of times/day: command palette, keyboard nav, shortcut-driven actions | No animation, or effectively instant |
| Tens of times/day: hover, row focus, common navigation | Minimal feedback only |
| Occasional: dialogs, drawers, toasts, settings panels | Standard motion is useful |
| Rare: onboarding, empty-state celebration, marketing demo | Delight is allowed |

Rule: keyboard-initiated actions should generally not animate. Keyboard users are asking for speed.

### 2. What is the purpose?

Valid purposes:

- Feedback: confirms an input was received
- Spatial continuity: explains where something came from or went
- State indication: connects previous state to next state
- Jarring-change prevention: avoids a broken-looking jump
- Education: demonstrates behavior or hierarchy
- Delight: only when rare enough not to become tax

If the purpose is only “looks cool,” cut it unless the moment is rare and intentionally expressive.

### 3. What easing should it use?

Default decisions:

- Entering/exiting surfaces: strong `ease-out`
- Moving objects already on screen: `ease-in-out`
- Hover/color/soft property changes: standard `ease` or tokenized curve
- Constant looping motion: `linear`
- Drag/release, physics, interrupts: spring/inertia model when available

Useful CSS curves:

```css
:root {
  --ease-out: cubic-bezier(0.23, 1, 0.32, 1);
  --ease-in-out: cubic-bezier(0.77, 0, 0.175, 1);
  --ease-drawer: cubic-bezier(0.32, 0.72, 0, 1);
}
```

Avoid `ease-in` for most UI entrances. It withholds motion at the moment the user is watching, so it feels sluggish.

### 4. How fast should it be?

| Interaction | Duration |
| --- | --- |
| Press feedback | 80–160ms |
| Hover/focus feedback | 100–180ms |
| Tooltip | 100–160ms, often no delay after first hover |
| Popover/menu | 140–220ms |
| Toast enter | 180–260ms |
| Dialog/drawer | 200–320ms |
| Page/route transition | 200–400ms, only if it preserves orientation |
| Exit of temporary UI | Usually faster than enter |

Perceived performance matters more than raw duration. The first visual response should happen immediately, even if the full transition takes longer.

## Motion Rules with Teeth

### Never animate `all`

Bad:

```css
.card { transition: all 300ms ease; }
```

Good:

```css
.card {
  transition:
    transform 180ms var(--ease-out),
    opacity 180ms var(--ease-out),
    border-color 140ms ease;
}
```

### Prefer transform and opacity

Animate `transform` and `opacity` by default. Treat layout properties (`height`, `width`, `top`, `left`, `margin`) as expensive and suspicious unless the element count is small and you have tested it.

If animating layout is necessary, isolate the damage:

- Limit the animated subtree
- Use containment where appropriate
- Avoid animating large lists
- Test under CPU throttling or real low-end devices

### Do not appear from `scale(0)`

Bad:

```css
.popover[data-state='open'] { transform: scale(1); }
.popover[data-state='closed'] { transform: scale(0); }
```

Good:

```css
.popover {
  transform-origin: var(--surface-origin, top center);
  transition: transform 160ms var(--ease-out), opacity 120ms ease;
}
.popover[data-state='closed'] {
  transform: scale(.96) translateY(-2px);
  opacity: 0;
}
```

Objects should feel like they enter the world, not spawn from zero dimension.

### Origin matters

- Popovers, dropdowns, menus: scale/slide from the trigger or attachment edge.
- Modals: stay centered; do not pretend they came from a random button unless the design is explicitly morphing.
- Tooltips: small offset from anchor; fast and calm.
- Drawers/sheets: move from their physical screen edge.

### Make press states physical

Buttons and pressable controls need a visible response.

Good patterns:

```css
.button {
  transition: transform 120ms var(--ease-out), background-color 120ms ease;
}
.button:active {
  transform: scale(.97);
}
```

For dense professional tools, reduce the scale change: `.99` or a color/brightness response can be enough.

### Prefer interruptible motion

Users change their mind mid-transition. UI should survive interruption.

Prefer:

- CSS transitions for simple state changes
- Web Animations API for controllable imperative animation
- Spring/inertia libraries for gestures
- Timeline libraries only when sequencing is truly needed

Avoid brittle keyframe choreography for basic UI state changes.

### Use `@starting-style` for enter transitions when supported

For native CSS enter animations without mounting hacks:

```css
.panel {
  opacity: 1;
  transform: translateY(0);
  transition: opacity 160ms ease, transform 160ms var(--ease-out);

  @starting-style {
    opacity: 0;
    transform: translateY(4px);
  }
}
```

Use progressive enhancement. Do not make the component depend on unsupported behavior without a fallback.

## Component Design Principles

### Components own behavior, not just pixels

A polished component defines:

- States: idle, hover, focus, active, loading, disabled, selected, invalid
- Input modes: mouse, touch, keyboard, screen reader
- Timing: how it enters, exits, interrupts, and repeats
- Layout behavior: overflow, wrapping, density, container constraints
- Accessibility: roles, names, focus movement, reduced motion

A component that only defines colors and border radius is not done.

### Design APIs around intent

Prefer component props/options that encode product intent over raw styling knobs.

Better:

```ts
variant: 'primary' | 'secondary' | 'danger'
density: 'comfortable' | 'compact'
emphasis: 'strong' | 'subtle'
```

Worse:

```ts
borderColor: string
paddingX: number
animationDuration: number
```

Expose escape hatches only when real use cases demand them.

### Empty, loading, and error states are part of the component

Do not bolt them on later. A component should know how it behaves when:

- Data has not loaded
- Data is empty
- Data failed
- Data is stale while refetching
- The user lacks permission
- The viewport is cramped

### Focus is a design material

Keyboard focus should be visible, tasteful, and predictable. Never remove outlines without replacing them. Focus movement must match user intent.

Checklist:

- Tab order follows visual/task order
- Escape closes temporary surfaces
- Focus returns to the trigger after dismissal
- Dialogs trap focus only while modal
- Roving focus is used for composite widgets when appropriate

## Gestures and Drag Interactions

### Dismiss by intent, not distance alone

Use a combination of distance and velocity. A quick confident flick should dismiss even if distance is short; a slow accidental drag should recover.

### Boundaries should damp, not brick-wall

When dragging past a limit, apply resistance/friction instead of hard stopping. Hard stops feel cheap and broken.

### Capture the pointer

For web drag interactions, use pointer events and pointer capture so dragging remains coherent even if the pointer leaves the element.

```ts
function onPointerDown(event: PointerEvent): void {
  event.currentTarget.setPointerCapture(event.pointerId);
}
```

Guard multi-touch if the gesture is single-pointer.

### Release must settle somewhere believable

On release, animate to the nearest intentional state: closed, open, dismissed, snapped, reordered. Do not leave fractional awkward states unless the UI is truly continuous.

## Layout and Visual Polish

### Alignment beats decoration

Before adding effects, fix:

- Uneven spacing
- Optical alignment
- Baseline rhythm
- Hit target size
- Text contrast
- Icon/text balance
- Ragged wrapping in common widths

### Use hierarchy before chrome

If a screen feels confusing, do not add borders everywhere. Clarify hierarchy with grouping, spacing, type scale, and content order first.

### Defaults should be opinionated

A high-quality component should look good with minimal configuration. If every usage needs five props/classes to look acceptable, the abstraction is weak.

### Responsive behavior is not just breakpoints

Check:

- Long labels
- Narrow containers
- Tall content
- Touch targets
- Safe areas / notches
- Reduced pointer precision
- Text zoom
- OS/browser font rendering differences

## Accessibility and User Preferences

### Reduced motion is not optional

Implement `prefers-reduced-motion`. Reduced does not always mean zero; it means remove vestibular motion and long transitions.

```css
@media (prefers-reduced-motion: reduce) {
  *, *::before, *::after {
    scroll-behavior: auto !important;
  }

  .animated-surface {
    transition-duration: 1ms;
    animation-duration: 1ms;
  }
}
```

Prefer component-scoped reductions when possible so you do not destroy useful non-motion affordances globally.

### Touch devices do not hover

Do not hide essential actions behind hover. Hover effects should enhance, not reveal required functionality.

Use pointer/hover media queries when needed:

```css
@media (hover: hover) and (pointer: fine) {
  .card:hover { transform: translateY(-1px); }
}
```

### Disabled is not the only unavailable state

A disabled button with no explanation is often bad UX. Prefer visible requirements, inline validation, or enabled actions that explain what is missing.

## Performance Rules

### Measure the feeling, not just the profiler

A transition can be technically cheap and still feel wrong. Test with real interaction speed, repeated use, and interruption.

### CSS usually wins for simple state animation

For simple enter/exit/hover/press states, CSS transitions are smaller, interruptible, and resilient. Use JavaScript when animation depends on runtime measurement, gestures, sequencing, physics, or canvas/WebGL.

### Use JS animation deliberately

If using a JS animation library:

- Know whether it writes transform/opacity or layout properties
- Ensure animations are interruptible
- Avoid rerendering the app every frame
- Keep animation state out of broad reactive/global state when possible
- Test with CPU throttling

### Beware inherited CSS variables in hot paths

Animating a CSS custom property can invalidate more than expected because variables inherit. It can be fine; just test it in real DOM shape before assuming it is free.

## Debugging and QA

### Slow it down

Temporarily multiply durations by 4–10x. Bad origins, weird easing, cross-fades, and layout jumps become obvious.

### Inspect frame by frame

Use browser/devtool animation inspection or screen recording scrub. Verify:

- First frame responds immediately
- No flash of wrong state
- No layout jump at end
- Exit does not lag
- Origin matches trigger/surface relationship

### Test interruption

Repeatedly open/close, hover/unhover, drag/cancel, route away, resize, and spam-click. Polished UI survives rude users.

### Test input modes

Before calling UI done, test:

- Mouse
- Keyboard
- Touch or touch emulation
- Screen reader semantics when relevant
- Reduced motion
- Narrow viewport
- Slower device / CPU throttle

## Common Fix Recipes

### Dropdown feels sluggish

- Remove `ease-in`
- Cut duration to 140–200ms
- Start with opacity + 2–6px translate, not a huge slide
- Scale from trigger origin only if it helps

### Modal feels cheap

- Keep modal centered
- Fade/soft-scale backdrop and content separately
- Avoid huge zooms
- Ensure focus trap and Escape behavior are correct
- Make exit slightly faster than enter

### Button feels dead

- Add active state
- Tighten hover feedback
- Check disabled/loading states
- Make loading preserve width to avoid layout shift
- Confirm focus style is visible

### List reorder feels confusing

- Preserve object identity visually
- Animate moved items, not the entire list opacity
- Avoid cross-fading text unless content actually changed
- Keep duration short under repeated interactions

### Page transition feels slow

- Remove it if navigation is frequent
- Animate only persistent shell/context, not every piece of content
- Show immediate feedback first
- Preserve scroll/focus expectations

## Final Review Checklist

Before shipping UI, answer yes:

1. Does the first frame respond immediately to user input?
2. Does every animation have a purpose besides decoration?
3. Are frequent actions instant or nearly instant?
4. Are transitions limited to exact properties?
5. Are transform origins physically believable?
6. Are press, hover, focus, loading, empty, error, and disabled states handled?
7. Does keyboard behavior match mouse behavior where appropriate?
8. Is reduced motion handled?
9. Does the UI survive interruption and spam-clicking?
10. Has it been tested in the smallest realistic container/viewport?
11. Has it been tested on touch or touch emulation if touch users exist?
12. Is the component API expressing intent rather than leaking arbitrary styling knobs?
13. Did you remove ornamental motion that becomes annoying under repetition?

If any answer is no, fix it before calling the interface polished.

## Source Note

This skill is generalized from Emil Kowalski-style design engineering principles and the source skill Scott provided, with framework-specific React/Framer assumptions removed. Keep the craft, lose the framework lock-in.
