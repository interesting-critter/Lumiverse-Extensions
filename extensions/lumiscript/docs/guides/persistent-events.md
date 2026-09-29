# Persistent events

`api.events.*` is a **persistent log of custom events** your script writes (`track`) and reads back (`query` / `replay` / `getLatestState`). The host stores each event with a server-side timestamp, optional chat association, and severity level. The log survives across script reloads, extension toggles, Lumiverse restarts — anywhere your script needs durable, queryable history.

Four methods:

```ts
interface EventsAPI {
  track(eventName: string, payload?: Record<string, unknown>, options?: EventTrackOptions): Promise<void>;
  query(filter?: EventQueryFilter):  Promise<EventRecord[]>;   // newest-first
  replay(filter?: EventQueryFilter): Promise<EventRecord[]>;   // oldest-first
  getLatestState(keys: string[]):    Promise<Record<string, unknown>>;
}
```

All four require the `event_tracking` permission.

## What this is NOT

This is the most-confused namespace name in LumiScript, so let's get the disambiguation out of the way first:

| `api.events.*` | NOT |
|---|---|
| A persistent log you write to (`track`) and read back (`query` / `replay`) | A subscription mechanism for Lumiverse host events (MESSAGE_SENT, GENERATION_ENDED, etc.) |
| For audit trails, state recovery, analytics-style observability | For reacting to chat / generation events |

If you want to **subscribe to host events** (MESSAGE_SENT, CHAT_SWITCHED, etc.), wire your script via the editor's event-picker — see the [Trigger model](../concepts/trigger-model.md). There is no `api.events.on()` or `api.events.subscribe()`; those would be a category error. The script body itself runs as the handler when a wired event fires.

If you want **inter-script messaging** (script-to-script pub/sub between scripts running inside the same LumiScript extension), use `api.broadcast.*` — see the [Broadcast bus guide](broadcast.md).

`api.events.*` is exclusively for **scripts recording their own custom events** that need to be queryable later. Think of it as a structured, persistent `console.log` with an HTTP-shaped log query API on top.

## Surface

```ts
type EventLevel = 'debug' | 'info' | 'warn' | 'error';

interface EventTrackOptions {
  level?:         EventLevel;       // Default 'info'
  chatId?:        string;            // Default: active chat
  retentionDays?: number;            // Default: host-side default
}

interface EventQueryFilter {
  eventName?: string;
  chatId?:    string;
  since?:     string;                // ISO 8601 timestamp
  until?:     string;                // ISO 8601 timestamp
  level?:     EventLevel;
  limit?:     number;
}

interface EventRecord {
  id:        string;
  ts:        string;                 // ISO 8601 timestamp
  eventName: string;
  level:     EventLevel;
  chatId?:   string;
  payload?:  Record<string, unknown>;
}
```

## Permissions

All four methods require **`event_tracking`**. The permission gate is in the LumiScript wrapper itself — when missing, all methods throw:

```
Error: api.events requires the event_tracking permission
```

**Heads up — different error format from other LS APIs.** Most LumiScript permission gates use `assertPerm()` and emit `Error: PERMISSION_DENIED:<perm> — grant this permission to use this API`. `api.events` throws its own message instead (the wrapper guards inline rather than via the shared helper). Don't `msg.includes('PERMISSION_DENIED')` to detect it — match on the literal `'requires the event_tracking permission'` substring, or just on the string `'event_tracking'`.

## Quick start

Track a few events from a chat-driven trigger, then query them back:

```js
// @triggers MESSAGE_SENT
// @description Track every user message length for an analytics dashboard.

if (!data.message.is_user) return;

await api.events.track('user-message', {
  length:    data.message.content.length,
  hasMention: /@\w+/.test(data.message.content),
});
```

Later — from a different script, or the same one on a different trigger — read them back:

```js
// @triggers ls:startup

// Newest 50 user-message events from the active chat:
const recent = await api.events.query({
  eventName: 'user-message',
  limit:     50,
});

const totalChars = recent.reduce((sum, e) => sum + (e.payload?.length ?? 0), 0);
console.log(`User has typed ${totalChars} chars across the last ${recent.length} messages`);
```

`track` is fire-and-forget from the caller's perspective — it returns a Promise that resolves after the host persists the event, but you don't need to do anything with it beyond awaiting (which you should, to surface persistence errors).

## `query` vs `replay` — choose by direction

The two read methods differ only in **ordering**:

| Method | Returns | Use for |
|---|---|---|
| `query(filter?)` | Newest-first | Pull recent activity, paginate from the present backward, "last N events" patterns |
| `replay(filter?)` | Oldest-first (chronological) | Reconstruct sequenced state, replay history forward, "from-the-beginning" walks |

Filter shape is identical. Both respect `limit` (default is host-side, typically 100). For large histories, prefer `replay` with explicit `since` / `until` windows over unbounded `query` — chronological replay over a known window is cheaper to reason about than ad-hoc newest-first scans.

```js
// Reconstruct quest state from the very beginning of the chat:
const activeChat = await api.chats.getActive();
if (!activeChat) return;

const allQuests = await api.events.replay({
  eventName: 'quest-updated',
  chatId:    activeChat.id,
});

let questState = {};
for (const event of allQuests) {
  questState = { ...questState, ...event.payload };
}
```

## `getLatestState` — fast resume after restart

A common pattern: your script tracks state changes incrementally via `track`, then on startup wants to re-derive the latest known value for each key without replaying the entire history. `getLatestState(keys)` returns just that — a `Record<string, unknown>` mapping each requested event name to its most-recent payload:

```js
// @triggers ls:startup

const state = await api.events.getLatestState([
  'player-hp',
  'player-mp',
  'quest-active',
]);

console.log(`Resumed: HP=${state['player-hp']?.value}, MP=${state['player-mp']?.value}`);
```

`getLatestState` is keyed by **event name**, not chat. The value you get back is whatever payload was most recently `track()`'d under that name across the entire history. If you need chat-scoped latest-state, fold the chat ID into the event name (`player-hp:${chatId}`) or filter via `query` with `chatId` + `limit: 1`.

Use this over an unbounded `query` or `replay` when:
- You only care about the **current** value, not the trajectory
- You're restoring state at boot / after a script reload
- You have many tracked events and want O(N keys) lookup instead of O(N history) scan

## Event levels — pick deliberately

The four levels (`'debug' | 'info' | 'warn' | 'error'`) work like a logging framework's severity tiers. They have no behavioural effect on whether or how events are stored — every level persists identically. The level field is **for filtering and surfacing** in query results.

Conventional usage:
- **`debug`** — high-volume "what is the script doing right now" trace events. Useful during development; consider lower `retentionDays` (e.g. 1–3) so they self-clean.
- **`info`** — the default. Normal application events.
- **`warn`** — recoverable issues (a fallback path was taken, an unexpected input was massaged).
- **`error`** — failures the script wants to surface in audit / debug-tooling views.

A script that has a "show me recent warnings" debug button would query with `level: 'warn'` to filter out routine `info` noise.

## Chat association

The optional `chatId` on both `track` and the filter is the canonical way to scope events to a particular chat:

- **Omit on `track`** → defaults to the active chat at the moment of the call (via the engine's per-fire active-context view). For events fired from `MESSAGE_SENT`, `CHAT_SWITCHED`, etc., this is almost always what you want.
- **Pass an explicit `chatId`** → useful when you're cross-chat (e.g. a script that aggregates stats across all chats) and want to explicit attribution rather than relying on the active-chat default.

**Heads up — there's no clean way to track an explicitly chat-less event from inside an active-chat context.** The wrapper at `events.ts:36` uses nullish coalescing: `options?.chatId ?? activeContext.chatId ?? undefined`. Passing `chatId: undefined` explicitly is indistinguishable from omitting the option — both fall through to the active-chat default. The only way to get a chatId-less event today is to fire `track` from a context where there IS no active chat (e.g. a startup handler before a chat is open).

On the read side, `filter.chatId` matches only events stored with that exact chatId. Events tracked when no chat was active (`activeContext.chatId === null`) end up with no chat association and won't match a chat-filtered query.

## Retention

`retentionDays` is an optional auto-expire hint on the host. The host applies a default retention policy (set per Lumiverse install, typically generous — events live indefinitely unless explicitly capped). Use `retentionDays` when:

- High-volume `debug`-level events would otherwise accumulate forever — set 1–3 days to keep the working set small.
- A specific event class is auditable but inherently transient (e.g. session-scoped diagnostics).
- You want to be a good citizen on shared hosts where storage growth is a concern.

Don't set `retentionDays` for events you might need historically — there's no API to extend retention after the fact, and expired events are gone.

## Use cases

### State recovery on restart

```js
// @triggers ls:startup, MESSAGE_SENT

const KEY = 'session-counter';

if (data.__event === 'ls:startup') {
  const state = await api.events.getLatestState([KEY]);
  globalThis.sessionCount = state[KEY]?.value ?? 0;
  console.log(`Resumed at count=${globalThis.sessionCount}`);
}

if (data.__event === 'MESSAGE_SENT' && data.message.is_user) {
  globalThis.sessionCount = (globalThis.sessionCount ?? 0) + 1;
  await api.events.track(KEY, { value: globalThis.sessionCount });
}
```

The script body's `globalThis.sessionCount` survives within a worker subprocess lifetime, but `getLatestState` is what makes it survive a Lumiverse restart.

### Audit trail for sensitive operations

```js
// @triggers ls:startup

// Wrap any destructive operation in an audit-track call:
async function deleteWithAudit(databankId, reason) {
  await api.events.track('databank-deleted', {
    databankId,
    reason,
    deletedBy: globalThis.__lsActiveUserId,
  }, { level: 'warn' });

  return api.databanks.delete(databankId);
}
```

Now `api.events.query({ eventName: 'databank-deleted' })` is the audit log; events persist independently of script lifetime so even disabled-then-reenabled scripts retain their history.

### Analytics-style observability

```js
// @triggers GENERATION_ENDED

await api.events.track('generation-completed', {
  contentLength: data.content?.length ?? 0,
  generationId:  data.generationId,
  messageId:     data.messageId,
}, { level: 'info' });
```

(Note: `GENERATION_ENDED`'s payload is `{ generationId, chatId, messageId, content }` — no `model` field. Pair with a `GENERATION_STARTED` tracker if you also need the model, since `GENERATION_STARTED`'s payload is `{ generationId, chatId, model }`. Join the two by `generationId` at query time.)

A separate "dashboard" script can `replay` these events and roll them up by hour / chat / model to surface usage patterns.

## When NOT to use this

| Need | Use |
|---|---|
| Current value of a config / setting | `api.scriptStorage` or `api.variables.*` (cheaper, faster, no permission needed) |
| Inter-script messaging (script A wants to notify script B in real-time) | `api.broadcast.*` |
| Reacting to host events (MESSAGE_SENT etc.) | Editor's event-picker wiring (NOT `api.events.*`) |
| Encrypted secrets at rest | `api.enclave.*` |
| Per-key set/get without history | `api.scriptStorage` |
| Per-script JSON micro-DB with operators | `api.db.*` |

`api.events.*` is the right answer when you need **persistent history with structured query**. If you only ever read the latest value and don't care about the trajectory, scriptStorage / variables are simpler and don't need a permission grant.

## Error handling

The most-common failure surfaces:

| Cause | Error |
|---|---|
| Permission denied | `Error: api.events requires the event_tracking permission` |
| Host validation failure on `track` (malformed payload, oversized fields) | Whatever the host emits — message varies |
| Network / host unavailability | Bubble up from `spindle.events.*` |

Robust pattern:

```js
try {
  await api.events.track('my-event', payload);
} catch (err) {
  const msg = err instanceof Error ? err.message : String(err);
  if (msg.includes('event_tracking')) {
    // Permission missing — fall back to scriptStorage or skip the audit.
    await api.scriptStorage.set('my-event-fallback', payload);
    return;
  }
  console.error('[my-script] events.track failed:', err);
}
```

Note the `msg.includes('event_tracking')` form — the substring `'event_tracking'` is reliable across both the custom error message (`requires the event_tracking permission`) and any host-side messages that mention the permission name.

## Version notes

Implemented in **v0.11.0+** via native `spindle.events.*`. All four methods (`track`, `query`, `replay`, `getLatestState`) have been stable since introduction. The LumiScript wrapper is a thin pass-through with permission gating and chat-default plumbing on `track`.
