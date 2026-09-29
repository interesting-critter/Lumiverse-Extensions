# Handler lifetime

When your body registers a `broadcast.on(...)`, a `tools.register(...)`, a `macros.register(...)`, or a `DOMHandle.on(...)` — what happens to that callback after the body returns? Why does it still work hours later? What does it see? And what's the closure-capture gotcha that bites every script with a long-lived handler?

## The core idea

The script body is the *event* handler (one body, fires every time a wired event arrives — see [Trigger model](trigger-model.md)). Inside the body, your code can register *additional* callbacks for things the host doesn't route as Lumiverse events: peer-script broadcasts, command-palette invocations, OAuth callbacks, LLM tool calls, macro resolutions, DOM events on injected elements. These are **registered handlers**, and they have a fundamentally different lifetime story than the body.

Three things to internalise up front:

1. **A registered handler survives past the run that registered it.** The body's AsyncFunction returns, the run completes, and the handler closure is still alive — held by the engine, waiting for whatever it's supposed to fire on.
2. **When the handler fires, it runs in its own ephemeral *handler run*** — a fresh `runId` of the form `handler-<ts>-<seq>` (distinct from body runs, which look like `run-<ts>-<seq>`). The engine sets up an activeRun for the handler-fire, the closure dispatches `api.*` calls against it, the run ends when the closure returns.
3. **The `api` reference the closure captured at registration time still works.** That's by construction — the proxy captures script identity and the dispatch path, not a specific body run's state. So `api.chat.getMessages()` inside a handler that fires three hours after its body returned still gets the current chat's messages.

The handler closure doesn't get a `data` global the way the body does (handlers receive their payload as a function argument), but everything else flows the same way — `api.*`, `script.*`, the storage tiers from [Storage model](storage-model.md), all available.

## The surfaces

Handler-registration APIs grouped by lifetime semantics. Each group's lifetime applies to every surface in it.

### Wiped between body fires

- **`api.broadcast.on(event, fn)`** — register a subscriber to a peer-script broadcast (see also the `LumiScript Events` section in the in-app Reference for the built-in `ls:*` event names). The host wipes broadcast subscriptions for a given script at the *start* of that script's next body fire. So a subscription registered in fire N is gone by fire N+1's first line. The canonical mitigation is the [bottom-of-body re-registration idiom](#the-bottom-of-body-re-registration-idiom) below.

### Last-write-wins (one handler per scope)

- **`api.commands.onInvoked(fn)`** — one handler per script. Registering a new one replaces the prior. Fires when the user invokes **any** command LumiScript has registered (not just the ones this script registered) — every script's `onInvoked` handler sees every commandId. Filter on the `commandId` argument yourself if you only care about a specific command. Lives until script unregister.
- **`api.oauth.onCallback(fn)`** — one handler per *extension* (not per script — LumiScript's OAuth surface is shared across every script inside it). Registering a new one replaces the prior; LumiScript emits a `spindle.log.warn` on cross-script collisions so you can spot accidental clobbers. Lives until script unregister.

### Many handlers, persist until script unregister

- **`api.macros.register(name, def, fn)`** (pull-mode) — many handlers per script, keyed by macro name. The handler fires when the host resolves a `{{name}}` macro. Re-registering with the same name replaces the prior handler for that name.
- **`api.tools.register(name, def, fn)`** — many handlers per script, keyed by tool name. Fires when an LLM invokes the tool (either Council or inline function-calling).
- **`api.chat.registerContentProcessor(fn, opts?)`** — many handlers per script. Fires per processed message during chat-message rendering.
- **`api.worldInfo.registerInterceptor(fn, opts?)`** — many handlers per script. Fires during world-info activation.
- **`api.macros.registerInterceptor(fn, opts?)`** — many handlers per script. Fires during macro resolution.

### Many handlers, persist until script unregister OR explicit cleanup

- **`api.ui.dom.delegate(selector, event, fn, opts?)`** — event-delegated DOM listener on the host's chat area, scoped to a selector. Lives until script unregister, or until `api.ui.dom.cleanup()` is called (which tears down all of this script's DOM-related registrations).
- **`DOMHandle.on(event, fn, opts?)`** — per-element DOM event listener on a specific injected element. Lives until the parent `DOMHandle` is `.remove()`'d, or until script unregister.

## "Script unregister" includes Reload (v1.0.0-rc.8+)

Throughout this doc, "until script unregister" means **any of**:

- **User disables the script** in the script manager (via the per-script toggle).
- **User deletes the script.**
- **User clicks the editor's Reload button** — per-script state wipe, then body re-run. New as of v1.0.0-rc.8.
- **Autosave with `// @ls:reload-on-edit` directive** — same wipe + re-run path as the Reload button. New as of v1.0.0-rc.8.
- **LumiScript extension itself unloads** (extension toggle off, Lumiverse shutdown) — Spindle-level cascade.

The first two paths fire `ls:teardown` before wiping. The Reload-button and autosave-reload paths do NOT fire `ls:teardown` — they wipe state silently before re-running the body. From the handler's perspective the result is the same: the closure is gone, and any state needed for the next body run must be re-established.

The wipe covers all of LumiScript's pinning registries (tools, macros, injections, drawerTabs, inputBarActions, worldInfoInterceptors, messageProcessors, macroInterceptors, rpcEndpoints, floatWidgets, advancedModals, handlerClosures, userBroadcastSubs) AND broadcast subscriptions + command handlers. Preserved across the Reload wipe (NOT across disable/delete): `api.scriptStorage`, `api.theme.*` contributions, and the worker's `script.require()` cache. The full list lives in `src/backend.ts:wipeScriptStateForReload`.

Practical implication: scripts that combine interactive UI surfaces with background state MUST be re-fire-safe. The bottom-of-body re-registration idiom (next section) is the canonical pattern.

## Handler-fire vs body-fire

Inside the handler's closure body, the runtime shape differs from a body fire in a few specific ways:

| Thing | In body | In handler |
|---|---|---|
| `data` global | Event payload from Lumiverse | Still in lexical scope via closure capture, but **frozen to the registering body's payload** (stale-closure trap — see below). The handler's actual incoming payload arrives via its own declared function argument(s), not via `data`. |
| `script.id` / `name` / `type` | Bound at run start | Bound at registration time, still valid |
| `api.*` | Bound at run start | Captured by closure at registration, still dispatch-able |
| `runId` (visible only via diagnostics) | `run-<ts>-<seq>` | `handler-<ts>-<seq>` |
| `console.log`, `console.warn`, `console.error` | Captured + routed to editor console | Same |
| Active-context-aware reads (e.g. `api.variables.character.get(...)`, `api.db.collection({scope: 'character'})`) | Read against active context at run start | Read against the **current** active context at handler-fire time |

That last row is the one most worth knowing about. Scope-aware storage reads inside a handler resolve their scope at handler-fire time, not at registration time. So if your body registered a `broadcast.on` handler while character A was active, and the handler later fires while character B is active, `api.variables.character.get('x')` inside the handler returns character B's `x` — not character A's. Same for `api.db.collection({scope: 'character'})`. The proxy reads active-context live; it's not snapshotted at registration.

## What the closure captures

Standard JS closure semantics. Variables referenced from inside the handler closure are captured by *reference* to their lexical scope. That has a non-obvious consequence in the body-is-the-handler model:

```js
// Body, fire 1:
let counter = 0;

switch (data.__event) {
  case 'MESSAGE_SENT':
    counter += 1;
    break;
}

api.broadcast.on('peer:ping', () => {
  console.log(`saw ${counter} messages`);
});
```

When `peer:ping` fires later, what does the handler print? `0` — because each body fire has its own `counter` (re-declared `let counter = 0` at the top), and the handler captured the `counter` of whichever fire's body registered it. The next body fire creates a fresh `counter`, registers a fresh handler that captures it, but the OLD handler (if it weren't wiped — `broadcast.on` happens to be — or another handler type like `commands.onInvoked`) still references the OLD `counter`, which is gone.

The mitigation:

- **For broadcast handlers**, the wipe-and-re-register-on-every-body-fire cycle (next section) means the handler is always closed over the *current* body's locals. That's why the pattern works for `broadcast.on` — every fire installs a fresh closure capturing fresh state.
- **For handlers that persist longer** (`commands.onInvoked`, `tools.register`, `macros.register`, etc.), don't capture per-body mutable state in the closure. Read from the storage tiers ([Storage model](storage-model.md)) at handler-fire time:

```js
api.commands.onInvoked(async (commandId, ctx) => {
  // GOOD: read live state at handler-fire time.
  const counter = await api.variables.character.get('messageCount', 0);
  // ...
});
```

What's always safe to capture:

- `api` itself (proxy is bound to script identity, not to a body run).
- `script.id` / `name` / `type` (immutable for a given script).
- Constants and pure functions defined at the top of the body.

## The bottom-of-body re-registration idiom

`api.broadcast.on(...)` typically lives at the *very bottom* of the body — after the `switch`, after all the event-specific work. Three reasons:

1. **Subscriptions are wiped at the start of each body fire.** Re-registering at the bottom of every fire keeps them alive. (See the [LumiScript broadcast bus](../) memory note for the original rationale.)
2. **Bottom-of-body code runs on every fire, including `ls:startup` and `ls:teardown`.** That makes the subscription self-healing on extension re-enable, since `ls:startup` fires after enable and re-runs the bottom-of-body code.
3. **Closure captures whatever was defined above.** If your body declares helper functions or constants up top and then references them from inside the `broadcast.on` handler, putting the registration at the bottom guarantees the references resolve to the current fire's definitions.

The canonical shape:

```js
// ── Top of body (helpers + always-on registrations) ───────────────────────

const PEER_EVENT = 'my-tracker:state-changed';

async function handlePeerEvent(payload) {
  await api.variables.chat.set('lastSeenState', payload);
}

api.macros.register('myMacro', { /* def */ }, async (ctx) => { /* ... */ });

// ── Switch (event-specific work) ─────────────────────────────────────────

switch (data.__event) {
  case 'MESSAGE_SENT':
    await reactToUserMessage(data.message);
    break;
  case 'GENERATION_ENDED':
    await reactToAssistantMessage(data.messageId);
    break;
}

// ── Bottom of body (re-registered on every fire) ─────────────────────────

api.broadcast.on(PEER_EVENT, (payload) =>
  handlePeerEvent(payload).catch(err =>
    console.warn(`[my-script] peer-event handler failed: ${err.message}`)
  )
);
```

`api.macros.register`, `api.tools.register`, `api.commands.onInvoked`, and the rest of the "persist until script unregister" group don't *need* this pattern — re-registering them every fire is a no-op (or a same-id-replace) — but many scripts put them up top anyway for symmetry and because top-of-body code also runs on every fire.

## Permission denial in long-lived handlers

Same try/catch shape as in [Permissions](permissions.md), but the stakes are higher here: an uncaught permission-denial inside a registered handler surfaces as an unhandled rejection *every time the handler fires*. With handlers that fire frequently (DOM event listeners, broadcast subscribers with chatty publishers), that's editor-console spam at best and rate-limited-and-dropped at worst (see the unhandled-rejection rate limit — 10 rejections per 60 s per script, then silent drops). Always wrap gated calls inside handlers:

```js
api.broadcast.on('something:happened', async (payload) => {
  try {
    await api.images.upload({ data: payload.bytes, mimeType: 'image/png' });
  } catch (err) {
    if (err.message.startsWith('PERMISSION_DENIED:images')) {
      console.warn('[my-script] images not granted; ignoring upload');
      return;
    }
    throw err;
  }
});
```

## Anti-patterns

- **Registering handlers inside `switch` cases.** `api.broadcast.on(...)` placed inside `case 'MESSAGE_SENT':` only re-registers on `MESSAGE_SENT` fires. If the next fire is anything else (say, `GENERATION_ENDED`), the broadcast subscription was wiped at the start of that fire and never re-registered, so the script silently loses its subscription until the next user message. Keep `broadcast.on` registrations at the bottom (or top) of the body, outside the switch.
- **Capturing per-body mutable state in closures of handlers that persist past one fire.** The classic stale-closure trap. Read from storage tiers at handler-fire time instead.
- **Capturing references to entities (chat, character, message) at registration time.** Those entities can be deleted; references go stale silently. Capture *IDs*, then `await api.characters.get(id)` / `api.chats.get(id)` / `api.chat.getMessages()` at handler-fire time.
- **Forgetting `broadcast.on`'s wipe-on-next-fire semantics.** A common assumption is "I registered it once, it's there forever." It isn't. The other persist-until-unregister surfaces don't have this property; broadcast does. Lean on the bottom-of-body idiom.
- **Not handling permission denials inside handlers.** Surfaces as recurring unhandled rejections per the section above.

## What's next

You now have the full conceptual surface for writing non-trivial scripts: the [Trigger model](trigger-model.md), [Permissions](permissions.md), the [Storage model](storage-model.md), and handler lifetime. From here:

1. **[DOM injection](../guides/dom-injection.md)** — building UI inside the host app shell. The single highest-leverage capability if you want LumiScript to feel like a first-class part of Lumiverse.
2. **The in-app Reference's API Functions section** — for the method signatures of every handler-registration surface in this doc.
3. **[The cookbook](../cookbook/index.md)** — common script shapes worked through end-to-end.
