# Databanks

`api.databanks.*` is full CRUD over user databanks (vectorised document collections) plus per-document upload, fetch, rename, reprocess, and delete. Use it to programmatically manage the reference material the host pulls into prompts via vector retrieval — knowledge bases, transcript libraries, character backstories, anything text-shaped that benefits from semantic lookup rather than keyword matching.

Two layers — `api.databanks.*` for the bank-level CRUD, `api.databanks.documents.*` for the per-document operations:

```ts
interface DatabanksAPI {
  list(options?):       Promise<{ data: DatabankInfo[]; total: number }>;
  get(databankId):      Promise<DatabankInfo | null>;
  findByName(name, scope?): Promise<DatabankInfo | null>;
  create(input):        Promise<DatabankInfo>;
  update(databankId, input): Promise<DatabankInfo>;
  delete(databankId):   Promise<boolean>;

  documents: {
    list(databankId, options?):     Promise<{ data: DatabankDocumentInfo[]; total: number }>;
    get(documentId):                Promise<DatabankDocumentInfo | null>;
    findByName(databankId, name):   Promise<DatabankDocumentInfo | null>;
    create(databankId, input):      Promise<DatabankDocumentInfo>;
    update(documentId, input):      Promise<DatabankDocumentInfo>;
    delete(documentId):             Promise<boolean>;
    getContent(documentId):         Promise<{ content: string } | null>;
    reprocess(documentId):          Promise<{ success: true; status: 'processing' }>;
    waitUntilReady(documentId, options?): Promise<DatabankDocumentInfo>;
  };
}
```

All 15 methods require the `databanks` permission. They throw `Error: PERMISSION_DENIED:databanks — grant this permission to use this API` when missing.

## Where the retrieval actually happens

**Scripts don't retrieve from databanks at runtime.** This is the most important framing distinction from `api.worldInfo` — both are "context-pulling" surfaces but they're operated differently:

- **Worldinfo entries** are matched against chat content at prompt-assembly time by the host's activation pipeline. Scripts CAN interpose via `api.worldInfo.registerInterceptor` to gate/mutate the activation result.
- **Databank documents** are vectorised by the host on upload, then matched by cosine similarity to the prompt at prompt-assembly time. Scripts manage the document store (upload, rename, delete, reprocess) but do NOT drive retrieval. The retrieval is opaque to scripts.

So `api.databanks.*` is, fundamentally, a **document-store management surface**. The vector search that turns those documents into prompt context is a host-side pipeline that scripts can populate but not interrogate directly. Scripts that need to "search the databank from a trigger handler" don't have an API for that — the closest equivalent is `documents.getContent(documentId)` which returns the raw text, but you'd have to do the matching yourself.

## Scopes — global, character, chat

Every databank belongs to exactly one scope, set at create time and immutable:

| Scope | `scopeId` requirement | Active when |
|---|---|---|
| `'global'` | omit (or pass `null`) | Always available for the active user |
| `'character'` | character UUID | The matching character is in context |
| `'chat'` | chat UUID | The matching chat is in context |

Documents inherit their parent databank's scope — there's no per-document override. The scope decides whether the host considers the databank during retrieval at prompt-assembly time. A `'character'`-scoped databank with `scopeId: 'abc-123'` only contributes to prompts when character `abc-123` is the active speaker; a `'chat'`-scoped bank only fires inside one specific chat session.

For most narrative use cases the choice is straightforward:
- **Global** — encyclopedia-style references the user always wants in scope.
- **Character** — backstory, voice notes, lore tied to one character.
- **Chat** — session-specific worldbuilding, NPC notes, accumulated narrative state.

## The async ingestion lifecycle

Document creation is **non-blocking**. `documents.create(...)` returns immediately with a `DatabankDocumentInfo` whose `status: 'pending'`. The host then queues the document for parsing + chunking + vectorisation, moving it through:

```
pending → processing → ready
                    ↘ error
```

Most scripts that upload then immediately consume the document want to wait for `'ready'` before proceeding. Use `documents.waitUntilReady(documentId, options?)` — it polls `documents.get()` at `pollIntervalMs` (default 500ms) until status reaches `'ready'`, with a `timeoutMs` ceiling (default 60s). On failure modes it throws with a descriptive message:

```ts
options?: {
  timeoutMs?:      number;   // Default 60_000 (60s)
  pollIntervalMs?: number;   // Default 500
}
```

- Throws `… document <id> failed processing: <errorMessage>` if status reaches `'error'`.
- Throws `… document <id> did not reach 'ready' within Nms (last status: …)` on timeout.
- Throws `… document <id> disappeared (deleted?) before reaching 'ready'` if the document is deleted mid-poll.
- Fast path: checks status once before the first sleep, so already-ready documents return instantly.

## Quick start

Create a global databank, upload a document, wait for it to be ready, confirm it landed:

```js
// @triggers ls:startup

const lore = await api.databanks.create({
  name:        'Campaign Lore',
  description: 'Setting notes that should always be reachable.',
  scope:       'global',
});

const doc = await api.databanks.documents.create(lore.id, {
  data:     '# The Drunken Owl\n\nA dimly-lit roadside tavern…',
  filename: 'drunken-owl.md',
});

// Block until parsing + vectorisation finishes (or timeout / error).
const ready = await api.databanks.documents.waitUntilReady(doc.id, {
  timeoutMs: 30_000,
});

console.log(`Document "${ready.name}" ready with ${ready.totalChunks} chunks`);
```

`documents.create` accepts `string | Uint8Array` for `data`. Strings are UTF-8 encoded internally — no need to wrap with `new TextEncoder()`. Use `Uint8Array` directly when your source is already binary (e.g. bytes from `api.utils.http` with `responseType: 'arraybuffer'`).

## File-type constraint

Lumiverse accepts **text-oriented uploads only**. The supported extensions:

`.txt` `.md` `.markdown` `.csv` `.tsv` `.json` `.xml` `.html` `.htm` `.yaml` `.yml` `.log` `.rst` `.rtf`

PDFs, images, archives, audio, and other binary payloads are rejected at ingestion even though `DatabankDocumentCreateInput.data` is typed `string | Uint8Array` (the `Uint8Array` is for already-binary text formats, not for arbitrary bytes). **Max 10 MB per document.**

For non-text persistence:
- `api.files.*` — arbitrary text files (base64-encode binary first).
- `api.images.*` — raw image bytes.
- For PDFs / Office formats that you need vectorised: extract text yourself first (host doesn't ship a parser), then upload the extracted text as `.txt` or `.md`.

## findByName helpers

Both `databanks.findByName(name, scope?)` and `databanks.documents.findByName(databankId, name)` are convenience wrappers over `list()` with a name filter applied locally. They page through results (`limit: 100` per page) and return the first match, or `null` if nothing matches:

```js
// Find a global databank by name, ignore scope-mismatched matches:
const lore = await api.databanks.findByName('Campaign Lore', 'global');

// Find a document within a known bank:
const doc = await api.databanks.documents.findByName(lore.id, 'drunken-owl.md');
```

**Cost**: O(banks) and O(documents-in-bank) respectively. The implementation has a 1000-entry hard cap to bound worst-case latency on pathological accounts — if you have more than 1000 banks (or more than 1000 documents in one bank), the cap kicks in before you'd realistically need it. Fine for hot-path use at typical scales.

When you have the ID already, prefer `get(databankId)` / `documents.get(documentId)` — direct lookup, no list traversal.

## Reading document content

`documents.getContent(documentId)` returns the parsed plain-text content as `{ content: string } | null`:

- Returns `null` if the document doesn't exist OR if its status is not yet `'ready'` (still pending/processing, or it errored before chunks were created).
- Returns `{ content: '...' }` once the document is ready. Note the wrapping object — callers must read `result.content`, not the result directly.

Common pattern: pair with `waitUntilReady` to guarantee the content is available:

```js
await api.databanks.documents.waitUntilReady(doc.id);
const result = await api.databanks.documents.getContent(doc.id);
if (result) {
  console.log(`Document content (${result.content.length} chars):`, result.content);
}
```

`getContent` is useful for scripts that need to re-derive something from the document (extract specific paragraphs, run a custom LLM call against the content, hash the body for change detection). It's NOT how prompt-time retrieval works — see [Where the retrieval actually happens](#where-the-retrieval-actually-happens) above.

## Reprocess — when to use it

`documents.reprocess(documentId)` resets a document to `'pending'`, drops its existing vectors, and queues it for full reingestion. Returns immediately with `{ success: true, status: 'processing' }`. Use it when:

- **The embedding model changed.** Documents vectorised under the old model don't blend cleanly with newly-uploaded ones; reprocess to re-vectorise under the new model.
- **The document errored on first ingestion and you want to retry.** Status moves `error → pending → processing → ready`.
- **You suspect index corruption** (rare). Reprocess rebuilds chunks + vectors from scratch.

Don't reprocess casually — it's a full re-vectorisation. Pair with `waitUntilReady(documentId)` if you need to block on completion.

## Worked example — character-scoped backstory ingestion

A practical pattern: a tool script that uploads a character's backstory text into a character-scoped databank, replacing any prior version:

```js
// @triggers ls:startup
// Imagine this script is wired to a UI button that uploads backstory.

async function ingestBackstory(characterId, name, text) {
  const BANK_NAME = `${name} — Backstory`;

  // Idempotency: re-find existing bank or create fresh.
  let bank = await api.databanks.findByName(BANK_NAME, 'character');
  if (bank && bank.scopeId !== characterId) {
    // Name collision with a different character's bank — bail cleanly.
    api.ui.toast(
      `Bank "${BANK_NAME}" exists but belongs to another character`,
      'error',
    );
    return null;
  }
  if (!bank) {
    bank = await api.databanks.create({
      name:        BANK_NAME,
      description: `Backstory + lore for ${name}.`,
      scope:       'character',
      scopeId:     characterId,
    });
  }

  // Replace prior backstory document if present (name-based dedup).
  const existing = await api.databanks.documents.findByName(bank.id, 'backstory.md');
  if (existing) {
    await api.databanks.documents.delete(existing.id);
  }

  // Upload the fresh content. Note: `data` accepts strings directly.
  const doc = await api.databanks.documents.create(bank.id, {
    data:     text,
    filename: 'backstory.md',
    mimeType: 'text/markdown',
  });

  try {
    const ready = await api.databanks.documents.waitUntilReady(doc.id, {
      timeoutMs: 30_000,
    });
    return ready;
  } catch (err) {
    const msg = err instanceof Error ? err.message : String(err);
    api.ui.toast(`Backstory ingestion failed: ${msg}`, 'error');
    return null;
  }
}
```

A few patterns worth noting:
- **`findByName` first, `create` only if missing** — the dedup pattern for "ensure this bank exists with these properties."
- **Delete-then-recreate for content replacement** — Lumiverse doesn't support in-place content updates; `documents.update` only renames. To replace content, delete + recreate.
- **`waitUntilReady` in a try/catch** — ingestion can fail (file-type rejection, parser error, vector backend down); fold the error into a user-facing toast.

## Error handling

Three failure surfaces:

| Cause | Error shape |
|---|---|
| Permission denied | `Error: PERMISSION_DENIED:databanks — grant this permission to use this API` |
| `waitUntilReady` ingestion failure | `Error: api.databanks.documents.waitUntilReady: document <id> failed processing: <upstream-message>` |
| `waitUntilReady` timeout | `Error: api.databanks.documents.waitUntilReady: document <id> did not reach 'ready' within <N>ms (last status: <pending\|processing>)` |
| `waitUntilReady` mid-poll deletion | `Error: api.databanks.documents.waitUntilReady: document <id> disappeared (deleted?) before reaching 'ready'` |

Beyond these, host-side errors (validation failures, rate limits, ingestion-pipeline errors) bubble up with whatever message the host emits. Wrap mutation calls in try/catch and surface the message via toast or console.

## When to use this vs `api.worldInfo`

The two namespaces look similar at the surface level but address different needs:

| Need | Use |
|---|---|
| Hand-curated lore that activates on keyword match in chat | `api.worldInfo` |
| Document-style reference material retrieved by host-side vector similarity | `api.databanks` |
| Programmatic activation gating (turn-based, character-state-driven) | `api.worldInfo.registerInterceptor` |
| Programmatic content management (upload/delete/rename via tool scripts) | Either — but `api.databanks` is built for document-shaped content; `api.worldInfo` for entry-shaped |
| Re-running retrieval on changed embedding settings | `api.databanks.documents.reprocess` |

Both surfaces are script-side CRUD. The retrieval logic — which entries activate, which documents get matched — runs host-side at prompt-assembly time. Scripts populate; the host retrieves. The two coexist cleanly: many setups use world-info for hand-curated narrative beats and databanks for long-form reference text.

## Version notes

Implemented in **v0.25.2+** via native `spindle.databanks.*` (lumiverse-spindle-types ≥0.4.48). The `findByName` helpers and `documents.waitUntilReady` polling helper are LumiScript-side conveniences; everything else is a thin pass-through over the Spindle base.
