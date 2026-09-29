# Share state between two scripts

Have one script produce state and another react to it — in real time, without either script knowing about the other's triggers. This is the backbone pattern for "a worker computes something, a UI script displays it," and it sidesteps the classic co-triggered-scripts race.

## What you'll use

- [`api.broadcast.*`](../guides/broadcast.md) — `emit` to announce a change, `on` to react. In-process, synchronous, **no permission**.
- `api.scriptStorage.*` — a free, session-scoped key/value store the two scripts share. ([Storage model](../concepts/storage-model.md))
- Two triggers wired in the editor: `MESSAGE_SENT` (producer) and `CHAT_SWITCHED` (consumer cold-start). ([Trigger model](../concepts/trigger-model.md))

The key idea up front: **`api.broadcast.on` subscriptions outlive the trigger run that registered them.** So a consumer needs *some* trigger to fire once (to run its body and subscribe), and after that its handler fires every time the producer emits — no matching trigger required.

## The scripts

Two separate scripts. The producer computes a metric per message and announces it; the consumer reacts to every announcement.

**Producer** — wire it to `MESSAGE_SENT`:

```js
// @triggers MESSAGE_SENT
// Producer: derive a metric from each USER message, store it, announce the change.
// No permission required.

// MESSAGE_SENT also fires for the assistant's empty placeholder — skip it.
if (!data.message?.is_user) return;

const text  = data.message?.content ?? '';
const words = text.trim() ? text.trim().split(/\s+/).length : 0;

// Write the shared state FIRST...
await api.scriptStorage.set('msg-metric', { words, at: Date.now() });

// ...THEN announce it. Emitting after the write is what makes the consumer race-free.
api.broadcast.emit('myapp:metric-updated', { source: 'message-sent' });
```

**Consumer** — wire it to `CHAT_SWITCHED` (just so its body runs once to subscribe):

```js
// @triggers CHAT_SWITCHED
// Consumer: subscribe once; the handler then fires on every producer emit.
// No permission required.

// Subscribe at the top of the body, unconditionally — the subscription survives
// between this script's own trigger fires.
api.broadcast.on('myapp:metric-updated', (payload) => {
  // Handlers run synchronously inside emit; do async work in a self-contained IIFE.
  void (async () => {
    try {
      const metric = await api.scriptStorage.get('msg-metric');
      console.log(`[consumer] ${metric?.words ?? '?'} words (via ${payload?.source})`);
    } catch (err) {
      console.warn(`[consumer] handler failed: ${err.message}`);
    }
  })();
});

// Cold-start path: render whatever's already stored when the chat opens.
if (data.chatId) {
  const metric = await api.scriptStorage.get('msg-metric');
  console.log(`[consumer] (cold start) ${metric?.words ?? 'nothing yet'}`);
}
```

Enable both, open a chat (the consumer cold-starts), and send messages. Each send: the producer stores + emits, and the consumer's handler logs the new metric — even though no `MESSAGE_SENT` trigger ever fires the consumer.

## How it works

**Emit *after* the write — that's the whole point.** The naïve version wires both scripts to `MESSAGE_SENT` and has the consumer read storage directly. But Lumiverse fires co-triggered scripts concurrently: the consumer's read can complete before the producer's `await … .set(…)` resolves, so it reads stale state. Moving the read into a broadcast handler and emitting *after* the write resolves orders them deterministically — the handler can't run until `emit` is called, and `emit` is called only after the write lands.

**Subscriptions persist between fires.** When the consumer's body runs (on `CHAT_SWITCHED`), `api.broadcast.on(...)` registers the handler — and it stays registered. The host wipes a script's subscriptions at the **start** of that script's *next* run, not the end of the current one, so the handler lives in the window between fires and catches every producer emit. That's why the consumer only needs one cold-start trigger.

**`scriptStorage` is the shared channel; the broadcast is the doorbell.** The payload (`{ source }`) is a lightweight notification, not the data — the actual state rides in `api.scriptStorage`, which both scripts can read. Keep payloads small (there's a 1 MiB cap and a 100-emit/sec rate limit) and let storage carry the weight.

**Async handlers need the IIFE + try/catch.** `emit` calls handlers synchronously and doesn't await them. A bare `async` handler that throws becomes an unhandled rejection that escapes the bus's per-handler error catching. The `void (async () => { try { … } catch { … } })()` wrapper keeps the async work and its errors contained.

## Make it yours

- **Drive a UI panel.** Replace the consumer's `console.log` with `content.update(...)` on a [draggable panel](draggable-panel.md) — now the producer's work refreshes the panel live. (Capture the unsubscribe `on()` returns and call it when the panel closes.)
- **Richer producers.** The producer's "work" can be anything async: an `api.llm.generateStructured` extraction, an `api.utils.http` fetch, an `api.db` write. Emit once it resolves; the consumer doesn't care what the work was.
- **Multiple event kinds.** Use a `source` field (`'message-sent'`, `'user-edit'`, `'rollback'`) so consumers can render selectively — flash on one source, ignore another.
- **More than two scripts.** `emit` fans out to *every* subscriber across all your scripts. A second consumer (a logger, an analytics sink) just subscribes to the same event — the producer is unaffected.

## Gotchas

- **Subscribe unconditionally at the top of the body.** Wrapping `api.broadcast.on(...)` in an `if` that's only sometimes true means the subscription vanishes on the next wipe-and-replay. If you need conditional behaviour, put the condition *inside* the handler, not around the subscription.
- **Emitting an `ls:` name throws.** That prefix is reserved for engine-internal broadcasts. Namespace yours with your script's name (`myapp:`, `<script>:`). You *can* subscribe to `ls:*` events, though — see [Observing engine state](../guides/broadcast.md#observing-engine-state-via-ls).
- **Non-`ls:*` subscriptions pin the worker.** A live cross-script subscription keeps the consumer's worker warm (so emits don't drop into an evicted worker). That's intended — but call the captured unsubscribe function when you're truly done, or the worker stays pinned indefinitely.
- **`scriptStorage` is session-scoped, not durable.** It survives worker eviction, hot reload, and script edits, but is cleared on a full backend restart. For state that must outlive a restart, use `api.variables.*` or `api.db.*` instead — see [Storage model](../concepts/storage-model.md).

## See also

- [Broadcasting events between scripts](../guides/broadcast.md) — the full `api.broadcast` surface, the subscription-lifecycle timeline, and the pinning policy.
- [Storage model](../concepts/storage-model.md) — picking the right tier for the shared state (`scriptStorage` vs `variables` vs `db`).
- [Build a draggable info panel](draggable-panel.md) — a natural consumer for the emitted updates.
