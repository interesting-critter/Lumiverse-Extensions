# Custom macros

A LumiScript script can register custom `{{macros}}` that Lumiverse resolves wherever a macro can appear — in prompts assembled for the LLM, in chat-title regeneration, in preset previews, anywhere the host's macro resolver runs. This is the cleanest way to inject script-computed content into the prompt pipeline without touching `api.chat.inject` or content processors.

`api.macros.register` itself is **free-tier** — any script can register a macro, no permission required. The side effects each handler causes (DB reads, LLM calls, etc.) are gated by whatever `api.*` permission they touch. Macro **interceptors** (covered later) require `macro_interceptor`.

## What macros are for

Lumiverse already ships built-in macros: `{{user}}` and `{{char}}` resolve to the active persona / character names, `{{getvar::x}}` reads a stored variable, and so on. Your script can register new ones with arbitrary computed values:

```
{{moodScore}}                      → "0.74"
{{getCharNote::theme}}             → "Tone: melancholic, midwinter setting"
{{lastSummary}}                    → "User asked Alice about her childhood..."
```

Once registered, the user can type the macro name into a system prompt / preset block / world-info entry, and your script's value gets substituted at generation time. The user controls *where* the macro fires; you control *what* it resolves to.

## Push vs pull mode

There are two ways to register a macro. The decision is about *when the value is computed*:

- **Push mode** — register without a handler. The macro starts with no value (resolves to `""`). Your script computes the value somewhere else (a trigger fire, a UI handler, a periodic refresh) and calls `api.macros.updateValue(name, newValue)` to push it. The host stores the pushed value; macro resolution reads it from cache with no further RPC.

- **Pull mode** — register *with* a handler function. The host calls your handler every time the macro resolves, passing a context object. The handler returns the string value. Async handlers are supported — `await` directly in the body, no globalThis pre-caching.

**Prefer push when possible.** Push mode avoids RPC latency at resolution time, which matters when a macro fires deep inside a heavily-templated prompt (each pull-mode resolve is a worker round-trip). Use pull only when the value genuinely depends on resolution-time context (the macro arguments, the active character, the active chat).

## Quick start — push mode

A macro that tracks how many user messages have been sent in the current session:

```js
// Trigger: ls:startup (runs once when the script becomes active)
if (data.__event === 'ls:startup') {
  api.macros.register('messagesSent', {
    description: 'Number of messages the user has sent in this session.',
    category:    'extension:lumiscript:my-script',
    returnType:  'integer',
  });
  // Seed the initial value.
  await api.macros.updateValue('messagesSent', '0');
}

// Trigger: MESSAGE_SENT
if (data.__event === 'MESSAGE_SENT' && data.message.is_user) {
  const current = parseInt(await api.variables.global.get('messagesSent', '0'), 10);
  const next    = current + 1;
  await api.variables.global.set('messagesSent', String(next));
  await api.macros.updateValue('messagesSent', String(next));
}
```

Now `{{messagesSent}}` resolves to the current count anywhere a macro can be used. The user can put it in a system prompt, a world-info entry, anywhere.

## Quick start — pull mode

A macro that resolves to a per-character mood the script computes on demand:

```js
api.macros.register(
  'characterMood',
  {
    description: 'Current mood of the active character (computed from recent messages).',
    category:    'extension:lumiscript:my-script',
    returnType:  'string',
  },
  async (ctx) => {
    if (ctx.commit === false) return '';                // dry resolve — skip work
    const chat = await api.chats.getActive();
    if (!chat?.characterId) return 'neutral';
    return await computeMood(chat.characterId);         // your async function
  },
);
```

Two things worth noticing in that handler:

1. **`ctx.commit === false` guard.** Lumiverse re-resolves macros for *previews* (chat-title regen, prompt-preview UI, etc.) — calling them with `commit: false`. Handlers with side effects should skip writes on dry resolves. Read-only handlers can ignore the field. (See [`ctx.commit`](#ctxcommit--handling-dry-resolves) below.)

2. **`await api.chats.getActive()` for the character ID** — not `ctx.env?.character?.id`. The macro context's `env.character` carries card data (name, description, etc.) but the source explicitly notes the `id` field isn't reliably populated there. Use `api.chats.getActive()` (or read `globalThis.__lsActiveCharId` set by the engine).

## `register(name, def, handler?)` — the unified call

```ts
register(
  name:    string,
  def:     MacroDefinition,
  handler?: (ctx: MacroContext) => string | Promise<string>,
): void
```

One call covers both modes — passing a `handler` makes it pull-mode, omitting makes it push-mode.

Two things the engine validates at registration time:

- **Reserved names are rejected.** LumiScript reserves the names of its own built-in macros (`lumiScriptActive`, the character-variable family `getcvar` / `setcvar` / `addcvar` / `inccvar` / `deccvar` / `hascvar` / `deletecvar` plus their `*char*` aliases). Trying to register one of these throws.

- **Cross-script collisions are rejected.** Two scripts can't register the same macro name. The second registration throws an error naming the owning script. Use a per-script prefix (`my-script:mood`, `agents:current-task`) to namespace.

There is no permission gate on `register` itself.

### `MacroDefinition` shape

```ts
interface MacroDefinition {
  description: string;
  category?:   string;          // default: 'extension:lumiscript:user'
  returnType?: 'string' | 'integer' | 'number' | 'boolean';
  args?: Array<{
    name:         string;
    description?: string;
    required?:    boolean;
  }>;
  volatile?: boolean;           // default: true (pull) / false (push)
}
```

| Field | Purpose |
|---|---|
| `description` | Shown in the macro browser + preset editor when a user is picking macros to include. Make it actionable: "Returns the user's current mood as a 0-1 score" beats "mood". |
| `category` | Groups your macros in the macro browser. Defaults to `extension:lumiscript:user`, which clusters all script-registered macros together. Override with your own prefix (e.g. `extension:lumiscript:my-script`) for visual grouping. |
| `returnType` | Hint for value coercion. Default `'string'`. Set to `'integer'`/`'number'`/`'boolean'` when you want the host to coerce the macro's resolved string into a typed value (used by `{{if}}`-style scoped macros). |
| `args` | Argument schema for the preset editor's macro-insertion UI. Doesn't enforce anything at resolution time (the handler still sees raw `ctx.args` strings); purely for the UI's autocomplete. |
| `volatile` | Opt out of Lumiverse's display-regex cache. Set `false` for push-mode macros (already cached on the host); set `true` for pull-mode macros whose value can change per-resolution. |

### Pull-mode handlers

#### The `ctx` parameter

```ts
interface MacroContext {
  name:      string;
  args:      string[];
  env?: {
    character?: { name?: string; [k: string]: unknown };
    chat?:      { id?: string;   [k: string]: unknown };
    names?:     { char?: string; user?: string; [k: string]: unknown };
    variables?: { local?: Record<string, string>; global?: Record<string, string> };
    [k: string]: unknown;
  };
  isScoped?: boolean;
  body?:     string;
}
```

- **`ctx.name`** — the macro name (without `{{}}`, without arguments).
- **`ctx.args`** — array of string arguments parsed from the invocation. `{{moodFor::Alice::strict}}` arrives as `['Alice', 'strict']`. **Always strings** — the engine doesn't type-coerce based on `def.args`.
- **`ctx.env`** — environment snapshot. `env.character` carries card data but **the `id` field isn't reliably populated** (it's character card metadata, not chat-level state). Use `await api.chats.getActive()` for active character resolution.
- **`ctx.env.variables`** — the user's `{{getvar}}` / `{{getglobalvar}}` stores, populated from `api.variables.local` / `api.variables.global`. Read-only snapshot.
- **`ctx.isScoped`** — `true` when resolved inside a scoped block like `{{if::condition}}…{{/if}}`.
- **`ctx.body`** — body text for scoped macros (the content between the open and close tags).

#### `ctx.commit` — handling dry resolves

The macro context also carries a `commit?: boolean` field at runtime that's used to signal whether the resolution will actually be consumed:

- **`commit === undefined` or `commit === true`** — real resolution; whatever your handler does has effect.
- **`commit === false`** — dry resolve. The host is previewing what the prompt would look like (chat-title regen, prompt-preview UI, etc.) and isn't going to use the result. Your handler should *skip side effects* — disk writes, event emissions, external HTTP, mutating `api.*` calls.

```js
async (ctx) => {
  if (ctx.commit === false) return '';        // dry — skip the work
  await api.events.track('mood_lookup', { ... });
  return computeMood();
}
```

`ctx.commit` is part of the typed `MacroContext` as of v1.0.0-rc.8 — TypeScript autocomplete inside Monaco resolves it cleanly. (The field had been delivered at runtime since Lumiverse spindle-types v0.4.32 but the public type definition lagged behind; the rc.8 docs pass closed the gap.)

#### Async handlers

Function-reference handlers can be sync or async — the type is `(ctx) => string | Promise<string>`. The host awaits the returned promise natively; you don't need to pre-load async data into `globalThis` and read it sync (that pattern only applies to the legacy string-form handler shape, which `api.macros.register` doesn't accept anyway).

```js
api.macros.register('lastReview', { description: '...' }, async (ctx) => {
  const events = await api.events.query({ eventName: 'character_review', limit: 1 });
  return events[0]?.payload?.summary ?? 'No reviews yet';
});
```

#### Getting the active character in a handler

```js
// Wrong — env.character is card data, .id isn't reliably populated.
const charId = ctx.env?.character?.id;

// Right — query the active chat.
const chat = await api.chats.getActive();
const charId = chat?.characterId;

// Also right — read the engine-published global.
const charId = globalThis.__lsActiveCharId ?? null;
```

The `globalThis.__lsActiveCharId` global is set by the LumiScript backend after every active-context refresh (it's the same pattern LumiScript's own built-in character-variable macros use — see `src/macros.ts:80`). Reading it is sync, which matters in tight handler bodies where you want to avoid an `await` round-trip.

### Push-mode value updates

```ts
updateValue(name: string, value: string): void
```

Pushes a new value for a push-mode macro. The argument must be a string — the host coerces typed-return values via `def.returnType` on the resolve side, not on the push side.

```js
await api.macros.updateValue('messagesSent', '42');
```

**Calling `updateValue` on a pull-mode macro throws:**

```
Error: api.macros.updateValue: "<name>" was registered with a handler (pull mode) — updateValue is only valid for push-mode macros.
```

This is a deliberate fail-fast — silently doing nothing would mask a mode-mismatch bug.

### Listing + cleanup

```ts
list(): RegisteredMacroInfo[]
```

Returns a snapshot of *all* macros across all scripts — the data backing the macro browser. Each entry includes `name`, `description`, `category`, `mode: 'push' | 'pull'`, `scriptId` / `scriptName` (which script owns it), and (for push-mode) `lastValue`. Free-tier; use it for inspection / debug UIs.

```ts
unregister(name: string): void
```

Removes a macro your script registered. Silent no-op if the macro doesn't exist or isn't owned by this script.

**You usually don't need to call `unregister` explicitly.** When a script is disabled or deleted, LumiScript's teardown walks its registered macros and unregisters all of them automatically. Reach for `unregister` only when you want to remove a specific macro mid-life without disabling the whole script.

## Macro interceptors — `registerInterceptor`

A separate, more powerful surface. Interceptors run *before* Lumiverse's normal macro resolver — they receive the **raw template string** for a given prompt block / response / etc. and can return a transformed template. Use when per-macro RPC cost would dominate the resolution path (think `{{#each LARGE_LIST}}…{{my_macro}}…{{/each}}` patterns where the per-macro round-trip is the bottleneck) — or when you want to rewrite templates wholesale before any macros resolve.

```ts
registerInterceptor(
  handler: (ctx: MacroInterceptorCtx) => string | void | Promise<string | void>,
  options?: MacroInterceptorOptions,
): { id: string; remove(): void }
```

### The handler

The handler receives:

```ts
interface MacroInterceptorCtx {
  readonly template:    string;                  // the raw template before macro resolution
  readonly env:         MacroInterceptorEnv;     // same env shape as MacroContext.env
  readonly commit:      boolean;                 // false on dry resolves (always present on this ctx)
  readonly phase:       'prompt' | 'display' | 'response' | 'other';
  readonly sourceHint?: string;                  // host-side hint, e.g. block id
  readonly userId?:     string;
}
```

Return **a string** to replace the template with your transformation, or **void / undefined** to pass through unchanged. (Returning the original template is wasted work — just return void.)

### Options

```ts
interface MacroInterceptorOptions {
  id?:            string;                          // stable id for de-dup / re-registration
  priority?:      number;                          // default 100; LOWER runs first
  phase?:         MacroInterceptorPhase | MacroInterceptorPhase[];
  matchTemplate?: string | string[] | RegExp;      // pre-filter on the raw template
  timeoutMs?:     number;                          // default 2000
}
```

- **`priority`** — interceptors run in **ascending** priority order — lower values run first. Default 100; **drop below 100** to run before other extensions' interceptors, **bump above 100** to run after. (Same convention as `api.worldInfo.registerInterceptor` and other LumiScript interceptor surfaces. Verified at `macro-interceptor-registry.ts:374-381` and `script.ts:5487`.)
- **`phase`** — pre-filter on resolution context. Only the listed phases invoke this handler. Omit to run for everything.
  - `'prompt'` — assembling the LLM prompt.
  - `'display'` — rendering messages in the chat UI.
  - `'response'` — post-processing the LLM's response.
  - `'other'` — anything that doesn't fit the above (preset previews, etc.).
- **`matchTemplate`** — pre-filter on the template string. Matched as:
  - `string` → `template.includes(s)`
  - `string[]` → any string in the array matches via `includes`
  - `RegExp` → `re.test(template)`
- **`timeoutMs`** — interceptor handlers that take longer than this throw and the template passes through unchanged. Default 2 seconds; bump up for slow async work, but keep in mind the prompt assembly is blocked waiting.

### Permission

`registerInterceptor` is gated by `macro_interceptor` — a separate permission from `interceptor` (which gates `api.chat.inject` and `api.chat.registerContentProcessor`). Performance-sensitive surface, so it's gated independently.

`listInterceptors()` is free-tier — diagnostic / debug surface.

### Cleanup

The handle returned from `registerInterceptor` has a `remove()` method. Like macros, interceptors are auto-cleared on script disable / delete — explicit `remove()` is for mid-life sweeps.

## The non-committing macro race

This is the macro-specific gotcha worth knowing about up front. The worker that runs your script has a module-scope flag that tracks whether *any* macro is currently being resolved with `commit: false`. While that flag is set, every *mutating* `spindle.*` call from the same worker throws something like:

```
Error: spindle.X.Y is not allowed during non-committing macro resolution
```

The host does this because dry resolves are supposed to be side-effect-free — you can't safely mutate state while previewing a prompt that may never actually be sent. Reads are fine; writes throw.

**The most exposed surface is `CHAT_SWITCHED`** — opening a chat triggers a prompt regen, which fires `CHAT_SWITCHED` triggers while macro resolution is mid-flight with `commit: false`. Scripts that mutate on chat-switch can hit this.

### What auto-retries vs what doesn't

The LumiScript engine handles this race for two surfaces transparently:

- **`api.theme.apply` / `applyPalette` / `clear`** — wrapped in `withMacroRaceRetry`. Five attempts with exponential backoff (15ms / 30ms / 60ms / 120ms / 240ms). Your script doesn't need to do anything.

- **`api.macros.register`** — has a targeted try/catch inside the registration path. If the underlying `spindle.registerMacro` call hits the race, the engine logs at info level and treats the previous registration as still valid. The macro stays registered.

Every other mutating surface — `api.chat.sendMessage`, `api.variables.global.set`, `api.events.track`, etc. — currently throws and you need to catch + retry in user code if you want race-resilience. Common shape:

```js
async function retryOnMacroRace(fn, attempts = 5) {
  let delayMs = 15;
  for (let i = 0; i < attempts; i++) {
    try {
      return await fn();
    } catch (err) {
      const msg = String(err?.message ?? err);
      if (!msg.includes('non-committing macro resolution')) throw err;
      if (i === attempts - 1) throw err;
      await new Promise(r => setTimeout(r, delayMs));
      delayMs *= 2;
    }
  }
}

// Usage:
await retryOnMacroRace(() => api.variables.global.set('mood', String(score)));
```

The proper upstream fix is to scope the worker's `macroInvocationStack` per-async-context (so concurrent unrelated work doesn't see the flag), but that requires host-side changes — currently tracked, not yet landed.

## LumiScript built-in macros

These are registered at extension boot and you can use them from scripts (or as macros in prompts) without registering anything yourself:

| Macro | Aliases | Returns | Notes |
|---|---|---|---|
| `{{lumiScriptActive}}` | — | `"true"` / `"false"` | Push-mode. Mirrors the extension's master enable toggle. |
| `{{getcvar::key}}` | `{{getcharvar}}` | Stored string value | Reads from `variables/characters/<id>.json`. Returns `""` if no character active or key not set. |
| `{{setcvar::key::value}}` | `{{setcharvar}}` | `""` | Writes. Skips on `ctx.commit === false`. |
| `{{addcvar::key::number}}` | `{{addcharvar}}` | New numeric value | Adds to current value (numeric). Skips on dry resolve. |
| `{{inccvar::key}}` | — | New value | `+1` shortcut for `addcvar`. |
| `{{deccvar::key}}` | — | New value | `-1` shortcut for `addcvar`. |
| `{{hascvar::key}}` | `{{hascharvar}}` | `"true"` / `"false"` | Read-only check. |
| `{{deletecvar::key}}` | `{{deletecharvar}}` | `""` | Removes the key. Skips on dry resolve. |

All the character-variable macros operate on a per-character JSON store. They're sync from the user's perspective (just type the macro in a prompt) but their handlers are async (`spindle.userStorage.*`). The names are reserved — user scripts can't override them.

## Common patterns

### Per-character status macro (push mode, recomputed on relevant events)

```js
// Trigger: MESSAGE_SENT, GENERATION_ENDED, ls:startup
if (data.__event === 'ls:startup') {
  api.macros.register('characterStatus', {
    description: 'Active character status — emotional/situational summary.',
    category:    'extension:lumiscript:my-script',
  });
}

// Recompute on every assistant message.
if (data.__event === 'GENERATION_ENDED') {
  const summary = await api.llm.generateStructured(/* analysis prompt */, /* schema */);
  await api.macros.updateValue('characterStatus', summary.text);
}
```

The user can then put `{{characterStatus}}` in the system prompt and it'll always reflect the most recent analysis.

### Databank lookup macro (pull mode, async)

```js
api.macros.register(
  'lookupFact',
  {
    description: 'Look up a fact from the project databank by topic.',
    args: [{ name: 'topic', required: true }],
  },
  async (ctx) => {
    if (ctx.commit === false) return '';
    const topic = (ctx.args[0] ?? '').trim();
    if (!topic) return '';
    const doc  = await api.databanks.documents.findByName('proj-databank-id', topic);
    const fact = doc ? (await api.databanks.documents.getContent(doc.id))?.content : null;
    return fact?.slice(0, 200) ?? `(no fact for "${topic}")`;
  },
);
```

Usage in a prompt: `Background: {{lookupFact::Vex's home village}}`.

### Interceptor pattern for iteration-heavy templates

```js
// Some prompt block uses {{#each items}}…{{itemDetail::X}}…{{/each}} with hundreds of items.
// Per-iteration macro resolves dominate. Pre-compute the whole expansion once.

api.macros.registerInterceptor(
  (ctx) => {
    if (!ctx.template.includes('itemDetail::')) return;
    if (ctx.commit === false) return ctx.template;   // skip work on dry resolves
    // Pre-expand all itemDetail references in this template at once.
    return ctx.template.replace(/\{\{itemDetail::([^}]+)\}\}/g, (_, id) => lookupDetail(id));
  },
  { matchTemplate: 'itemDetail::', phase: 'prompt' },
);
```

## Common pitfalls

- **`updateValue` on a pull-mode macro throws.** Pick one mode at registration time and stick with it. If you're not sure, push mode is the safer default — easier to reason about, no resolution-time RPC.

- **`ctx.env?.character?.id` isn't reliable.** Use `await api.chats.getActive()` (canonical) or `globalThis.__lsActiveCharId` (sync, set by the engine). The `env.character` field carries card data — `name`, `description`, etc. — but the source comment explicitly notes the id isn't populated there.

- **Reserved names are rejected at registration.** `lumiScriptActive` and the character-variable family (`getcvar` / `setcvar` / etc. + their `*char*` aliases) are reserved. Namespace your own macros (`my-script:mood`, etc.) to avoid future collisions when more built-ins ship.

- **Two scripts can't register the same name.** First registration wins; second throws naming the owning script. Use script-prefixed names if you're publishing scripts other people may install.

- **Mutating `api.*` calls during non-committing macro resolution throw.** `CHAT_SWITCHED` triggers are the canonical exposure. Wrap with a retry helper or restructure the script to defer the mutation until after the macro race window clears.

- **Don't pull when you can push.** Pull-mode handlers add per-resolution RPC latency. If the value can be computed ahead of time and refreshed on a known event (`MESSAGE_SENT`, `GENERATION_ENDED`, `ls:startup`, etc.), push it. Pull-only when resolution-time arguments (`ctx.args`) or per-resolution state (mid-stream snapshots, etc.) genuinely matter.

- **`args` is descriptive, not enforced.** The schema you provide in `def.args` powers the preset editor's UI but doesn't validate at resolution time. Handlers receive raw string arguments; validate / coerce / clamp them yourself.

- **Broadcast on register/unregister.** Every successful `register` fires `ls:macro:registered` with `{ name, scriptId, mode }`; every `unregister` fires `ls:macro:unregistered` with `{ name, scriptId }`. Other scripts can subscribe via `api.broadcast.on('ls:macro:registered', fn)` for cross-script awareness. See [`concepts/handler-lifetime.md`](../concepts/handler-lifetime.md) for the `ls:*` prefix policy.

## See also

- **In-app Reference, "API Functions → api.macros" section** — auto-generated method list with full signatures.
- **[`concepts/permissions.md`](../concepts/permissions.md)** — `macro_interceptor` (gated) vs `register/updateValue/...` (free-tier).
- **[`concepts/trigger-model.md`](../concepts/trigger-model.md)** — how the body becomes the handler that fires `api.macros.updateValue` on relevant events.
- **[`concepts/handler-lifetime.md`](../concepts/handler-lifetime.md)** — closure persistence: pull-mode handlers capture state at registration; covered there in detail.
- **[Calling the LLM](llm.md)** — `api.llm.dryRun()` is invaluable when debugging macro resolution — it returns the assembled message array so you can see exactly what your macros end up contributing.
