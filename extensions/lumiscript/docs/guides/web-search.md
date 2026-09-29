# Web search

`api.webSearch.*` runs a search against the user's configured search provider (SearXNG today) and hands you back ranked results — and, optionally, the scraped page text plus a prompt-ready context block you can drop straight into an LLM call. Requires the `web_search` permission.

Two methods:

- `query(options)` — run a search.
- `getSettings()` — read the safe provider config (never the API key).

## Check first, then query

Web search only works when the user has actually configured a provider. `query()` **rejects** with `"Web search is disabled"` if there's none — so branch on `getSettings().enabled` before you search:

```js
const settings = await api.webSearch.getSettings();
if (!settings.enabled) {
  api.ui.toast('Web search is not configured.', 'warning');
  return;
}
const res = await api.webSearch.query({ query: 'site reliability engineering' });
```

`getSettings()` returns a **safe** view — `enabled`, `provider`, `apiUrl`, `defaultResultCount`, `maxResultCount`, `maxPagesToScrape`, `maxCharsPerPage`, `language`, `safeSearch`, `engines`, `requestTimeoutMs`, and `hasApiKey`. There is deliberately **no `apiKey` field** — you can see *whether* a key is set (`hasApiKey`), never the key itself.

## `scrape` is the important knob

```js
// Default: scrape ON — results + scraped page text + a prompt-ready context block.
const full = await api.webSearch.query({ query: 'mars helicopter', count: 5 });

// Fast path: results only (titles / URLs / snippets), no page fetches.
const quick = await api.webSearch.query({ query: 'mars helicopter', scrape: false });
```

`query(options)` takes:

- `query` (required) — the free-text query. Trimmed by the host; empty is rejected.
- `count?` — desired result count. Clamped to `maxResultCount`; omit for `defaultResultCount`.
- `scrape?` — **defaults to `true`**. When on, the top results are fetched and their text extracted.

The response shape depends on `scrape`:

```js
{
  query:    'mars helicopter',           // the trimmed query that ran
  results:  [ { title, url, snippet, engine?, score? }, ... ],
  documents?: [ { title, url, snippet, content?, contentLength?, sourceType?, error? }, ... ],
  context?:   '…assembled, prompt-ready text…',
}
```

`documents` and `context` are **present only when `scrape` is on** (the default). With `scrape: false` you get just `results`.

## Grounding an LLM reply (the main use case)

The `context` block is built for exactly this — paste it into a prompt and let the model answer from fresh sources:

```js
const { context } = await api.webSearch.query({ query: userQuestion, count: 5 });

const answer = await api.llm.generate(
  [
    { role: 'system', content: 'Answer using only the search context provided. Cite URLs.' },
    { role: 'user',   content: `CONTEXT:\n${context}\n\nQUESTION: ${userQuestion}` },
  ],
  { connectionName: 'fast' },
);
```

(Calling the LLM additionally needs the `generation` permission — see [Calling the LLM](llm.md).)

If you'd rather assemble your own prompt, iterate `documents` (each carries `content`, the extracted page text, plus an `error` field when a particular page couldn't be scraped):

```js
const { documents } = await api.webSearch.query({ query: topic });
for (const doc of documents ?? []) {
  if (doc.error) continue;            // skip pages that failed to scrape
  addToPrompt(`## ${doc.title}\n${doc.url}\n${doc.content}`);
}
```

## As a Council tool

Web search pairs naturally with `api.tools.register` — give the Council a "look it up" capability that grounds its reasoning in current information:

```js
api.tools.register(
  'web_lookup',
  { display_name: 'Web lookup', description: 'Search the web for current information', council_eligible: true },
  async (args) => {
    const { context } = await api.webSearch.query({ query: args.query, count: 4 });
    return context ?? 'No results.';
  },
);
```

See [Registering tools](tools.md) for the full tool surface.

## Notes

- **Permission.** Both `query` and `getSettings` require `web_search`. Without it they throw `PERMISSION_DENIED`.
- **The active user is implicit.** You don't pass a user id — the call is scoped to the active user automatically.
- **Results aren't persisted.** Each `query` is a live fetch. Cache the response yourself (`api.scriptStorage` / `api.db`) if you need to reuse it.

See the in-app **Reference tab** for the full `WebSearchOptions`, `WebSearchResponse`, `WebSearchResult`, `WebSearchDocument`, and `WebSearchSettings` field lists.
