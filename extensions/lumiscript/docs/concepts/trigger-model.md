# Trigger model

The mental model that LumiScript actually runs on. Five minutes of reading, then you can stop fighting the framework and start using it.

## The core idea: the body IS the handler

There is no `api.on(event, handler)`. There is no `addEventListener`, no subscription registry, no callback you attach during setup that fires later. **The script body itself runs as the handler.** Every time a wired event fires, Lumiverse wraps your script source in a fresh `AsyncFunction` and runs it top-to-bottom. The event payload arrives on the `data` global. When the body's last line returns, the run is over.

That's the whole model. Everything else falls out of it.

The wiring — *which* events run *which* scripts — lives entirely in the script editor's UI. Open a script, look at the events multi-select up top, that's it. There's no script-source syntax that subscribes to events.

A small consequence worth internalising up front: the `// @triggers MESSAGE_SENT, ...` comment some scripts carry near the top of the file is **documentary only**. The host doesn't parse it. Writing `@triggers` doesn't wire anything; deleting it doesn't unwire anything. It's a convention for telling future-you (or a reader) what the script is *intended* to be wired to. The actual wiring is in the editor UI.

## What a fire looks like step by step

1. A wired event happens somewhere in Lumiverse (user sends a message, chat switches, etc.).
2. The host packages the event payload, picks the scripts wired to that event, and dispatches each one.
3. For each dispatched script, under the default **AsyncFunction** engine: a fresh `AsyncFunction` is constructed around the script body. The function takes eight positional parameters that are exposed to the body as in-scope names: `api`, `data`, `script`, `__console` (re-bound to `console` by a preamble the engine prepends), `z` (the Zod library), `fetch` (a guarded `fetch` — an SSRF-hardened wrapper over `api.utils.http.*` — when the script has `allowDangerous` on, a stub that throws otherwise), `Bun`, and `process` (the last two always `undefined` — they're in the parameter list to shadow the global so a user reference resolves to `undefined` rather than reaching the host runtime). Under the opt-in **QuickJS** isolate the same names are in scope (`api` / `data` / `script` / `console` / `z` / `fetch`), reached as VM globals rather than eight positional params — and a fresh WASM realm simply never had `Bun` / `process` to shadow. Same names and behaviour to your body, different plumbing. See [Execution engine](engine.md).
4. The body executes top-to-bottom. `await` works normally; the run waits for it. Any handler closures the body registers (more on this below) get captured for later.
5. When the body returns (or throws), a *flush* step drains any fire-and-forget chains the body initiated (e.g. unawaited `api.broadcast.emit(...)`) so they complete before the run is reported done.
6. The function instance is discarded.

Step 6 is the one that bites first-timers. The function instance is **gone**. Local variables declared in the body don't survive to the next fire. See "What does NOT persist across fires" below.

## The `data` global

`data` is the event payload. It's a plain JSON-ish object, read-only-by-convention, with two guaranteed properties:

- `data.__event` — the event name (e.g. `'MESSAGE_SENT'`, `'GENERATION_ENDED'`, `'CHAT_SWITCHED'`). Always present.
- The rest depends on the event. For `MESSAGE_SENT` it's `{ chatId, message }`. For `CHAT_SWITCHED` it's `{ chatId }`. For `ls:teardown` it's `{ reason, scriptId, scriptName }`. The full per-event shape table is in the in-app **Reference** under "Lumiverse Events".

Don't mutate `data` — it's not yours, and mutating it has no observable effect anyway (the next fire gets a fresh payload from the host).

## Multi-event wiring and the `__event` discriminator

Most non-trivial scripts wire to multiple events. The body runs for *all* of them, so you'll typically branch on `data.__event`:

```js
switch (data.__event) {
  case 'CHAT_SWITCHED':
    if (data.chatId) await initialiseForChat(data.chatId);
    break;
  case 'MESSAGE_SENT':
    // Fires for every message creation (incl. the assistant's empty
    // placeholder) — guard on is_user to react to the user's send only.
    if (data.message.is_user) await reactToUserMessage(data.message);
    break;
  case 'GENERATION_ENDED':
    await reactToAssistantMessage(data.messageId, data.content);
    break;
}
```

A common newcomer instinct is "I'll split this into three scripts, one per event." Don't — the model is the opposite. One script per *concern*, wired to whichever events that concern needs to react to. The body is the integration point.

## Lifecycle events: `ls:startup`, `ls:reload`, `ls:teardown`

Three LumiScript-internal events mark the moments where a script (re)initialises or shuts down. None come from Lumiverse — the engine synthesises them:

- **`ls:startup`** — *cold start*. Fires when the script enters the active state: at extension boot, and again any time the user toggles it from disabled → enabled. The place for setup that should run whenever the script becomes runnable: registering tools, pre-warming caches, re-asserting broadcast subscriptions that disable wiped.
- **`ls:reload`** — *hot restart*. Fires when you press the editor's **Reload** button, or — with the `@ls:reload-on-edit` directive (below) — when you save an edit. The engine does a full per-script state wipe, then re-runs the body into a clean slate: semantically a fresh `ls:startup`, except `data.__event` is `'ls:reload'` and `data.reason` tells you which path fired it — `'manual'` (Reload button) or `'autosave'` (save-with-directive). `scriptStorage`, theme contributions, and the `script.require()` cache survive the wipe, so you can re-initialise without a disable/enable round-trip. **It's not a picker trigger** — it fires off the Reload button / directive, not a checkbox (the same way the Run button has none).
- **`ls:teardown`** — *end of life*. Fires when the script is disabled or deleted (`data.reason` is `'disabled'` or `'deleted'`). 10-second budget. **The handler cannot veto** — the script is going away regardless; this is your last chance to flush state, notify subscribers, log a goodbye. Anything you don't finish in 10 seconds gets dropped.

Two things to know about `ls:startup`:

- At cold boot, the host hasn't necessarily dispatched the active chat to your script yet. Don't assume `api.chats.getActive()` returns anything useful at `ls:startup` time on a fresh extension load. Use `CHAT_SWITCHED` for chat-context-dependent setup; use `ls:startup` for chat-agnostic setup.
- Top-of-body code (above your `switch` statement) runs on *every* fire, including `ls:startup`. Things like `api.commands.register(...)` and `api.broadcast.on(...)` typically live above the switch precisely so they re-register on every fire including startup, which makes the script self-healing on re-enable.

### Splitting init across cold start and reload

Because `ls:reload` re-runs the body exactly like `ls:startup` — only `data.__event` and `data.reason` differ — you route initialisation by *where it should run*:

```js
// Top of body — runs on EVERY fire (startup, reload, and any real trigger).
// Idempotent (re-)registration lives here so the script self-heals after a reload wipe.
api.commands.register([{ id: 'my-command', label: 'My Command', description: 'What it does' }]);

switch (data.__event) {
  case 'ls:startup':
    // COLD START only (boot, or disable→enable). One-shot, non-idempotent work:
    // seed a default config into scriptStorage, run a data migration, stamp a boot time.
    break;

  case 'ls:reload':
    // HOT RELOAD only. Narrow further with data.reason:
    if (data.reason === 'manual') {
      // The Reload button specifically — e.g. drop a dev cache, re-pull fixtures.
    }
    break;
}
```

Keeping non-idempotent cold-start work (seeding defaults, migrations) inside the `ls:startup` case is the practical reason the split matters — it stops that work re-firing every time you press Reload while iterating.

## `@ls:reload-on-edit`

A directive (not an event) for the iteration loop. `ls:reload` always fires on the editor's **Reload** button; putting `// @ls:reload-on-edit` anywhere in your source *additionally* fires it **automatically when you save**, after a ~500 ms debounce. The autosave fire carries `data.reason === 'autosave'`; the full payload is `{ reason, previousCodeHash, currentCodeHash, previousLength, currentLength, triggeredAt }`. (The Reload-button fire carries `data.reason === 'manual'` — see the lifecycle section above for the full `ls:reload` semantics.)

It's opt-in by design. Auto-re-running a body that does non-idempotent module-scope work — an LLM call, a DB insert, a leaked timer — on every save is a foot-gun, so you opt in per-script when the script is safe to live-reload (UI scripts, idempotent observers) and leave it off otherwise. Without the directive, edits don't fire anything; the new code takes effect on the next *natural* trigger (next `MESSAGE_SENT`, chat switch, etc.), or whenever you press Reload.

The full directive list (and other `// @ls:*` runtime directives) is in the in-app **Reference** under "Directives".

## Handler registrations vs the body

The body model handles "react to a Lumiverse host event". A different shape — "react to something LumiScript doesn't know about" — uses registered handlers:

```js
api.broadcast.on('tracker:state-changed', (payload) => { /* ... */ });
api.commands.onInvoked(async (commandId, ctx) => { /* ... */ });
api.macros.register('myMacro', { /* def */ }, async (ctx) => '...');
api.tools.register('myTool', { /* def */ }, async (args) => '...');
```

These register *closures* that the engine fires later — when a peer script emits a broadcast, when the user invokes a command via Cmd/Ctrl+K, when the host resolves a `{{myMacro}}` in a prompt, when an LLM calls your tool. Closures capture the proxy's `api`, so they keep working even after the run that registered them is over.

Three things to know about registrations:

- **They persist across body fires.** Once registered, a handler stays alive until the script is disabled / deleted, or until the next body fire wipes the prior registrations (depending on the surface — broadcast subs and commands have different wipe semantics; the in-app Reference is authoritative).
- **Their dispatches don't share lifetime with the body that registered them.** A `broadcast.on` handler that fires three hours after the body that registered it ran will still get a working `api` — handler dispatches route through the script's current activeRun via fallback rules.
- **They're cleaned up on disable.** The 10-second `ls:teardown` window includes registry cleanup; user code that registered handlers doesn't need to explicitly unregister them unless it wants to.

The detail of which surface persists how, and the closure-lifetime story, is in [`concepts/handler-lifetime.md`](handler-lifetime.md).

## What does NOT persist across fires

Local declarations don't. This bites everyone once:

```js
let counter = 0;
counter += 1;
console.log(counter);
```

This logs `1`, every time. The `let counter` is local to the AsyncFunction wrapping the body — when the function returns, `counter` is collected.

Three escape valves, in increasing durability:

- **`globalThis.<key>`** — persists for the lifetime of the script-runner subprocess (≈ until the extension reloads). Cheapest option; good for in-memory caches. `globalThis.myCounter ??= 0; globalThis.myCounter += 1;` Shared across scripts in the worker under AsyncFunction and QuickJS *shared* isolation (so namespace your keys); under QuickJS *per-script* isolation each script has its own `globalThis`. See [Execution engine](engine.md).
- **`api.scriptStorage.*`** — session-scoped per-script KV store. Survives worker eviction/respawn and script edits, but cleared on disable/delete and lost on backend restart. 1 MB hard cap.
- **`api.variables.{local, global, character, chat}`** — durable JSON-serialised stores, four scopes. Survives extension reloads. This is the "real" persistence tier.

The full decision matrix lives in [`concepts/storage-model.md`](storage-model.md).

## The sandbox boundary

User scripts run in a hardened sandbox, not on the bare Bun runtime. The short version:

- Some literal source patterns are **rejected at dispatch time** — your script won't run, and the editor console shows a `[security]` entry. The patterns include `import('...')`, bare `require('...')`, `new Function('...')` / `Function('...')`, `.constructor.constructor`, and literal `globalThis.Bun` / `globalThis.process`. Each pattern has a documented replacement (use `script.require('library-name')` instead of `import()`, define functions with normal syntax instead of `new Function`, etc.).
- Many `globalThis` properties are **replaced with `undefined`** at runtime — `Worker`, `WebSocket`, `XMLHttpRequest`, browser dialogs (`alert`/`prompt`/`confirm`), Node-compat module globals (`fs`, `http`, `net`, ...). Standard ES built-ins (`Object`, `Array`, `Promise`, `JSON`, `Math`, `Date`, ...), Web data carriers, Web Crypto, streams, and timers all work normally. (`fetch` is the exception worth calling out: `globalThis.fetch` is locked, but a script with `allowDangerous` receives a top-level `fetch` — the guarded wrapper from step 3 above, on both engines. See [Network egress](../guides/network.md).)

For the full list of rejected patterns + accessible globals + the rationale for each: in-app **Reference** → **Sandbox hardening** section. The canonical accessible-globals list is `SAFE_GLOBALS` in `src/script-runner/child-entry.ts` — when in doubt, that's the source of truth.

Two things to internalise about the sandbox:

- **It's not configurable per script or from script source.** There's no opt-out or escape hatch in your code — the sandbox is part of the runtime contract. (The runtime *engine* — AsyncFunction vs the stronger QuickJS isolate — and, under QuickJS, the context-*isolation* model are user-configurable, but globally in Settings, and neither weakens the sandbox. See [Execution engine](engine.md).)
- **The script's outbound capabilities flow through `api.*` instead.** HTTP via `api.utils.http.*` (gated by `cors_proxy` + `allowDangerous`); filesystem via `api.files.*` (`allowDangerous`); secrets via `api.enclave.*` (`allowDangerous`); etc. The sandbox doesn't lock you *out* of these capabilities — it routes you through permission gates that the user grants explicitly when enabling the extension.

## What's next

You now have the mental model. Suggested next reads:

1. **[Permissions](permissions.md)** — which methods need which permissions, what `allowDangerous` is, how to handle denial gracefully.
2. **[Storage model](storage-model.md)** — picking the right tier from `variables` / `db` / `scriptStorage` / `enclave` / `files`.
3. **[Handler lifetime](handler-lifetime.md)** — the closure-persistence story for `broadcast.on`, `commands.onInvoked`, `macros.register`, `tools.register`.
4. **The in-app Reference tab** — for everything else. Open it once, skim the section list, you'll know where to look later.
