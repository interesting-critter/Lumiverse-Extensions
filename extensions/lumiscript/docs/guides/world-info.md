# World info

`api.worldInfo.*` is full CRUD over Lumiverse's world books (lorebooks) and their entries, plus two prompt-assembly hooks: `getCapturedActive` to inspect which entries fired for a given chat, and `registerInterceptor` to dynamically disable / enable / mutate entries at activation time. It's the LumiScript surface for everything from "create lore entries from a story-tracker script" through "feature-flag-gate lore by turn number" to "rewrite entry content from an external retrieval index."

The namespace name (`worldInfo`) preserves the TavernScript terminology — internally Lumiverse calls these **world books** and their **entries**. They're the same thing.

## Surface

Three layers:

- **Book CRUD** — `list` / `get` / `create` / `update` / `delete`. Books are the containers; each has a name, description, and metadata.
- **Entry CRUD** (nested under `entries`) — `list` / `get` / `create` / `update` / `delete` / `listByAutomationIdPrefix`. Entries are the lore records that get matched against chat content during prompt assembly.
- **Prompt-assembly hooks** — `getCapturedActive` (read which entries fired) and `registerInterceptor` (mutate the activation result).

```ts
interface WorldInfoAPI {
  list(options?): Promise<{ data: WorldInfo[]; total: number }>;
  get(ref):       Promise<WorldInfo | null>;
  create(input):  Promise<WorldInfo>;
  update(ref, input): Promise<WorldInfo>;
  delete(ref):    Promise<boolean>;

  entries: {
    list(ref, options?):         Promise<{ data: WorldInfoEntry[]; total: number }>;
    get(entryId):                Promise<WorldInfoEntry | null>;
    create(ref, input):          Promise<WorldInfoEntry>;
    update(entryId, input):      Promise<WorldInfoEntry>;
    delete(entryId):             Promise<boolean>;
    listByAutomationIdPrefix(prefix): Promise<WorldInfoEntry[]>;
  };

  getCapturedActive(chatId?):    Promise<ActivatedWorldInfoEntry[]>;

  // Global (all-chats) activation — new in v2.0
  getGlobal():                   Promise<string[]>;
  setGlobal(refs):               Promise<string[]>;
  activateGlobal(ref):           Promise<string[]>;
  deactivateGlobal(ref):         Promise<string[]>;

  registerInterceptor(handler, options?): WorldInfoInterceptorHandle;
  listInterceptors():            RegisteredWorldInfoInterceptorInfo[];
}
```

## Permissions

- **`world_books`** — everything in the namespace EXCEPT `registerInterceptor` and `listInterceptors`. Throws `Error: PERMISSION_DENIED:world_books — ...` when missing.
- **`generation`** — `registerInterceptor` rides on the same gate as `api.llm.*` and the macro / content-processor interceptors. The interceptor chain fires inside the prompt-assembly pipeline, so the gate is the same as anything else that touches prompts. `listInterceptors` is a diagnostic surface and is un-gated.

A typical world-info script needs `world_books` for CRUD and `generation` if it registers an interceptor. Most CRUD-only scripts need just `world_books`.

## Refs: UUID or name

Every method that takes a book reference accepts either the UUID or the human-readable book name as a string:

```js
// Equivalent — both resolve to the same book:
await api.worldInfo.get('e7a9b1c3-4f2d-…');
await api.worldInfo.get('My Campaign Lore');
```

Resolution is per-execution cached — the first name lookup triggers a `list()` round-trip, subsequent lookups in the same script run hit the cache for free. UUIDs skip the cache entirely.

Don't worry about distinguishing UUIDs from names yourself; the API does it via regex (`^[0-9a-f]{8}-[0-9a-f]{4}-…$`). Names are validated to never look like UUIDs at creation time, so the dispatch is unambiguous.

If you pass a name that doesn't resolve, you get `Error: api.worldInfo: world book "<name>" not found`.

## Quick start

Create a book, add an entry, list active entries:

```js
// @triggers ls:startup

const book = await api.worldInfo.create({
  name:        'Story Tracker',
  description: 'Auto-generated lore from chat events',
});

await api.worldInfo.entries.create(book.id, {
  key:     ['the tavern', "Drunken Owl"],
  content: 'The Drunken Owl is a dimly-lit roadside tavern, popular with mercenaries and traveling bards.',
  comment: 'Tavern setting',
  // Activate whenever any of the keys appears in recent messages.
  // (Default `selective: false` — any single primary `key` hit activates; see "Entries" below.)
});

// Later — see which entries actually fired in this chat:
const active = await api.worldInfo.getCapturedActive();
console.log(active.map(e => `${e.comment} (${e.source})`));
// → ["Tavern setting (keyword)", ...]
```

That's the basic loop. Books own entries; entries match against chat content; matches get assembled into the LLM prompt.

## Entries — the key concepts

Most entry fields are optional with sensible defaults; just `key` + `content` is usually enough. The interesting ones to know:

| Field | Meaning |
|---|---|
| `key: string[]` | Primary keys. Entry activates when any key appears in scanned chat content. |
| `keysecondary: string[]` | Optional secondary keys. Combined with `key` per `selectiveLogic` rule. |
| `selective: boolean` | When true, require BOTH a primary AND secondary key hit (subject to `selectiveLogic`). Default false. |
| `selectiveLogic: 0 \| 1 \| 2` | Combination rule when `selective: true`: 0 = AND, 1 = NOT (secondary must NOT be present), 2 = OR. |
| `constant: boolean` | Always active, no key match needed. Useful for global lore. |
| `position: number` | Where the entry text gets injected in the prompt. 0 = before main prompt, 1 = after main prompt, 4 = at specific depth. |
| `depth: number` | Distance from latest message when `position: 4`. |
| `priority: number` | Tie-breaker when budget gets tight. Higher = kept first. |
| `probability: number` | 0–100. When `useProbability: true`, roll the dice each scan. |
| `disabled: boolean` | Skip during activation entirely. The interceptor surface can override this. |
| `automationId: string \| null` | **The script-script integration field.** See below. |
| `vectorized: boolean` | When true, entry participates in vector-similarity matching in addition to keywords. |

`comment` is your free-form description — it shows up in the UI as the entry's label and in `getCapturedActive` results, but doesn't affect activation.

## Dynamic entries via `automationId`

The convention for scripts that **own** entries dynamically: tag each entry's `automationId` with a script-namespaced prefix. The host doesn't interpret the field; LumiScript provides `listByAutomationIdPrefix(prefix)` to enumerate, update, and clean up your script's entries:

```js
// @triggers ls:startup, MESSAGE_SENT, ls:teardown

const NS = `lumiscript:${script.id}:`;

if (data.__event === 'ls:teardown') {
  // Symmetric cleanup — delete every entry this script created.
  const owned = await api.worldInfo.entries.listByAutomationIdPrefix(NS);
  for (const entry of owned) {
    await api.worldInfo.entries.delete(entry.id);
  }
  return;
}

if (data.__event === 'MESSAGE_SENT' && data.message.is_user) {
  // Detected a new character mention; promote it to a lore entry.
  const name = extractCharacterName(data.message.content);
  if (!name) return;

  const book = await api.worldInfo.get('Story Tracker');
  if (!book) return;

  // The automationId encodes both ownership AND content key — re-running
  // the script with the same name will UPDATE rather than duplicate
  // (we look it up first).
  const automationId = `${NS}character:${name}`;
  const owned = await api.worldInfo.entries.listByAutomationIdPrefix(automationId);
  if (owned.length > 0) return;   // already tracked, skip

  await api.worldInfo.entries.create(book.id, {
    key:          [name],
    content:      `${name} is a character mentioned in this story. Track and develop them as needed.`,
    comment:      `Auto-tracked: ${name}`,
    automationId,
  });
}
```

Two reasons this pattern matters:
1. **No duplicate accumulation across reruns.** Scripts that re-fire on every message and just `create` without checking would multiply entries forever. The `listByAutomationIdPrefix` lookup makes "does this already exist?" a one-liner.
2. **Clean teardown.** When the script is disabled or deleted, the same prefix-walk + `delete` loop wipes everything cleanly. Without `automationId` tagging there's no way to know which entries the script created vs. which the user authored.

`listByAutomationIdPrefix` is O(books × entries-per-book). Fine for typical world-book sizes (most users have a few dozen entries total); avoid calling it on hot paths like `STREAM_TOKEN_RECEIVED`.

## Reading activated entries — `getCapturedActive`

`getCapturedActive(chatId?)` runs the **full Lumiverse activation pipeline** — keyword matching, selective logic, probability rolls, sticky/cooldown/delay state, group competition, budget enforcement, vector similarity — and returns the entries that would activate for the chat right now. Each result is a full `WorldInfoEntry` plus two activation-metadata fields:

```ts
type ActivatedWorldInfoEntry = WorldInfoEntry & {
  source: 'keyword' | 'vector';
  score?: number;   // cosine similarity, present only when source: 'vector'
};
```

Falls back to the active chat if `chatId` is omitted. Throws if no chat is open.

Common use: surface "what lore is going into the prompt right now?" in a UI sidebar:

```js
const active = await api.worldInfo.getCapturedActive();
const items = active.map(e =>
  `<li><strong>${e.comment || 'Untitled'}</strong> — ${e.source}${e.score != null ? ` (${e.score.toFixed(3)})` : ''}</li>`,
);
await widget.root.update(`<ul>${items.join('')}</ul>`);
```

Performance: one `getActivated` host call + one `entries.get` per activated entry (in parallel). For typical activations (5–15 entries) this is fast. For unusually large activation sets, consider caching the result if you call it more than once per turn.

## Globally-active world books

> **New in v2.0.** Four methods on `api.worldInfo` read and edit the user's set of **global** world books.

A **global** world book applies to **every chat**, regardless of which character or chat it's bound to. It's the "always in scope, everywhere" tier of world info — the user normally manages it in the Lumiverse UI; v2.0 exposes it to scripts.

```ts
getGlobal():                Promise<string[]>;              // read the global set
setGlobal(refs):            Promise<string[]>;              // replace the whole set
activateGlobal(ref):        Promise<string[]>;              // add one book
deactivateGlobal(ref):      Promise<string[]>;              // remove one book
```

All four need the **`world_books`** permission.

### Global books vs. `constant: true` entries

These are two different mechanisms — don't confuse them:

- **`constant: true`** (an *entry* field, [see the entries table above](#entries--the-key-concepts)) makes a single **entry** always active, but only within the books that are actually in scope for the current chat. It's per-entry, and it's still gated by which books the chat pulls in.
- **Global activation** (this section) makes an entire **book** in scope for **every** chat — nothing is bound to a character or chat, it just always applies. Whether each entry inside that book then activates still depends on that entry's own rules (keywords, `constant`, selective logic, etc.).

So global activation controls *which books are in scope everywhere*; `constant` controls *which entries fire once a book is in scope*.

### Refs and return values

`setGlobal`, `activateGlobal`, and `deactivateGlobal` take book references as either a **UUID or the book name** — the same [name-or-UUID resolution](#refs-uuid-or-name) as `get` / `update` / `delete`. Names are resolved through the per-run cache; a name that doesn't resolve throws `Error: api.worldInfo: world book "<name>" not found`.

Every one of the four returns the **updated set of global book IDs** — a `string[]` of UUIDs, *not* book objects. If you want names or full records, map the IDs back through `get`:

```js
const ids   = await api.worldInfo.getGlobal();
const books = await Promise.all(ids.map(id => api.worldInfo.get(id)));
console.log(books.map(b => b?.name));
```

A few resolution details worth knowing:

- **`setGlobal(refs)` replaces the entire set.** Pass `[]` to clear all global books. Each ref is resolved to an ID first, so you can mix names and UUIDs freely. The host silently drops any resolved ID that isn't an existing book, so the returned array can be **shorter than what you passed** — treat the return value as the source of truth, not your input list.
- **`activateGlobal(ref)` adds one book** to the set and throws if the book doesn't exist (the ref fails to resolve).
- **`deactivateGlobal(ref)` removes one book** and is a **no-op** if the book wasn't in the global set — it still returns the (unchanged) ID list rather than throwing.

### Worked example

Make a "House Rules" book apply to every chat, then confirm it landed:

```js
// @triggers ls:startup

// Ensure the book exists (get returns null if it doesn't).
let book = await api.worldInfo.get('House Rules');
if (!book) {
  book = await api.worldInfo.create({
    name:        'House Rules',
    description: 'Table conventions that apply to every chat',
  });
}

// Activate it globally — by name; the ref resolves to the book's UUID.
const active = await api.worldInfo.activateGlobal('House Rules');
console.log('Global book IDs:', active);
// → ["e7a9b1c3-4f2d-…", ...]

// getGlobal returns IDs; map through get() for names.
const names = await Promise.all(
  active.map(async id => (await api.worldInfo.get(id))?.name),
);
console.log('Globally active:', names.filter(Boolean));
// → ["House Rules", ...]
```

To take it back out later:

```js
await api.worldInfo.deactivateGlobal('House Rules');   // no-op if it wasn't global
```

Because these edit **user-level** state (not chat- or character-scoped), the change persists across chats and sessions until a script or the user reverses it. If your script activates a book globally, mirror it with a `deactivateGlobal` on `ls:teardown` unless the global binding is meant to outlive the script.

## Interceptors — dynamic activation control

`registerInterceptor(handler, options?)` is the seam for **modifying activation results before they hit prompt assembly**. The handler receives a context with the candidate entries + chat state, and returns vote-style decisions:

```ts
type WorldInfoInterceptorHandler = (ctx: WorldInfoInterceptorCtx) =>
  | { disabled?: string[]; enabled?: string[]; forced?: string[]; mutated?: WorldInfoInterceptorMutation[] }
  | void
  | Promise<...>;
```

Four lists, all independent:
- **`disabled: string[]`** — entry ids to suppress this turn. **Vote-off is absolute**: once any handler votes disabled for an entry id, the entry stays disabled. No `enabled` or `forced` vote — from any handler, including the one that voted disabled — can revive it. The accumulator gates `enabled`/`forced` inside an `else` branch from `disabled`, and the final result assembly filters both lists against the disabled set as a second layer.
- **`enabled: string[]`** — entry ids to allow through despite being stored-disabled (`entry.disabled === true` in the stored data). Bypasses the stored-disabled flag but does NOT override another handler's `disabled` vote.
- **`forced: string[]`** — entry ids that bypass selective-logic checks and probability rolls (always treated as if they matched). Same precedence rule: does NOT override another handler's `disabled` vote.
- **`mutated: { id, content }[]`** — replace entry content on the fly. Last-write-wins per id.

Return `void` (or omit all four arrays) to pass through unchanged. Async handlers are awaited.

### Turn-based gating example

```js
// @triggers ls:startup

api.worldInfo.registerInterceptor(async (ctx) => {
  if (ctx.chatTurn < 5) {
    // Hide "endgame-spoiler" entries for the first 5 turns.
    const disabled = ctx.entries
      .filter(e => (e.extensions)?.gate === 'endgame-only')
      .map(e => e.id);
    return { disabled };
  }
}, { priority: 50 });
```

### Content rewrite from extension state

```js
// @triggers ls:startup

api.worldInfo.registerInterceptor(async (ctx) => {
  // Pull dynamic overrides from a script-owned database.
  const overrides = await api.db.collection('wi-overrides').find({});
  return {
    mutated: overrides.map(o => ({ id: o.entryId, content: o.content })),
  };
}, { priority: 100 });
```

### Composition + priority

Multiple handlers form a chain. **Lower priority runs first** (default 100, tie-broken by registration order). Each handler sees prior handlers' decisions reflected in `ctx.entries.disabled`, so a low-priority "gate" handler can run before a high-priority "force everything" handler and have its disabled-votes respected.

`options.id` is a stable identifier — re-registering with the same id replaces the prior entry instead of accumulating. Useful when re-running the script body (e.g. via Reload or `ls:reload`):

```js
api.worldInfo.registerInterceptor(myHandler, {
  id: 'turn-gate',
  priority: 50,
});
```

### Critical performance constraint

The interceptor chain runs **before** activation, which runs **before** prompt assembly, which runs **before** the LLM call. **Anything slow here adds visible latency before the first streamed token.** Each invocation has a 2-second soft timeout (configurable via `options.timeoutMs`); the host's outer 10-second budget is shared across ALL extensions' interceptors.

**Hard rule: don't call `api.llm.*` or `api.utils.http.*` from an interceptor.** The latency stacks visibly. Pre-compute via a trigger handler (e.g. `MESSAGE_SENT`), store the result in `api.db.*` or `api.scriptStorage`, and just READ from the interceptor.

### Listing registered interceptors

`listInterceptors()` returns the current chain as `{ scriptId, scriptName, id, priority, timeoutMs }` records. Diagnostic surface — un-gated. Use for "what's wired up?" debugging:

```js
console.log(api.worldInfo.listInterceptors());
```

## Stale-cleanup behavior

Interceptors auto-clean across script reruns. If a handler with id `'turn-gate'` was registered in run N and the script's re-run in run N+1 doesn't re-register it, LumiScript's post-run `diffAndCleanStale` pass drops it from the chain. Same mechanism as macros and content-processors. Net effect: scripts can re-register interceptors freely without accumulating stale ones.

Note: `registerInterceptor` calls without `options.id` get auto-ids (`auto-1`, `auto-2`, …). Auto-id'd interceptors are subject to the same stale cleanup, so re-running won't accumulate them — but you also can't reliably look them up later.

## Error handling

The CRUD methods can throw on:

- **Permission denied** — `Error: PERMISSION_DENIED:world_books — grant this permission to use this API` (or `generation` for the interceptor surface).
- **Unknown ref** — `Error: api.worldInfo: world book "<name>" not found` when a name-based lookup fails.
- **Validation** — host-side validation of create/update payloads. Error message includes the field name.

Robust pattern:

```js
try {
  await api.worldInfo.entries.create(bookRef, { key: [name], content: '...' });
} catch (err) {
  const msg = err instanceof Error ? err.message : String(err);
  if (msg.includes('PERMISSION_DENIED')) {
    await api.ui.toast({
      message: 'World info access requires the world_books permission.',
      kind:    'warning',
    });
    return;
  }
  console.error('[my-script] world-info write failed:', err);
}
```

## When to use this vs `api.databanks.*`

Two overlapping namespaces, two different jobs. Important framing first: **both surfaces are script-side CRUD**. The retrieval logic — what content actually reaches the prompt — runs host-side at prompt-assembly time in both cases. The difference is what each surface stores and how the host retrieves from it:

| Need | Use |
|---|---|
| Lore that activates on keyword match inside the chat | `api.worldInfo` |
| Lore retrieved by vector similarity (in addition to keyword match) | `api.worldInfo` with `vectorized: true` on entries |
| Reference material attached to a chat / character / global scope, retrieved by host-side vector similarity at prompt time | `api.databanks` |
| Hand-curated narrative lore (factions, locations, characters) | `api.worldInfo` |
| Document-style reference material (PDFs, transcripts, knowledge bases) | `api.databanks` |

World info is the **keyword-and-rules + optional vector-augmentation** surface; databanks is the **document-store-with-host-side-vector-retrieval** surface. Scripts manage records in either. The host pulls activated entries / vector-matched chunks into the assembled prompt at LLM-call time. Scripts don't directly retrieve from databanks at runtime — they manage the document store; the host does the retrieval. Scripts CAN manipulate world-info activation via `api.worldInfo.registerInterceptor`, including reading from databanks at script-trigger time (NOT inside the interceptor — see the perf rule above) to drive world-info content rewrites.

## Version notes

Fully implemented since v1.0.0-rc.1+ (via native `spindle.world_books.*`). The `registerInterceptor` surface landed in **v0.27.0+** along with the diff-cleanup of stale entries across re-runs. Entry fields track Lumiverse's own world-book schema 1:1 in camelCase — when new entry fields ship in future Lumiverse versions, they appear in `WorldInfoEntry` automatically (mapping is generated, not hand-written).
