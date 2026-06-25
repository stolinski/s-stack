---
name: loop-me
description: Grill the user about recurring personal or work loops and turn them into implementable workflow specs in the current workspace. Use when designing repeatable workflows, delegatable loops, or personal automation routines.
argument-hint: "A workflow to design, or nothing to go find one"
---

# Loop Me

Run a stateful grilling session whose only output is workflow specs. Use the `grilling` discipline: one question at a time, with a recommended answer attached to each question. Create, edit, and delete workflow specs as the grilling resolves things.

## The Loop Lens

A **loop** is a recurring pattern in the user's life: their career, their week, their morning, or a single repeated activity. Picturing a life as loops within loops reveals how predictable its activities really are, which is what makes them worth delegating.

A **workflow** is the spec of one loop, made real. You run a workflow on a loop; the loop is its running instantiation. Workflows live in `workflows/*.md` and are the source of truth.

Use the loop lens to find loops worth specifying, including loops the user has not noticed yet.

## Workspace Files

- `workflows/*.md` - one spec per workflow.
- `NOTES.md` - raw notes on the user's world: tools, channels, repeated activities, and the user's terminology.

If `NOTES.md` is empty or thin, interview the user about their world before specifying workflows. Sharpen fuzzy terms into canonical ones as they surface, and record them there.

## Vocabulary

Reach for this vocabulary only when a workflow calls for it. Do not force every workflow to include every concept.

- **Trigger** - what fires each run: an event, such as a new email or issue, or a schedule, such as every morning. Event-triggering is usually more efficient.
- **Checkpoint** - a human-in-the-loop point where the user verifies or decides. Some workflows have none and run autonomously; some use no AI at all.
- **Push right** - defer the checkpoint as far as it will go. Do maximal work before involving the human, so they are asked once, late, with everything prepared.
- **Brief** - what a checkpoint presents: a tight, decision-ready summary of what was produced, why, and where to inspect it. The user reads a brief, not a draft.

## Workflow Spec Standard

A workflow spec is done when an implementer agent could build it without asking a single question. Grill until then; nothing is done while a load-bearing question remains.

Keep specs implementation-ready but not bloated. Mandate nothing structural: a workflow needs no AI, no checkpoint, and no schedule unless the grilling shows it does.

## Process

1. Inspect `NOTES.md` and existing `workflows/*.md` if they exist.
2. If the user provided a workflow idea, start there. Otherwise, propose one likely loop from the notes or workspace context.
3. Grill one consequential decision at a time.
4. Update `NOTES.md` when durable terminology or context lands.
5. Create or update the relevant `workflows/*.md` spec as decisions crystallize.
6. Stop when the spec is implementable without another question.
