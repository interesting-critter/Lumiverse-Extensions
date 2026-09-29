# Broadcasting events between scripts

`api.broadcast.*` is the in-process pub/sub bus that lets two LumiScript scripts coordinate. Use it when one script produces state and another renders it, when a worker script wants to notify a UI script that an async operation completed, or when you want to decouple co-triggered scripts from same-trigger races.

The bus is in-process, synchronous on emit, and requires no permission. It complements Lumiverse triggers rather than replacing them — you still need a trigger to fire each script at least once so its body can register subscriptions, but once subscribed, a handler can fire between trigger runs in response to another script's emit.

## The big idea

`api.broadcast.emit(event, payload)` fires synchronously to every subscriber matching the event name, across all LumiScript scripts.

`api.broadcast.on(event, handler)` subscribes a handler and returns an unsubscribe function. **Subscriptions persist between trigger fires** — they're wiped at the start of the script's *next* run, not the end of the current one. That's the key behaviour: a script can listen for events from any other script without needing its own Lumiverse trigger to fire each time.

## Quick start

Two scripts that talk:

**`producer.js`** — does async work after generation, emits when the result lands:

```js
// @triggers GENERATION_ENDED
// @description Producer: extracts state, emits when ready.

const extracted = await extractFromContext(data);
await api.scriptStorage.set('latest-state', extracted);

api.broadcast.emit('myapp:state-changed', { source: 'extraction' });
```

**`viewer.js`** — subscribes at the top of its body; the subscription survives between fires:

```js
// @triggers CHAT_SWITCHED
// @description Viewer: re-renders on emit; cold-start on chat-open.

api.broadcast.on('myapp:state-changed', () => {
  void (async () => {
    try {
      const state = await api.scriptStorage.get('latest-state');
      rerender(state);
    } catch (err) {
      console.warn(`[viewer] rerender failed: ${err.message}`);
    }
  })();
});

// Cold-start path — render whatever's in storage when the chat just opened.
if (data.chatId) {
  const state = await api.scriptStorage.get('latest-state');
  rerender(state);
}
```

`viewer.js`'s handler fires every time `producer.js` emits — even though no Lumiverse trigger fired the viewer. The two scripts are decoupled: the producer doesn't know who listens, the viewer doesn't know what triggered the producer.

## Surface

```ts
interface BroadcastAPI {
  emit(event: string, payload?: unknown): void;
  on(event: string, handler: (payload: unknown) => void): () => void;
}
```

Both methods are sync. No permission required. There's no `off()` — unsubscribe by calling the function `on()` returns.

### `emit(event, payload?)`

Fire-and-forget. The bus iterates current subscribers in registration order and calls each handler synchronously. Per-handler errors are caught and logged so one bad handler doesn't break sibling handlers. Returns void.

- **`event`** — the event name. Use a namespace prefix to avoid cross-script collisions (`myapp:state-changed`, `tracker:rerun`). Names starting with `ls:` are reserved (see [Reserved `ls:*` prefix](#reserved-ls-prefix) below); emitting one throws.
- **`payload`** — optional. Any structured-clone-compatible value (strings, numbers, plain objects, arrays). Don't include non-serialisable types like functions or class instances if you expect handlers in other scripts to receive them with full fidelity.

### `on(event, handler)`

Subscribe a handler to an event. Returns an unsubscribe function.

- **`event`** — the event name. You CAN subscribe to `ls:*` events (the emit-side restriction is one-way); see [Observing engine state via `ls:*`](#observing-engine-state-via-ls).
- **`handler`** — called with the `payload` from the emit. Return value is ignored. Handlers fire synchronously inside `emit` — see [Async handlers](#async-handlers) for the standard async pattern.

```js
const unsub = api.broadcast.on('myapp:state-changed', (payload) => {
  console.log(payload?.source);
});

// Later, when the subscription is no longer needed:
unsub();
```

### Limits

`emit` is rate-limited and payload-capped at the api-proxy boundary:

- **Payload cap**: 1 MiB (`1_048_576` bytes) per emit. Over-cap emits throw with a message naming the limit.
- **Rate limit**: 100 emits/sec sustained, 1000-emit burst (token-bucket per script). Bursts above the sustained rate spend the burst budget; exhaust it and subsequent emits throw until the bucket refills.

The caps are per-script — multiple scripts can each hit the limit independently without affecting each other. For most use cases (chat-event reactions, periodic state pings) you'll never approach them; if you do, you're probably hot-looping inside a handler and should rethink the pattern.

## Subscription lifecycle

This is the part to internalise. Subscriptions live longer than a single trigger run. They persist across fires for the script's lifetime, and the host wipes them at the **start** of the script's next trigger run, not the end of the current one.

The timeline:

1. Trigger fires for script A.
2. Host wipes A's subscriptions from the bus (`busClearByScriptId` in `src/script-runner/host-dispatcher.ts`, before the body runs).
3. A's body executes. Any `api.broadcast.on(...)` call re-adds the subscription.
4. A's body finishes. **Subscriptions stay in the bus untouched.**
5. Script B emits an event A subscribes to. A's handler fires.
6. Another trigger fires for A → wipe → body re-runs → re-subscribes.

That window between step 4 and the next wipe is the whole point — A's handler can fire any number of times in response to B's emits without A's own trigger firing. The wipe is at run-*start* (step 2) rather than run-*end* (step 4) so a subscription registered in run *N* survives until run *N+1* starts.

### What this enables

- **A subscriber-only script.** Script B can subscribe to `myapp:state-changed` without subscribing to whatever trigger fires script A. B just needs *some* trigger to fire once — a minimal `CHAT_SWITCHED` cold-start, for instance — to get its body running so it can register the subscription. After that, B's handler fires whenever A emits, indefinitely.
- **Race-free coordination.** Two scripts both subscribing to `GENERATION_ENDED`, where one does an async write and the other does a sync read, race the read against the write. Move the read into a broadcast handler and emit from the writer *after* the write resolves — race eliminated. Worked example below.

## Decoupling co-triggered scripts

The motivating case: an `extractor.js` script subscribes to `GENERATION_ENDED` and runs an async LLM extraction (multi-second network call) that writes to `api.db.*`. A companion `panel.js` also subscribes to `GENERATION_ENDED` to read that state and re-render a visible panel. Lumiverse fires both concurrently; `panel.js`'s read completes in milliseconds while `extractor.js`'s write doesn't complete for seconds. Result: the panel renders pre-extraction state and never updates.

Fix — emit-after-write in the producer, subscribe in the consumer:

```js
// extractor.js — emit after each state mutation
async function runExtraction(...) {
  const events = await llmExtract(...);
  await insertEvents(events);
  api.broadcast.emit('myapp:state-changed', { source: 'llm-extraction' });
}
```

```js
// panel.js — subscribe at top of body; no GENERATION_ENDED dependency
api.broadcast.on('myapp:state-changed', () => {
  void (async () => {
    try { await rerender(); }
    catch (err) { console.warn(`[panel] ${err.message}`); }
  })();
});

// Cold-start path — render whatever's in storage when the chat just opened.
if (data.chatId) {
  await rerender();
}
```

The viewer's trigger list shrinks to just `CHAT_SWITCHED` (for cold-start). The broadcast handler covers all post-generation updates without the race.

## Reserved `ls:*` prefix

Event names starting with `ls:` are reserved for engine-internal broadcasts. Calling `emit('ls:anything', ...)` from a user script throws:

```
Error: api.broadcast.emit: "ls:" prefix is reserved for LumiScript internal events. Use a different event name.
```

(Enforced at `src/engine/api/broadcast.ts:34-40`.)

Scripts CAN subscribe to `ls:*` events — the restriction is emit-only.

### Observing engine state via `ls:*`

The engine emits several `ls:*` events you can subscribe to for observability and cross-script reactions. The tool-lifecycle events (`ls:tool:registered`, `ls:tool:unregistered`, `ls:tool:invoked`) are documented in [Registering tools](tools.md); other engine events include `ls:startup` (fired when a script enters the active state — on boot or enable) and database / theme events. For the full current list with payload shapes, see the in-app Reference's "Broadcast Events" section.

Typical use cases for subscribing to `ls:*`:

- **Cross-script observability** — a debugging script logs every `ls:tool:invoked` for a session-wide tool-fire audit.
- **Re-init on engine startup** — a script that needs to re-establish DOM bindings or open connections on every cold-start subscribes to `ls:startup`.
- **React to other scripts' database mutations** — a viewer subscribes to relevant `ls:db:*` events to refresh without polling.

## Async handlers

`emit` is synchronous — the bus iterates handlers and calls each one synchronously. If your handler is async, **don't `await` it inside emit** (emit doesn't await handlers anyway, and a top-level `await` in the handler creates an unhandled rejection if it throws).

The standard pattern is an IIFE-wrapped async block with its own try/catch:

```js
api.broadcast.on('my:event', () => {
  void (async () => {
    try {
      await doAsyncWork();
    } catch (err) {
      console.warn(`[my-script] handler failed: ${err.message}`);
    }
  })();
});
```

The inner try/catch is load-bearing. Without it, the async error becomes an unhandled rejection that doesn't reach the bus's own per-handler error-catching, so you lose the per-script attribution in logs.

## Pinning policy

(Cross-link: see [`concepts/handler-lifetime.md`](../concepts/handler-lifetime.md) for the full eviction-pinning surface.)

Worker eviction-pinning has a per-script policy that asks "does this script have load-bearing background state?". Non-`ls:*` broadcast subscriptions count — a forwarded broadcast into a dead worker silently drops, so the policy pins the worker to keep it alive for future cross-worker events.

- **Non-`ls:*` subscriptions PIN.** The worker stays warm until you explicitly call the unsubscribe function returned by `on()`, or the script is disabled / deleted.
- **`ls:*` subscriptions DON'T PIN.** Engine-lifecycle events only fire as side-effects of local activity inside the worker — by definition, the worker is alive when they fire. Pinning on `ls:*` subs would over-pin every script that listens for `ls:startup`.

Practical implication: a script that combines an `ls:startup` re-init handler with a cross-script user-event handler is pinned by the latter. A script with *only* `ls:*` subscriptions can be evicted between fires when idle.

## Conventions

### Namespacing

Use a `<scriptName>:<verb>` shape for event names:

- `tracker:state-changed`
- `weather-provider:cache-invalidated`
- `myapp:filters-applied`

This makes it obvious in logs where an event originated and avoids collisions when two unrelated scripts both want a `state-changed` event.

### `source` field on payloads

By convention, include a `source` field identifying what kind of activity produced the event:

```js
api.broadcast.emit('tracker:state-changed', { source: 'llm-extraction' });
api.broadcast.emit('tracker:state-changed', { source: 'user-edit' });
api.broadcast.emit('tracker:state-changed', { source: 'rollback' });
```

Subscribers don't have to read it, but it's a free debugging surface and lets future viewers do source-selective rendering (flash on extraction, skip on user-edit, etc.).

## Common pitfalls

- **Conditional subscriptions evaporate on the next wipe.** Wrapping `api.broadcast.on(...)` in an `if` that's only true on some fires means the subscription disappears on the next trigger run. Either subscribe unconditionally at the top of the body, or accept the conditional-lifetime semantic explicitly.

- **`emit` doesn't await async handlers.** The handler runs synchronously inside emit; any async work inside runs detached. There's no return value to collect across handlers.

- **Emitting `ls:*` throws.** If you mean to publish an internal-namespace event from one of your scripts, pick a non-`ls:*` prefix like `myapp:` or `<scriptName>:internal:`.

- **Handlers fire in subscription order, not script-name order.** Don't rely on ordering across scripts. Across worker restarts the order can flip.

- **Cross-worker broadcasts cross IPC.** If two scripts run in different worker subprocesses, emits are forwarded via host IPC. Forwarding into a dead worker silently drops — the pinning policy is what prevents this in practice (so non-`ls:*` subs keep their target worker alive). Worth knowing if you ever see a "ghost subscription" not firing.

- **Subscriptions don't survive script-source edits, but the wipe-and-replay model makes that invisible.** Editing the body and saving doesn't restart the worker; the next trigger fire wipes-and-re-registers naturally. As long as your new body re-registers what it needs, the transition is clean.

## When to use broadcast vs. other event surfaces

LumiScript has four event-shaped surfaces. They overlap visually but solve different problems:

| Need | Use | Permission |
|---|---|---|
| React to Lumiverse host events (MESSAGE_SENT, GENERATION_ENDED, CHAT_SWITCHED, etc.) | Editor-UI event-picker wiring on the script — see [Trigger model](../concepts/trigger-model.md). The script body runs as the handler when a wired event fires | Free |
| Real-time script-to-script pub/sub between LumiScript scripts in the same install | `api.broadcast.*` — what this guide covers | Free |
| Persistent log of custom events that can be queried later (audit trails, state recovery, analytics) | `api.events.*` — see [Persistent events](persistent-events.md) | `event_tracking` |
| Cross-extension RPC where another Lumiverse extension reads named values from your script | `api.rpc.*` — see [Cross-extension RPC](rpc.md) | Free |

**The category-error to avoid**: `api.events.*` is NOT a subscription surface. It's a persistent log with `track` / `query` / `replay` / `getLatestState`. Writing `api.events.on(...)` is a common confusion — it doesn't exist. For event SUBSCRIPTION inside LumiScript, the choices are exactly two: the editor-picker (for HOST events) and `api.broadcast.on` (for SCRIPT-emitted custom events).

## See also

- **In-app Reference, "API Functions → api.broadcast" section** — auto-generated method list.
- **[`concepts/handler-lifetime.md`](../concepts/handler-lifetime.md)** — the full eviction-pinning model; where the `ls:*` vs non-`ls:*` policy lives in context.
- **[`concepts/trigger-model.md`](../concepts/trigger-model.md)** — when scripts fire and what `data.__event` looks like. Broadcasts complement triggers; they don't replace them.
- **[Registering tools](tools.md)** — uses `ls:tool:*` broadcasts as the observability surface for tool registry changes.
