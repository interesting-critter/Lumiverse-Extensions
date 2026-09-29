# Memory Cortex & chat memory

`api.memories.*` bridges Lumiverse's hybrid memory architecture — everything the app automatically *remembers* about a chat — under a single `memories` permission. Use it to visualise, edit, or augment that memory: entity dashboards, relation editors, vault management UIs, alternate retrieval strategies, memory-aware sidebars.

Two halves live under the namespace:

- **Memory Cortex** — a structured graph (entities + typed relations), narrative-arc consolidations, per-chunk salience, frozen **vaults**, chat **interlinks**, and a **fused retrieval** pipeline.
- **Long-Term Chat Memory** — the vectorized chunk store behind the `{{memories}}` macro.

It's distinct from [`api.databanks`](databanks.md): databanks are documents *you* upload; the cortex is what Lumiverse builds from the conversation itself.

> **Ownership & scoping.** The active userId is folded in automatically (you never pass it). Every chat-scoped call is ownership-checked host-side — referencing a chat you don't own returns `null` (reads) or throws (writes). Get the active chat id with `api.chat.getChatId()`.

## The nine sub-namespaces

| Sub-namespace | What |
|---|---|
| `cortex` | config, fused `query`, `queryLinked`, the warm cache |
| `entities` | the entity graph (characters / locations / items / …) |
| `relations` | typed edges between entities |
| `consolidations` | narrative-arc summaries |
| `salience` | per-chunk importance records |
| `vaults` | frozen cortex snapshots |
| `links` | vault attaches + chat interlinks |
| `chatMemory` | the `{{memories}}` chunk store |
| `stats` | usage counts + ingestion telemetry |

## Fused retrieval (the headline call)

`cortex.query` fuses semantic search with salience, recency, reinforcement, emotional, and entity signals — the same ranking the host uses during prompt assembly:

```js
const chatId = api.chat.getChatId();
const result = await api.memories.cortex.query({
  chatId,
  queryText: 'recent conflict with Selene',
  topK: 8,
  emotionalContext: ['betrayal', 'fury'],
  includeConsolidations: true,
  includeRelationships: true,
});

for (const m of result.memories) console.log(m.finalScore.toFixed(2), m.content);
console.log('Entities in context:', result.entityContext.map((e) => e.name).join(', '));
console.log('Active arc:', result.arcContext ?? 'none');
```

Results are **server-cached ~5 minutes** per chat + query shape. `cortex.getCached(chatId)` reads that warm cache without re-running retrieval (returns `null` if empty/expired) — handy when you just want what the active generation saw.

### Grounding an LLM reply

Drop the retrieved content into a prompt (needs the `generation` permission too — see [Calling the LLM](llm.md)):

```js
const { memories } = await api.memories.cortex.query({ chatId, queryText: question, topK: 6 });
const context = memories.map((m) => `- ${m.content}`).join('\n');
const answer = await api.llm.generate(
  [
    { role: 'system', content: 'Answer using only the remembered context below.' },
    { role: 'user',   content: `MEMORY:\n${context}\n\nQUESTION: ${question}` },
  ],
  { connectionName: 'fast' },
);
```

## The `{{memories}}` chunk store

`chatMemory.get` runs the same top-K hybrid (vector + BM25) retrieval the `{{memories}}` macro uses:

```js
const mem = await api.memories.chatMemory.get(chatId, { topK: 6 });
console.log(mem.formatted);            // prompt-ready block
console.log(mem.count, 'of', mem.chunksAvailable, 'chunks');
```

This is **equivalent to `api.chats.getMemories()`** — if you don't otherwise need the `memories` permission, that lighter alias under the `chats` permission does the same retrieval. `chatMemory.listChunks(chatId)` inspects the raw index; `warm(chatId, { force })` rebuilds stale chunks (a no-op `status: 'skipped'` when chat vectorization is disabled); `invalidate(chatId)` drops the cached result.

## The entity & relation graph

```js
// Entities — upsert is a smart merge against canonical name + aliases.
const selene = await api.memories.entities.upsert(chatId, {
  name: 'Selene', type: 'character', aliases: ['the Lady of Ashes'], confidence: 0.95,
});
await api.memories.entities.addFacts(selene.id, ["Carries a dagger inscribed with her mother's name"]);
await api.memories.entities.updateStatus(selene.id, { status: 'deceased' });
await api.memories.entities.updateEmotionalValence(selene.id, { betrayal: 0.6, grief: 0.4 });

const list = await api.memories.entities.list(chatId);                 // active, by salience
const found = await api.memories.entities.findByName(chatId, 'Selene'); // name or alias
```

Relations use entity **names** (the host resolves ids), but **both endpoints must already exist** — upsert the entities first, or the edge is silently dropped and `upsert` returns `null`:

```js
await api.memories.entities.upsert(chatId, { name: 'Marcus', type: 'character' });
const edge = await api.memories.relations.upsert(chatId, {
  source: 'Selene', target: 'Marcus', type: 'rival', label: 'duel pending', sentiment: -0.4,
});
if (!edge) console.warn('relation dropped — an endpoint was missing');

const active = await api.memories.relations.list(chatId);             // active edges
const seleneEdges = await api.memories.relations.forEntity(chatId, selene.id);
```

## Consolidations & salience

```js
const arcs    = await api.memories.consolidations.list(chatId, { tier: 2 });
const current = await api.memories.consolidations.latestArc(chatId);
await api.memories.consolidations.run(chatId);   // background, extractive-only — see note

const salience = await api.memories.salience.list(chatId, { limit: 50 });
```

## Vaults & links

Snapshot a chat's cortex state into a vault, then attach it to another chat as read-only knowledge:

```js
const vault = await api.memories.vaults.create({ chatId, name: 'Campaign — Act 1' });
// entities + relations copy synchronously; the chunk copy runs in the background.

await api.memories.links.attach({ chatId: otherChatId, linkType: 'vault', vaultId: vault.id });

// Or interlink two chats so each sees the other's live graph:
await api.memories.links.attach({
  chatId: 'chat-a', linkType: 'interlink', targetChatId: 'chat-b', bidirectional: true,
});
```

Linked data surfaces through `cortex.queryLinked(chatId, { queryText })`. `vaults.reindex(vaultId)` re-runs the chunk copy after an embedding-model swap; `vaults.delete` / `links.remove` clean up.

## Telemetry

```js
const usage = await api.memories.stats.usage(chatId);        // entity/relation/consolidation/salience counts
const phase = await api.memories.stats.ingestionStatus(chatId); // live phase + pending jobs, or null
```

## Notes & gotchas

- **`relations.upsert` needs both endpoints first** — it returns `null` (not the row) when an endpoint is missing. Always upsert entities before edges, and null-check.
- **`consolidations.run` is extractive only** — heuristic, no sidecar LLM. Sidecar-driven consolidation is host-owned and automatic during ingestion. `run` is fire-and-forget; poll `list()` for results.
- **Vault `create` is half-async** — entities + relations are there immediately; chunk retrieval is structural-only until the background LanceDB copy finishes. Use `reindex` to redo it.
- **`chatMemory.warm` no-ops when vectorization is off** (`status: 'skipped'`, `reason: 'chat_vectorization_disabled'`) — check the user's embedding config first.
- **Cortex caches have a ~5-minute TTL** per chat. `getCached` never triggers a re-query.
- **No `processChunk` / `runMaintenance` / synthetic-chunk injection** — cortex ingestion stays the host's responsibility; the bridge is read + structured-edit, not pipeline control.

See the in-app **Reference tab** for the full method list and the `CortexQuery` / `CortexResult` / `MemoryEntity` / `MemoryRelation` / `Vault` / `ChatLink` field shapes.
