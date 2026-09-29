# Cross-extension RPC

`api.rpc.*` lets your script publish lightweight state on a named channel that **other Lumiverse extensions** can read. It wraps Spindle's `rpcPool` — a host-side registry of endpoints any extension on the install can `read` from. Use it to expose script-owned data (state snapshots, computed values, on-demand answers) to other extensions running in the same Lumiverse install, without setting up your own HTTP server or coordinating via files.

Four methods, no permission required:

```ts
interface RpcAPI {
  sync<T>(channel: string, value: T, options?: { as?: string; policy?: RpcPolicy }): Promise<string>;
  handle<T>(channel: string, handler: (ctx: RpcRequestContext) => T | Promise<T>, options?: { as?: string; policy?: RpcPolicy }): Promise<string>;
  read<T>(endpoint: string): Promise<T>;
  unregister(channel: string, options?: { as?: string }): Promise<void>;
}
```

`sync` and `handle` both return the **fully-qualified endpoint string** (e.g. `'lumiscript.tracker.state'`) so you can log it or pass it onward without recomputing the prefix.

## The big idea

Spindle's RPC pool is a host-side keyspace that every extension can publish to + read from. LumiScript wraps it with two-tier namespacing so user-scripts inside the same LumiScript install don't collide:

```
lumiscript.<scriptSlug>.<channel>
```

- **`lumiscript.`** — added by Spindle automatically (the extension identifier prefix).
- **`<scriptSlug>`** — added by LumiScript, derived from your script's name (slugified to `[a-z0-9_-]+`). Override via `options.as` when the auto-derived form is unsuitable.
- **`<channel>`** — the part you choose (can contain `.` for multi-segment paths).

So a script named "Story Tracker" calling `api.rpc.sync('state', currentState)` publishes to `lumiscript.story-tracker.state`. Two scripts with different names publishing the same `'state'` channel end up on distinct endpoints — no collision.

## No permission required

`api.rpc.*` is **free tier**. No `assertPerm` gate, no permission declaration needed in `spindle.json`. But registrations are observable: LumiScript logs every `sync` / `handle` to the backend console (one line per registration, server-side, NOT user-script console) so cross-extension exposure is auditable:

```
[Spindle:lumiscript] [LumiScript] script "Story Tracker" registered lumiscript.story-tracker.state (sync) for cross-extension visibility
```

Reads are NOT logged — only registrations.

## `sync` vs `handle` — snapshot vs on-demand

The two publishing methods differ in **when the value is computed**:

| Method | Semantics | Use for |
|---|---|---|
| `sync(channel, value, options?)` | Stores `value` in the pool. Readers get it immediately, no callback to your script. | Latest-value snapshots, presence flags, computed state you re-publish on changes |
| `handle(channel, handler, options?)` | Stores a callback. Each `read` invokes the callback live; readers get whatever the handler returns. | Per-caller-tailored responses, expensive queries you don't want to recompute on every change, gating on `requesterExtensionId` |

Calling `sync` on a channel already registered as `handle` (or vice versa) **replaces** the prior registration. Same script, same channel → one slot.

### `sync` — publish a snapshot

```js
// @triggers ls:startup, MESSAGE_SENT

// Maintain a "messages-this-session" counter, publish on every change.
globalThis.sessionCount ??= 0;
if (data.__event === 'MESSAGE_SENT' && data.message.is_user) {
  globalThis.sessionCount += 1;
}

await api.rpc.sync('session-stats', {
  messageCount: globalThis.sessionCount,
  startedAt:    globalThis.sessionStartedAt ??= Date.now(),
});
```

Any other extension can now read `lumiscript.<your-script-slug>.session-stats` and get the current snapshot. The value lives in the host registry — your script doesn't have to be running when the read happens, as long as your registration is still active.

### `handle` — answer reads on demand

```js
// @triggers ls:startup

await api.rpc.handle('recent-events', async (ctx) => {
  // ctx.endpoint:             'lumiscript.<your-slug>.recent-events'
  // ctx.requesterExtensionId: e.g. 'tracker_dashboard'
  // ctx.effectivePermissions: ['chat_mutation', ...] per the endpoint's policy

  const events = await api.events.query({ limit: 50 });
  return {
    caller: ctx.requesterExtensionId,
    events,
  };
});
```

The handler runs every time another extension `read`s the endpoint. Async handlers are awaited; thrown exceptions surface to the calling extension as a rejected `read()` promise.

## Reading from another extension

`read(endpoint)` is the receive side. You pass the **fully-qualified** endpoint name — there's no auto-prefixing on the read path, because reads typically target OTHER extensions whose prefix isn't `lumiscript.`:

```js
// Read from a hypothetical weather extension:
const weather = await api.rpc.read('weather_ext.current');

// Read from another LumiScript script (use the lumiscript.<slug>. prefix):
const trackerState = await api.rpc.read('lumiscript.story-tracker.state');
```

Reads reject with descriptive errors on:

- **Invalid endpoint name** (malformed format)
- **Endpoint not registered** (producer disabled, never registered, or unloaded)
- **Handler exception** (handler-mode endpoint threw — message carries the upstream throw)
- **Handler timeout** (Spindle-side default ceiling)

The reading extension doesn't need any special permission to call `read` — but the host enforces whatever `RpcPolicy` the publisher attached (see below).

## `RpcPolicy` — controlling permission delegation

Optional `policy` on `sync` and `handle` controls how OWNER permissions flow into requester-driven calls. Three modes:

| Policy form | Owner permission delegation | Use for |
|---|---|---|
| `options.policy` omitted | Legacy default: requester must hold **every** gated permission the owner currently has. Defends against confused-deputy proxying. | Backward-compatible default for pre-RC2 scripts; not recommended for new endpoints |
| `policy: { requires: [] }` | No permissions delegated. Endpoint is readable by any extension; the handler runs with zero owner-permission scope. | Public / narrow endpoints (presence flags, version snapshots, status pings) where requester permissions are irrelevant |
| `policy: { requires: ['perm', ...] }` | BOTH owner AND requester must hold every listed permission. Inside the handler, gated `api.*` calls are restricted to this declared set. | Explicit narrow delegation — "this endpoint hands the requester exactly these permissions, no more" |

The `requires` array is the **explicit contract**: it's the set of permissions that flow into the handler's execution context. Owner permissions outside the list don't bleed through.

### Worked example — public presence beacon

A simple "is this script alive?" endpoint that any extension can read without permission gymnastics:

```js
// @triggers ls:startup

await api.rpc.sync('alive', true, {
  policy: { requires: [] },   // intentionally public
});
```

Readers do `await api.rpc.read('lumiscript.<your-slug>.alive')`; they get `true` regardless of their own permission grants.

### Worked example — explicit permission scoping

```js
// @triggers ls:startup

// Expose the last 50 messages — but ONLY to readers that hold chat_mutation.
// Inside the handler, only chat_mutation-gated api.* calls are usable;
// other owner permissions don't leak.
await api.rpc.handle('recent-messages', async (ctx) => {
  return api.chat.getMessages({ last: 50 });
}, {
  policy: { requires: ['chat_mutation'] },
});
```

A reader extension without `chat_mutation` gets a rejection. A reader extension with `chat_mutation` gets the messages, and any gated work the handler does is checked against `['chat_mutation']` — owner permissions like `databanks` or `image_gen` are NOT delegated through.

### `RpcRequestContext.effectivePermissions`

Handlers receive `ctx.effectivePermissions: readonly string[]` — the permissions actually delegated to THIS call after policy application. It's an informational signal handlers can branch on:

```js
await api.rpc.handle('audit-feed', async (ctx) => {
  if (!ctx.effectivePermissions.includes('event_tracking')) {
    // Caller can't read tracked events; return a placeholder.
    return { available: false, reason: 'event_tracking not delegated' };
  }
  return { available: true, events: await api.events.query({ limit: 100 }) };
}, {
  policy: { requires: ['event_tracking'] },
});
```

The host enforces the actual restriction — this is just so handler logic can degrade gracefully when permissions are narrower than the full path would need.

## Slugs and channels

Two character constraints, slightly different:

| Field | Allowed | Auto-derivation | If invalid |
|---|---|---|---|
| `scriptSlug` (auto from name, or `options.as`) | `[a-z0-9_-]+` | Lowercase, whitespace → `-`, strip unmapped chars, collapse runs of `-` / `_`, trim leading/trailing | Empty result throws; user-supplied `options.as` throws on any non-match (no auto-correction) |
| `channel` | `[a-z0-9_.-]+`, non-empty | None — channel is always literal | Throws |

The slugifier is opinionated: a script named "Story Tracker!" becomes `story-tracker`. A script named "!!!" (no letters) slugifies to `""` and throws — pass `options.as` to override.

`options.as` is **strictly validated, not auto-normalised**. The theory: a typo in an override should fail loudly rather than silently mangle into something else. `options.as: 'Story Tracker'` throws because of the uppercase + space; pass `'story-tracker'` instead.

Channels can contain `.` for multi-segment paths under the slug — so `'chat.messages.recent'` is a valid channel, producing `lumiscript.<slug>.chat.messages.recent`.

## Lifecycle

Endpoints auto-unregister on:

- **Script disabled** — via the same teardown that fires `ls:teardown` (the RPC store walks all endpoints owned by the script and calls `spindle.rpcPool.unregister` for each).
- **Script deleted** — same teardown path.
- **Stale-after-re-run** — the diff-and-clean pass that applies to macros, tools, and other registered entities. If your script re-fires and doesn't re-register a previously-published endpoint, the orphan gets cleaned up automatically.
- **Extension unload** — Spindle's lifecycle guarantee tears down ALL LumiScript-owned endpoints when LumiScript itself unloads (extension toggle off, app shutdown, etc.).

You can also explicitly `unregister(channel, options?)` — idempotent, no-op if the channel was never registered. Pass the same `options.as` you used at registration time.

## Use cases

### Cross-extension state sharing

A LumiScript story-tracker maintains rich chat state. A separate non-LumiScript dashboard extension wants to render a sidebar showing that state. Publisher:

```js
// @triggers ls:startup, MESSAGE_SENT, GENERATION_ENDED

await api.rpc.sync('tracker-state', currentState, {
  policy: { requires: [] },   // public — no permission gating
});
```

Reader (in the dashboard extension):

```js
const state = await spindle.rpcPool.read('lumiscript.story-tracker.tracker-state');
renderSidebar(state);
```

### Per-caller-tailored RPC

A script exposes "current quest status" but wants to filter by requester:

```js
await api.rpc.handle('quest-status', (ctx) => {
  if (ctx.requesterExtensionId === 'trusted-dashboard') {
    return { full: true, ...allQuestState };
  }
  return { full: false, activeQuestId: allQuestState.activeId };
});
```

### Public version / capability advertisement

```js
await api.rpc.sync('version', {
  scriptVersion:  '2.3.1',
  apiVersion:     'rpc-v1',
  features:       ['quest-tracking', 'inventory'],
}, {
  policy: { requires: [] },   // anyone can read
});
```

Other extensions read this to detect what's available and degrade gracefully if expected features are missing.

## When to use this vs `api.broadcast`

Two pub/sub-shaped surfaces, different scopes:

| Need | Use |
|---|---|
| Real-time event push between LumiScript scripts running in the same install | `api.broadcast.*` (event emit/on with payload, in-process) |
| Pull-based state queries across extensions (LumiScript ↔ other Lumiverse extensions) | `api.rpc.*` |
| Latest-value snapshot another extension can poll | `api.rpc.sync` |
| Caller-driven computation that may involve script state / API calls | `api.rpc.handle` |
| Cross-script messaging without cross-extension exposure | `api.broadcast` (cheaper, no host-side registry) |

The two coexist cleanly. A common pattern: use `api.broadcast` for in-LumiScript script-to-script chatter, and `api.rpc.sync` to publish aggregated state that OUT-of-LumiScript extensions can consume.

## Error handling

| Cause | Error shape |
|---|---|
| Invalid channel format | `Error: api.rpc: invalid channel "<channel>" — must match /^[a-z0-9_.-]+$/ and be non-empty.` |
| Empty slug from auto-derivation | `Error: api.rpc: cannot derive a slug from "<name>" — pass options.as with a valid slug (lowercase letters, numbers, _, -).` |
| Invalid `options.as` override | `Error: api.rpc: invalid slug override "<slug>" — must match /^[a-z0-9_-]+$/.` |
| `read` against unregistered / disabled endpoint | Host-emitted error (message varies) |
| `read` handler exception | Rejection carries the upstream throw's message |
| `read` handler timeout | Host-emitted timeout error |

`unregister` is idempotent and doesn't throw on missing channels.

## Version notes

Implemented in **v0.26.0+**. The two-tier namespacing (`lumiscript.<scriptSlug>.<channel>`) has been stable since introduction; `RpcPolicy` + `effectivePermissions` were added in **RC2** (along with the host-side permission-delegation enforcement). Scripts that omit `options.policy` get the pre-RC2 legacy behavior (requester must inherit every owner permission) — backward-compatible by design.
