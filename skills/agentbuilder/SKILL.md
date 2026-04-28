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

The effective agents guide is a living record that is not part of the specification, but intended to help humans and agents use the specification to craft effective agents. It is updated regularly and not bound to any specific version of the specification.

Note: in this guide we occasionally discuss commands like `npmx` or `pnpm` — use whatever package manager the user prefers, these are just placeholders for the correct command.

## What is a Standard Agent?

In the Standard Agent paradigm agents are the atomic unit of an AI system and it is the composition of many domain-specific agents that produces efficacy. How large should the concerns of a given agent be? There is no hard or fast rule here, the question is a bit like how big should a React component be. However, in general Standard Agents can be effective using smaller and cheaper models — but smaller and cheaper models typically suffer from poor tool discernment when presented with a broad range of tools, especially a broad range of similar tools.

A "gmail" agent, for example, will typically do better than a "google apps" agent. Those higher-level agents would instead be composed of many sub-service agents, and a larger coordinator agent would be the one with the primary objectives, business goals and logic, and ultimate authority. This is a fractal that can be scaled up to create even larger coordinators and larger agents. These agent-graphs, or agent-trees, are the core philosophy of Standard Agents and solves many industry problems such as progressive tool discovery, model-prompt tuning, context dilution, task resumability, and even compaction.

However, just because we can use subagents to create complex interactions doesn't mean we have to. A subagent is always a two-sided conversation where each side can perform multiple steps per turn. Sometimes you just need to use a feature of a different model, without it being a full subagent, for example using nano bananna to generate an image. If you just need to have a tool call that generates an image, use a subprompt. You may want an asset generator subagent if, for example, the agent needs to perform QA on the generated asset to ensure it meets certain criteria.

Another example may be using an instruct model variant to run an agent, but a more eloquent model to write the actual text of the email that is sent to people. Again, that could be a subprompt. Having another model proofread the email that is written — that would be a subagent. One small note on that: when performing QA in a subthread, try to use different models from different providers on the agent side that is doing the proofreading. Models from the same labs tend to think their own output is quite good, even if it is not.

Example structure:

- `personal_assistant_coordinator`
  - central dual-ai coordinator
  - decides which domain owns the work
  - aggregates state, plans, and responses
  - has subprompts for producing images, or using a more eloquent model for writing
- `communications_coordinator`
  - owns all inbound and outbound communications work
  - delegates to `gmail_agent`, `slack_agent`, and `text_message_agent`
  - receives escalations from those channel agents and routes the next action
- `research_assistant`
  - owns information gathering and synthesis
  - delegates to `browser_use_agent`, `google_drive_agent`, and `notes_agent`
- optional additional branches
  - `scheduling_coordinator`
  - `travel_coordinator`
  - `finance_ops_coordinator`
  - `crm_coordinator`

## Standard Agent Stack

The standard agent spec defines exactly what an agent is made up of:

- Providers (define how to interact with different LLM providers, and different model variants.)
- Prompts (system instructions, and how to use tools, subagents, and subprompts)
- Tools (define the functions that an agent can call to interact with the world, and other agents)
- Agents (define the agent itself, how it uses prompts and tools)
- Hooks (define custom code that can perform various internal operations such as modifying the prompt or changing the message history)
- Effects (custom code that executes at a certain time)
- Threads (the instance of a given agent, with its own state, message history, and filesystem)
- Endpoints (built-in and custom endpoints exposed by an agent thread)

Each of these is defined by the Standard Agent Specification.

## Models

The standard agent spec is model-agnostic, but it is important to understand how to use different models effectively within the paradigm. The most effective agents use a combination of smaller and larger models. The smallest model that produces sufficiently good results is always the best choice. Smaller models are cheaper, faster, and more likely to run on local hardware (eventually). However smaller models often struggle with discernment, coordination, and higher order reasoning like business objectives. So typically we'll want to use larger models for coordinators, and smaller models for discrete subagents.

Different models are good at different things. The Standard Agents organization has identified a number of well-suited models for various tasks. To get the latest curated list run:

```bash
pnpm exec agents current-models
```

When writing an agent, use `current-models` to choose a model that fits the use case, then use `available-models` to check the exact model string your installed provider exposes. If you have more than one configured provider, pass `--provider=<name>`.

```bash
pnpm exec agents available-models --provider=openai
```

Note: the `name` of a model should indicate the use case for the model, for example: `extra_reasoning`, or `fast_tool_calls` or `image_generation`. Fallback models can be created for similar use cases, and should typically use different providers to increase the chances of at least one model producing good results. For example, you may have `extra_reasoning` models from both OpenAI and Anthropic.

### Prompts

Prompts define what actually gets sent to the LLM. They encompass model instructions, tool definitions, subprompts, and subagent tools, history inclusion, model selection, and more. The typescript definition of a prompt is as follows:

```ts
export interface PromptDefinition<
  N extends string = string,
  S extends ToolArgs = ToolArgs,
> {
  /**
   * Unique name for this prompt.
   * Used as the identifier when referencing from agents or as a tool.
   * Should be snake_case (e.g., 'customer_support', 'data_analyst').
   */
  name: N;

  /**
   * Description shown when this prompt is exposed as a tool.
   * Should clearly describe what this prompt does for LLM tool selection.
   */
  toolDescription: string;

  /**
   * The system prompt content sent to the LLM.
   * Can be either a plain string or a structured array for composition.
   */
  prompt: PromptContent;

  /**
   * Model to use for this prompt.
   * Must reference a model defined in agents/models/.
   */
  model: StandardAgentSpec.Models;

  /**
   * Include full chat history in the LLM context.
   * @default false
   */
  includeChat?: boolean;

  /**
   * Include results from past tool calls in the LLM context.
   * @default false
   */
  includePastTools?: boolean;

  /**
   * Allow parallel execution of multiple tool calls.
   * @default false
   */
  parallelToolCalls?: boolean;

  /**
   * Tool calling strategy for the LLM.
   *
   * - `auto`: Model decides when to call tools (default)
   * - `none`: Disable tool calling entirely
   * - `required`: Force the model to call at least one tool
   *
   * @default 'auto'
   */
  toolChoice?: 'auto' | 'none' | 'required';

  /**
   * Zod schema for validating inputs when this prompt is called as a tool.
   */
  requiredSchema?: S;

  /**
   * Declared variables for this prompt.
   */
  variables?: VariableDefinition[];

  /**
   * Tools available to this prompt.
   * Can be:
   * - string: Simple tool name (custom or provider tool)
   * - SubpromptConfig: Sub-prompt used as a tool
   * - PromptToolConfig: Tool with environment values and/or options
   * - SubagentToolConfig: `dual_ai` subagent invocation behavior
   *
   * To enable handoffs, include ai_human agent names in this array.
   *
   * @example
   * ```typescript
   * tools: [
   *   'custom_tool',                                    // Simple tool name
   *   { name: 'other_prompt' },                         // Sub-prompt as tool
   *   { name: 'file_search', env: { VECTOR_STORE_ID: 'vs_123' } },  // Tool with env values
   * ]
   * ```
   */
  tools?: (
    | StandardAgentSpec.Callables
    | SubpromptConfig
    | PromptToolConfig
    | SubagentToolConfig
  )[];

  /**
   * Environment values provided by this prompt.
   * Prompt values are the lowest-precedence source in runtime resolution.
   */
  env?: Record<string, string>;

  /**
   * Reasoning configuration for models that support extended thinking.
   */
  reasoning?: ReasoningConfig;

  /**
   * Number of recent messages to keep actual images for in context.
   * @default 10
   */
  recentImageThreshold?: number;

  /**
   * Provider-specific options passed through to the provider.
   * These override model-level providerOptions for this prompt.
   *
   * Options are merged in order (later wins):
   * 1. model.providerOptions (defaults)
   * 2. prompt.providerOptions (this field - overrides)
   *
   * @example
   * ```typescript
   * providerOptions: {
   *   response_format: { type: 'json_object' },
   * }
   * ```
   */
  providerOptions?: Record<string, unknown>;

  /**
   * Hook IDs to run when this prompt is active.
   * References hooks by their unique `id` property from defineHook().
   * If not specified, falls back to agent-level hooks.
   *
   * @example
   * ```typescript
   * hooks: ['limit_to_20_messages', 'log_tool_calls']
   * ```
   */
  hooks?: StandardAgentSpec.HookIds[];
}
```

The `prompt` property is the system message for that LLM step. It is defined as a string or array of parts. If defined with parts, the prompt is rendered before being sent to the LLM and can contain dynamic content. For example, let's say our agent is powering a chatbot for a website that sells athletic shoes. We want the chatbot to be able to answer questions about the shoes, billing, shipping etc. To do this, we have many prompts, but we always want to use the same friendly and helpful tone.

To share "tone" prompt that just defines the tone and style of the chatbot, and then reference that prompt in all our other prompts as a part. This allows us to change the tone of the chatbot across all its functions by just changing one prompt.

```ts
const tonePrompt: PromptDefinition = {
  name: 'company_tone',
  toolDescription: 'Defines the tone and style of the chatbot.',
  prompt: `You are a friendly and helpful customer support assistant for an athletic shoe company. You always respond in a positive and upbeat tone, even when the customer is upset. You use simple language and avoid technical jargon. Your goal is to help the customer with their issue and make them feel good about shopping with us.`,
  model: 'tiny_model',
};
```

Then in another prompt we can reference that tone prompt:

```ts
const shippingPrompt: PromptDefinition = {
  name: 'shipping_inquiries',
  toolDescription: 'Handles customer questions about shipping.',
  prompt: [
    { type: 'include', prompt: 'company_tone' }, // Reference the tone prompt as a part
    { type: 'text', content: `Details about the products:...` }
  ],
  model: 'tiny_model',
};
```

Additionally, we can use the `env` property to provide dynamic content that can be referenced in the prompt. This can be especially helpful for packed agents that get distributed to other people, businesses, or industries. For example, we could create a generic ecommerce assistant agent that uses a `PRODUCT_INVENTORY` env to customize the agent to the specific products of the business that is using it.

```ts
{
// ...
  name: 'ecommerce_assistant',
  toolDescription: 'An assistant for ecommerce businesses that can answer questions about products, inventory, and orders.',
  prompt: [
    { type: 'text', content: `You are an assistant for an ecommerce business. You help customers with their questions about products, inventory, and orders. Here is the current product inventory: ` },
    { type: 'env', property: 'PRODUCT_INVENTORY' }, // Reference the PRODUCT_INVENTORY env variable
  ],
  model: 'tiny_model',
  variables: [
    {
      /** Environment variable/property name */
      name: 'PRODUCT_INVENTORY',
      /** Value type: 'text' or 'secret' */
      type: 'text',
      /** Whether this variable is required to execute */
      required: true;
      /**
      * Whether this variable is scoped to the declarer agent subtree.
      *
      * Scoped variables do not inherit parent thread env values. Descendants of
      * the declarer still inherit scoped values from that declarer thread.
      *
      * @default false
      */
      scoped: false;
      /** Human-readable description (empty string when not provided) */
      description: 'The full inventory of products, including names, descriptions, and stock levels.',
    }
  ]
}
```

## Tools

Tools are a generic term for anything an agent can "call". In practice there are 3 types of tools:

1. Callables: these are TypeScript functions that are defined via `defineTool` in the `tools` directory. These should be created as the primary mechanism for interfacing with the outside world, interacting with APIs, database, business logic, and often even other agents. Each callable receives a `ThreadState` object. You should not assume that the function is being executed on a node server; it may be an edge function, for example, so do not use node imports if possible. Instead use the `ThreadState` to perform any necessary operations (see the ThreadState documentation for more details and examples).

2. Subprompts: these are prompts that can be called as tools. They are defined in the `prompts` directory and have a `toolDescription` property that indicates they can be used as tools, and are useful for switching models for a small task, like writing some text, or generating an image.

3. Subagents: these are full agents that can be called as tools. They are defined in the `agents` directory and have `exposeAsTool: true` in their agent definition. Subagents are useful when you need a more complex interaction that requires multiple steps, or when you want to leverage the unique capabilities of a different model that is better suited for a specific task.

There are lots of options for how tools are defined on their respective prompt:

```ts
{
  // ...
  tools: {
    /**
    * Agent callable name.
    *
    * Must reference a `dual_ai` agent with `exposeAsTool: true`.
    */
    name: T;

    /**
    * Whether parent execution blocks until the subagent returns a result.
    *
    * - `true`: Parent waits for completion (tool-call style)
    * - `false`: Parent continues immediately and receives results asynchronously
    *
    * @default true
    */
    blocking?: boolean;

    /**
    * Property from tool-call arguments used as the initial message sent to the
    * subagent on invocation.
    *
    * Uses the same semantics as {@link SubpromptConfig.initUserMessageProperty}.
    */
    initUserMessageProperty?: StandardAgentSpec.SchemaFields<T>;

    /**
    * Property from tool-call arguments containing attachment path(s) that should
    * be sent to the subagent on invocation.
    *
    * Uses the same semantics as {@link SubpromptConfig.initAttachmentsProperty}.
    */
    initAttachmentsProperty?: StandardAgentSpec.SchemaFields<T>;

    /**
    * Property from tool-call arguments used to assign a human-readable name for
    * each spawned child thread instance.
    *
    * Implementations SHOULD store this as a thread tag in the form
    * `name:<value>` so UIs can render a concise per-instance title.
    */
    initAgentNameProperty?: StandardAgentSpec.SchemaFields<T>;

    /**
    * Execute this tool immediately when the prompt becomes active.
    *
    * - `true`: Execute immediately using runtime defaults.
    * - Object: Execute immediately with explicit per-instance env relationships.
    *
    * When the object form is used:
    * - `scopedEnv` names the per-instance env values copied into the child thread.
    * - `nameEnv` and `descriptionEnv` identify the only per-instance env values
    *   that runtimes may expose to an internal bootstrap model when deriving
    *   initial child arguments.
    *
    * Runtimes MUST NOT expose `scopedEnv` values to the model unless the same env
    * name is explicitly designated by `nameEnv` or `descriptionEnv`.
    *
    * Immediate tools run before the first LLM step for that activation.
    */
    immediate?:
      | boolean
      | {
          /**
          * Scoped env name whose value may be used as the safe per-instance name
          * hint for child bootstrap.
          */
          nameEnv?: string;

          /**
          * Scoped env name whose value may be used as the safe per-instance
          * description hint for child bootstrap.
          */
          descriptionEnv?: string;

          /**
          * Scoped env names that should be copied into the child thread for each
          * immediate instance group.
          */
          scopedEnv?: string[];
        };

    /**
    * Optional branch flag env name.
    *
    * When set, this subagent is only enabled when the named env resolves to
    * `true`, `1`, or `yes` (case-insensitive).
    */
    optional?: string;

    /**
    * Resumability configuration.
    *
    * - `false` (default): Non-resumable subagent
    * - Object: Resumable subagent with message routing and instance limits
    *
    * When resumable mode is enabled, runtimes SHOULD provide a built-in create
    * and message lifecycle interface instead of exposing raw agent callables for
    * new instance creation.
    */
    resumable?:
      | false
      | {
          /**
          * Which side of the child `dual_ai` conversation receives parent messages.
          *
          * - `side_a`: Messages are queued as `role: 'user'`
          * - `side_b`: Messages are queued as `role: 'assistant'`
          */
          receives_messages: 'side_a' | 'side_b';

          /**
          * Maximum concurrent instances for this subagent tool.
          *
          * When reached, implementations may remove this tool from subsequent LLM
          * requests and route new messages to existing instances.
          *
          * @default unlimited
          */
          maxInstances?: number;

          /**
          * How this child reports back to its parent.
          *
          * - `implicit` (default): Child completion is automatically queued to the parent.
          * - `explicit`: The runtime does not auto-queue child completion; tools/hooks may
          *   use thread APIs such as `state.notifyParent()` when they choose to escalate.
          */
          parentCommunication?: 'implicit' | 'explicit';
        };
    } |
    {
        /**
        * Name of the tool (custom tool or provider tool).
        */
        name: StandardAgentSpec.Callables;

        /**
        * Environment variable values for this tool.
        */
        env?: Record<string, string>;
        /**
        * @deprecated Use `env` instead.
        */
        tenvs?: Record<string, unknown>;

        /**
        * Static options for this tool.
        * Passed to the tool handler at execution time.
        */
        options?: Record<string, unknown>;
      } | {
      /**
      * Name of the sub-prompt or agent to call.
      * Must be a prompt defined in agents/prompts/ or an agent in agents/agents/.
      */
      name: T;

      /**
      * Include text response content from sub-prompt execution in the result string.
      * @default true
      */
      includeTextResponse?: boolean;

      /**
      * Serialize tool calls made by the sub-prompt (and their results) into the result string.
      * @default true
      */
      includeToolCalls?: boolean;

      /**
      * Serialize any errors from the sub-prompt into the result string.
      * @default true
      */
      includeErrors?: boolean;

      /**
      * Property from the tool call arguments to use as the initial user message
      * when invoking the sub-prompt or agent.
      *
      * Autocompletes to fields from the prompt's requiredSchema (or agent's side_a prompt schema).
      *
      * @example
      * If the tool is called with `{ query: "search term", limit: 10 }` and
      * `initUserMessageProperty: 'query'`, the sub-prompt will receive
      * "search term" as the initial user message.
      */
      initUserMessageProperty?: StandardAgentSpec.SchemaFields<T>;

      /**
      * Property containing attachment path(s) to include as multimodal content
      * when invoking the sub-prompt or agent.
      *
      * Autocompletes to fields from the prompt's requiredSchema (or agent's side_a prompt schema).
      * Supports both a single path string or an array of paths.
      *
      * @example
      * If the tool is called with `{ image: "/attachments/123.jpg" }` and
      * `initAttachmentsProperty: 'image'`, the sub-prompt will receive
      * the image as an attachment in the user message.
      *
      * @example
      * If the tool is called with `{ images: ["/attachments/a.jpg", "/attachments/b.jpg"] }` and
      * `initAttachmentsProperty: 'images'`, the sub-prompt will receive
      * both images as attachments.
      */
      initAttachmentsProperty?: StandardAgentSpec.SchemaFields<T>;
    } |
    string /* simple tool name with no extra options * /;
}
```

Excellent orchestration is the creation of models definitions, prompts, tools, subprompts, subagents, and agent definitions to tie everything together.


## Subprompts vs Handoffs vs Subagents

A subprompt allows a `prompt` definition to be run with a single step as if it was a tool. The result of the subprompt is returned to the thread as if it is a tool result. Subprompts are a useful way to use a different model for a specific task. For example, an agent may use a subprompt to leverage a better image generation model.

Subprompts can receive the entire context of the parent thread if the prompt definition has `includeChat: true` and `includePastTools: true`. If those are set to false, then the subprompt receives no context at all, and it should use `initUserMessageProperty` and optionally `initAttachmentsProperty` instead. These properties specify what explicit context from the parent thread should be exposed, and their values are provided as if it is a `user` requesting the subprompt to run. The exact text the user provides will be the `initUserMessageProperty` value, and optionally any file attachments specified using the `/attachments/{filename}.{ext}` convention (`initAttachmentsProperty` should be an array of strings).

A subagent on the other hand is a full agent thread, with its own `ThreadState`. Subagents can be:

- Blocking and non-resumable
- Blocking and resumable
- Non-blocking and non-resumable
- Non-blocking and resumable

These distinctions indicate how a subagent appears to its parent agent. A non-resumable subagent is very much like a tool call from the parent's perspective. It calls the "tool" the subagent runs for a while and returns a result. A resumable subagent however, becomes an addressable part of the agent graph, the parent can communicate with it receive results from it, and inspect its status. Resumable subagents can be as long-lived as the parent itself. For example a LinkedIn subagent may be permanently monitoring a particular linked in account and advising the parent coordinator when messages that would be important for the owner are received.
Blocking and non-blocking indicates whether the parent thread waits for the subagent to finish before it continues. A non-blocking subagent allows the parent thread to continue its work while the subagent is running, and the parent can receive messages from the subagent.

A handoff is when an `ai_human` agent is used as a tool by another `ai_human` agent. This is a special case and rather than spawn a new thread, it changes which prompt "owns" that thread — in other words it hands off control from one agent to another. This is very useful in a number of applications where a human is being led through a series of steps, for example, an onboarding agent may collect necessary information from a user, and then "handoff" to another agent which performs a different function, for example, a scheduling agent that books meetings based on the information collected by the onboarding agent.

Subagents should always be `dual_ai`. `dual_ai` agents are fully autonomous and require no input from a human. Use `side_a` and `side_b` to perform reflection and reasoning on the results of the communication. For example if a subagent is tasked with generating assets for a video game, and those assets need to be on a green background, the `side_a` of the subagent can generate the assets, and the `side_b` can review the assets and determine if they meet the criteria. If they do, the `side_b` can communicate back to the parent coordinator via the tool defined as the `sessionStop` tool. If they do not meet the criteria, `side_b` can inform `side_a` in which ways the image needs to change before it can be approved.

The following is the official shape of the agent definition by which these kinds of interactions can be configured:

```ts
export interface AgentDefinition<
  N extends string = string,
  Prompt extends string = StandardAgentSpec.Prompts,
  Callable extends string = StandardAgentSpec.Callables,
> {
  /**
   * Unique name for this agent.
   * Used as the identifier for thread creation and handoffs.
   * Should be snake_case (e.g., 'support_agent', 'research_flow').
   */
  name: N;

  /**
   * Agent conversation type.
   *
   * - `ai_human`: AI conversing with a human user (default)
   * - `dual_ai`: Two AI participants conversing
   *
   * @default 'ai_human'
   */
  type?: AgentType;

  /**
   * Maximum total turns across both sides.
   * Only applies to `dual_ai` agents.
   * Prevents infinite loops in AI-to-AI conversations.
   */
  maxSessionTurns?: number;

  /**
   * Configuration for Side A.
   * For `ai_human`: This is the AI side.
   * For `dual_ai`: This is the first AI participant.
   */
  sideA: SideConfig<Prompt, Callable>;

  /**
   * Configuration for Side B.
   * For `ai_human`: Optional, the human side doesn't need config.
   * For `dual_ai`: Required, the second AI participant.
   */
  sideB?: SideConfig<Prompt, Callable>;

  /**
   * Expose this agent as a tool for other prompts.
   * Enables agent composition and handoffs.
   * When true, other prompts can invoke this agent as a tool.
   * @default false
   */
  exposeAsTool?: boolean;

  /**
   * Description shown when agent is used as a tool.
   * Required if exposeAsTool is true.
   * Should clearly describe what this agent does.
   */
  toolDescription?: string;

  /**
   * Brief description of what this agent does.
   * Useful for UIs and documentation.
   *
   * @example 'Handles customer support inquiries and resolves issues'
   */
  description?: string;

  /**
   * Icon URL or absolute path for the agent.
   * Absolute paths (starting with `/`) are converted to full URLs in API responses.
   *
   * @example 'https://example.com/icon.svg' or '/icons/support.svg'
   */
  icon?: string;

  /**
   * Environment values provided by this agent.
   * Agent values are lower priority than thread/account/instance values.
   */
  env?: Record<string, string>;

  // ============================================================================
  // Package Metadata (for packing/unpacking)
  // ============================================================================

  /**
   * npm package name for this agent when packed.
   * Used by the packing system to maintain consistent package identity
   * across pack/unpack cycles.
   *
   * @example 'standardagent-support-agent', '@myorg/support-agent'
   */
  packageName?: string;

  /**
   * Package version (semver format).
   * Used by the packing system to track versions across pack/unpack cycles.
   * When re-packing, this version is auto-incremented by the pack modal.
   *
   * @example '1.0.0', '2.3.1-beta.1'
   */
  version?: string;

  /**
   * Package author/copyright holder.
   * Used by the packing system for the LICENSE file and package.json author field.
   *
   * @example 'John Doe', 'Acme Corp'
   */
  author?: string;

  /**
   * License identifier (SPDX format).
   * Used by the packing system for LICENSE file generation.
   *
   * @example 'MIT', 'Apache-2.0', 'ISC'
   */
  license?: string;

  /**
   * Hook IDs to run for this agent.
   * References hooks by their unique `id` property from defineHook().
   * These run when prompts don't specify their own hooks.
   *
   * @example
   * ```typescript
   * hooks: ['log_messages', 'track_tool_usage']
   * ```
   */
  hooks?: StandardAgentSpec.HookIds[];
}
```

## Receiving and sending input

Within an agent graph, input can come directly via a user, a tool call, or even subagents. In its simplest form, you may have an ai+human agent at the top of the graph, although this works well in simple cases, it's often preferable to have the highest order agent be a high-level coordinator that uses a more intelligent discerning model and is guided by prompting to achieve high-level objectives (ex: "You are  project manager for the Jichael Mordan line of shoes at an athletic shoe company. You are in charge of...").

It is quite possible to have an entire agent graph with no `ai_human` agent types, where the only i/o with a human is through a subagent that performs tool calls to, for example, Slack. New messages are queued and posted by `side_b` as "Message from user: "Mow the lawn clanker", and responses would be sent via a tool call send_to_slack(channel, "I don't know how to do that"). However, for a chat-bot-like implementation it's often preferable to use `ai_human` as the top-level coordinator.

## Inter-agent communication

A standard agent will often be communicating with many other agents. In almost every situation the agent graph is a tree, meaning each agent can have children, and can have parents. There are a few options that are important architectural decisions when creating these parent/child relationships. The shape of the options is this:

```ts
interface SubagentToolConfig<T extends string = StandardAgentSpec.Callables> {
    /**
     * Agent callable name.
     *
     * Must reference a `dual_ai` agent with `exposeAsTool: true`.
     */
    name: T;
    /**
     * Whether parent execution blocks until the subagent returns a result.
     *
     * - `true`: Parent waits for completion (tool-call style)
     * - `false`: Parent continues immediately and receives results asynchronously
     *
     * @default true
     */
    blocking?: boolean;
    /**
     * Property from tool-call arguments used as the initial message sent to the
     * subagent on invocation.
     *
     * Uses the same semantics as {@link SubpromptConfig.initUserMessageProperty}.
     */
    initUserMessageProperty?: StandardAgentSpec.SchemaFields<T>;
    /**
     * Property from tool-call arguments containing attachment path(s) that should
     * be sent to the subagent on invocation.
     *
     * Uses the same semantics as {@link SubpromptConfig.initAttachmentsProperty}.
     */
    initAttachmentsProperty?: StandardAgentSpec.SchemaFields<T>;
    /**
     * Property from tool-call arguments used to assign a human-readable name for
     * each spawned child thread instance.
     *
     * Implementations SHOULD store this as a thread tag in the form
     * `name:<value>` so UIs can render a concise per-instance title.
     */
    initAgentNameProperty?: StandardAgentSpec.SchemaFields<T>;
    /**
     * Execute this tool immediately when the prompt becomes active.
     *
     * - `true`: Execute immediately using runtime defaults.
     * - Object: Execute immediately with explicit per-instance env relationships.
     *
     * When the object form is used:
     * - `scopedEnv` names the per-instance env values copied into the child thread.
     * - `nameEnv` and `descriptionEnv` identify the only per-instance env values
     *   that runtimes may expose to an internal bootstrap model when deriving
     *   initial child arguments.
     *
     * Runtimes MUST NOT expose `scopedEnv` values to the model unless the same env
     * name is explicitly designated by `nameEnv` or `descriptionEnv`.
     *
     * Immediate tools run before the first LLM step for that activation.
     */
    immediate?: boolean | {
        /**
         * Scoped env name whose value may be used as the safe per-instance name
         * hint for child bootstrap.
         */
        nameEnv?: string;
        /**
         * Scoped env name whose value may be used as the safe per-instance
         * description hint for child bootstrap.
         */
        descriptionEnv?: string;
        /**
         * Scoped env names that should be copied into the child thread for each
         * immediate instance group.
         */
        scopedEnv?: string[];
    };
    /**
     * Optional branch flag env name.
     *
     * When set, this subagent is only enabled when the named env resolves to
     * `true`, `1`, or `yes` (case-insensitive).
     */
    optional?: string;
    /**
     * Resumability configuration.
     *
     * - `false` (default): Non-resumable subagent
     * - Object: Resumable subagent with message routing and instance limits
     *
     * When resumable mode is enabled, runtimes SHOULD provide a built-in create
     * and message lifecycle interface instead of exposing raw agent callables for
     * new instance creation.
     */
    resumable?: false | {
        /**
         * Which side of the child `dual_ai` conversation receives parent messages.
         *
         * - `side_a`: Messages are queued as `role: 'user'`
         * - `side_b`: Messages are queued as `role: 'assistant'`
         */
        receives_messages: 'side_a' | 'side_b';
        /**
         * Maximum concurrent instances for this subagent tool.
         *
         * When reached, implementations may remove this tool from subsequent LLM
         * requests and route new messages to existing instances.
         *
         * @default unlimited
         */
        maxInstances?: number;
        /**
         * How this child reports back to its parent.
         *
         * - `implicit` (default): Child completion is automatically queued to the parent.
         * - `explicit`: The runtime does not auto-queue child completion; tools/hooks may
         *   use thread APIs such as `state.notifyParent()` when they choose to escalate.
         */
        parentCommunication?: 'implicit' | 'explicit';
    };
}
```

1. Parents always create children.
  1. Parents explicitly create children by calling the tool `subagent_create`.
  2. Parents implicitly create children by having a subagent tool call with `immediate: { ... }` which creates a child thread as soon as the parent thread is activated, without the parent having to explicitly call the tool.
2. Children only communicate back to their parents:
  1. Implicit communication: the child thread automatically queues a message to the parent thread when it ends a "session". A session ends, typically, when one side calls the tools assigned to `sessionStop` or `sessionFail` properties in the agent definition, or the `maxSessionTurns` is reached. All subagents communicate implicitly, unless they are resumable and have `resumable.parentCommunication` set to `explicit`.
  2. Explicit communication: the child thread never communicates with the parent unless `state.notifyParent()` is called. Typically this is done in the tools that are given to the child thread. These calls are independent from session management. Subagents that receive a lot of inbound traffic, for example a Slack subagent, may want to use explicit communication to have more control over when the parent is notified and reduce noise.
3. Resumable subagents can receive messages from their parents. Which side receives the message is indicated by the `resumable.receives_messages`.
4. When sending messages back to the parent, subagents MUST indicate if they require the parent to provide a response or not (in plain english, for example "After researching you must provide a response back so I can continue with sending this email."). For example, a Gmail subagent may ask its parent for guidance on how to respond to an email regarding company policy, the coordinator may ask another subagent with expertise in legal matters and company policy for advice. The legal subagent does not need a response, it is just providing information, but the gmail subagent does require a response in order to proceed. Thus it is critical that the gmail subagent indicates to the parent that it needs a response in order to continue.
5. File attachments are represented by simple strings. Anytime a file is added to a standard agent's filesystem by generation or upload it is given an explicit attachment "path" `/attachments/{filename}.{ext}`. This MUST be used anytime an agent is coordinating with subagents. The tool definition for a subagent can indicate an `initAttachmentsProperty`, which should be an array of strings — if these strings are valid attachments in the parent's file system, then those attachments will be copied into the subagent's file system and attached along with the `initUserMessageProperty` when the subagent is created (note: this also is true for sub-prompts).
6. Resumable agents communicate from one agent to another via "silent" user messages. These messages indicate which agent instance they are from (via uuid) as well as the content of the message. The parent agent can then decide what to do with the message, whether to respond to it, or just use the information in the message to make a decision.
7. If a message is sent to a parent agent or a sub agent that is currently busy, it will be queued and sent when the agent is free.
8. When writing the language for prompts that include resumable subagents, ensure you clearly describe when the subagent should be created (assuming it's not immediate) via `subagent_create` vs when it should just be given a message via `subagent_message`. For example, if you have a research subagent that is researching a given topic, and another subagent needs additional information on that topic, just send a message back to the same resumable subagent, so its existing context can be useful. But if it's requesting information on a totally new topic, then perhaps it should use `subagent_create`.


## Coordinators and the agent graph

The coordinator pattern is akin to the actor model. It can work well when a number of related subagents need to work together. Coordinators provide two significant benefits:

1. Provide subagent communication for similar domains. For example, a coding agent with a `research_agent` and a `bash_agent`. The coordinator could request the `research_agent` research a given operation. Once the `research_agent` completes its task, it communicates back to the coordinator. The coordinator, being satisfied with the results, can then instruct the `bash_agent` to run certain commands and provide the results. In this scenario, the bash subagent didn't even need to know the research subagent existed in order to benefit from it.
2. Filtering. Imagine a `gmail_agent`. On its own it can probably do a pretty good job of filtering out spam. But the `gmail_agent` shouldn't be responsible for the higher-level objectives of an organization. A coordinator that sits above a `gmail_agent` could have directives for what kind of email really matters to the CEO of the sprinkler hose company and can further reduce clutter before it passes email results to it's parent.

The agent graph is critical to get right. The deeper a graph, the more latency is introduced and the more chances for inter-agent communication to fail. However, deep graphs can also allow for better parallelization. A `marketing_communications_agent` may have a social media subagent: `social_media_agent` which may in turn have a `twitter_agent`, `linkedin_agent`, and `facebook_agent` as subagents. This subtree can perform many of its functions without ever even involving the top-level coordinator, allowing it additional flexibility.

## Hooks

Hooks are a powerful mechanism for extending the functionality of agents without having to modify their core logic. They can be used for a variety of purposes, such as logging, monitoring, modifying inputs/outputs, and more. Hooks are defined via `defineHook` and can be referenced in both agent and prompt definitions by their unique `id`.

A common use for a hook is to truncate the conversation history, or inject artificial tool calls into the conversation. This can be useful, for example, to give a model awareness of the real time world clock, or to provide additional context without an explicit tool call. The following hooks are supported:

| Hook | Execution Point | Purpose |
|------|-----------------|---------|
| filter_messages | Before LLM context assembly | Filter/transform message history |
| prefilter_llm_history | After context assembly | Final adjustments before LLM request |
| before_create_message | Before message insert | Transform message before storage |
| after_create_message | After message insert | Side effects after storage |
| before_update_message | Before message update | Transform update data |
| after_update_message | After message update | Side effects after update |
| before_store_tool_result | Before tool result storage | Transform tool results |
| after_tool_call_success | After successful tool call | Post-process success results |
| after_tool_call_failure | After failed tool call | Handle/recover from errors |

Note: Do be careful when applying hooks to ensure matching tool calls and results are not separated or truncated off as this can lead to errors in many models.

## Variables and Environment

Variables allow tools, prompts, and agents to specify dynamic content they require to function properly. For example a `weather_agent` may have a variable `LOCATION` that it needs in order to provide accurate weather information. Variables can be one of two types: `text` or `secret`. Text variables are simple strings that can be used for any purpose. Secret variables are encrypted and MUST only be used in tools to prevent exposing them to a model. For example, an `GMAIL_API_KEY` variable should be of type `secret` and only used in a tool that makes API calls, it should never be included in a prompt or exposed to the model in any way.

When creating a new thread, all required variables used within the agent graph must be provided. Variables can have their values provided by prompts, tools, or threads. An instance of a variable is called an "environment variable" or `env` whereas the definition of the variable is just called a variable.

## ThreadState

The `ThreadState` object is a powerful tool that is passed to all callables, hooks, and endpoints. It provides access to the current thread's context, including its messages, tools, variables, filesystem, and more. To increase portability, callable tools should only use the `ThreadState` APIs for interacting with the execution environment. For example, rather than using `node:fs` to read files, use `state.readFile()` which will work regardless of the underlying runtime environment.

```ts
/**
 * Thread state interface - the unified API for thread interactions.
 * Available to tools, hooks, and endpoints.
 */
interface ThreadState {
  // ─────────────────────────────────────────────────────────────────────────
  // Identity (readonly)
  // ─────────────────────────────────────────────────────────────────────────
  readonly threadId: string;
  readonly agentId: string;
  readonly userId: string | null;
  readonly createdAt: number;
  readonly children: SubagentRegistryEntry[];
  readonly terminated: number | null;

  // ─────────────────────────────────────────────────────────────────────────
  // Messages
  // ─────────────────────────────────────────────────────────────────────────
  getMessages(options?: GetMessagesOptions): Promise<MessagesResult>;
  getMessage(messageId: string): Promise<Message | null>;
  injectMessage(input: InjectMessageInput): Promise<Message>;
  queueMessage(input: QueueMessageInput): Promise<void>;
  updateMessage(messageId: string, updates: MessageUpdates): Promise<Message>;

  // ─────────────────────────────────────────────────────────────────────────
  // Logs
  // ─────────────────────────────────────────────────────────────────────────
  getLogs(options?: GetLogsOptions): Promise<Log[]>;

  // ─────────────────────────────────────────────────────────────────────────
  // Resource Loading
  // ─────────────────────────────────────────────────────────────────────────
  loadModel<T = unknown>(name: string): Promise<T>;
  loadPrompt<T = unknown>(name: string): Promise<T>;
  loadAgent<T = unknown>(name: string): Promise<T>;
  getChildThread(referenceId: string): Promise<ThreadState | null>;
  getParentThread(): Promise<ThreadState | null>;
  getPromptNames(): string[];
  getAgentNames(): string[];
  getModelNames(): string[];
  env(propertyName: string): Promise<string>;
  setEnv(propertyName: string, value: string): Promise<void>;
  notifyParent(content: string): Promise<void>;
  setStatus(status: string): Promise<void>;

  // ─────────────────────────────────────────────────────────────────────────
  // Tool Invocation
  // ─────────────────────────────────────────────────────────────────────────
  queueTool(toolName: string, args: Record<string, unknown>): void;
  invokeTool(toolName: string, args: Record<string, unknown>): Promise<ToolResult>;

  // ─────────────────────────────────────────────────────────────────────────
  // Effect Scheduling
  // ─────────────────────────────────────────────────────────────────────────
  scheduleEffect(name: string, args: Record<string, unknown>, delay?: number): Promise<string>;
  getScheduledEffects(name?: string): Promise<ScheduledEffect[]>;
  removeScheduledEffect(id: string): Promise<boolean>;

  // ─────────────────────────────────────────────────────────────────────────
  // Events
  // ─────────────────────────────────────────────────────────────────────────
  emit(event: string, data: unknown): void;

  // ─────────────────────────────────────────────────────────────────────────
  // Context Storage
  // ─────────────────────────────────────────────────────────────────────────
  context: Record<string, unknown>;

  // ─────────────────────────────────────────────────────────────────────────
  // File System
  // ─────────────────────────────────────────────────────────────────────────
  writeFile(path: string, data: ArrayBuffer | string, mimeType: string, options?: WriteFileOptions): Promise<FileRecord>;
  readFile(path: string): Promise<ArrayBuffer | null>;
  readFileStream(path: string, options?: ReadFileStreamOptions): Promise<AsyncIterable<FileChunk> | null>;
  statFile(path: string): Promise<FileRecord | null>;
  readdirFile(path: string): Promise<ReaddirResult>;
  unlinkFile(path: string): Promise<void>;
  mkdirFile(path: string): Promise<FileRecord>;
  rmdirFile(path: string): Promise<void>;
  getFileStats(): Promise<FileStats>;
  grepFiles(pattern: string): Promise<GrepResult[]>;
  findFiles(pattern: string): Promise<FindResult>;
  getFileThumbnail(path: string): Promise<ArrayBuffer | null>;

  // ─────────────────────────────────────────────────────────────────────────
  // Execution State
  // ─────────────────────────────────────────────────────────────────────────
  execution: ExecutionState | null;
  terminate(): Promise<void>;

  // ─────────────────────────────────────────────────────────────────────────
  // Runtime Context (Non-Portable)
  // ─────────────────────────────────────────────────────────────────────────
  readonly _notPackableRuntimeContext?: Record<string, unknown>;
}
```


## Further reading

To get an even more detailed understanding of how to create agents, read the following:

Specification Docummentation: https://standardagentspec.org/llms.txt
Agent Builder Documentation: https://docs.standardagentbuilder.com/llms.txt


## Implementation checking

When you edit `agents/`, `prompts/`, `models/`, `tools/`, `hooks/`, or `worker/`,
  validate before claiming the change is done.

  Validation order:
  1. Read `package.json` and prefer project scripts if they exist.
  2. Refresh Cloudflare types:
     - use `pnpm cf-typegen` if that script exists
     - otherwise run `npx wrangler types`
  3. Regenerate AgentBuilder types by running the project's build command:
     - usually `pnpm build`, `npm run build`, `pnpm run dev` or equivalent
  4. Run the project's type checker:
     - prefer `pnpm type-check` / `npm run type-check` if present
     - otherwise use the installed checker directly, such as `pnpm exec vue-tsc --build`, `pnpm exec tsc -b`, or `pnpm exec tsc --noEmit`
  5. If any validation step cannot run, state exactly what is missing.
