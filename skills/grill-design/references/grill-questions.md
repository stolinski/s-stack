# Grill Questions + Anti-Slop Guardrails

The decision tree for extracting a distinctive identity. Ask ONE question per turn with a recommended answer attached. Reject generic answers using the guardrails. Explore the repo before asking anything the code already answers.

## Contents
- How to reject a generic answer
- The question tree
- Anti-slop guardrails

## How to Reject a Generic Answer

Vague adjectives ("modern", "clean", "minimal", "sleek", "professional") describe almost every app and commit to nothing. When you get one, do not accept it — push:

- Ask for a **concrete reference**: "Name a product whose look you'd be happy to be compared to, and one you'd hate."
- Ask for a **contrast**: "Clean like Linear's dense precision, or clean like Apple's airy space? They're opposites."
- Ask for a **feeling + audience**: "What should a first-time user feel in the first second — and who is that user?"
- Force a **trade-off**: "Pick one: calm and quiet, or bold and confident. You can't lead with both."

Keep pushing a branch until the answer would change a token.

## The Question Tree

Ask in this order. Each turn: one question + a recommended answer + a one-line reason.

1. **Product & feeling** — What is the app, who uses it, and what single emotion should it evoke in the first second? (Get 2–3 specific adjectives, not "clean".)
2. **The distinctive angle** — What is the one thing that should make this look unmistakably *this* app and not a template? (This is the uniqueness anchor. Do not skip.)
3. **References** — One product whose look you admire and want to be compared to. Why that one specifically?
4. **Anti-references** — One product (or the generic "AI landing page" look) you want to avoid. What exactly about it?
5. **Foundation tone** — Light or dark foundation? Warm or cool neutrals? (e.g. warm limestone `#F7F5F2` vs. cool slate vs. true dark.)
6. **Accent & interaction color** — The single color that drives interaction. Push away from reflexive indigo/violet toward something owned by the brand. What is it and where is it *allowed* to appear?
7. **Semantic colors** — Success / warning / danger / info — inherit sensible defaults or brand-tune them?
8. **Type personality** — Should type feel editorial, technical, humanist, geometric, or characterful/display? Name at least one family with a point of view for display; a readable family for body.
9. **Type scale & hierarchy** — How many levels, and how sharp is the jump between them? (Big editorial contrast vs. tight utilitarian steps.)
10. **Density & spacing** — Airy and generous, or dense and efficient? This sets the spacing scale.
11. **Shape & radius language** — Sharp/architectural (0–4px), soft/friendly (8–16px), or pill/round? Consistent or mixed by component?
12. **Depth & elevation** — How is depth expressed: flat, hairline borders, subtle shadows, or layered surfaces? (Steer away from colored glow.)
13. **Motion posture** — Calm and instant, or expressive? (Brief — feeds Do's/Don'ts; defer craft detail to `design-engineering`.)
14. **Do's and Don'ts** — Capture 3–5 explicit rules, including the anti-slop lines this brand commits to (from anti-references above).

Stop when answers 1–12 are specific enough to assign token values and 2/4/14 give the identity a point of view.

## Anti-Slop Guardrails

Actively steer away from the generic AI look. Flag and push back when an answer drifts toward any of these — accept them only as a deliberate, reasoned choice:

| Default drift | Push toward |
|---------------|-------------|
| Indigo/violet/purple-blue accent; untouched Tailwind palette | A committed, owned accent color chosen for the brand's feeling |
| Inter/Geist/system by reflex | At least one family with character; a deliberate body face |
| Gradient text, mesh/blob backgrounds, gradient CTAs | Flat, confident color; hierarchy from type and space |
| Dark mode + neon glow shadows | Depth from layering, borders, and contrast |
| Glassmorphism everywhere | Solid, intentional surfaces; blur only where it means something |
| `rounded-2xl shadow-lg border` on everything | A stated radius/elevation language applied by role, not by reflex |
| "Modern and clean", everything centered and equal-weight | A stated compositional stance and real emphasis hierarchy |
| Emoji as icons; ✨ "AI-powered" pills | Restrained, brand-appropriate iconography and specific copy |

If, after grilling, the identity still matches the slop column, say so directly and re-open the relevant branch. Uniqueness is the deliverable.
