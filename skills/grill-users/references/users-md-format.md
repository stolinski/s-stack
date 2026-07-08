# Authoring a USERS.md

How to write a `USERS.md` from the grilled persona set. Like `DESIGN.md`, it has two layers: structured YAML front matter (machine-readable, so critique tools can pick a scenario and priority lens per type) and prose sections (the human nuance behind each type).

## Contents
- File structure
- Schema
- Field semantics
- Full example
- How downstream skills read it

## File Structure

```md
---
# YAML front matter: one entry per user type + anti-users
---

## Overview          # what the product does; the shape of its audience
## <User Type>       # one section per type, in priority order — nuance/story
## Who This Is Not For
```

## Schema

```yaml
name: <product>
description: <one line — the core job the product does>
users:
  - id: <slug>                 # stable handle, e.g. daily-operator
    label: <string>            # human name for the type
    priority: primary | secondary | edge   # exactly one primary
    share: <string>?           # rough proportion, optional (e.g. "~70% of sessions")
    proficiency:
      tech: novice | intermediate | expert
      domain: novice | intermediate | expert
    context:
      device: <string>         # where/how they use it
      environment: <string>    # focused/mobile/on-the-go/high-stress
      frequency: <string>      # daily / occasional / one-time
      stakes: low | medium | high   # cost of a mistake
    goals:                     # jobs-to-be-done → become critique scenarios
      - <observable outcome>
    cares_about:               # → the UX priority lens for this type
      - <e.g. speed, control, safety, confidence, cost, trust>
    frustrations:              # bail conditions
      - <what makes them give up or distrust>
    success: <string>          # what "done well" looks/feels like
    resonance_cue: <string>    # → the design "this is for me" test
anti_users:
  - <who it is deliberately not for>
```

`goals`, `cares_about`, `success`, and `resonance_cue` are the load-bearing fields — never leave them empty.

## Field Semantics

- **priority** — exactly one `primary`; it wins when needs conflict. `edge` = served but not optimized for.
- **cares_about** — what to weight most for this type: an expert who cares about speed makes accelerators/recognition matter most; a high-stakes novice makes error-prevention matter most.
- **goals / success** — a concrete job-to-be-done and its observable "done", reusable directly as a test scenario.
- **resonance_cue** — what makes this type feel the product is for them ("would this user feel it's for them?").
- **proficiency** — tech and domain can differ (a domain expert new to software, or vice versa). Both shape expectations.

## Full Example

```md
---
name: Ledger
description: Reconcile a business's daily transactions against bank feeds.
users:
  - id: daily-operator
    label: "Daily Operator"
    priority: primary
    share: "~70% of sessions"
    proficiency: { tech: expert, domain: expert }
    context:
      device: "desktop, dual monitor"
      environment: "focused back-office work"
      frequency: "many times daily"
      stakes: high
    goals:
      - "clear the day's unreconciled queue"
    cares_about: [speed, keyboard control, not re-entering data]
    frustrations:
      - "modal interruptions mid-task"
      - "losing filters on navigation"
    success: "queue hits zero without reaching for the mouse"
    resonance_cue: "dense, calm, precise; zero marketing gloss"
  - id: reviewing-owner
    label: "Reviewing Owner"
    priority: secondary
    proficiency: { tech: intermediate, domain: expert }
    context:
      device: "laptop or phone"
      environment: "checking in between other work"
      frequency: "weekly"
      stakes: medium
    goals:
      - "confirm the books are reconciled and spot anomalies"
    cares_about: [confidence, clarity, trust]
    frustrations: ["having to understand the operator's workflow to get a summary"]
    success: "sees 'all reconciled' and any exceptions at a glance"
    resonance_cue: "reassuring, legible summaries; no jargon"
anti_users:
  - "casual consumers tracking personal budgets"
  - "one-time visitors evaluating without a real dataset"
---

## Overview
Ledger serves a small number of high-frequency professional users. The Daily
Operator lives in it; the Reviewing Owner drops in for assurance. It is a tool of
record, not a consumer app — density and trust beat delight.

## Daily Operator
[nuance: their day, their pressure, why speed and keyboard control dominate…]

## Reviewing Owner
[nuance: what "confidence at a glance" means, why summaries must stand alone…]

## Who This Is Not For
Personal-finance hobbyists and tire-kickers without real data — optimizing for
them would add gloss and hand-holding that slows the operators who matter.
```
