# AI Slop Detection (CRITICAL)

The single most important check. "AI slop" is the interchangeable, template-generated look shared by most AI-produced interfaces of the 2024–2025 era. It reads as generic, untrustworthy, and unconsidered even when technically clean.

## The Core Test

> If you showed this interface to someone and said "AI made this," would they believe you immediately?

If yes, that is the finding. Slop is not one bug — it is the compounding of the fingerprints below. Two or three tells is a smell; five or more is Severe. Score honestly.

## Risk Levels

| Level | Signal |
|-------|--------|
| **None** | Distinct point of view; nothing on the fingerprint list, or every match is a deliberate, well-executed choice |
| **Low** | 1–2 mild tells, otherwise intentional |
| **Moderate** | 3–4 tells; the design leans on defaults |
| **High** | 5+ tells; recognizably generated |
| **Severe** | The look IS the template — indistinguishable from a hundred other AI outputs |

Report the level in the Verdict. Each distinct tell is at least a P1 finding; a Severe overall read is P0.

## The Fingerprints (DON'T list)

Each row is a tell. Flag matches unless the choice is clearly deliberate and executed with intent.

| Fingerprint | What it looks like | Why it reads as slop |
|-------------|--------------------|----------------------|
| **AI color palette** | Indigo/violet/purple-blue (`#6366F1`, Tailwind `indigo`/`violet`), blue→purple, teal accents. Default Tailwind palette untouched. | The default of the default. Signals no color decision was made. |
| **Gradient text** | Headings with `bg-clip-text` purple→pink/blue gradients. | The #1 "look at my AI landing page" tell. |
| **Dark mode + glow** | Dark background with neon/colored glow shadows, halos behind buttons and cards. | Decoration masquerading as depth. |
| **Glassmorphism everywhere** | Frosted `backdrop-blur`, translucent cards stacked on gradients. | Applied as reflex, not for layering meaning. |
| **Hero metric row** | A row of 3–4 big stats ("10k+ users", "99.9% uptime", "24/7") with no real data behind them. | Fills space, communicates nothing. |
| **Identical card grid** | 3-column feature cards, each icon + heading + one line, all equal weight. | No hierarchy; every feature "equally important." |
| **Generic type** | Inter / Geist / system default with no type personality or pairing. | Zero typographic voice. |
| **Emoji as icons** | Emoji in headings, buttons, or as feature icons; ✨ "AI-powered" pills. | Reads as placeholder, not design. |
| **Rounded + shadow + border on all** | `rounded-2xl shadow-lg border` on every surface. | Uniform softness; nothing earns emphasis. |
| **Everything centered** | Center-aligned hero, center text blocks, perfect symmetry throughout. | Safe, flat, no compositional intent. |
| **Gradient blobs / mesh bg** | Blurry gradient blobs, mesh-gradient backgrounds, floating orbs. | Ambient noise standing in for art direction. |
| **Gradient CTA + generic copy** | Purple gradient button reading "Get Started" / "Try it free". | Interchangeable with every other generated page. |
| **Bento grid as decoration** | Asymmetric tile grid used because it's trendy, not because content needs it. | Form with no function. |
| **Uniform spacing, no rhythm** | One spacing value everywhere; no tight/loose contrast to group content. | Even ≠ designed. |
| **Fake social proof** | Avatar stacks, invented testimonials, logo walls of unknown brands. | Hollow trust signals. |
| **Icon-in-circle rows** | Lucide/Heroicons in colored circles, repeated identically. | Iconography as filler. |

## How to Report Slop

Do not just say "looks AI-generated." Name the specific fingerprints and prescribe the antidote.

**Bad:** "This looks like AI made it."

**Good:**
> **[F-01] Reads as generic AI output** · Inconsistency · Dimension: AI Slop · P0
> **Evidence:** Indigo-500 primary (`#6366F1`, untouched Tailwind default), gradient-text hero heading, three equal-weight feature cards with Lucide-in-circle icons, and a purple gradient "Get Started" CTA.
> **Impact:** Indistinguishable from hundreds of generated landing pages; undermines trust and brand distinctiveness.
> **Recommendation:** Replace the default palette with brand tokens (a committed primary + warm neutral, not indigo); drop the gradient text for a real type hierarchy; give the three features unequal weight based on actual importance; swap circle icons for content or restrained marks.

## The Antidote (what "not slop" looks like)

Use these as the bar and to phrase recommendations:

- A committed, non-default palette — often high-contrast neutrals plus one intentional accent.
- Real typographic hierarchy and at least one font with a point of view.
- Emphasis earned by content importance, not applied uniformly.
- Whitespace and spacing rhythm used to group and separate, not spread evenly.
- Depth from layering and contrast, not glow and blur.
- Composition with intentional asymmetry and a clear focal point.
- Copy specific to the product, not "Get Started" boilerplate.
