# Sentry beyond the CLI: AI Agent Monitoring + Seer auto-PRs

The sentry-cli skill is for inspection and triage. This file covers the
two adjacent setups that come up when a user asks for "Sentry for my app"
or "Sentry for my agent" — both are SDK + dashboard configuration, not
CLI work, but they share the same org/project context.

## 1. App error monitoring with Seer auto-PRs (the self-healing pattern)

User asks for "self-healing" / "auto-fix PRs" → do NOT build a cron +
sentry-cli wrapper first. Sentry already ships this as a native feature.

Setup order:
1. Install the framework SDK (SvelteKit example: `npx @sentry/wizard@latest -i sveltekit`).
   The wizard writes `hooks.client.ts`, `hooks.server.ts`, the Vite
   plugin, and source-map upload config.
2. Verify capture with a synthetic throw before touching automation.
3. Sentry → Settings → Integrations → GitHub → connect, grant repo access.
4. Project → Settings → Source Maps + Code Mappings (stack frame path →
   repo path). Seer cannot open useful PRs without code mappings.
5. Project → Automation (or Seer settings) → enable "Automatically open
   PRs for fixes". Gate by issue criteria (event count, path prefix,
   level) so you don't get a PR for every transient.

Only reach for a cron + `sentry issue plan` wrapper when the native
automation isn't expressive enough (e.g. you want fixes routed through
a Hermes subagent, or you want a Telegram digest of pending Seer PRs
instead of GitHub notifications). Run the native loop for a few weeks
first and let real data tell you if the wrapper is worth building.

Hard rule: closed-loop autonomy ends at PR. Never wire auto-merge for
Seer fixes — human review is the safety bar.

## 2. AI Agent Monitoring (Sentry's gen_ai dashboards)

Docs: https://docs.sentry.io/ai/monitoring/agents/

OpenTelemetry-based, auto-instruments OpenAI / Anthropic / others. For
Python:

```python
import sentry_sdk
from sentry_sdk.integrations.anthropic import AnthropicIntegration
from sentry_sdk.integrations.openai import OpenAIIntegration

sentry_sdk.init(
    dsn=...,
    traces_sample_rate=1.0,
    send_default_pii=False,  # see PII section below
    integrations=[AnthropicIntegration(), OpenAIIntegration()],
)
```

Per-run tagging so the dashboard groups usefully — apply BEFORE the
top-level transaction:

```python
sentry_sdk.set_tag("hermes.profile", active_profile_name)
sentry_sdk.set_tag("hermes.model", model)
sentry_sdk.set_tag("hermes.provider", provider)
with sentry_sdk.start_transaction(op="gen_ai.agent",
                                  name=f"hermes:{profile}"):
    ...
```

Transaction name drives the agent-name grouping in the AI dashboard.
For Hermes specifically, use `hermes:<profile>` so each profile
(default, syntax, courtney-personal-assistant, phases-podcast, etc.)
gets its own lane — that's what the tuning data hangs off.

### PII / prompt content trade-off

`send_default_pii=True` ships prompt + completion content to Sentry.
That's the useful tuning data (you can actually see what went wrong),
but it also means user messages and any injected context (Honcho
memory, RAG chunks) leave the box. Default to OFF for any profile that
handles personal/private data; turn ON only for profiles where the
content is fair game (your own default profile, dev/test profiles).
Make the call per profile, not globally.

### When NOT to wire Seer auto-PRs on an agent runtime

Agent errors are mostly tool failures, context overflows, provider
rate limits, malformed model output. None of those have a "missing
null check" shape that Seer can patch. The monitoring data is the
lever — humans read it and tune prompts / tool defs / provider
choices. Seer auto-PRs belong on the *application* code, not on the
agent harness.
