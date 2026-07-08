# Task Walkthrough

UX is judged by doing, not looking. Define the task, then step through it the way the intended user would — including the unhappy paths. A single static screenshot cannot reveal flow, feedback, or recovery problems.

## Contents
- Define the task first
- Tool selection
- Find the right URL first
- Walk the happy path
- Walk the unhappy paths
- What to record
- Working from screenshots/recordings only

## Define the Task First

Before touching the UI, state:
- **Who** the user is (novice, returning, expert, assistive-tech user).
- **The goal** — one concrete outcome ("check out with a saved card", "invite a teammate", "recover a forgotten password").
- **Success state** — the observable signal that the goal is done.
- **Starting point** — where the user realistically begins (cold, logged-out, deep link).

Every finding is weighed by how much it harms this task.

## Tool Selection

| Situation | Tool |
|-----------|------|
| Web app (preferred) | The `agent-browser` skill — navigate, click, type, screenshot, extract |
| Web app (DevTools needs) | `chrome-devtools` MCP — snapshots, computed state, console/network, latency |
| Mobile app | `argent` — `describe` + gestures/taps on a booted simulator/emulator |
| Nothing runnable | Reconstruct from screenshots/recording; mark unverified steps |

Prefer `agent-browser`; fall back to `chrome-devtools` MCP for state/console/network inspection during the walk.

## Find the Right URL First

Do not guess the port. Check the dev config (e.g. `vite.config.ts` → `server.port`), then running dev-server output, then confirm with `lsof -i :<port>` (defaults: 5173 Vite, 3000 Next/React, 4173 Vite preview, 8080).

## Walk the Happy Path

Step through the task to the success state. At each step record: what you did, what the system did, how long it took, and whether the next action was obvious. Note any hesitation, ambiguity, or extra click.

## Walk the Unhappy Paths

This is where real UX breaks. Deliberately exercise:
- **Bad/edge input** — wrong format, empty required field, too long, duplicate.
- **Reversal** — back button, browser back, cancel, escape, close mid-flow; does work survive?
- **Interruption / return** — leave and come back; refresh mid-flow.
- **System failure** — slow network (throttle), server error, no results, empty account.
- **Input modes** — keyboard-only pass; touch (or emulation) for target size and hover-dependence.

Record where the user would get stuck, lose data, or be unable to recover.

## What to Record

Feed the report's **Flow Walkthrough** section and each finding's **Evidence**:
- Step-by-step: action → system response → friction.
- Click/step count vs. the necessary minimum.
- Exact error/confirmation copy.
- The point of failure, or the success state reached.
- Which paths/states/input modes you covered (for the **Verified** line) and which you could not.

Use `chrome-devtools` MCP where objective signals help: `list_console_messages` (silent errors), `list_network_requests` (failed/slow calls behind a hang), `performance_start_trace` (latency at a janky step).

## Working From Screenshots / Recordings Only

- Reconstruct the sequence from the frames; do not assume steps or states you cannot see.
- List unverified paths/states/input modes explicitly in the **Verified** line.
- Where a missing state matters (no visible error/empty/loading), record it as a Gap with the caveat that it could not be confirmed live.
