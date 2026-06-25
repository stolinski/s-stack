---
name: agentbuilder
description: >
  Use this skill when creating, editing, debugging, reviewing, or documenting
  Standard Agents or AgentBuilder projects. Apply it for work on agents,
  prompts, models, tools, hooks, threads, APIs, subagents, provider setup,
  model selection, environment variables, and spec-aligned architecture, even
  if the user only says things like "build an agent", "write a prompt",
  "choose a model", or "fix my AgentBuilder app" without explicitly naming
  Standard Agents.
---

# AgentBuilder Skill

This guide is a living record. It is not part of the Standard Agent Specification — it exists to help humans and coding agents *use* the spec to build effective agents. It is updated regularly and not bound to any spec version.

> Commands like `pnpm exec agents …` are placeholders. Use whichever package manager (`npm`, `pnpm`, `yarn`, `bun`) the project actually uses.

For deeper reference material that this skill links to, see:

- `agents/agents/AGENTS.md` — agent definition reference (created by `agents scaffold`)
- `agents/prompts/AGENTS.md` — prompt and tool config reference
- `agents/tools/AGENTS.md` — tool-writing patterns and `ThreadState` examples
- `agents/models/AGENTS.md` — recommended model list (authoritative)
- `agents/hooks/AGENTS.md` — hook reference
- Canonical TypeScript types (full signatures): read `node_modules/@standardagents/spec/dist/` in your project, or browse `packages/spec/src/` on GitHub at https://github.com/standardagents/agentbuilder
- Specification: https://standardagentspec.org/llms.txt
- Builder docs: https://docs.standardagentbuilder.com/llms.txt

> If the `agents/*/AGENTS.md` files don't exist in your project yet, run `pnpm exec agents scaffold` to create them.

---

## Before you write any code

Most failed Standard Agent projects fail in the same six ways. Read this section first and answer all six questions explicitly before creating any code. Each links to the deeper section that explains *how*.

**1. Architect the agent graph before picking types.**
List the domains the system touches. Determine the tree (coordinator → domain agents → sub-domain agents). *Then* decide what each node is. Do not start by writing one agent and bolting tools onto it. → [Architecture & decomposition](#architecture--decomposition)

**2. `dual_ai` is the default. `ai_human` is the exception.**
`ai_human` is correct only when the thread itself is the chat surface (chat UI, website widget, AgentBuilder admin, direct API chat). For Slack, email, SMS, Discord, webhooks, polled inboxes, schedulers, or any other mediated channel, the human is just a tool target — the agent is `dual_ai`. The whole graph being `dual_ai` is fine and often cleaner. → [Interaction type](#interaction-type-dual_ai-is-the-default)

**3. Every `dual_ai` agent must have explicit session boundaries.**
Name its `sessionStop` tool. Name its `sessionFail` tool. Set a finite `maxSessionTurns`. Give side_b a real, non-redundant job. If you cannot say in one sentence what tool call ends the session, the agent is not designed yet. → [Session boundary discipline](#session-boundary-discipline)

**4. Pick models from `current-models` only.**
Never invent model strings from memory. Run `pnpm exec agents current-models`, choose by category, then run `pnpm exec agents available-models --provider=<name>` to confirm the exact string. → [Model selection](#model-selection)

**5. Research every third-party API before writing the tool.**
Fetch the official current docs. Confirm auth, base URL, endpoint paths, payload shapes, rate limits, error codes. Do not write a tool from memory of how an API "usually" works. APIs change. → [API research checklist](#api-research-checklist)

**6. Check `ThreadState` before adding any dependency.**
Before reaching for S3, Redis, an external cron, a queue service, or even `node:fs`, confirm the framework does not already provide the capability via `ThreadState`. It almost always does. → [ThreadState first](#threadstate-first)

---

## What is a Standard Agent?

In the Standard Agent paradigm, agents are the atomic unit of an AI system, and it is the *composition* of many domain-specific agents that produces efficacy. Standard Agents can be effective using small and cheap models — but small and cheap models suffer from poor tool discernment when presented with a broad, undifferentiated set of tools. Decomposition is what makes them work.

A "gmail" agent will outperform a "google apps" agent. A higher-level "communications" coordinator composes the gmail, slack, and SMS agents. A still-higher "personal assistant" coordinator composes communications, research, scheduling, and finance. This is fractal: the same shape scales up to teams, departments, and entire products. These agent-graphs solve real industry problems — progressive tool discovery, model/prompt tuning, context dilution, task resumability, and compaction.

Just because subagents *can* compose complex behavior doesn't mean every step needs one. A subagent is a two-sided conversation where each side may take multiple steps per turn. Sometimes you only need a feature of a different model — generating an image, rewriting a paragraph in a more eloquent voice. For those, use a **subprompt** (a single LLM step exposed as a tool), not a subagent. Use a **subagent** when you need iteration, QA, reflection, or long-lived addressable behavior.

When a subagent does QA on another model's output, prefer a *different provider* on the reviewing side. Same-lab models tend to rate their own output too generously.

### Example graph

```
personal_assistant_coordinator         (dual_ai, top-level reasoning model)
├── communications_coordinator         (dual_ai, resumable, explicit parent communication, note: children are explicit because they receive inbound messages and should filter out noise before returning)
│   ├── gmail_agent                    (dual_ai, resumable, immediate, explicit parent communication)
│   ├── slack_agent                    (dual_ai, resumable, immediate, explicit parent communication)
│   └── sms_agent                      (dual_ai, resumable, immediate, explicit parent communication)
├── research_assistant                 (dual_ai, resumable, explicit parent communication)
│   ├── browser_use_agent              (dual_ai, resumable, implicit parent communication)
│   ├── google_drive_agent             (dual_ai, resumable, implicit parent communication)
│   └── notes_agent                    (dual_ai, resumable, implicit parent communication)
├── scheduling_coordinator             (subprompt)
└── finance_ops_coordinator            (dual_ai)
```

Note: no `ai_human` anywhere. The "human" enters via the gmail/slack/sms tool calls. The whole graph is autonomous.

## The Standard Agent stack

The spec defines an agent as a composition of:

- **Providers** — how to talk to LLM providers and model variants
- **Models** — named model definitions referencing a provider
- **Prompts** — system instructions plus the tools, subprompts, and subagents available at that step
- **Tools** — functions the agent can call to interact with the world (and other agents)
- **Agents** — bind it all together: name, type, sides, session bindings
- **Hooks** — custom code that intercepts lifecycle events (history filtering, message injection, tool result transforms)
- **Effects** — custom code scheduled to run later
- **Threads** — runtime instances of an agent, each with its own state, message history, and filesystem
- **Endpoints** — built-in and custom HTTP endpoints exposed by a thread

---

## Architecture & decomposition

**Procedure (do this in order, every time):**

1. List the domains the system actually touches.
2. Draw the tree on paper or in a comment block. Coordinators on top, leaf agents at the bottom.
3. Pick interaction types. (Default `dual_ai`. Use `ai_human` only at chat-surface entry points.)
4. Name session boundaries for every `dual_ai` node.
5. Pick a model category for every node.
6. List the third-party APIs each leaf needs and queue them for research.
7. Map each capability to a `ThreadState` primitive before writing any custom plumbing.

Only then start writing files.

### Mega-agent smell test

If a single agent has **more than ~8 tools across unrelated domains**, decompose it. Decomposition is rarely wrong; flattening usually is.

Other smells that mean "decompose now":

- One prompt has tools from two clearly different worlds (e.g., `send_email` and `query_postgres` and `generate_image`).
- The system prompt is trying to teach the model when to use which tool by writing rules in English. Rules-in-prose are coordinator logic; promote them to a coordinator that picks subagents.
- A model keeps calling the wrong tool because two tools have similar names or overlapping descriptions. Different domains, different agents.
- The agent's prompt is over ~150 lines. That's almost always context dilution — split.

### Worked example: wrong vs. right

**Wrong** — one mega-agent with tools from unrelated domains:

```
personal_assistant (ai_human)
  tools: send_gmail, read_gmail, search_ebay_listings, place_ebay_order,
         track_ebay_shipment, create_calendar_event, list_calendar_events,
         post_slack_message, read_slack_channel, query_stock_price,
         execute_stock_trade, search_recipes, generate_image
```

Gmail, eBay, calendar, Slack, stock trading, recipes, image gen — seven unrelated worlds in one tool list. The model picks `send_gmail` when it meant `post_slack_message`. It calls `execute_stock_trade` with eBay listing IDs. The system prompt grows 80 lines of rules trying to teach it which tool belongs to which world. Every bug fix breaks two unrelated flows.

**Right** — coordinator + domain subagents:

```
personal_assistant_coordinator (dual_ai, reasoning model)
├── gmail_agent        tools: send_gmail, read_gmail
├── ebay_agent         tools: search_ebay_listings, place_ebay_order,
│                             track_ebay_shipment
├── calendar_agent     tools: create_calendar_event, list_calendar_events
├── slack_agent        tools: post_slack_message, read_slack_channel
├── trading_agent      tools: query_stock_price, execute_stock_trade
└── content_helpers    subprompts: search_recipes, generate_image
```

`personal_assistant_coordinator` runs a reasoning model and decides which domain owns the next step. Each leaf is a fast tool-calling model with a tight tool set it can actually discern between. A fix to the eBay flow can't break Gmail.

### Coordinators

Coordinators provide two things flat agents can't:

1. **Inter-domain communication.** A `coding_coordinator` can ask `research_agent` to investigate a library, then hand the findings to `bash_agent` to run commands. The bash agent never needed to know the research agent existed.
2. **Filtering.** A `gmail_agent` can filter spam well on its own. But the higher-level objectives of the organization — "what email actually matters to the CEO of the sprinkler hose company" — belong to a coordinator above it. The coordinator filters again before escalating.

### Graph depth tradeoffs

Deeper graphs add latency and create more inter-agent communication that can fail. But they also enable parallelism and isolation. A `marketing_communications_agent` can have a `social_media_agent` which has `twitter_agent`, `linkedin_agent`, and `facebook_agent` as children. That subtree handles posting workflows entirely without involving the top coordinator. Aim for depth that matches the natural hierarchy of the work, not depth for its own sake.

---

## Interaction type: `dual_ai` is the default

> **`dual_ai` is the default agent shape. `ai_human` is the exception**, used only when the thread itself is the chat surface — chat UI, website widget, AgentBuilder admin, or an API the user is talking to directly.

For **Slack, email, SMS, Discord, webhooks, polled inboxes, schedulers, or any other mediated channel**, the human is just a tool target. The agent on the framework side is `dual_ai`. The whole graph being `dual_ai` is fine — often cleaner.

### The single decision

Ask: **"Is the human typing directly into this thread?"**

- **Yes** → the top-level agent is `ai_human`.
- **No** → the top-level agent is `dual_ai`. The human enters the graph via a tool somewhere (e.g., a `send_slack_message` call inside a slack subagent).

Do not default to `ai_human` just because a human is eventually involved. Ask where messages physically arrive.

### The two shapes

**Shape A — chat surface (`ai_human` at top):**

```
website_chatbot (ai_human)         ← user types in a chat widget
  tools: search_products, lookup_order, escalate_to_human
```

**Shape B — mediated (`dual_ai` everywhere):**

```
slack_research_assistant (dual_ai, reasoning model)
├── slack_agent (dual_ai, resumable) ← messages arrive via tool calls,
│                                      not as thread messages
└── research_agent (dual_ai)
```

In Shape B, side_a of `slack_research_assistant` plans the work and dispatches subagents. Side_b reviews and decides when the work is done. The "user" is a Slack channel reachable through `slack_agent`'s tools.

### Handoffs (a special case of `ai_human`)

When an `ai_human` agent calls another `ai_human` agent as a tool, the runtime does **not** spawn a new thread — it changes which prompt "owns" the existing thread. This is a *handoff*. Useful for stepwise human flows: an `onboarding_agent` collects information, then hands off to a `scheduling_agent` that books a meeting. The user keeps talking to the same thread.

### Subprompts vs. subagents vs. handoffs

- **Subprompt** — one LLM step exposed as a tool. Use to switch models for a focused task: image generation, polished writing, JSON extraction.
- **Subagent** — full child thread with its own `ThreadState`. Use when you need iteration, QA, reflection, or long-lived addressable behavior. Always `dual_ai`.
- **Handoff** — `ai_human` → `ai_human`, swaps prompt ownership of the same thread. Use for stepwise human-driven flows.

A subagent can be:

- **Blocking + non-resumable** — the parent waits, the child runs once, returns, and is gone (tool-call style)
- **Blocking + resumable** — the parent waits but the child remains addressable for future calls
- **Non-blocking + non-resumable** — fire-and-forget one-shot
- **Non-blocking + resumable** — long-lived addressable child the parent can re-message later (e.g., a Slack monitor that lives as long as the parent)

---

## Session boundary discipline

This section is a hard rule, not a suggestion. **Every `dual_ai` agent in the graph must explicitly define its session boundaries.** This is what makes whole-graph `dual_ai` safe.

### Required for every `dual_ai` agent

- **`sessionStop` tool binding** — names the tool call that ends the session with a result. Side_a or side_b invokes it when the work is done. Common patterns: `report_findings`, `submit_to_parent`, `final_answer`, `done`.
- **`sessionFail` tool binding** — names the tool call that ends the session with a failure the parent should see. Common patterns: `escalate_blocked`, `report_unresolvable`, `give_up_with_reason`.
- **`maxSessionTurns`** — a finite integer sized to the realistic upper bound for the task. Never omit this. A research subagent might be 20; an asset QA loop might be 6; a tight reflection loop might be 3.
- **A real job for side_b** — reflection, QA, judging, alternative-perspective driving, devil's advocate. If side_b is "another instance of side_a," you don't have a `dual_ai` agent — collapse it to a single-side prompt or a coordinator pattern.
- **Cross-provider QA** — when side_b reviews side_a's output, pick a model from a *different provider* than side_a. Same-lab models systematically over-rate their own work.

### The smell test

> If you cannot articulate in one sentence what tool call ends the session, the agent is not designed yet.

Examples that pass:

- "Side_a calls `submit_video_assets` once side_b approves the renders."
- "Side_b calls `report_findings` after the research turn count exceeds 5 or it judges the topic exhausted."
- "Either side calls `escalate_to_parent` if the customer policy question is unresolvable."

Examples that fail:

- "It just stops when it's done." (How does the runtime know?)
- "Whichever side decides." (Decides by calling *what*?)

### Why this matters

A `dual_ai` agent with no `sessionStop`, no `sessionFail`, and no `maxSessionTurns` will burn turns until it hits a runtime cap, then fail in a way the parent can't interpret. Coding agents that have been bitten by this once will start defaulting to `ai_human` to "feel safer" — and then we're back to mega-agents and chat-surface confusion. Boundary discipline is what keeps `dual_ai` the default.

---

## Model selection

**Rule: never write a model string the user did not request and `current-models` did not produce.**

### Required workflow

1. Run `pnpm exec agents current-models` to see the curated category list (e.g. `extra_reasoning`, `fast_tool_calls`, `writing`, `image_generation`, `tiny`).
2. Choose the category that fits the role (table below).
3. Run `pnpm exec agents available-models --provider=<name>` to confirm the exact provider model string.
4. Define the model in `agents/models/<name>.ts` using `defineModel`.

If you have multiple configured providers, pass `--provider=<name>` explicitly.

### Role → category mapping

| Role | Category | Notes |
|---|---|---|
| Top-level coordinator | `extra_reasoning` | The discerning brain. Don't skimp here. |
| Domain subagent (gmail, slack, etc.) | `fast_tool_calls` | High volume, narrow tool set, latency matters. |
| Eloquent text generation | `writing` | Use as a subprompt from a fast-tool-calling agent. |
| Image generation | `image_generation` | Use as a subprompt. |
| QA / reviewer side_b | `extra_reasoning` from a *different provider* than side_a | Same-lab QA is biased. |
| Cheap classification, tagging, routing | `tiny` | Where speed and cost dominate quality. |

### Fallback strategy

Define more than one model per category, on different providers. The model `name` should describe the *use case*, not the provider, so a fallback can substitute transparently:

```
agents/models/extra_reasoning.ts          → primary (provider A)
agents/models/extra_reasoning_fallback.ts → secondary (provider B)
```

Prompts and agents reference `extra_reasoning`; if the primary provider is down, the fallback is one rename away.

For the authoritative list of which models currently fill each category, see `agents/models/AGENTS.md` in your project (created by `agents scaffold`). Do not embed the list here — it drifts.

---

## API research checklist

This is the single biggest gap in most coding-agent-built tools. **Do not write a `defineTool` that touches a third-party API from memory.** APIs change. Auth flows change. Endpoints get deprecated. Read the current docs every time.

### Before writing the tool

1. **Fetch the official current docs.** Use WebFetch, the read-website skill, or a browser. Confirm:
   - Auth method (bearer, OAuth, signed request, API key in header vs. query)
   - Base URL and current API version
   - Endpoint paths and HTTP methods
   - Request payload shape (and required vs. optional fields)
   - Response payload shape (success and error)
   - Rate limits and `Retry-After` handling
   - Pagination model (cursor, offset, link header)
   - Idempotency rules — does retrying duplicate side effects?
2. **Confirm SDK availability and Workers compatibility.** Is there a first-party JS/TS SDK? Does it run in Cloudflare Workers (no Node built-ins, no `fs`, no native modules, no long-lived sockets)? If not, use `fetch` directly. Most provider SDKs are not Workers-compatible out of the box.
3. **Identify required secrets.** Declare them as `secret` variables, never `text`. Secrets must never be referenced in prompt text or returned to the model.
4. **Map error modes explicitly.** Each gets handled, not ignored:
   - `401` / `403` — auth failed. Surface a clear message; do not retry blindly.
   - `404` — missing resource. Often a user-facing error; surface it.
   - `409` — conflict. Often means the operation already happened; check before retrying.
   - `429` — rate limited. Honor `Retry-After`. Retry with backoff.
   - `5xx` — server error. Retry with exponential backoff, finite cap.
   - `4xx` (other) — surface to the model so it can correct its arguments.
5. **Prototype the raw call** in isolation — a single `fetch` against the real endpoint with a real token. Confirm the response shape *as observed*, not as documented. Docs lag.
6. **Return shapes the model can actually use.** Strip noise. Surface IDs, names, statuses, and the fields the next step needs. Don't return the entire 12 KB JSON blob and hope the model picks the right field.

> **Anti-pattern:** "I know the Slack API, I'll just write it." You don't. It changed last quarter. Read the docs.

---

## ThreadState first

> **Rule:** Before adding any dependency or external service, check whether `ThreadState` already provides the capability.

`ThreadState` is the unified API passed to every callable, hook, and endpoint. It abstracts the runtime — your tool may run on the edge, on a Worker, or on a Node server, and the same `state.readFile()` call works in all of them. Tools should **not** import `node:fs`, read `process.env`, or assume Node-shaped APIs.

### Capability lookup table

| What you need | Use this | Don't use |
|---|---|---|
| Store a file between turns | `state.writeFile` / `state.readFile` | S3, external blob store |
| Persist small structured data across turns | `state.getValue` / `state.setValue` durable KV | External KV, Redis |
| Persist large or binary data across turns | `state.writeFile` / `state.readFile` | External blob store |
| Trigger work later | `state.scheduleEffect` | External cron, queue service |
| Invoke another tool from inside a tool | `state.invokeTool` / `state.queueTool` | Re-implementing tool logic inline |
| Read/write config and secrets | `state.env` / `state.envType` / `state.setEnv` | `process.env`, `.env` files |
| Run isolated deterministic code | `state.runCode` with explicit bridges | `eval`, child processes, implicit runtime access |
| Search files the thread has seen | `state.grepFiles` / `state.findFiles` | Reimplementing search |
| Escalate / report status to the parent | `state.notifyParent` / `state.setStatus` | Custom message bus |
| Load a sibling prompt / agent / model | `state.loadPrompt` / `state.loadAgent` / `state.loadModel` | Duplicating the definition |
| Inspect or message a child thread | `state.children` / `state.getChildThread` | External orchestration |
| Inject context the model should see | `state.injectMessage` / `state.queueMessage` | Stuffing the system prompt |
| Read the thread's message history | `state.getMessages` / `state.getMessage` | Reimplementing storage |
| Update an existing message | `state.updateMessage` | Mutating storage directly |
| Read execution logs | `state.getLogs` | External observability shim |
| Emit a runtime event | `state.emit` | Console logging |
| Stop the thread | `state.terminate` | Throwing and hoping |

### Method cheat sheet

Discoverability reference. For full signatures, read the spec types from `node_modules/@standardagents/spec/dist/` (or browse `packages/spec/src/` on GitHub). Worked examples live in `agents/tools/AGENTS.md`.

```
Identity     threadId, agentId, userId, createdAt, children, terminated
Messages     getMessages, getMessage, injectMessage, queueMessage, updateMessage
Logs         getLogs
Resources    loadModel, loadPrompt, loadAgent,
             getChildThread, getParentThread,
             getPromptNames, getAgentNames, getModelNames
Env          env, envType, setEnv
Parent       notifyParent, setStatus
Tools        queueTool, invokeTool
Effects      scheduleEffect, getScheduledEffects, removeScheduledEffect
Events       emit
Context      context (Record<string, unknown>, in-memory only)
KV           getValue, setValue
Files        writeFile, readFile, readFileStream, statFile, readdirFile,
             unlinkFile, mkdirFile, rmdirFile, getFileStats,
             grepFiles, findFiles, getFileThumbnail
Execution    execution, terminate, runCode
```

### Durable key-value store

Use `state.getValue()` and `state.setValue()` for small per-thread durable JSON values such as counters, checkpoints, cursors, tool state, and user preferences. Values survive restarts and are scoped to the current thread.

```ts
const count = (await state.getValue<number>('invocation_count')) ?? 0;
await state.setValue('invocation_count', count + 1);
```

`setValue(key, null)` and `setValue(key, undefined)` delete the key. For larger payloads, binary data, user-visible artifacts, or content that should be shared as a file, use `state.writeFile()` / `state.readFile()` instead.

### Sandboxed code execution

Use `state.runCode()` for model- or user-authored JavaScript/TypeScript instead of `eval` or `new Function`. The sandbox runs in Cloudflare Dynamic Workers, has no implicit thread state, filesystem, network, timers, or host globals, and only receives capabilities you explicitly bridge through `imports` or `globals`.

```ts
const run = state.runCode(
  `
    import { readFile } from "fs";

    export async function summarize(path: string) {
      const text = await readFile(path);
      return text.slice(0, 200);
    }
  `,
  {
    language: 'typescript',
    execute: { fn: 'summarize', args: ['/notes/input.txt'] },
    imports: {
      fs: {
        readFile: async (path: string) => {
          const file = await state.readFile(path);
          return file ? new TextDecoder().decode(file) : '';
        },
      },
    },
  },
);

const result = await run;
```

By default, `runCode()` executes the `default` export with no args. Use `execute: { fn, args }` to call a named export or pass arguments; `fn: 'default'` calls the default export with args. Use `modules` to provide local relative ES modules. The result is a status object: successful runs return `status: 'success'`, `result`, `logs`, `reports`, and `durationMs`; failed runs return an error status and an `error` object. Call `run.terminate(reason)` from your own timeout budget when needed.

### Notes on a few that are easy to misuse

- **`state.context`** is in-memory for the *current execution*. It is not durable across thread restarts. For durable structured state, use `state.getValue` / `state.setValue`.
- **`state.getValue` / `state.setValue`** are durable per-thread JSON storage. Use them for small structured state; use files for larger content or artifacts.
- **`state.env` / `state.envType` / `state.setEnv`** are for runtime configuration and secrets. `state.env(name)` resolves thread -> account -> instance -> agent -> prompt. `state.envType(name)` returns `'secret'` by default; `'text'` means the value may be shown in tool output. `state.setEnv(name, value, { type: 'text' | 'secret' })` writes thread-scoped env and propagates to active descendants. Omit `type` only when preserving the existing type is intentional; new keys default to secret.
- **`state.runCode`** runs JavaScript or TypeScript in an isolated Dynamic Worker sandbox. The sandbox does not receive `ThreadState`, env, files, network, or host globals implicitly. Pass exact capabilities through `imports`, `globals`, `modules`, and `execute`. It executes exported values/functions; code such as `console.log(fibonacci(5))` can log, but it must still `export default ...` or export a named function/value for the host to receive a result.
- **`state.scheduleEffect`** runs a named effect after a delay. It survives restarts. This is your cron, your queue, and your retry timer all in one.
- **`state.invokeTool` vs `state.queueTool`** — `invokeTool` runs synchronously and returns the result; `queueTool` schedules the call to run later in the normal tool-call flow. Prefer `queueTool` when the model should see the result as a regular tool call.
- **`state.notifyParent`** — for resumable subagents with `parentCommunication: 'explicit'`, this is the only way the child talks to the parent. Use it sparingly; every notification interrupts the parent.
- **File attachments** use the path convention `/attachments/{filename}.{ext}`. Always use this path when passing files between agents — the runtime copies them across thread filesystems automatically.

### Sandboxed code env bridge pattern

When building a coding agent that runs user-authored code, do **not** expose all thread env and do **not** rely on `process.env`. Use an explicit allowlist stored in durable KV:

1. A setup tool such as `set_code_envs` (called `set_code_run_envs` in the built-in sandboxed coding agent) receives the required env names and stores the allowlist with `state.setValue(...)`.
2. The execution tool reads that allowlist with `state.getValue(...)`.
3. For each allowed name, the execution tool resolves `await state.env(name)` and `await state.envType(name)`. If a value is missing, it may create an empty thread env entry with `state.setEnv(name, '', { type: 'secret' })` so the UI can prompt the user.
4. The execution tool calls `state.runCode(source, { imports: { env: { env: allowedValues } }, ... })`, exposing only the whitelisted env object.
5. The tool redacts values whose `envType` is `secret` from results, reports, logs, and errors. Values marked `text` may appear.

The prompt for a coding agent must explain this exact interface. Tell the model to call the allowlist setup tool before running code, then import a static object from `"env"`:

```ts
import { env } from "env";

const apiKey = env.WEATHER_API_KEY;
```

Also tell it what **not** to do: do not read `process.env`, do not call `env()` as a function, do not `await` env values, do not use named env imports, and do not pass env values around as tool arguments.

---

## Tools

A "tool" is anything an agent can call. There are three kinds:

1. **Callables** — TypeScript functions defined via `defineTool` in `agents/tools/`. The primary way to interface with the outside world: APIs, databases, business logic, and (sometimes) other agents. Each callable receives a `ThreadState`. Do not assume Node APIs are available — your code may run on the edge.
2. **Subprompts** — prompts exposed as tools via their `toolDescription`. A single-step LLM call. Use for switching models on a focused task (image generation, polished writing, JSON extraction).
3. **Subagents** — full agents exposed as tools via `exposeAsTool: true` on the agent definition. Use when you need iteration, QA, reflection, or long-lived addressable behavior. Always `dual_ai`.

### Provider-visible argument schemas

Tool argument schemas are sent to model providers, and strict tool-calling providers commonly reject JSON Schema objects unless every object schema has `additionalProperties: false`. This applies recursively, not just at the top level. If a tool has nested object args, arrays of objects, `anyOf` object branches, prompt `requiredSchema`, or an agent exposed as a tool, verify the model-facing schema contains `additionalProperties: false` for every `type: "object"`.

Common failure modes:

- `z.record(...)` can emit `propertyNames`, which some providers reject for tool schemas. Prefer explicit `z.object({ ... })` shapes for provider-visible tool args.
- `z.object({}).catchall(...)` can emit `additionalProperties: {}`. Strict providers expect the boolean `false`, not an object schema.
- "Arbitrary object" tool args are a poor fit for strict provider tool schemas. Prefer a JSON string for truly arbitrary payloads, or define the object properties explicitly.

### `PromptDefinition` cheat sheet

A prompt is what actually gets sent to the LLM at one step. Set on each prompt file in `agents/prompts/`. For full signatures, read the spec types from `node_modules/@standardagents/spec/dist/` (or browse `packages/spec/src/` on GitHub), and see `agents/prompts/AGENTS.md`.

```
PromptDefinition
  name                   string                          unique snake_case identifier
  toolDescription        string                          shown when this prompt is exposed as a tool
  prompt                 string | PromptContent[]        system prompt (string, or composable parts)
  model                  ModelName                       references agents/models/<name>
  includeChat            boolean (default false)         pass full chat history to this LLM step
  includePastTools       boolean (default false)         pass past tool call results
  parallelToolCalls      boolean (default false)         allow multiple tool calls per turn
  toolChoice             'auto' | 'none' | 'required'    tool calling strategy (default 'auto')
  requiredSchema         ZodSchema                       validate args when called as a tool
  variables              VariableDefinition[]            declared text/secret variables
  tools                  (string | SubpromptConfig | PromptToolConfig | SubagentToolConfig)[]
                                                         tools available at this step
  env                    Record<string, string>          prompt-level env values (lowest precedence)
  reasoning              ReasoningConfig                 extended thinking config (for models that support it)
  recentImageThreshold   number (default 10)             how many recent messages keep real images
  providerOptions        Record<string, unknown>         passthrough to the provider (overrides model defaults)
  hooks                  HookId[]                        prompt-scoped hooks (overrides agent hooks)
```

### Composable prompts: the `tone` pattern

`prompt` can be a string or an array of parts. Use parts to compose a shared "tone" or "persona" across many prompts so changes flow through one place.

```ts
const tonePrompt: PromptDefinition = {
  name: 'company_tone',
  toolDescription: 'Defines the tone and style of the chatbot.',
  prompt: `You are a friendly and helpful customer support assistant for an athletic shoe company. You always respond in a positive and upbeat tone, even when the customer is upset. You use simple language and avoid technical jargon.`,
  model: 'tiny',
};

const shippingPrompt: PromptDefinition = {
  name: 'shipping_inquiries',
  toolDescription: 'Handles customer questions about shipping.',
  prompt: [
    { type: 'include', prompt: 'company_tone' },
    { type: 'text', content: 'Details about shipping policies: ...' },
  ],
  model: 'tiny',
};
```

Use `{ type: 'env', property: 'PRODUCT_INVENTORY' }` parts to inject runtime values into the prompt. Combined with `variables`, this lets a generic agent be specialized per-thread without code changes:

```ts
{
  name: 'ecommerce_assistant',
  toolDescription: 'Assistant for ecommerce businesses.',
  prompt: [
    { type: 'text', content: 'You help customers with products and orders. Current inventory: ' },
    { type: 'env', property: 'PRODUCT_INVENTORY' },
  ],
  model: 'tiny',
  variables: [
    {
      name: 'PRODUCT_INVENTORY',
      type: 'text',
      required: true,
      description: 'Full product inventory with names, descriptions, and stock levels.',
    },
  ],
}
```

### `AgentDefinition` cheat sheet

The agent definition binds prompts and sides together. Set on each file in `agents/agents/`. For full signatures, see `agents/agents/AGENTS.md`.

```
AgentDefinition
  name              string                       unique snake_case identifier
  type              'ai_human' | 'dual_ai'       default 'ai_human' — but see "dual_ai is the default"
  maxSessionTurns   number                       REQUIRED for dual_ai. Finite turn cap.
  sideA             SideConfig                   AI side (or first AI in dual_ai)
  sideB             SideConfig                   second AI side; required for dual_ai
  exposeAsTool      boolean (default false)      enables this agent to be called as a tool by other prompts
  toolDescription   string                       required if exposeAsTool: true
  description       string                       brief human description for UIs
  icon              string                       URL or absolute path
  env               Record<string, string>       agent-level env values
  hooks             HookId[]                     agent-scoped hooks (when prompts don't specify their own)
```

`SideConfig` is where you bind `defaultPrompt`, `defaultModel`, and the **session lifecycle bindings**:

- `sessionStop` — name of the tool whose call ends the session with a result
- `sessionFail` — name of the tool whose call ends the session with a failure
- `sessionStatus` — optional, a tool used to update the session status mid-run

These are the bindings the [Session boundary discipline](#session-boundary-discipline) section refers to. Every `dual_ai` agent must set them.

Packaging fields (`packageName`, `version`, `author`, `license`) exist for the agent packing system; ignore them unless you're publishing.

### `SubagentToolConfig` cheat sheet

This is where parent/child architecture is actually configured — it lives on the *parent prompt's* `tools` array. Knowing every field exists is essential; full signatures live in the spec types (`node_modules/@standardagents/spec/dist/`, or `packages/spec/src/` on GitHub).

```
SubagentToolConfig — entry on a parent prompt's `tools` array
  name                       string                      dual_ai agent to invoke (must have exposeAsTool: true)
  blocking                   boolean (default true)      parent waits for result, vs. fire-and-forget
  immediate                  bool | object               spawn child when prompt activates, before any LLM step
                                                         object form: { nameEnv, descriptionEnv, scopedEnv }
                                                         scopedEnv values are copied into the child but NOT
                                                         exposed to the bootstrap model unless named in
                                                         nameEnv/descriptionEnv
  resumable                  false | object              false = tool-call style (default)
                                                         object = long-lived addressable child; runtime
                                                         exposes built-in subagent_create / subagent_message
    receives_messages          'side_a' | 'side_b'       which side hears parent messages
                                                         side_a = queued as 'user'
                                                         side_b = queued as 'assistant'
    maxInstances               number                    cap on concurrent instances; when reached the tool
                                                         is hidden and new messages route to existing instances
    parentCommunication        'implicit' | 'explicit'   implicit (default) auto-queues completion to parent
                                                         explicit requires state.notifyParent() in code
  initUserMessageProperty    schema field name           tool arg used as child's first user message
  initAttachmentsProperty    schema field name           tool arg holding /attachments/* paths to copy to child
  initAgentNameProperty      schema field name           tool arg used to tag the child thread (UI label)
  optional                   string (env name)           subagent disabled unless this env resolves truthy
```

A second tool-config form exists for plain callables (`{ name, env, options }`), and a third for subprompts (`{ name, includeTextResponse, includeToolCalls, includeErrors, initUserMessageProperty, initAttachmentsProperty }`). Both are documented in full at `agents/prompts/AGENTS.md`.

### Inter-agent communication rules

These are correct as written in the spec — internalize them.

1. **Parents always create children.**
   - Explicitly via the built-in `subagent_create` tool, which requires a non-empty `name` for the child instance.
   - Implicitly via `immediate: { ... }` — the child spawns the moment the parent prompt activates, before any LLM step.
2. **Children only communicate back to their parents.** Two flavors:
   - **Implicit**: the child auto-queues a message to the parent when the session ends (via `sessionStop`, `sessionFail`, or `maxSessionTurns`). Default for all subagents.
   - **Explicit**: only when `state.notifyParent()` is called. Set with `resumable.parentCommunication: 'explicit'`. Use for high-traffic resumables (e.g., a Slack monitor) where you want control over when the parent is interrupted.
3. **Resumable subagents can receive messages from their parents.** `resumable.receives_messages` chooses which side hears them.
4. **When a child sends back to the parent, it MUST indicate whether it needs a response** — in plain English, e.g. "After researching, you must respond so I can continue with the email." A research subagent providing FYI info doesn't need a response; a gmail subagent waiting on guidance does.
5. **File attachments use path strings.** `/attachments/{filename}.{ext}`. Use this convention everywhere — the runtime copies attachments across thread filesystems automatically when listed in `initAttachmentsProperty`.
6. **Resumable agents communicate via "silent" user messages** that carry the source instance UUID. The receiving agent decides whether to respond or just absorb the information.
7. **Messages to busy agents are queued** and delivered when the agent is free.
8. **Prompt language must distinguish `subagent_create` from `subagent_message`.** When writing prompts that include resumable subagents, explicitly tell the model: *"To research a new topic, use `subagent_create`. To follow up on a topic an existing instance is already researching, use `subagent_message`."* Without this guidance, models will pick wrong.

---

## Hooks

Hooks extend agent behavior without modifying core logic. Defined via `defineHook`, referenced by `id` from agent or prompt definitions. Common uses: truncating history, injecting synthetic tool calls (e.g., real-time clock awareness), logging, and adapting tool results.

| Hook | Execution Point | Purpose |
|---|---|---|
| `after_thread_created` | After thread creation, before execution | Initialize thread state |
| `after_subagent_created` | On parent after child thread creation | Track or initialize child relationships |
| `after_system_message` | After system message render | Transform dynamic system instructions |
| `filter_messages` | Before LLM context assembly | Filter/transform message history |
| `prefilter_llm_history` | After context assembly | Final adjustments before LLM request |
| `before_create_message` | Before message insert | Transform message before storage |
| `after_create_message` | After message insert | Side effects after storage |
| `before_update_message` | Before message update | Transform update data |
| `after_update_message` | After message update | Side effects after update |
| `before_store_tool_result` | Before tool result storage | Transform tool results |
| `after_tool_call_success` | After successful tool call | Post-process success results |
| `after_tool_call_failure` | After failed tool call | Handle/recover from errors |

> **Caution:** Hooks that filter or truncate messages must keep matching tool calls and tool results together. Separating them produces hard-to-debug failures in many models.

## Variables and environment

Variables let tools, prompts, and agents declare dynamic values they need. Declare them on prompts, tools, and agents with `variables`:

```ts
variables: [
  {
    name: 'LOCATION',
    type: 'text',
    required: true,
    description: 'City or ZIP code to use for weather lookups.',
  },
  {
    name: 'WEATHER_API_KEY',
    type: 'secret',
    required: true,
    description: 'Weather API credential used only inside tools.',
  },
];
```

Two value types are supported:

- **`text`** — simple string. Safe to render in prompts.
- **`secret`** — encrypted; **MUST only be used inside tools**. Never reference a secret in prompt text and never return it to the model. A `GMAIL_API_KEY` is `secret`; a `LOCATION` is `text`.

When a thread is created, all required variables in the agent graph must be provided. The instance of a variable on a thread is called an "environment variable" or `env`. Resolution precedence (low → high): prompt → tool → agent → thread.

Scoped variables (`scoped: true`) do not inherit from parent thread env — they reset for the declaring agent's subtree. Use this when a subagent must run with different config from its parent (e.g., a per-instance Slack channel ID).

Thread env values are editable from the thread metadata UI. `ThreadState.setEnv(name, "")` intentionally creates a blank thread env entry that still appears in that UI. Prefer the explicit form when writing new values:

```ts
await state.setEnv('LOCATION', 'Charlottesville, VA', { type: 'text' });
await state.setEnv('WEATHER_API_KEY', token, { type: 'secret' });
```

In code, read values through `await state.env('NAME')`, check display policy with `await state.envType('NAME')`, and write thread-scoped values with `await state.setEnv('NAME', value, { type: 'text' | 'secret' })`. Use `text` only for values that are safe in prompts, tool output, logs, and errors. Use `secret` for tokens, API keys, credentials, and anything that should be redacted.

Undeclared thread-only env keys are treated as write-only secrets when scanned for the UI, so the key is visible and editable but arbitrary stored values are not echoed back.

---

## Implementation checking

When you edit `agents/`, `prompts/`, `models/`, `tools/`, `hooks/`, or `worker/`, validate before claiming the change is done.

Validation order:

1. Read `package.json` and prefer project scripts if they exist.
2. Refresh Cloudflare types:
   - use `pnpm cf-typegen` if that script exists
   - otherwise run `npx wrangler types`
3. Regenerate AgentBuilder types by running the project's build command:
   - usually `pnpm build`, `npm run build`, `pnpm run dev`, or equivalent
4. Run the project's type checker:
   - prefer `pnpm type-check` / `npm run type-check` if present
   - otherwise use the installed checker directly: `pnpm exec vue-tsc --build`, `pnpm exec tsc -b`, or `pnpm exec tsc --noEmit`
5. If any validation step cannot run, state exactly what is missing.
