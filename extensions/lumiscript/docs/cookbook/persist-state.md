# Persist state the right way

One script usually holds several *kinds* of state at once — a throwaway session counter, a value that must survive forever, an append-only log you'll query later. The mistake this recipe exists to prevent is reaching for one storage tier for all of it. Here's a tracker that uses three, and the reasoning for each.

## What you'll use

- `api.scriptStorage.*` — a session-scoped key/value store. Survives worker eviction and script edits; lost on backend restart. **No permission.** ([Storage model](../concepts/storage-model.md))
- `api.variables.character.*` — durable, per-character JSON values. Survives everything until the character is deleted. **No permission.**
- `api.db.*` — collection-shaped persistent storage you can query. **No permission.**
- `GENERATION_ENDED` as the trigger — it fires once the assistant's reply has landed. ([Trigger model](../concepts/trigger-model.md))

The idea up front: **durability, scope, and shape are three independent questions, and each piece of state answers them differently.** Pick the tier that matches the answer — don't default everything to the most durable option (the single most common storage mistake) or cram records into a place with no query layer.

## The script

A relationship tracker. It reacts to each assistant reply, nudges a per-character affinity score, and logs every change. Three pieces of state, three tiers.

```js
// @triggers GENERATION_ENDED
// A relationship tracker — three kinds of state, three storage tiers.
// No permission required.
//
// GENERATION_ENDED fires after the assistant's reply lands. Its payload is
// { generationId, chatId, messageId, content } — read the reply via `data.content`
// and its id via `data.messageId` (this event has no `data.message` object).

// ─── Derive an affinity delta (self-contained keyword heuristic) ──────────
const text  = (data.content ?? '').toLowerCase();
const delta =
  /\b(thank|love|smile|glad|appreciate)\b/.test(text) ?  1 :
  /\b(hate|angry|leave|never|refuse)\b/.test(text)    ? -1 : 0;

// ─── Tier 1: api.scriptStorage — a session stat, fine to lose on restart ──
// "How many assistant turns this process?" is a cheap counter we don't need
// to outlive a backend restart. The session-scoped tier is the right call —
// reaching for durable variables here would be over-engineering.
const seen = ((await api.scriptStorage.get('turns-seen')) ?? 0) + 1;
await api.scriptStorage.set('turns-seen', seen);

if (delta === 0) return;

// ─── Tier 2: api.variables.character — the one value that MUST persist ────
// Affinity is a single number, scoped to the active character, that has to
// survive extension reloads and backend restarts. Character-scoped variables
// are exactly that: durable, per-character, no query layer needed.
const score = ((await api.variables.character.get('affinity')) ?? 0) + delta;
await api.variables.character.set('affinity', score);

// ─── Tier 3: api.db — an append-only log you'll want to QUERY later ───────
// "What changed her affinity, and when?" is a question over many records.
// That's the collection tier (scoped to this chat) — never a single variable.
const log = await api.db.collection('affinity-log', { scope: 'chat' });
await log.insert({ ts: Date.now(), delta, score, messageId: data.messageId });

console.log(`[rel] turn ${seen}: affinity ${delta > 0 ? '+' : ''}${delta} → ${score}`);
```

Enable it and chat. Affinity climbs and dips with the assistant's tone; every change lands as a record in the log. Open the LumiScript panel's **Storage** tab to watch both the `scriptStorage` entry and the `affinity-log` collection fill in live.

## How it works

The whole lesson is in *why each piece of state went where it did*:

**The session counter → `api.scriptStorage`.** It's a single value, it's cheap to recompute, and nothing breaks if it resets when the backend restarts. `scriptStorage` is exactly this niche: per-script, session-scoped, survives the things that happen often (worker eviction, editing and re-saving the script) but not a full restart. Putting it in durable `variables` would work — and quietly imply the count matters more than it does.

**The affinity score → `api.variables.character`.** This is the opposite: one value that *must* survive everything and belongs to a specific character. The four `api.variables.*` scopes differ only in *what owns the value* — `local`/`chat` per chat, `global` per user, `character` per character. Affinity is per-character, so `variables.character` it is. Reads default cleanly when there's no active character (preview, cold start), which is why `(… ?? 0)` is safe.

**The change log → `api.db`.** "Many records you'll filter later" is the collection tier's whole reason to exist. `api.db.collection(name, { scope })` hands back a handle; `insert` appends, and later `find` / `query` read with a Mongo-style filter subset. Scoping it to `'chat'` keeps each conversation's log separate. Crucially, this is the one piece you should *not* fold into `variables` — a variable holding a growing array rewrites the entire blob on every append and gives you nothing to query against.

**What we deliberately didn't use.** No `globalThis` (it would lose the score on restart — wrong for durable state, though it's the right home for a hot, non-serialisable cache). No single big `variables` array for the log (no query layer; rewrites everything per append). No `api.enclave` (affinity isn't a secret, and encryption isn't free). Matching each need to its tier *is* "persisting state the right way."

## Make it yours

- **Read the state back.** Surface the current score with `await api.variables.character.get('affinity')`, or answer "what were the last three positive beats?" with `await log.find({ delta: { $gt: 0 } })` — the collection tier is there precisely so questions like that are one call.
- **Re-scope the log.** `api.db.collection` takes `scope: 'script'` (default — shared across all chats and characters), `'character'`, or `'chat'`. Switch it to `'character'` if you want one affinity history per character rather than per conversation.
- **Reach for `globalThis` when serialisation would be wasteful.** A compiled matcher, a live `Map`, a class instance — things that can't round-trip through JSON — belong in a namespaced `globalThis` key (`globalThis.lsRelTracker_cache ??= new Map()`), accepting that they reset on restart.
- **Add a secret the right way.** If a variant calls an external API, keep the token in `api.enclave` (`put` / `get`, AES-256-GCM at rest) rather than a plaintext variable — it needs `allowDangerous`. See the enclave row in the [storage model](../concepts/storage-model.md#per-tier-reference).

## Gotchas

- **`scriptStorage` is *not* durable.** It survives eviction, hot reload, and edits, but a backend restart wipes it. Anything that must outlive a restart (the score) belongs in `variables` or `db` — that's the line the recipe is drawn around.
- **`variables` round-trip through JSON.** Functions, `Map`s, class instances won't survive a `set`/`get`. Store plain JSON-shaped data; keep the live objects in `globalThis`.
- **It's `insert`, not `insertOne`.** The `api.db` collection surface is `insert` / `insertMany` for writes and `find` / `findOne` / `count` / `query` for reads — the Mongo-ish *names* don't all match Mongo. Filters use a subset (`$gt`, `$gte`, `$lt`, `$in`, `$regex`, …), not arbitrary Mongo operators.
- **Mind the caps.** `scriptStorage` hard-stops at 1 MB per script; an `api.db` collection soft-warns at 10 MB and hard-stops at 50 MB. A high-volume log wants periodic pruning (`delete` old records) or a tighter scope.

## See also

- [Storage model](../concepts/storage-model.md) — the full storage-tier reference, the decision tree, and the per-tier anti-patterns. This recipe is the goal-oriented sibling of that concept doc; read it when you need a tier this recipe didn't use (`globalThis`, `enclave`, `api.files`).
- [Share state between two scripts](share-state-broadcast.md) — uses `scriptStorage` as a *shared* channel between a producer and a consumer.
- [Handler lifetime](../concepts/handler-lifetime.md) — when state is captured by a long-lived handler, the tier you chose for it matters.
