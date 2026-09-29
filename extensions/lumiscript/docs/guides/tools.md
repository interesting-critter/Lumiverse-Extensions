# Registering tools

`api.tools.*` is the inversion of `api.llm.generateWithTools` — the LLM-guide covers how a script *consumes* tools mid-generation; this guide covers how a script *publishes* tools the LLM (or other scripts) can invoke.

Tools registered via `api.tools.register` show up in two distinct surfaces:

- **Lumiverse's Council** — the host's tool-use system where assigned "council members" with personality fields can invoke your tools mid-generation, with the result inlined into the assistant's response. Mark a tool `council_eligible: true` to opt in.
- **Inline function-calling and direct script invocation** — `api.tools.invoke(name, args?)` calls a registered tool's handler directly. Use inside `api.llm.generateWithTools` loops, or cross-script when one extension wants to expose a capability another extension can lean on.

Registering requires the `tools` permission. The list / unregister / invoke surfaces are free-tier.

## The big idea

A tool is a named function with a JSON-Schema-typed `parameters` definition that the LLM (or another script) can call. Your script's handler runs server-side inside the LumiScript subprocess, has the full `api.*` surface available, and returns a string that Lumiverse uses as the tool result.

The two invocation surfaces share **one** handler dispatch path. The only thing that differs is whether `ctx.councilMember` is populated:

![Tool dispatch paths: three sources — Lumiverse host TOOL_INVOCATION event (Council path, ctx populated), api.tools.invoke from a script (inline path, ctx undefined), and api.llm.generateWithTools tool_calls (routed through api.tools.invoke) — all converging on entry.handler](../diagrams/tools.svg)

Either way, the same handler runs. Council-aware tools use `ctx?.councilMember` to personalise the output; non-Council-aware tools ignore `ctx` entirely and just operate on `args`.

## Quick start — minimal Council-eligible tool

Register a "roll dice" tool that any Council member can use:

```js
api.tools.register(
  'roll_dice',
  {
    display_name: 'Roll dice',
    description:  'Rolls a die. Use when the scene calls for a random outcome.',
    parameters: {
      type: 'object',
      properties: {
        sides: { type: 'integer', description: 'Number of sides on the die.', minimum: 2 },
      },
      required: ['sides'],
    },
    council_eligible: true,
  },
  async (args, api, ctx) => {
    // The Council path doesn't fill your `parameters` (see the note below), so on a
    // Council fire `args.sides` is undefined — default to a d20 rather than rolling NaN.
    // Direct callers (api.tools.invoke / generateWithTools) supply it.
    const sides  = typeof args.sides === 'number' ? args.sides : 20;
    const result = 1 + Math.floor(Math.random() * sides);
    if (ctx?.councilMember) {
      return `${ctx.councilMember.name} rolled a ${sides}-sided die and got ${result}.`;
    }
    return `Rolled a ${sides}-sided die: ${result}.`;
  },
);
```

Save the script, enable it, assign the tool to a Council member in Lumiverse's Council UI. The tool now fires whenever that member's chance roll succeeds during generation — with the result inlined into the assistant message.

> **Heads-up — the Council doesn't fill your `parameters`.** Direct callers (`api.tools.invoke` / `generateWithTools`, below) supply `sides`, but on the **Council** path Lumiverse invokes extension tools with `{ context, __deadlineMs }` only — it does *not* run a sidecar LLM to populate your schema, so `args.sides` is `undefined` there. That's why the handler defaults it (a Council fire just rolls a d20). A *real* Council tool does better: it reads `args.context` and **decides** its own arguments from the scene. See the cookbook recipe [Register a tool the Council can use](../cookbook/council-tool.md) for that dual-mode pattern.

You can also invoke it directly from any script:

```js
const text = await api.tools.invoke('roll_dice', { sides: 20 });
console.log(text);  // "Rolled a 20-sided die: 14."
```

## `register(name, def, handler)` reference

```ts
register(name: string, def: ToolDefinition, handler: ToolHandler): void
```

Sync. Requires the `tools` permission. Throws `Error: PERMISSION_DENIED:tools — grant this permission to use this API` when the permission isn't granted.

- **`name`** — the bare tool name (no namespace prefix). Convention is `lowercase_snake_case`. Collisions across scripts throw `Error: tool "<name>" already registered by "<scriptName>"`. Same-script re-registration replaces the prior handler silently — useful for hot-reloading the implementation during dev. There is no reserved-name list for tools (unlike macros).
- **`def`** — see [`ToolDefinition`](#tooldefinition-shape) below.
- **`handler`** — see [The handler signature](#the-handler-signature) below.

### `ToolDefinition` shape

```ts
interface ToolDefinition {
  display_name:      string;
  description:       string;
  parameters?:       Record<string, unknown>;
  council_eligible?: boolean;
}
```

| Field | Purpose |
|---|---|
| `display_name` | Human-readable label shown in Lumiverse's Council UI when assigning tools to members. Don't repeat the snake_case name here. |
| `description` | LLM-facing description — the model reads this to decide when to call the tool. Be specific about the *trigger condition* ("Use when the scene calls for a random outcome") rather than the implementation ("Generates a random number"). |
| `parameters` | JSON Schema for the args object. Standard shape — `{ type: 'object', properties: { … }, required: [ … ] }`. Lumiverse forwards this to providers' native function-calling surface. Omit when the tool takes no arguments. |
| `council_eligible` | Set `true` to make the tool assignable in the Council UI. Defaults to `false` — tools that are useful only for inline function-calling don't need to clutter the Council picker. |

### The handler signature

```ts
type ToolHandler = (
  args: ToolInvocationArgs,
  api:  LumiScriptAPI,
  ctx?: ToolInvocationContext,
) => string | Promise<string>;
```

Async handlers are supported — return `Promise<string>` and the dispatcher awaits naturally.

**Why `api` is passed as a parameter** — even though `api` is a global inside your script body, the engine wraps your handler at registration time with a closure that calls `getApi()` at invocation time and injects it as the second argument. This means a long-lived tool handler always sees the *current* live API surface, even if registration and invocation are minutes apart. The wrapper is documented at `src/engine/api/tools.ts:45-55` if you want to read the rationale; in practice, just use the `api` parameter inside your handler — it does the right thing.

#### `ToolInvocationArgs`

```ts
interface ToolInvocationArgs {
  context?:      string;             // Host-supplied chat context string (Council path).
  __userId?:     string;             // User ID, for scoped api.* calls.
  __deadlineMs?: number;             // Wall-clock deadline by which the handler must return.
  [key: string]: unknown;            // Your tool's schema-defined parameters.
}
```

The well-known fields (`context`, `__userId`, `__deadlineMs`) are populated by the host on Council invocations. Your script's schema-defined parameters arrive as additional fields on the same object (`args.sides` from the roll-dice example above).

`__deadlineMs` is a wall-clock timestamp, not a relative duration — compare against `Date.now()` to decide whether you still have time for an expensive LLM call:

```js
async (args, api, ctx) => {
  const remainingMs = (args.__deadlineMs ?? Infinity) - Date.now();
  if (remainingMs < 2000) {
    return 'Insufficient time for analysis.';
  }
  // ... do the work
}
```

#### `ToolInvocationContext`

```ts
interface ToolInvocationContext {
  requestId?:       string;            // Host correlation ID. Populated on Council path.
  councilMember?:   CouncilMemberContext;  // Populated on Council path only.
  contextMessages?: LLMMessage[];      // Structured chat context. Populated on Council path.
}
```

`ctx` itself is `undefined` when invoked via `api.tools.invoke()` (no Council context applies). On Council invocations, `ctx` is always an object but its fields fall through to `undefined` on older Lumiverse hosts. Always treat the fields as optional:

```js
if (ctx?.councilMember) { /* Council-aware branch */ }
```

`requestId` is useful for cross-system correlation — log it alongside any external API calls so you can match handler-side traces against host-side logs.

`contextMessages` (Council-only, host-version-gated) is the structured equivalent of `args.context` — same chat history, but with role boundaries preserved. Prefer this over the flattened `args.context` string when available; the `ls:council-prompt` helper auto-prefers it when both are provided.

## Council-aware patterns

When a Council member invokes your tool, you have access to the full member identity:

```ts
interface CouncilMemberContext {
  memberId:       string;
  itemId:         string;
  packId:         string;
  packName:       string;
  name:           string;
  role:           string;             // Free-form role description.
  chance:         number;             // 0–100 probability of participation per generation.
  avatarUrl:      string | null;
  definition:     string;             // Lumia "definition" field — physical/identity description.
  personality:    string;             // Free-form personality description.
  behavior:       string;             // Behavioural patterns.
  genderIdentity: 0 | 1 | 2;          // 0=unspecified, 1=feminine, 2=masculine.
}
```

The natural pattern is to use these fields to personalise the tool's output to the invoking member:

```js
async (args, api, ctx) => {
  const member = ctx?.councilMember;
  if (!member) {
    return 'Insufficient context — this tool runs best from a Council member.';
  }

  // Personalise the LLM call with the member's identity.
  const result = await api.llm.generate(
    [
      { role: 'system', content: `You are ${member.name}. ${member.personality}` },
      { role: 'user',   content: `Analyse the recent scene: ${args.context ?? ''}` },
    ],
    { connectionName: 'fast' },
  );
  return result;
}
```

### The `ls:council-prompt` built-in library

The Council UI's *built-in* tools (the ones Lumiverse ships) use a specific prompt shape — identity block + role directive + tool-specific instruction + brevity note + user-control note. To replicate that exact shape inside your own LumiScript-registered tool, use the `ls:council-prompt` helper:

```js
async (args, api, ctx) => {
  if (!ctx?.councilMember) {
    return await api.llm.generate([{ role: 'user', content: args.context ?? '' }]);
  }
  const { buildCouncilMessages } = await script.require('ls:council-prompt');

  const messages = buildCouncilMessages({
    councilMember:   ctx.councilMember,
    contextMessages: ctx.contextMessages,  // structured chat context, preferred over args.context
    args,                                  // fallback for `args.context` string
    tool: {
      display_name: 'Tone Analyzer',
      description:  'Analyse the emotional register of the scene.',
    },
    maxWordsPerTool: 100,
  });

  return await api.llm.generate(messages);
}
```

The helper exports include:

| Helper | What it builds |
|---|---|
| `buildCouncilMessages(opts)` | Complete `LLMMessage[]` ready for `api.llm.generate()` — `[system, context?, user]`. |
| `buildCouncilSystemPrompt(opts)` | Just the system-prompt string used inside `buildCouncilMessages`. |
| `buildCouncilIdentity(member)` | The "You are a council member named …" identity block + personality directives. |
| `roleNote(role)` | Role-aware directive ("As the {role}, focus on …"). Empty when `role` is empty. |
| `brevityNote(maxWords)` | Word-budget directive. Empty when `maxWords ≤ 0`. |
| `userControlNote(allow)` | User-character guidance block — permissive when `allow=true`, restrictive when `false`. |
| `debug.format*` | Snapshot helpers for diagnosing what got built — useful in test fixtures. |

See the in-app Reference's "Built-in Libraries" section for the full surface.

### Reading Council settings via `api.council.*`

For tools that need to introspect Council state outside a specific invocation:

```ts
api.council.getSettings():           Promise<CouncilSettings>             // No permission required.
api.council.getMembers():            Promise<CouncilMemberContext[]>      // Currently-assigned members.
api.council.getAvailableLumiaItems(): Promise<LumiaItem[]>                // Superset — all items in installed packs.
```

`getMembers()` returns the *currently-assigned* Council members (the same `CouncilMemberContext` shape `ctx.councilMember` carries inside a tool handler — only with all members listed). `getAvailableLumiaItems()` returns the broader pool of available items across the user's installed packs, including unassigned ones.

**Inside a tool handler, prefer `ctx.councilMember` over `api.council.getMembers().find(...)`** — it's faster (no IPC roundtrip) and tied directly to the invocation that triggered your handler.

## Council invocation lifecycle — what your script controls vs what the host does

Important boundary for understanding what your handler can rely on:

- **The host decides when to invoke a tool.** Council members roll their `chance` percentage on each generation; on success, Lumiverse fires `TOOL_INVOCATION` with the member context populated. Your script does not influence the chance roll — that's a per-member setting under the user's control.

- **Result retention across regens / swipes is a host setting.** `CouncilSettings.toolsSettings.retainResultsForRegens` is a boolean the user sets via the Council UI. When `true`, Council tool results are persisted in chat metadata and reused on swipes / regens (rather than re-firing every time). When `false`, the tool re-fires. Your handler doesn't participate in the retain logic — it just gets invoked when the host decides. Read the setting via `api.council.getSettings()` if your handler's behavior should adapt (e.g. log "first-run" vs "retained" semantics in your output).

- **The sidecar-LLM dimension is host-side.** Lumiverse-built-in Council tools may route through a sidecar LLM (configured via `CouncilToolsSettings.mode`). Extension-registered tools — yours — are invoked directly with their args; LumiScript doesn't run a sidecar pass between the dispatch and your handler. Your handler is just *called*, period.

## Direct invocation via `api.tools.invoke`

```ts
invoke(name: string, args?: Record<string, unknown>): Promise<string>
```

Free-tier; works across scripts. Use cases:

- **Inside `api.llm.generateWithTools` loops** — dispatch the model's `tool_calls` to the appropriate handler:

  ```js
  for (const tc of result.tool_calls) {
    const out = await api.tools.invoke(tc.name, tc.args);
    messages.push({ role: 'user', content: [{ type: 'tool_result', tool_use_id: tc.call_id, content: out }] });
  }
  ```

- **Cross-script** — script A invokes a tool registered by script B. From B's handler's perspective: `ctx` is undefined, `args` is whatever A passed. Useful when one extension publishes a reusable capability and other scripts want to consume it programmatically.

- **Testing** — call your own tool with synthetic args during dev to verify the handler logic without spinning up a Council invocation.

`invoke()` throws if the name doesn't resolve:

```
Error: api.tools.invoke: no handler registered for tool '<name>'
```

This is sync-throw (the rejection is on the returned Promise but the absence-of-tool check happens synchronously).

## Listing + cleanup

```ts
list(): RegisteredToolInfo[]
```

Returns a snapshot of *all* registered tools across all scripts. Each entry includes `name`, `display_name`, `description`, `parameters`, `council_eligible`, `scriptId`, `scriptName`. Free-tier; use for diagnostic UIs and "what other tools are available" introspection.

```ts
unregister(name: string): void
```

Ownership-verified — only removes the tool if your script owns it. Silent no-op when the name doesn't match a tool your script registered. Free-tier.

**Auto-cleanup on script disable / delete.** When your script is disabled or deleted, LumiScript walks its registered tools and `unregister`s each one synchronously (`src/backend.ts:814-816`). Both the LumiScript store and Spindle's host-side tool registry are cleared. You don't need to call `unregister` explicitly for the disable/delete path — it's only for mid-life removals.

## `TOOL_INVOCATION` event + `ls:tool:*` broadcasts

Two distinct event surfaces fire during tool lifecycle:

- **`TOOL_INVOCATION`** — the Lumiverse host event that drives tool dispatch. LumiScript subscribes to it **internally** and routes each invocation to the matching registered handler (the Council path — see `dispatchToolInvocation`). You do **not** wire it yourself: it is **not** in the editor's events selector, and the way you receive invocations is by registering a handler with `api.tools.register`. To **observe** invocations from a *different* script — or log them globally — without owning the tool, use the `ls:tool:invoked` broadcast (below): it carries the `result` + `callMs` timing and fires on both the Council and `invoke()` paths. (For reference, the payload the host hands the dispatcher is `{ toolName, args, councilMember, requestId, contextMessages }` — `councilMember` / `contextMessages` populated on Council-driven invocations; `requestId` correlates with the eventual `tool_invocation_result` post-back.)

- **`ls:tool:registered`** — fires inside `api.tools.register` after successful registration. Payload: `{ name, scriptId }`. Subscribe via `api.broadcast.on('ls:tool:registered', fn)` if you want to watch the tool registry from another script.

- **`ls:tool:unregistered`** — symmetric to `:registered`. Payload: `{ name, scriptId }`.

- **`ls:tool:invoked`** — fires after a tool handler returns successfully, on both the Council and `invoke()` paths. Payload: `{ name, args, result, scriptId, callMs, councilMember? }`. `councilMember` is populated on the Council path, `undefined` on `invoke()`. The `callMs` timing makes this useful for tool-performance dashboards. (See `concepts/handler-lifetime.md` for the broader broadcast lifecycle.)

## Common patterns

### Council-aware analyser tool

```js
api.tools.register(
  'analyse_tone',
  {
    display_name: 'Tone analyser',
    description:  'Read the recent scene and report on the emotional register.',
    council_eligible: true,
  },
  async (args, api, ctx) => {
    if (!ctx?.councilMember) {
      // Fallback for direct invocation — return a less personality-driven analysis.
      const text = await api.llm.generate(
        [{ role: 'system', content: 'You analyse emotional tone.' },
         { role: 'user',   content: args.context ?? '' }],
        { connectionName: 'fast' },
      );
      return text;
    }
    const { buildCouncilMessages } = await script.require('ls:council-prompt');
    const messages = buildCouncilMessages({
      councilMember:   ctx.councilMember,
      contextMessages: ctx.contextMessages,
      args,
      tool: { display_name: 'Tone analyser', description: 'Analyse emotional register.' },
      maxWordsPerTool: 80,
    });
    return await api.llm.generate(messages, { connectionName: 'fast' });
  },
);
```

### Tool that does an expensive operation only when deadlines allow

```js
api.tools.register('deep_summary', {
  display_name: 'Deep summary',
  description:  'Produce a detailed multi-paragraph summary of the scene.',
  council_eligible: true,
}, async (args, api, ctx) => {
  const remainingMs = (args.__deadlineMs ?? Infinity) - Date.now();
  if (remainingMs < 5000) {
    return 'Not enough time remaining for a deep summary; sticking to a brief.';
  }
  const text = await api.llm.generate(/* expensive call */, { connectionName: 'flagship' });
  return text;
});
```

### Cross-script tool exposing a capability

```js
// Script "weather-provider" registers:
api.tools.register('lookup_weather', {
  display_name: 'Weather lookup',
  description:  'Get the weather for a location.',
  parameters: { type: 'object', properties: { location: { type: 'string' } }, required: ['location'] },
}, async (args) => {
  const resp = await api.utils.http.get(`https://wttr.in/${encodeURIComponent(args.location)}?format=3`);
  return resp.body;
});

// Script "weather-consumer" calls it cross-script:
const summary = await api.tools.invoke('lookup_weather', { location: 'Tokyo' });
// summary: "Tokyo: ⛅️  +18°C"
```

The consumer doesn't need its own `app_manipulation` / `cors_proxy` permission for the lookup — the registered tool's owning script does. This is a useful encapsulation point.

## Common pitfalls

- **`council_eligible` defaults to `false`.** If your tool doesn't show up in the Council picker, it's because you didn't opt in. Set `council_eligible: true` explicitly in the definition.

- **`ctx` is `undefined` when invoked via `api.tools.invoke`.** Don't write `ctx.councilMember` — write `ctx?.councilMember` so the optional chaining handles the direct-invoke and inline-function-calling paths gracefully.

- **Handler must return a string.** Returning an object (or anything that's not a string / `Promise<string>`) ends up stringified inelegantly. If you have structured data to surface, JSON.stringify it yourself and shape the description to tell the model what to do with the JSON.

- **Tool names share a single registry — no namespacing.** Two scripts can't both register `analyse_tone`. The second registration throws. Use a script-prefix convention (`my-script_analyse_tone`) if you publish scripts that other people install, so collisions stay rare.

- **`api` is a parameter, not the closure.** Inside the handler, `api` refers to the function parameter (which is the engine-injected current api). You CAN reference the closure-scoped `api` from the outer script body and it'll work for current invocations, but the parameter version is the official wiring — write `(args, api, ctx) => api.llm.generate(...)` and let the engine inject. This avoids a subtle bug where a long-lived registered handler captured an old `api` closure that has since been torn down.

- **Retention behavior is host-side.** Don't try to implement "skip re-run on swipe" inside your handler. The host already handles this when `CouncilSettings.toolsSettings.retainResultsForRegens` is true — your handler just gets called when the host decides it needs a fresh result. If you need to know whether *this* invocation is a first-run vs a retained-result rerun, the host doesn't currently expose that distinction; the safest design assumes every invocation might be either.

- **`__userId` is the invoking-user id, not the active-character id.** Don't confuse the two. For the active character, read `globalThis.__lsActiveCharId` (sync) or `await api.chats.getActive().then(c => c?.characterId)`.

## See also

- **In-app Reference, "API Functions → api.tools" section** — auto-generated method list with full signatures.
- **[Calling the LLM](llm.md)** — `api.llm.generateWithTools` is the consumer side of the tool surface; this guide is the publisher side. The two pair cleanly.
- **[`concepts/permissions.md`](../concepts/permissions.md)** — `tools` permission and the free-tier methods (`list`, `unregister`, `invoke`).
- **[`concepts/handler-lifetime.md`](../concepts/handler-lifetime.md)** — registered tool handlers persist across fires; covered there for the broader handler-lifetime picture.
- **`ls:council-prompt` built-in library** — the canonical helper for Council-shaped prompts inside your own LLM calls. Reference's "Built-in Libraries" section.
- **`api.events.track(...)`** with the `tool_invoked` event-kind — pair with `ls:tool:invoked` broadcasts for a persistent audit log of tool-fires.
