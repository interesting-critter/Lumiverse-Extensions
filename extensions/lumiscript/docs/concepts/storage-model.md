# Storage model

LumiScript has six places to put data. Each one trades a different combination of (durability, scope, permission, size). Pick by what you actually need — defaulting to "the most durable one" is the biggest single mistake people make here.

## The core idea

A script that needs to remember something across fires has six options, and they're not interchangeable. The differences that matter:

- **Durability** — how long the data survives. Until the next fire? Until the extension reloads? Until the backend restarts? Forever, across everything?
- **Scope** — what owns the data. The script alone? A specific character or chat? The whole extension? All scripts in LumiScript?
- **Permission** — do you need `allowDangerous`, a specific permission, or neither?
- **Shape** — do you need a single key-value store, an array of records you query, raw bytes on disk, or encrypted secrets?

The decision tree below maps "what I'm trying to store" to "which tier". The per-tier reference and patterns/anti-patterns sections that follow are for when you've already picked a tier and want to use it well.

## Decision tree

![Storage-tier decision tree: starts at "survive past current fire?", branches through "survive backend restart?" and "what shape?", terminates at one of the storage options (local let/const, globalThis, api.scriptStorage, api.variables, api.db, api.enclave, api.files)](../diagrams/storage-model.svg)

Most non-trivial scripts use two or three of these together. A typical pattern: globalThis for a per-process cache, `api.variables.character` for per-character persistent state, `api.db.*` for an append-only event log.

## Per-tier reference

| Tier | Durability | Scope | Permission | Size cap | Async? |
|---|---|---|---|---|---|
| `globalThis.<key>` | Until the script-runner subprocess restarts (extension reload / backend restart) | Per-script-runner subprocess, shared across all scripts running in it (per-script under QuickJS *per-script* isolation) | none | host RAM | sync |
| `api.scriptStorage` | Until script disable / delete / backend restart. Survives worker eviction + script-source edits. | Per script | none | 1 MB per script | async |
| `api.variables.local` | Persistent, until the user clears the chat's variables | Per chat | none | host limits | async |
| `api.variables.global` | Persistent, until the user clears | Per user (cross-chat, cross-character) | none | host limits | async |
| `api.variables.character` | Persistent, until the character is deleted or you clear it | Per character | none | host limits | async |
| `api.variables.chat` | Persistent, until the user clears the chat's variables | Per chat | none | host limits | async |
| `api.db.*` | Persistent, until you drop the collection | Three sub-scopes: `script` (default) / `character` / `chat` | none | 10 MB soft-warn / 50 MB hard-stop per collection | async |
| `api.enclave.*` | Persistent, AES-256-GCM encrypted at rest | Per user, per extension (cross-script within LumiScript) | `allowDangerous` | value ≤ 64 KB (printable ASCII), key matches `[A-Za-z0-9_\-.]{1,128}` | async |
| `api.files.{userRead,userWrite,...}` | Persistent | Per user, per extension | `allowDangerous` | host limits | async |
| `api.files.{sharedRead,sharedWrite,...}` | Persistent | Shared across all users of the extension | `allowDangerous` | host limits | async |
| `api.files.{tempRead,tempWrite,...}` | TTL-bound, auto-expires | Per user, per extension | `allowDangerous` + `ephemeral_storage` | host limits + quota | async |

**`globalThis.<key>`** is just JavaScript. The script-runner subprocess shares one `globalThis` across every script that runs in it, so namespace your keys aggressively (`globalThis.lsMyScript_cache ??= {}` is a good shape). One exception: under the QuickJS engine's *per-script* isolation each script gets its own `globalThis`, so cross-script collisions can't happen there — but the aggressive-namespacing habit is worth keeping regardless (it holds under the default engine and QuickJS *shared* isolation). See [Execution engine](engine.md). Values can be anything — including non-serialisable things like functions, class instances, or live `Map`s — which is the main reason to reach for it over `api.scriptStorage` when serialisation would be wasteful. Sync access: no `await`.

**`api.scriptStorage`** is the structured cousin of `globalThis`. Same lifetime ceiling (lost on backend restart), but per-script-isolated with an admin inspector in the **Storage** tab of the LumiScript panel. The 1 MB cap is a hard ceiling per script — `set` throws if you exceed it. Use when you want the inspector (debug-visibility into what your script is keeping in memory) or when multiple scripts could accidentally collide on `globalThis` keys.

**`api.variables.*`** is the persistent JSON-shaped tier with four scope variants. Three of them are macro-compatible — values you set are visible to prompts via Lumiverse's built-in macros: `local` and `global` pair with `{{getvar}}` / `{{setvar}}`, and `chat` pairs with `{{getchatvar}}` / `{{setchatvar}}`. `character` lives in a per-character userStorage file and is NOT exposed via macros — read it from script code only. Values round-trip through JSON, so non-serialisable types (functions, class instances, Maps, etc.) won't survive a set/get.

**`api.db.*`** is the collection-shaped persistent tier. Create a collection with `await api.db.collection(name, opts)`, then call methods on the returned handle: `insert` / `insertMany` (writes), `find` / `findOne` / `count` / `query` (reads), `update` / `delete` / `clear` (mutations). Filter predicates use a Mongo-style subset (`$gt`, `$gte`, `$lt`, `$lte`, `$ne`, `$in`, `$nin`, `$exists`, `$regex`) plus a `jsonquery` escape hatch for anything more complex. Three sub-scopes are available via `opts.scope`: `'script'` (default, shared across all chats/characters for this script), `'character'` (per-character), `'chat'` (per-chat). Optional Zod-schema validation on writes. Soft-warns at 10 MB per collection, hard-stops at 50 MB. The Storage tab's Collections section is the admin inspector — sortable, filterable, with per-record edit / delete actions.

**`api.enclave.*`** is the encrypted-at-rest tier. AES-256-GCM, key derived from a per-user-per-extension secret you don't control directly. Use this for OAuth refresh tokens, API keys, anything that would be bad to read off disk in plaintext. Requires `allowDangerous`. Surface is `put` / `get` / `delete` / `has` / `list` — note `put` (not `set`), and don't expect `clear` (it doesn't exist; iterate `list` + `delete` if you need to wipe). Constraints: values must be printable ASCII and ≤ 64 KB; keys match `^[A-Za-z0-9_\-.]{1,128}$`.

**`api.files.*`** is for actual files: bytes on disk. Three tiers:

- `api.files.userRead` / `userWrite` / `userDelete` / `userExists` / `userList` / `userMkdir` — per-user, per-extension. Persistent. (No `userStat`.)
- `api.files.sharedRead` / `sharedWrite` / `sharedDelete` / `sharedExists` / `sharedStat` / `sharedList` / `sharedMkdir` — shared across all users of the extension. Persistent.
- `api.files.tempRead` / `tempWrite` / `tempReadBinary` / `tempWriteBinary` / `tempDelete` / `tempStat` / `tempList` / `tempClearExpired` — TTL-bound. Text **and** binary I/O — use the `*Binary` pair (`tempReadBinary` → `Uint8Array`, `tempWriteBinary(path, bytes, opts)`) for images / PDFs / any non-text blob. The host auto-expires entries; use for large transient data (downloaded asset that's about to be processed, intermediate state during a long-running job). The ephemeral pool is **quota-bounded** (per-extension and global): `tempGetPoolStatus()` reads the usage snapshot, and for a large write you can reserve space up front — `tempRequestBlock(sizeBytes, { ttlMs?, reason? })` returns a `{ reservationId, … }` you pass to the write's `options.reservationId` so it can't fail partway through on a full pool (`tempReleaseBlock(reservationId)` frees an unused reservation).

All three need `allowDangerous`; the temp tier additionally needs the `ephemeral_storage` permission.

## Common patterns

**Per-process cache of an expensive computation** — keep it hot across fires, accept losing it on backend restart:

```js
async function getCharacterCardFor(charId) {
  globalThis.lsMyScript_charCache ??= new Map();
  const cache = globalThis.lsMyScript_charCache;
  if (cache.has(charId)) return cache.get(charId);
  const card = await api.characters.get(charId);
  cache.set(charId, card);
  return card;
}
```

**Persistent per-character state** — survives extension reload, scoped to one character:

```js
const lastReview = await api.variables.character.get('lastReviewTimestamp', 0);
if (Date.now() - lastReview > 24 * 60 * 60 * 1000) {
  await runDailyReview();
  await api.variables.character.set('lastReviewTimestamp', Date.now());
}
```

**Append-only event log** — `api.db.*` shines here. Many records, filterable, scoped to the chat:

```js
const events = await api.db.collection('events', { scope: 'chat' });
await events.insert({
  ts:   Date.now(),
  kind: 'message_processed',
  messageId: data.message.id,
  outcome: 'success',
});

// Later: query for everything in the last hour.
const recent = await events.find({ ts: { $gte: Date.now() - 60 * 60 * 1000 } });
```

**OAuth refresh token** — encrypted-at-rest because it's a secret:

```js
await api.enclave.put('spotify.refresh_token', refreshToken);
// ...later, in another fire:
const stored = await api.enclave.get('spotify.refresh_token');
```

**Downloaded binary processed across two fires** — write to temp, read from temp, let the host auto-expire it:

```js
// Fire 1: fetch and stash. resp.body is a Uint8Array → use the binary tier.
const resp = await api.utils.http.get(url, { responseType: 'arraybuffer' });
await api.files.tempWriteBinary('pending-upload.png', resp.body, { ttlMs: 10 * 60_000 });

// Fire 2: process from temp.
const bytes = await api.files.tempReadBinary('pending-upload.png');
await api.images.upload({ data: bytes, mimeType: 'image/png' });
```

## What NOT to use each tier for

- **`globalThis`** for data that must survive backend restart. The script-runner subprocess respawns on backend restart, and `globalThis` resets to empty. Use `api.scriptStorage` (one-restart-cycle resilience) or `api.variables.*` (true durability) instead.
- **`api.scriptStorage`** for data that must survive backend restart. Same trap, slightly later — `scriptStorage` is in-memory at the backend level, not on disk. Cleared on disable / delete / backend restart. Use `api.variables.*` if you need real durability.
- **`api.variables.*`** for high-volume records (hundreds-of-entries-and-growing). Each `set` rewrites the underlying file or metadata blob; performance degrades with size and there's no query layer. Use `api.db.*` for record-shaped data — that's exactly what it's for.
- **`api.variables.*`** for binary blobs. The store is JSON; bytes would have to be base64-encoded and you'd pay the size-tripling tax on every read/write. Use `api.files.*` instead.
- **`api.db.*`** for single-value flags or counters. Spinning up a Collection handle and inserting documents for "the user's currently-selected mode" is overkill. Use `api.variables.*`.
- **`api.enclave.*`** for non-sensitive data. Encryption isn't free — `get` and `set` both pay an AES-256-GCM round-trip. There's no policy reason to encrypt non-secrets, and it makes the data harder to inspect during debugging. Reserve enclave for actual secrets.
- **`api.files.*`** for structured JSON. There's no native query layer over files — you'd be reading the whole file, parsing JSON, mutating, writing back. `api.variables.*` (for small) or `api.db.*` (for large) handle structured data far better.

## See also

Three adjacent surfaces that share storage's shape but solve different problems:

- **`api.events.*`** — persistent event tracking with `track` / `query` / `replay` / `getLatestState`. Use when you want an append-only event log that the host indexes for you, with replay-to-current-state semantics. Requires `event_tracking`. Distinct from `api.db.*` because the API is event-shaped (you `track` an event with a payload; you don't get raw record CRUD).
- **`api.databanks.*`** — vectorised reference material attached to global / character / chat scopes. Documents inside a databank are embedded for semantic retrieval at prompt-assembly time. Use when you want LLMs to retrieve documents by meaning, not when you want a key-value or record store. Requires `databanks`.
- **`api.images.*`** — image-store CRUD specifically for image bytes. Returns an `ImageInfo` whose `id` is the canonical handle into Lumiverse's image table — passes cleanly into `api.theme.extractColors`, `api.characters.setAvatar`, etc. Use this (not `api.files.*`) when the bytes are an image you want Lumiverse to treat as one. Requires `images`.

## What's next

1. **[Handler lifetime](handler-lifetime.md)** — the closure-persistence story for `broadcast.on`, `commands.onInvoked`, `macros.register`, `tools.register`. When you register a handler in one fire and need it to work hours later, the storage tier you chose for its captured state matters; this doc explains why.
2. **The in-app Reference's API Functions section** — for the full method signatures of every namespace in this doc.
