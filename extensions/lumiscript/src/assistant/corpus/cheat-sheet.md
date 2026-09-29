# LumiScript API Cheat-Sheet

> Auto-generated from `src/components/reference/ReferenceTab.tsx` and `src/types/editor-lib.ts` by `scripts/gen-assistant-corpus.ts`. Do not edit by hand — re-run `bun run gen:corpus` instead.

> For deeper detail (full TS signature, examples, see-also), call the `lookup_api(method)` tool with the fully-qualified method name from any table below.

## Globals

Bindings available in every script body:

- `api` — full LumiScript API surface (`api.chat`, `api.llm`, `api.utils`, ...). See sections below.
- `script` — script self-info and built-in library loader. `script.id`, `script.name`, `script.type`, `script.require(name)`.
- `z` — [Zod](https://zod.dev) schema builder. Use for `api.llm.generateStructured(messages, schema)`.
- `data` — trigger-event payload object. The triggering event is identified by `data.__event` (e.g. `"MESSAGE_SENT"`). Shape varies per event — see the **Events** section.
- `console` — log to the LumiScript console panel. Standard `log`, `warn`, `error` methods.

## Trigger model

**LumiScript does NOT use runtime event subscription.** There is no `api.on()`, no `api.events.on()`, no `api.subscribe()`, no `api.listen()`, no `api.triggers.on()` — and no, you don't write `event.on('message', handler)` either. None of those exist. **Do not lookup_api on any of them.** The paradigm is completely different from Node.js EventEmitter or DOM event listeners.

**Event wiring is configured in the editor UI, not in the script source.** When you create or edit a script in LumiScript's script editor, an event-selector control lets you pick which Lumiverse events should run this script's body. There is no script-side syntax that subscribes — the wiring lives in the script's editor config, alongside its name, enabled flag, and binding.

**`// @triggers EVENT_NAME[, ...]` in a script header is INFORMATIVE ONLY.** It's a comment convention you may write at the top of your script to document which events the script is *intended* to be wired to. The host does not parse it — writing `@triggers` has zero runtime effect. The actual events that fire your script come from the editor-UI wiring, not from this comment. (A future LumiScript version may add a programmatic-subscription API; current versions do not.)

When a wired event fires, the **script body itself runs as the handler** — the entire body executes top-to-bottom with the event's payload available as the `data` global. No callback, no subscription object, no listener registry. `data.__event` carries the event name (e.g. `"MESSAGE_SENT"`); the rest of `data` is the event-specific payload (see the **Lumiverse Events** section for per-event payload shapes).

```js
// Optional documentary comment — has no effect on what triggers the script.
// Actual wiring (e.g. "MESSAGE_SENT, MESSAGE_EDITED") is set in the editor UI.
// @triggers MESSAGE_SENT, MESSAGE_EDITED

// The body runs every time a wired event fires.
// `data.__event` identifies which event triggered this invocation;
// the rest of `data` is the event-specific payload.
if (data.__event === 'MESSAGE_SENT') {
  const score = await api.llm.generateStructured(/* ... */);
  await api.databanks.documents.create('reviews-databank-id', {
    data: JSON.stringify(score),
    filename: `score-${data.message.id}.json`,
  });
}
```

The full list of available event names + their payload shapes + firing semantics is in the **Lumiverse Events** section below. To make a script react to one of those events, open it in the editor and select the event in the UI.

**Execution isolation — every fire is a fresh function scope.** When a wired event fires, the host wraps your script body in a brand-new `AsyncFunction` and invokes it ONCE. Module-scope `let` / `const` / `var` declarations at the top of your script body are LOCAL to that one invocation — they do NOT survive to the next fire of the same script. A pattern like:

```js
let bankId = null;
if (data.__event === 'ls:startup')  bankId = await ensureBank();
if (data.__event === 'MESSAGE_SENT' && bankId) { /* ... */ }
```

...does NOT work. The `ls:startup` fire writes to `bankId` and returns; the function instance is discarded; the next `MESSAGE_SENT` fire is a fresh `AsyncFunction` invocation with its own brand-new `bankId = null`. The two fires share no local state.

For state that needs to survive across fires, pick one of:

- **`globalThis.<key>`** — process-scoped, persists for the lifetime of the script-runner subprocess (i.e. until the extension reloads). Cheapest option; ideal for in-memory caches. Example: `globalThis.lsScoringBankId ??= await ensureBank();`. (Note: globalThis values survive *editor saves* too — see the saved memory note about globalThis-cache-invalidation traps if you cache anything keyed on script identity.)
- **`api.variables.{local,global,character,chat}`** — durable JSON-serialised stores with explicit scope semantics. Survives extension reloads.
- **Registered handlers** (`api.macros.register(...)`, `api.tools.register(...)`, `api.chat.registerContentProcessor(...)`, etc.) — these capture closures over the proxy and *do* survive across fires until the script is disabled or deleted. Useful for "subscriber-only" patterns where a script registers a handler in one fire and it fires later from a different source. **`api.broadcast.on(...)` is the one exception**: its subscriptions also persist between fires, but the owning script's next run clears them at its START and the body re-registers — so a re-firing script never stacks duplicate listeners, and a fire-once script (e.g. wired to `ls:startup`) keeps them until disabled or deleted.

**Common misconception**: "the local variable persists until the extension reloads." It does NOT. Each fire is its own scope. The boundary is per-fire, not per-extension-load.

**Three similar-sounding systems, three different problems** — keep them straight:

- **Editor-UI event wiring** — react to Lumiverse host *lifecycle* events (MESSAGE_SENT, GENERATION_ENDED, CHAT_CHANGED, ...). Configured per-script in the script editor.
- `api.broadcast.*` — real-time *script-to-script* pub/sub between user scripts running inside the same LumiScript extension. Use for custom in-extension messaging.
- `api.events.*` — *persistent log* of custom events (`track` / `query` / `replay` / `getLatestState`). Use for audit trails, state-resuming scripts, custom analytics. **NOT** for subscribing to host events.

**Lifecycle events (`ls:startup` / `ls:teardown` / `ls:reload`).** Three LumiScript-synthetic events that mark script-state transitions. `ls:startup` fires on cold-boot entry into active state (extension boot OR disable→enable transition). `ls:teardown` fires on exit (disable / delete). `ls:reload` fires on hot in-place body refresh (autosave-with-`// @ls:reload-on-edit`-directive OR manual Reload button click). The first two are symmetric subscription events — opt in via the editor's event-picker checkbox. **`ls:reload` is the exception**: it is NOT in the picker — it fires deterministically as a side-effect of the Reload button (always) or the autosave directive (always, for scripts that declared it). Parallel to the Run button: a user action, not a subscription. The body re-runs on all three events; branch on `data.__event` to differentiate handling. **Reload also wipes per-script state** (DOM, modals, widgets, drawer tabs, handler closures, interceptors, injections, ...) BEFORE the body re-runs, while preserving `api.scriptStorage`, `api.theme.*` contributions, and the worker's `script.require()` cache — so an `ls:reload` body run lands into a clean slate equivalent to a fresh `ls:startup`. Use `if (data.__event === "ls:startup")` branches for code that should run EXCLUSIVELY on cold boot (default-config seeding, persistent-data migrations) — those fires once per active-state entry, not on every Reload click. To make init code re-run on both events, branch on `(data.__event === "ls:startup" || data.__event === "ls:reload")` or put it at the top of the body un-gated. v1.0.0-rc.8+ for the wipe semantics.

**Sandbox hardening.** The script-runner sandbox locks down host capabilities that user scripts have no business reaching. Two layers gate this:

- **Dispatch-time source check.** Scripts containing any of the following patterns are REJECTED before they run; the editor console shows a `[security]` entry naming the rejected pattern:
  - `import('...')` / `await import('...')` — dynamic import. Use `script.require('library-name')` for inter-script dependencies (see the **Built-in Libraries** section).
  - bare `require('...')` — CommonJS-style global require. Same migration: `script.require('library-name')`. Note: `script.require(...)` and method-style `obj.require(...)` are NOT rejected (the source check uses `(?<!\.)` lookbehind to exclude method access).
  - `new Function('...')` / `Function('...')` — Function constructor. Define functions with normal syntax (`function foo() {}` / `const foo = () => {}`); same lookbehind exempts method-style `obj.Function(...)`.
  - `.constructor.constructor` — prototype-chain access to the Function constructor.
  - literal `globalThis.Bun` / `globalThis["Bun"]` — Bun runtime API. Use `api.utils.http.*` for HTTP, `api.files.*` (with `allowDangerous`) for filesystem.
  - literal `globalThis.process` / `globalThis["process"]` — host process. Use `api.enclave.*` for secrets, never read host env vars from a script.

- **Runtime globalThis lockdown.** At subprocess startup, every globalThis property not on the LumiScript allowlist is replaced with `undefined`. `typeof X` returns `'undefined'`; reading `X.method()` throws `TypeError`. Affects: `fetch`, `Worker`, `WebSocket`, `BroadcastChannel`, `XMLHttpRequest`, `EventSource`, `prompt`, `onerror`, `onmessage`, `postMessage`, `removeEventListener`, and Bun-specific Node-compat module globals (`fs`, `http`, `net`, `os`, `tls`, `vm`, `worker_threads`, `child_process`, `ffi`, `sqlite`, etc.). Standard ES built-ins (Object, Array, Promise, JSON, Math, Date, RegExp, Map, Set, etc.), timers (`setTimeout`, `setInterval`, `clearTimeout`, `clearInterval`, `queueMicrotask`, `structuredClone`), cancellation primitives (`AbortController`, `AbortSignal`), `crypto`, `WebAssembly`, `performance`, `Buffer`, base64 codecs (`atob`, `btoa`), web data carriers (Blob, File, FileReader, FormData, Headers, Request, Response), event types (Event, EventTarget, CustomEvent), and streams (ReadableStream/WritableStream/TransformStream) are all left accessible. The canonical list is `SAFE_GLOBALS` in `src/script-runner/child-entry.ts`.

- **Layer-2 setTimeout/setInterval string-form rejection.** `setTimeout` and `setInterval` are whitelisted but monkey-patched to reject the string-form callback (the legacy `setTimeout('foo()', 100)` API that some runtimes still honour). Passing a string throws `TypeError: setTimeout requires a function callback (string form is not supported in the LumiScript sandbox)`. Always pass a function.

**Console rate-limit.** Unhandled rejections from a single script are rate-limited to **10 per 60s window**; further rejections drop silently with a `"N additional rejection(s) were suppressed"` summary on the next-window rejection. Prevents a runaway loop from filling the editor console + backend stderr with millions of lines. The 10-per-minute cap is intentional — most legitimate scripts produce ≪ 1 rejection/minute under normal operation.

## Permission model

**DO NOT WRITE `// @permissions` OR `// @permission` IN YOUR SCRIPT.** Neither is a LumiScript directive. **LumiScript does not parse `@permissions`, `@permission`, or `@triggers` directives** — `// @triggers`, despite the name, is purely a documentary comment with no runtime effect (event wiring happens in the editor UI; see the **Trigger model** section). (The one script-header directive LumiScript *does* read is the unrelated `// @ls:reload-on-edit` hot-reload opt-in — nothing to do with permissions or wiring.) Writing `@permissions` or `@permission` in a script header is a **no-op** — it looks like it grants permissions but actually does nothing; your script will then fail at runtime when it calls a gated method. This is the single most common script-permission-bug we see; if you find yourself reaching for an `@permission` directive, stop and re-read this section.

How permissions actually work: LumiScript permissions are declared **at the extension level** in `spindle.json` and granted once by the user when the extension is enabled. **There are no per-script permission declarations** — every script inside the LumiScript extension shares the same grant set. The user (not the script author) controls what's granted. (Earlier mental models à la SillyTavern, where each script declares its own perms, do NOT apply here.)

**Permissions gate `api.*` method calls, NOT the `data` trigger global.** Reading `data.message.content` from a `MESSAGE_SENT` trigger does NOT require `chat_mutation` — the host already routed the event payload to your script for free. Permissions only kick in when your script reaches back through the API (e.g. `api.chat.getMessages`, `api.chat.editMessage`). Don't list a permission unless your script actually calls a gated method.

**`allowDangerous` is SEPARATE** — it's a per-script LumiScript-level UI toggle (in the script-list row), NOT a Spindle permission. It gates a **fixed set of surfaces**: outbound HTTP (`api.utils.http.*`), encrypted secrets (`api.enclave.*`), file I/O (`api.files.*`), and `api.chat.clearAllInjections`. **It does NOT gate any other surface.** Raw image bytes (`api.characters.setAvatar`, `api.images.upload`), DOM injection (`api.ui.dom.*`), theme manipulation (`api.theme.*`), character mutations (`api.characters.update`), OAuth callbacks (`api.oauth.*`), image generation (`api.imageGen.*`), and every other gated method flow through their own dedicated Spindle permissions only — no `allowDangerous` toggle required. If you find yourself reaching for `allowDangerous` to "unlock" a surface that isn't on the fixed list above, stop: the surface is gated by its own permission instead. When a method's permission tag below shows `[X, + allowDangerous]`, BOTH gates must be on: the extension must have permission `X` granted AND the calling script must have `allowDangerous` toggled on.

| Permission | What it gates |
|---|---|
| `app_manipulation` | Gates ONLY `api.ui.dom.*` (DOM injection, `addStyle`, delegation), `api.ui.showAdvancedModal`, and `api.theme.*`. Does NOT gate `api.chats.*` (use `chats`), `api.characters.*` (use `characters`), `api.ui.showContextMenu` (free-tier, system-themed), `api.ui.toast`, `api.ui.pushNotification`, or any other UI primitive — those have their own permissions. Mental model: this is the "script-owns-its-own-shell-pixels" gate. |
| `characters` | CRUD on characters via `api.characters.*`. |
| `chat_mutation` | Read / send / edit / delete chat messages. Required for most `api.chat.*` operations. |
| `chats` | Chat session metadata + CRUD on the chat list. Distinct from message content (chat_mutation). |
| `context_handler` | Paired with `interceptor` for `api.chat.inject`. Gates the host's context-handler stage where `mode: 'context'` injections are prepended at index 0 of the assembled prompt. LumiScript declares both by default. |
| `cors_proxy` | Outbound HTTP via `api.utils.http.*`. Paired with `allowDangerous` (both gates required). |
| `databanks` | CRUD on databanks + their documents via `api.databanks.*` (vectorised reference material attached to global / character / chat scopes). |
| `ephemeral_storage` | TTL-bound `api.files.temp*` file storage with auto-expiry. |
| `event_tracking` | Record + query persistent events via `api.events.*`. |
| `generation` | Call LLM providers via `api.llm.*`. Also required to register world-info interceptors that touch the assembled prompt. |
| `image_gen` | Generate images via `api.imageGen.*` against the user's configured image-gen connection profiles. Returns `ImageGenResult` with both a base64 data URL (immediate render) and (when persisted) a canonical `imageId` accepted by `api.images.get` / `api.theme.extractColors` / `characters.setAvatar`, plus an auth-free `imageUrl` for push notifications. Provider/connection metadata available for dynamic parameter UIs. |
| `images` | Persist + retrieve images in Lumiverse's image store via `api.images.*`. Returns `ImageInfo` whose `id` can be passed to `api.theme.extractColors`, stored on a character avatar, or attached to a databank document. |
| `interceptor` | Register prompt injections (`api.chat.inject`) and content processors (`api.chat.registerContentProcessor`) — anything that mutates host data mid-flight. **Note**: world-info interceptors (`api.worldInfo.registerInterceptor`) are gated by `generation` instead, since they run at prompt-assembly time. |
| `macro_interceptor` | Register macro-resolution interceptors (`api.macros.registerInterceptor`). Performance-sensitive; gated separately from `interceptor`. |
| `memories` | Full access to Lumiverse's hybrid memory architecture via `api.memories.*` — the Memory Cortex (entity/relation graph, narrative-arc consolidations, salience, vault snapshots, chat interlinks, fused retrieval) and Long-Term Chat Memory (the vectorized chunk store behind the `{{memories}}` macro). Read + write; every chat-scoped call is ownership-checked against the active user. |
| `oauth` | OAuth callback handling via `api.oauth.*` — the only inbound-HTTP hook Spindle exposes to extensions. Wrapper is intentionally thin: it covers the callback registration, CSRF state nonce, and the callback URL path. Constructing the authorize URL, exchanging the code for a token, and persisting + refreshing tokens are the script's responsibility (pair with `api.utils.http` + `api.enclave`). |
| `personas` | CRUD on personas via `api.personas.*`, plus read/update of global add-ons via `api.personas.addons.*`. |
| `presets` | CRUD on generation presets + their prompt blocks via `api.presets.*` (parameters, ordered prompt blocks with roles/positions/depth, behavior settings, metadata, plus host-derived category groupings). |
| `push_notification` | OS-level push notifications via `api.ui.pushNotification` (delivered when the app is unfocused). |
| `regex_scripts` | CRUD on regex find/replace scripts via `api.regexScripts.*`. |
| `tools` | Register Council-eligible LLM tools via `api.tools.*`. |
| `ui_panels` | Float widgets / dock panels — surfaces that hold their own persistent UI region in the app shell. |
| `web_search` | Run web searches via `api.webSearch.*` against the user's configured search provider (e.g. SearXNG). `query` returns ranked results plus optional scraped page content + an assembled context string; `getSettings` exposes the safe provider config (never the API key — only `hasApiKey`). |
| `world_books` | CRUD on world books and entries via `api.worldInfo.*`. |

## api.chat

| Method | Args | Description |
|---|---|---|
| async `getMessages` | options? | Get messages in the current chat. Pass `{ last: N }` for the N most recent or `{ first: N }` for the N oldest (omit for the full message list). Requires chat_mutation permission. [chat_mutation] |
| async `sendMessage` | content, options? | Append a new message. Options: role, metadata, triggerGeneration (when true, fires the host's full chat-orchestration pipeline after the append — same effect as the user pressing Enter on an empty input bar), generation (per-call ChatGenerationOptions overrides — connectionId / personaId / presetId / parameters / etc., consulted only when triggerGeneration is true). Requires chat_mutation permission. [chat_mutation] |
| async `editMessage` | id, contentOrPatch | Edit a message by ID. Pass a string to replace the active swipe's content, or a MessagePatch to update swipes / swipeId / swipeDates / reasoning / metadata. Patches touching swipe-shaped fields fire SWIPE_EDITED alongside MESSAGE_EDITED. [chat_mutation] |
| async `deleteMessage` | id | Delete a message by ID. [chat_mutation] |
| `getChatId` | — | Return the active chat ID, or null. |
| async `getMetadata` | key | Get a metadata value from the current chat. Requires chats permission (note: distinct from chat_mutation — chats gates session-level fields, chat_mutation gates message content). [chats] |
| async `setMetadata` | key, value | Set a metadata key (read-modify-write). Requires chats permission. [chats] |
| `inject` | id, content, options? | Register a prompt injection. Options: mode, role, depth, ephemeral. [interceptor] |
| `removeInjection` | id | Remove one injection by ID. |
| `getInjections` | — | List all active injections across all scripts. |
| `clearInjections` | — | Remove all injections from this script. Requires interceptor permission. [interceptor] |
| `clearAllInjections` | — | Remove ALL injections across all scripts (cross-script wipe — use sparingly). Requires interceptor permission + allowDangerous. [interceptor, + allowDangerous] |
| `registerContentProcessor` | handler, options? | Register a handler that fires on chat message origin events — `create` (user/assistant message write), `update`, `swipe_add`, `swipe_update`, auto-inserted greetings on the write side, AND on per-message display rendering (`render`). Return a patch { content?, extra? } to transform what gets stored or shown. Options: id, priority (default 100), origin filter, timeoutMs (default 2000). NOT invoked for api.chat.* mutations (loop safety). `extra` is IGNORED on swipe origins (swipes share the parent message's extra) and on `render` (no row to mutate). Returns handle { id, remove }. Requires chat_mutation permission. [chat_mutation] |
| `listContentProcessors` | — | List all currently registered message content processors across all scripts. |
| async `setMessageHidden` | id, hidden | Mark a single message as hidden or visible. Hidden messages are excluded from vector retrieval but still included in prompt assembly. Toggle pattern: pass `true` to hide, `false` to unhide. Persists on the message — survives reloads. Requires chat_mutation permission. [chat_mutation] |
| async `setMessagesHidden` | ids, hidden | Bulk variant of `setMessageHidden`. Max 500 IDs per call. Same hidden-flag semantics (excluded from vector retrieval, still included in prompt assembly). Requires chat_mutation permission. [chat_mutation] |
| async `isMessageHidden` | id | Check whether a message is hidden. Returns false for messages that have never had the flag set (default state). Requires chat_mutation permission. [chat_mutation] |
| async `setStyleMode` | mode | Set the active chat's CSS containment mode. 'bounded' (default) clamps extension- and card-injected content inside the message stream; 'extension-relaxed' lets a `position: fixed` element injected into a message paint at viewport scope — e.g. a full-bleed overlay from an injected-DOM or card script. Distinct from `api.ui.mountApp` (a host-owned document.body portal): this relaxes the in-chat container. Requires app_manipulation permission. [app_manipulation] |
| `onMessageTag` | tagName, handler, options? | Register a handler for a custom XML-like tag in chat messages (e.g. `<dice>3d6</dice>`). The handler receives a MessageTagEvent (tagName, content, attrs, messageId). `options.removeFromMessage` strips the tag from the rendered message. Returns an unsubscribe function. Requires chat_mutation permission. [chat_mutation] |

## api.llm

| Method | Args | Description |
|---|---|---|
| async `generate` | messages, options? | Generate a text response from the LLM. [generation] |
| `generateStream` | messages, options? | Streaming variant of generate. Async iterator of StreamChunk values (token / reasoning / done). Break out of for await or pass options.signal to cancel. [generation] |
| async `generateStructured` | messages, schema, options? | Generate and parse a structured JSON response against a Zod or JSON Schema. [generation] |
| async `generateWithTools` | messages, tools, options?, schema? | Generate with tool schemas. Returns text or function calls for an agentic loop. [generation] |
| async `dryRun` | options? | Assemble the full prompt without calling the LLM. Returns messages, token counts, WI stats. [generation] |

## api.connections

> **Concepts:** Read-only view of the user's LLM connection profiles — the same profiles `api.llm` resolves against. `list()` / `get(id)` / `getDefault()` (the `is_default` profile, else the first available) / `findByName(name)` (case-insensitive). Each `Connection` is snake_case (mirrors the host DTO: `id`, `name`, `provider`, `api_url`, `model`, `preset_id`, `is_default`, `has_api_key`, `metadata`, `reasoning_bindings`) and is a SAFE view — it carries `has_api_key: boolean`, NEVER the key. `id` / `name` map straight to `api.llm` options `connectionId` / `connectionName`, so you can let the user pick a connection (pair with `api.ui.components.mountSelect` / `mountModelCombobox`) and route a generation to it. **Intentionally read-only** — connections hold provider credentials and are managed by the user in Lumiverse settings, never by scripts (no create / update / delete). Free tier. The active userId is folded in implicitly.

| Method | Args | Description |
|---|---|---|
| async `list` | — | List the user's LLM connection profiles (read-only; never includes API keys — only has_api_key). Returns Connection[]. Free tier. Pair with api.ui.components.mountSelect/mountModelCombobox for pickers. |
| async `get` | connectionId | Get a connection profile by ID, or null if not found/accessible. Returns Connection \| null. |
| async `getDefault` | — | Get the user's default connection (is_default, or the first available), or null. |
| async `findByName` | name | Find a connection by name (case-insensitive), or null. The id/name map to api.llm options.connectionId/connectionName. |

## api.webSearch

> **Concepts:** Native web search against the user's configured provider (SearXNG today). `query({ query, count?, scrape? })` returns ranked `results` (title / url / snippet); with `scrape` (default **true**) it ALSO scrapes the top pages into `documents` (full text) and assembles a prompt-ready `context` string — drop `context` straight into an LLM prompt for grounding / RAG. `scrape: false` is the fast path: titles / URLs / snippets only, no page fetches. `query` REJECTS with "Web search is disabled" when the user has no provider configured — branch on `getSettings().enabled` first. `getSettings()` returns the SAFE config (provider, limits, language, safeSearch, engines, `hasApiKey`) — NEVER the API key itself. The active userId is folded in implicitly. Use cases: Council "look it up" tools, grounding a reply in current info, RAG over fresh results. Requires `web_search` permission.

| Method | Args | Description |
|---|---|---|
| async `query` | options | Search the user's configured provider (SearXNG). options: { query (required), count?, scrape? (default true) }. Returns WebSearchResponse { query, results: WebSearchResult[], documents?, context? }. With scrape (default) you also get scraped documents + a prompt-ready context block; scrape:false returns only results (titles/URLs/snippets). Rejects "Web search is disabled" if no provider configured. Requires web_search. [web_search] |
| async `getSettings` | — | Read the safe web-search config (NEVER the API key — only hasApiKey). Returns WebSearchSettings { enabled, provider, apiUrl, defaultResultCount, maxResultCount, maxPagesToScrape, maxCharsPerPage, language, safeSearch, engines, hasApiKey, requestTimeoutMs }. Branch on enabled before query(). Requires web_search. [web_search] |

## api.users

> **Concepts:** Active-user context probes. `isVisible()` — true if the user has the app visible in at least one session, false if every session is hidden / backgrounded (or there is no open session); use it to choose a push notification (when the user is away) vs. an in-app toast / UI update (when present), or to defer background work. `getRole()` — the user's Lumiverse role: `'operator' | 'admin' | 'user'` (internal owners report as `operator`); gate operator-only diagnostics or admin-only actions on it. Free tier; the active userId is folded in implicitly.

| Method | Args | Description |
|---|---|---|
| async `isVisible` | — | True if the active user has the app visible in at least one session; false if every session is hidden/backgrounded or there is no open session. Returns Promise<boolean>. Free tier. Use to gate push notifications (when hidden) vs. in-app UI (when visible). |
| async `getRole` | — | The active user's Lumiverse role: 'operator' \| 'admin' \| 'user' (internal owners report as operator). Returns Promise<UserRole>. Free tier. |

## api.version

> **Concepts:** The running Lumiverse versions: `getBackend()` and `getFrontend()` each return a semver string. Use for feature-gating — branch a script on whether the host is new enough for a given event or API. Compare numerically (split on `.`, strip any `-rc` / `+build` suffix) rather than string `===`; LumiScript's own `host-version.ts` `compareVersions` helper is the reference implementation. Free tier. (This is the script-facing view of the same version probe LumiScript uses internally for its `minimum_lumiverse_version` check.)

| Method | Args | Description |
|---|---|---|
| async `getBackend` | — | The running Lumiverse backend server's semantic version string (e.g. '1.2.0'). Returns Promise<string>. Free tier. Pair with feature gating / compatibility checks. |
| async `getFrontend` | — | The running Lumiverse frontend bundle's semantic version string. Returns Promise<string>. Free tier. |

## api.permissions

> **Concepts:** Read the extension's runtime permission grant set. `getGranted()` returns the Spindle permissions the user granted the extension (extension-level, not per-script); `has(permission)` checks one. Use as a pre-flight guard so a script degrades gracefully — `if (await api.permissions.has('images')) { … }` — instead of catching a `PERMISSION_DENIED` error after the fact. The reactive host handlers (onDenied / onChanged) are intentionally not exposed. Free tier.

| Method | Args | Description |
|---|---|---|
| async `getGranted` | — | The Spindle permissions currently granted to the LumiScript extension (e.g. ['chat_mutation', 'generation']). Extension-level, not per-script. Returns Promise<string[]>. Free tier. |
| async `has` | permission | Whether a specific permission is currently granted. Returns Promise<boolean>. Use as a pre-flight check before a gated call — if (await api.permissions.has('images')) { … } — so a script degrades gracefully instead of catching a PERMISSION_DENIED error after the fact. Free tier. |

## api.variables.local / api.variables.global / api.variables.character / api.variables.chat

_The same method set applies to each of the 4 namespaces above._

> **Concepts:** Four scopes with identical method surface (get / set / delete / has / clear). `local` is per-script and transient between trigger fires. `global` is extension-wide, persists across scripts. `character` follows the active character UUID. `chat` follows the active chat UUID. The `character` and `chat` scopes auto-resolve from the active context — no need to pass the UUID explicitly.

| Method | Args | Description |
|---|---|---|
| async `get` | key, defaultValue? | Get a variable. Returns defaultValue if the key does not exist. |
| async `set` | key, value | Set a variable (JSON-serialized). |
| async `delete` | key | Delete a variable. Returns true if it existed. |
| async `has` | key | Check if a variable exists. |
| async `clear` | — | Delete all variables in this store. |

## api.json

| Method | Args | Description |
|---|---|---|
| `parse` | text | Parse a JSON string. Throws on invalid JSON. |
| `stringify` | data, pretty? | Serialize to JSON. Pass true for formatted output. |
| `clone` | data | Deep clone a value. |
| `get` | data, path, defaultValue? | Get a nested value by dot-path (e.g. "user.address.city"). |
| `set` | data, path, value | Set a nested value by dot-path. |
| `merge` | ...objects | Deep merge objects. Later arguments override earlier ones. |
| `isValid` | text | Check if a string is valid JSON. |
| `filter` | data, predicate | Filter an array by predicate. |
| `sort` | data, key, direction? | Sort array by key (asc or desc). |
| `uniq` | data | Deduplicate array. |
| `flatten` | data | Flatten a nested array. |
| `query` | data, queryString | Run a jsonquery pipeline (jq-like). See jsonquerylang.org. |

## api.utils

| Method | Args | Description |
|---|---|---|
| `uuid` | — | Generate a UUID v4 string. Cryptographically random (uses crypto.randomUUID). |
| `shortId` | — | Generate a short random ID (8 chars, URL-safe). Cryptographically random (derived from crypto.randomUUID). |
| async `wait` | ms | Pause execution for ms milliseconds. |
| `random.int` | min, max | Random integer in [min, max] inclusive. **NOT cryptographically secure** — uses Math.random for gameplay/UI use cases. For tokens or security-sensitive identifiers use api.utils.uuid / shortId or globalThis.crypto.getRandomValues. |
| `random.float` | min, max | Random float in [min, max). **NOT cryptographically secure** (Math.random — see random.int). |
| `random.pick` | array | Pick a random element from an array. **NOT cryptographically secure** (Math.random — see random.int). |
| `random.bool` | — | Random true/false. **NOT cryptographically secure** (Math.random — see random.int). |
| `random.chance` | probability | Returns true with probability p (0–1). **NOT cryptographically secure** (Math.random — see random.int). |
| `random.shuffle` | array | Return a shuffled copy of the array. **NOT cryptographically secure** (Math.random — see random.int). |
| async `http.get` | url, options? | GET request via cors_proxy. Requires allowDangerous. [cors_proxy, + allowDangerous] |
| async `http.post` | url, body, options? | POST request via cors_proxy. Requires allowDangerous. [cors_proxy, + allowDangerous] |
| async `http.put` | url, body, options? | PUT request via cors_proxy. Requires allowDangerous. [cors_proxy, + allowDangerous] |
| async `http.delete` | url, options? | DELETE request via cors_proxy. Requires allowDangerous. [cors_proxy, + allowDangerous] |
| async `http.request` | url, options | Custom HTTP request via cors_proxy. Requires allowDangerous. [cors_proxy, + allowDangerous] |
| async `template.render` | template, data?, options? | Two-pass render: Lumiverse macros first, then Handlebars. Returns Promise<string>. |
| `template.compile` | template | Pre-compile a Handlebars template for sync reuse. No macro resolution. |
| `template.registerHelper` | name, fn | Register a custom Handlebars helper scoped to this script. |
| async `macros.resolve` | template, options? | Resolve Lumiverse macros without the Handlebars pass. Pass { commit: false } for a dry resolve — extension macro handlers that honour the flag skip their side effects (useful for template previews). chatId / characterId default to the active context. Returns Promise<{ text, diagnostics }>. |
| `image.detectMime` | bytes | Magic-byte sniff. Returns image MIME type (image/png, image/jpeg, image/webp, image/gif, image/bmp) or null. Pair with api.characters.setAvatar when the source format is unknown — the host defaults to image/png on missing mimeType. |
| `image.dataUrlToBytes` | url | Parse a base64 data URL (data:<mime>;base64,<payload>) into { data: Uint8Array, mimeType }. Returns null for malformed or non-base64 data URIs. |
| `image.bytesToDataUrl` | bytes, mimeType | Encode bytes + MIME into a base64 data URL. Useful for previewing proposed avatars in the UI before committing via setAvatar. |

## api.ui

> **Concepts:** User-facing UI surface — notifications (`toast`, `pushNotification`), dialogs (`prompt`, `confirm`, `editText`, `showModal`, `showContextMenu`), script-owned regions (`showAdvancedModal`, `createFloatWidget`, `registerDrawerTab`), the native file picker (`pickFile`), and **navigation**. The sub-namespaces `api.ui.dom.*` (DOM injection), `api.ui.components.*` (host components), and `api.ui.events.*` (reactive keyboard / drawer / settings state) have their own concept entries. `pickFile(options?)` opens the browser's native file picker and resolves with the selected `PickedFile[]` (bytes as `Uint8Array`) — free tier (the native dialog is the user-action gate); rejects if a file exceeds `maxSizeBytes`, resolves `[]` on cancel. **Script-owned UI regions** form a ladder of increasing scope: `showAdvancedModal` (a modal), `createFloatWidget` (a draggable overlay, `ui_panels`), `registerDrawerTab` (a sidebar tab), and `mountApp(options?)` — a route-persistent, full-bleed `document.body` portal (`position: 'start' | 'end' | 'app-overlay'`) for full-screen overlays / persistent chrome beyond the others. All hand back a `.root` DOMHandle you fill via `api.ui.dom.*`; `mountApp` / `showAdvancedModal` / DOM / components require `app_manipulation`. **Navigate vs. contribute** is the key distinction for the navigation methods: `registerDrawerTab` *contributes* a new tab your script owns, whereas `openDrawerTab(id)` / `openSettings(viewId?)` / `openCommandPalette()` *navigate* the user to surfaces that already exist (built-in OR contributed by any extension). Enumerate targets with `getDrawerTabs()` / `getSettingsTabs()` (each returns id + names + keywords + source) and deep-link by id — you can jump into another extension's tab if you know its id. Navigation calls resolve once the host dispatches the event (the frontend applies it asynchronously); `closeDrawer` / `closeSettings` / `closeCommandPalette` reverse them. Use for onboarding nudges ("open the Connections drawer"), agent-driven walkthroughs, and "fix it" deep links. The navigation methods are free tier — other `api.ui.*` methods carry their own permissions (see the per-method rows).

| Method | Args | Description |
|---|---|---|
| `toast` | message, type?, options? | Show a native Lumiverse toast notification. Fire-and-forget. Rate-limited 5/10s. Options: title, duration. |
| async `prompt` | message, defaultValue?, options? | Show a themed text input dialog. Returns entered string (trimmed) or null if cancelled. Options: placeholder, submitLabel, cancelLabel, multiline. |
| async `confirm` | message, title?, options? | Show a themed confirmation dialog. Returns true if confirmed. Options: variant (info/warning/danger/success), confirmLabel, cancelLabel. |
| `showModal` | items, options | Display structured read-only content in a themed modal. Returns ModalHandle { result, openRequestId, close() }. Await handle.result for dismissal. Options: title (required), width, maxHeight, persistent. |
| `showAdvancedModal` | options | Open a modal whose body is fully script-owned via a DOMHandle (handle.root). Use api.ui.dom.* on root.update/on/... to render and wire interactive UIs. Up to 2 concurrent modals per script (pre-checked backend-side). Returns AdvancedModalHandle { modalId, root, dismissed, setTitle, dismiss, onDismiss }. Requires app_manipulation. [app_manipulation] |
| async `showContextMenu` | options | Show a themed context menu at a screen position and await the user's selection. Resolves with the chosen item's key, or null if dismissed. Options: { position: { x, y }, items: [{ key, label, type?, disabled?, danger?, active? }] }. Pair with a contextmenu event listener using { preventDefault: true } to suppress the native browser menu. Free-tier. |
| `registerInputBarAction` | options | Register an action inside the chat input-bar Extras popover. Extension actions are visually grouped under a teal-badged extension header. Optional subtitle adds a second line under the label (status text, shortcut, etc.) — settable via setSubtitle for live updates. Limits: 4 per script (pre-checked backend-side), 12 global. Returns InputBarActionHandle { actionId, setLabel, setSubtitle, setEnabled, onClick, destroy }. Free-tier. |
| `createFloatWidget` | options | Create a small draggable widget overlaying the app. Body DOM is fully script-owned via handle.root (DOMHandle). Supports snap-to-edge, chromeless mode, drag-end callbacks for position persistence. Limits: 2 widgets per script (pre-checked backend-side), 8 global. Returns FloatWidgetHandle { widgetId, root, moveTo, getPosition, setVisible, isVisible, onDragEnd, destroy }. Requires ui_panels. [ui_panels] |
| `mountApp` | options? | Mount a route-persistent, full-bleed document.body portal — full-screen overlays or persistent chrome beyond dock / drawer / float. options: { className?, position? ('start'\|'end'\|'app-overlay') }. Body DOM is fully script-owned via handle.root (DOMHandle); render + wire it via api.ui.dom.*. Returns MountedAppHandle { mountId, root, setVisible, destroy }. Requires app_manipulation. [app_manipulation] |
| `registerDrawerTab` | options | Register a tab in the ViewportDrawer sidebar. Body DOM is script-owned via handle.root (DOMHandle). Tabs auto-appear in the command palette (Ctrl+K) searchable by title, shortName, description terms, keywords, and the extension name. Limits: 1 tab per script (LumiScript-enforced), 4 total across all LumiScript scripts (Spindle host cap), 8 global. Returns DrawerTabHandle { tabId, root, setTitle, setShortName, setBadge, activate, onActivate, destroy }. Free-tier. |
| async `editText` | title?, value?, options? | Open the native Lumiverse expanded text editor with macro syntax highlighting. Blocks until close. Returns edited text or null if cancelled. Options: placeholder. |
| async `pushNotification` | title, body, options? | Send an OS push notification. Only delivered when app is unfocused. Returns { sent }. Options: tag (dedup), url, icon, rawTitle, image. Requires push_notification. [push_notification] |
| async `getPushStatus` | — | Check if push notifications are available. Returns { available, subscriptionCount }. Requires push_notification. [push_notification] |
| async `getDrawerTabs` | — | List discoverable drawer tabs (built-in + extension-contributed) visible to the user. Returns UIDrawerTab[] { id, shortName, tabName, tabDescription, keywords, source ("builtin"\|"extension"), extensionId? }. Pair with openDrawerTab(id) to build a custom jump-to picker. Free-tier. |
| async `getSettingsTabs` | — | List discoverable settings tabs visible to the user (role-restricted tabs are filtered out). Returns UISettingsTab[] { id, shortName, tabName, tabDescription, keywords, role? }. Free-tier. |
| async `openDrawerTab` | tabId | Open the drawer to a specific tab id (built-in or extension-contributed — ids from getDrawerTabs). Resolves once the host dispatches the navigation; the frontend applies it asynchronously. Free-tier. |
| async `closeDrawer` | — | Close the drawer if it is currently open. Free-tier. |
| async `openSettings` | viewId? | Open the settings modal to a tab id (e.g. 'connections', 'display' — ids from getSettingsTabs). Omit viewId to land on 'display'. Free-tier. |
| async `closeSettings` | — | Close the settings modal if it is currently open. Free-tier. |
| async `openCommandPalette` | — | Open the command palette overlay (the Ctrl+K surface). Free-tier. |
| async `closeCommandPalette` | — | Close the command palette overlay if it is currently open. Free-tier. |
| async `pickFile` | options? | Open the browser's native file picker and return the selected file(s). options: { accept? (string[] of extensions/MIME types), multiple? (default false), maxSizeBytes? }. Returns Promise<PickedFile[]> where PickedFile is { name, mimeType, sizeBytes, bytes: Uint8Array }. Resolves [] if the user cancels; REJECTS if a file exceeds maxSizeBytes (mirrors the host throw). The native dialog is the user-action gate, so free-tier. Feed bytes into api.images.upload / api.db / api.files; decode text via new TextDecoder().decode(file.bytes). |

## api.ui.events

> **Concepts:** Reactive Lumiverse UI state — virtual keyboard, side drawer, settings modal. Each surface has BOTH a snapshot getter and a change subscription. **Snapshots** (`getKeyboardState` / `getDrawerState` / `getSettingsState`) resolve with the latest known state (keyboard `{ visible, insetBottom, viewportWidth, viewportHeight }`, drawer `{ open, tabId }`, settings `{ open, view }`) — async because they cross the worker boundary, but served from a cache the frontend keeps fresh (no per-call round-trip). **Subscriptions** (`onKeyboardChange` / `onDrawerChange` / `onSettingsChange`) fire on every change with the new state and return an unsubscribe fn. **Primary use case: mobile-safe positioning** — reposition float widgets / injected DOM when the on-screen keyboard opens (`insetBottom`) or the visual viewport changes. Free tier. Lifecycle: a live subscription keeps the script pinned (it won't be evicted while a handler is registered) and is torn down automatically on script disable — call the returned unsub when you're done to release the pin sooner. Keyboard events are most relevant on mobile / PWA; on desktop the drawer + settings channels are the active ones.

| Method | Args | Description |
|---|---|---|
| async `getKeyboardState` | — | The current virtual-keyboard snapshot. Returns Promise<UIKeyboardState> { visible, insetBottom, viewportWidth, viewportHeight }. Free-tier. Resolves from a backend cache the frontend keeps fresh. |
| `onKeyboardChange` | handler | Subscribe to keyboard visibility / safe-area changes. handler receives UIKeyboardState. Returns an unsubscribe fn. Free-tier. The subscription keeps the script alive while registered and is torn down on disable. Primary use: mobile-safe widget positioning (reposition on insetBottom). |
| async `getDrawerState` | — | The current side-drawer snapshot. Returns Promise<UIDrawerState> { open, tabId }. Free-tier. |
| `onDrawerChange` | handler | Subscribe to drawer open/close + tab changes. handler receives UIDrawerState. Returns an unsubscribe fn. Free-tier. |
| async `getSettingsState` | — | The current settings-modal snapshot. Returns Promise<UISettingsState> { open, view }. Free-tier. |
| `onSettingsChange` | handler | Subscribe to settings open/close + active-view changes. handler receives UISettingsState. Returns an unsubscribe fn. Free-tier. |

## api.ui.dom

> **Concepts:** DOM injection surface. `inject(target, html, position?)` returns a `DOMHandle`; subsequent calls go through the handle (`update`, `remove`, `on`, `injectChild`, `read`, `makeDraggable`). `addStyle(css)` adds a scoped stylesheet (wrapped in `@scope ([data-ls-script="<id>"])` — only matches script-injected DOM, doesn't cascade into the host app shell). `injectAtMessage(messageId, html, options?)` attaches DOM to a specific chat message (header, before, after, footer positions). `delegate(selector, event, handler, options?)` installs a capture-phase event-delegated listener at a known root — use to react to events on host DOM you didn't inject (e.g. LLM-emitted interactive elements inside `.mes_text` content). `cleanup()` removes ALL of this script's DOM in one call. Requires `app_manipulation` permission.

**Styling host UI requires inline `<style>` injection, NOT `addStyle`.** Because `addStyle` wraps its CSS in `@scope ([data-ls-script="<id>"])`, every rule only matches descendants of script-injected DOM — host elements (chat input bar, message bubbles, toolbar buttons, the body itself) are unreachable. To style host DOM (`button[aria-label="..."]`, `[class*="hostClass"]`, `body:has(.my-toggle:checked) ...` selectors, etc.), include a `<style>` block in your `inject()` HTML. CSS rules inside a `<style>` element are document-global regardless of where the tag sits in the DOM, so they reach host elements. The wrapper (`data-ls-script="<id>"`) still carries cleanup attribution — script disable removes the wrapper, the `<style>` goes with it, host UI returns to baseline. Same lifecycle as `addStyle`; different reach.

**Top-level at-rules don't survive `addStyle`.** `addStyle` wraps the whole stylesheet in its `@scope` block, and at-rules CSS only permits at the top level — `@font-face`, `@keyframes`, and similar — are invalid nested inside `@scope`, so the browser silently drops them (the surrounding scoped style rules still apply, which makes the failure easy to miss: a custom `@font-face` never registers and the text falls back to a default face, with no error and no console entry). Put any such at-rule in an inline `<style>` block inside an `inject()` HTML payload instead — that `<style>` is document-global (NOT `@scope`-wrapped), so the `@font-face` / `@keyframes` registers normally. Same split as host-UI styling above: descendant-scoped rules go through `addStyle`; anything that must live at stylesheet top level goes in an inline `<style>`.

**Inline `<style>` blocks and `opts.id`-dedup DON'T MIX.** Re-firing `inject(target, html, { id })` with an `id` that already exists triggers the `dom_update` IPC path on the frontend, which replaces the wrapper's `innerHTML` in place. Browsers don't reliably reactivate `<style>` elements added via `innerHTML` — the tag is in the DOM but its rules don't register into the active stylesheet list. Net effect: first fire works, second fire silently loses every CSS rule from the inline `<style>` (host elements re-appear; the script's own UI loses its styling too). For scripts that combine inline `<style>` blocks with re-firing triggers (`ls:startup` + `CHAT_SWITCHED` + manual Run-button), DROP the `id` and call `api.ui.dom.cleanup()` at the top of the body instead. `cleanup()` removes only THIS script's DOM + styles, leaves other scripts untouched, and forces every fire to take the fresh-inject path which parses `<style>` correctly.

**HTML sanitisation.** Every HTML payload — `inject`, `update`, `injectChild`, `injectAtMessage` — is run through DOMPurify with strict defaults plus `FORBID_TAGS: ['iframe', 'frame', 'object', 'embed', 'form']` (matching the host's three-layer CSP + DOMPurify + X-Frame-Options policy). DOMPurify defaults strip all `on*` event handler attributes (`onclick`, `onerror`, `onload`, `onmouseover`, etc.), `<script>` tags, `javascript:` URLs, `data:` URLs on dangerous elements, the `formaction` attribute, and other XSS vectors. **When content is stripped, the affected script's editor console gets a `[security]` entry** naming what was removed (tells you why your `<button onclick="...">` button doesn't work).

**Event handlers — use delegation, not inline.** Inline `onclick="..."` attributes are stripped by sanitisation. The replacement is `DOMHandle.on(event, handler)` event delegation with a `data-*` attribute on the trigger element:

```js
// Stripped at injection — button does nothing:
handle.update('<button onclick="doThing()">Click</button>');

// Recommended:
handle.update('<button data-action="do-thing">Click</button>');
handle.on('click', (ev) => {
  if (ev.target.dataset.action !== 'do-thing') return;
  // ...do thing
});
```

The delegation reads `event.target.dataset.action` (not a `closest()` walk), so when the visible button content is bigger than its padded text area (icon SVG, `<img>`, decorative `<span>`), the click target can be the inner element — which lacks the `data-*` attribute, so the handler silently no-ops. Standard fix: `pointer-events: none` on the decorative inner content so clicks pass through to the button itself.

**Read access.** `handle.read(options?)` returns a `SerializedDOMElement` snapshot of the bound element — `tag`, `attrs` (with internal `data-ls-*` and `data-spindle-ext` stripped), `text`, `childCount`, plus optional `html` for the inner markup. Async because it awaits a frontend roundtrip. Multi-root or text-only injections return the LumiScript wrapper snapshot; single-root injections return the user's element directly. Returns `null` if the element vanished on the frontend (live DOM raced ahead of script logic) — distinct from `DomHandleReleasedError` which throws after `handle.remove()`.

**Spindle wrapper nesting (gotcha).** `ctx.dom.inject` wraps every payload in a Spindle wrapper `<div data-spindle-ext>` containing a LumiScript wrapper `<div data-ls-el data-ls-script>` containing your HTML. Two levels of wrapper sit ABOVE your root element. Code that walks down to the user's root must do `wrapper.firstElementChild?.firstElementChild`. The `data-ls-script` attribute is what `addStyle`'s `@scope` rules match.

| Method | Args | Description |
|---|---|---|
| `inject` | target, html, options? | Inject sanitized HTML at a CSS selector. Returns DOMHandle { id, update, remove, on }. Options object (single arg — NOT `inject(target, html, position, options)`): `position` (default "beforeend"), `id` (stable ID for idempotent injection — re-firing with the same id triggers in-place innerHTML update via dom_update IPC instead of a fresh insert). **Note**: when the injected HTML contains an inline `<style>` block (the host-CSS-targeting pattern — see api.ui.dom NAMESPACE_CONCEPTS), do NOT use `id`-dedup. The dom_update path replaces wrapper innerHTML, and browsers don't reliably reactivate `<style>` blocks added that way — second fire silently loses every CSS rule. Pattern for inline-style scripts that re-fire: drop `id`, call `api.ui.dom.cleanup()` at body start instead. Requires app_manipulation. [app_manipulation] |
| `injectAtMessage` | messageId, html, options? | Inject sanitized HTML into a message bubble. Waits up to 5 s for the element if not yet rendered. Options: position ("footer" default / "header"), id (stable ID). Returns DOMHandle. Requires app_manipulation. [app_manipulation] |
| `addStyle` | css, opts? | Add a `<style>` element scoped to this script via `@scope ([data-ls-script="<id>"])`. Returns `{ remove() }`. Use `--lumiverse-*` CSS variables for theming. Pass `{ id: 'foo' }` for idempotent re-injection — calling `addStyle` again with the same id removes the prior stylesheet first (useful for dev-iteration where the CSS source changes between fires). Without an id, every call adds a fresh stylesheet. **Scope limitation**: rules ONLY match descendants of script-injected DOM. Cannot reach host elements (chat input bar, message bubbles, toolbar buttons, the body, etc.) because `@scope` excludes everything outside the script's wrappers. To style host UI, include an inline `<style>` block inside an `inject()` HTML payload instead — CSS rules in a `<style>` element are document-global regardless of where the tag sits. **Top-level at-rules** (`@font-face`, `@keyframes`) are likewise dropped here — they're invalid nested inside `@scope`, so the browser silently discards them; deliver those via an inline `<style>` block too, never `addStyle`. See api.ui.dom NAMESPACE_CONCEPTS for the full pattern. Requires app_manipulation permission. [app_manipulation] |
| `delegate` | selector, event, handler, options? | Attach an event-delegated listener at a known root, matching descendants by CSS selector. Lets scripts react to clicks/changes on DOM the script didn't inject — e.g. interactive elements emitted by the LLM in chat-message content. Single host-side capture listener per (root, event) tuple regardless of how many scripts subscribe; selector matching happens frontend-side via event.target.closest(). Default scope (options.root: "chat") restricts matching to chat content; "document" matches anywhere on the page. Returns an unsubscribe function. Requires app_manipulation. [app_manipulation] |
| `cleanup` | — | Remove all DOM injections, styles, and delegations created by THIS script (other scripts' DOM is untouched). Auto-fired on script disable / delete — manual call is for re-fire scenarios where you want to wipe and rebuild from scratch. **Canonical use**: at the top of a script body that combines re-firing triggers (`ls:startup` + `CHAT_SWITCHED` + manual Run) with inline `<style>` blocks in injected HTML. Calling `cleanup()` then `inject(...)` guarantees a fresh-inject path on every fire — which parses `<style>` correctly — instead of dom_update-via-id-dedup which doesn't reactivate inline styles. Requires app_manipulation. [app_manipulation] |

## api.ui.components

| Method | Args | Description |
|---|---|---|
| `mountBadge` | target, options? | Mount a themed host badge into a script-owned slot. `target` is a DOMHandle (inject an empty container via api.ui.dom.inject first). Options: text, color ("neutral"\|"primary"\|"success"\|"warning"\|"danger"\|"info"), size ("sm"\|"md"\|"pill"). Returns MountedComponentHandle { id, update(patch), destroy() } synchronously. update/destroy are fire-and-forget. Requires app_manipulation. [app_manipulation] |
| `mountSpinner` | target, options? | Mount a themed loading spinner into a script-owned slot (DOMHandle target). Options: size (px, default 16), fast (boolean). Returns MountedComponentHandle. Requires app_manipulation. [app_manipulation] |
| `mountSwitch` | target, options? | Mount a themed toggle switch into a script-owned slot (DOMHandle target). Options: checked, onChange(checked:boolean), size ("sm"\|"md"), disabled, ariaLabel. Returns MountedValueComponentHandle { id, update, destroy, getValue(): Promise<boolean> } — getValue() is ASYNC (a frontend round-trip), unlike the host's sync getValue. onChange fires into your script on every toggle. Requires app_manipulation. [app_manipulation] |
| `mountTextInput` | target, options? | Mount a themed single-line text input into a script-owned slot (DOMHandle target). Options: value, onChange(value:string), placeholder, autoFocus, disabled, className, ariaLabel. Returns MountedValueComponentHandle { ..., getValue(): Promise<string> } (async getValue). onChange fires on every user change. Requires app_manipulation. [app_manipulation] |
| `mountTextArea` | target, options? | Mount a themed multi-line text editor (DOMHandle target). Options: value, onChange(value:string), placeholder, rows (default 4), disabled, className, ariaLabel. Returns MountedValueComponentHandle (getValue(): Promise<string>). Requires app_manipulation. [app_manipulation] |
| `mountNumericInput` | target, options? | Mount a themed validated number input (DOMHandle target). Options: value (number\|null), onChange(value:number\|null), allowEmpty, integer, min, max, step, placeholder, disabled. Returns MountedValueComponentHandle (getValue(): Promise<number\|null>). Requires app_manipulation. [app_manipulation] |
| `mountNumberStepper` | target, options? | Mount a themed number input with +/- buttons (DOMHandle target). Like mountNumericInput minus integer; step defaults to 1. Returns MountedValueComponentHandle (getValue(): Promise<number\|null>). Requires app_manipulation. [app_manipulation] |
| `mountCheckbox` | target, options? | Mount a themed checkbox (DOMHandle target). Options: checked, onChange(checked:boolean), label, hint, disabled. Returns MountedValueComponentHandle (getValue(): Promise<boolean>). Requires app_manipulation. [app_manipulation] |
| `mountRangeSlider` | target, options | Mount a themed touch-friendly range slider (DOMHandle target). options is REQUIRED (min + max are mandatory). Options: min (required), max (required), value, step, integer, onCommit(v:number) [once per gesture], onDragValue(v:number\|null) [live], label, hint, format ({decimals,prefix,suffix}), disabled, className. Returns MountedValueComponentHandle (getValue(): Promise<number>). Requires app_manipulation. [app_manipulation] |
| `mountSelect` | target, options? | Mount a themed searchable single-select dropdown (DOMHandle target). Options: options (SpindleSelectOption[]), value, onChange(value:string), placeholder, searchPlaceholder, clearable, clearLabel, leading cells, grouping, portal/align/maxHeight/minWidth, disabled, etc. Returns MountedValueComponentHandle (getValue(): Promise<string>). Requires app_manipulation. [app_manipulation] |
| `mountMultiSelect` | target, options? | Mount a themed searchable multi-select (DOMHandle target). Like mountSelect but value/onChange use string[]. Returns MountedValueComponentHandle (getValue(): Promise<string[]>). Requires app_manipulation. [app_manipulation] |
| `mountFolderDropdown` | target, options? | Mount a themed folder picker with inline create-folder (DOMHandle target). Options: folders (string[]), value, onChange(folder:string), onCreateFolder(name:string), placeholder, disabled. Returns MountedValueComponentHandle (getValue(): Promise<string>). Requires app_manipulation. [app_manipulation] |
| `mountModelCombobox` | target, options? | Mount the themed connection-aware model picker (DOMHandle target). Connection-bound mode: pass connection {kind:"llm"\|"image"\|"tts"\|"embedding", id?} and the host manages the model list. Manual mode: pass models[] + onRefresh. Other options: value, onChange(model:string), appearance, placeholder, etc. Returns MountedValueComponentHandle (getValue(): Promise<string>). NOTE: the handle's refresh() method is a deferred follow-up; connection-bound mode auto-manages the list without it. Requires app_manipulation. [app_manipulation] |
| `mountPagination` | target, options | Mount themed page navigation (DOMHandle target). options is REQUIRED (currentPage, totalPages, onPageChange(page:number) are mandatory). Optional: perPage, perPageOptions, onPerPageChange(n:number), totalItems. Fully controlled — call handle.update({currentPage}) after navigation. Returns MountedComponentHandle (no getValue). Requires app_manipulation. [app_manipulation] |
| `mountCloseButton` | target, options? | Mount a themed close (X) button (DOMHandle target). Options: onClick(), size ("sm"\|"md"), variant ("subtle"\|"solid"), position ("static"\|"absolute"), iconSize. Returns MountedComponentHandle (no getValue). Requires app_manipulation. [app_manipulation] |
| `mountCollapsibleSection` | target, options | Mount a collapsible section (DOMHandle target). options is REQUIRED (title is mandatory). Options: title (required), iconSvg, iconUrl, badge, defaultExpanded (default true), onToggle(expanded:boolean). The host owns the header chrome; the returned MountedCollapsibleSectionHandle adds: body (a DOMHandle you inject/update content into), isExpanded(): Promise<boolean>, expand(), collapse(), toggle(). Requires app_manipulation. [app_manipulation] |

## api.files

> **Concepts:** Three storage tiers, each with its own method-name prefix. `user*` is per-user persistent (survives extension reload, scoped to the active user). `shared*` is extension-wide persistent (shared across users). `temp*` is TTL-bound (deleted after `ttlMs` expires; requires `ephemeral_storage` permission). Same operations across tiers — read / write / delete / exists / list / mkdir / stat / move — just with the tier prefix on each method name.

| Method | Args | Description |
|---|---|---|
| async `userRead` | path | Read a file as UTF-8 text. |
| async `userWrite` | path, data | Write UTF-8 text (creates dirs as needed). |
| async `userDelete` | path | Delete a file. |
| async `userExists` | path | Check if a path exists. |
| async `userList` | prefix? | List files under a prefix. |
| async `userMkdir` | path | Create a directory. |

## api.files

| Method | Args | Description |
|---|---|---|
| async `sharedRead` | path | Read a file as UTF-8 text. |
| async `sharedWrite` | path, data | Write UTF-8 text (creates dirs as needed). |
| async `sharedDelete` | path | Delete a file. |
| async `sharedExists` | path | Check if a path exists. |
| async `sharedList` | prefix? | List files under a prefix. |
| async `sharedStat` | path | Get file metadata (size, modifiedAt, isFile, isDirectory). |
| async `sharedMkdir` | path | Create a directory. |
| async `sharedMove` | from, to | Move or rename a file. |

## api.files

> **Concepts:** TTL-bound ephemeral storage (`spindle.ephemeral`), gated by `allowDangerous` + `ephemeral_storage`. Text I/O (`tempRead` / `tempWrite`) AND **binary I/O** (`tempReadBinary` → `Uint8Array`, `tempWriteBinary(path, Uint8Array, options?)`) — use the binary pair for images / PDFs / any non-text blob; bytes cross the IPC intact (no base64 hop). `tempWrite` / `tempWriteBinary` options are `{ ttlMs?, reservationId? }`. Files auto-expire after `ttlMs`; `tempClearExpired()` sweeps expired entries early. **Quota subsystem**: the ephemeral pool is bounded per-extension AND globally. `tempGetPoolStatus()` returns the snapshot (`TempPoolStatus` — global + this-extension max/used/reserved/available bytes + fileCount / fileCountMax). For a large write, **reserve up front** with `tempRequestBlock(sizeBytes, { ttlMs?, reason? })` → `TempReservation { reservationId, sizeBytes, expiresAt }`, pass that `reservationId` to the write options so it can't fail partway through on a full pool, and `tempReleaseBlock(reservationId)` if you end up not using it. **Use cases**: cache a downloaded asset across two fires, buffer intermediate state during a long job — anything large + transient where you don't want to spend durable storage. For persistent bytes use `api.files.user*` / `shared*`; for images Lumiverse should own use `api.images.*`.

| Method | Args | Description |
|---|---|---|
| async `tempRead` | path | Read a file as UTF-8 text. |
| async `tempWrite` | path, data, options? | Write UTF-8 text. Options: { ttlMs?, reservationId? } — ttlMs sets expiry; reservationId charges the write against a tempRequestBlock reservation. |
| async `tempReadBinary` | path | Read a file as raw bytes. Returns Promise<Uint8Array>. Pair with tempWriteBinary for caching images / PDFs / other binary blobs within quota. |
| async `tempWriteBinary` | path, data, options? | Write raw bytes (Uint8Array). Options: { ttlMs?, reservationId? }, same as tempWrite. Bytes cross the IPC intact (no base64). |
| async `tempDelete` | path | Delete a file. |
| async `tempList` | prefix? | List files under a prefix. |
| async `tempStat` | path | Get file metadata (sizeBytes, createdAt, expiresAt?). |
| async `tempClearExpired` | — | Remove all expired files. Returns count removed. |
| async `tempGetPoolStatus` | — | Read the ephemeral-storage quota snapshot. Returns TempPoolStatus — global + this-extension max/used/reserved/available bytes plus fileCount / fileCountMax. Check available before a large write. |
| async `tempRequestBlock` | sizeBytes, options? | Reserve sizeBytes of quota up front (so a large write can't fail partway). Options: { ttlMs?, reason? }. Returns TempReservation { reservationId, sizeBytes, expiresAt } — pass reservationId to tempWrite/tempWriteBinary options, or tempReleaseBlock to free it. |
| async `tempReleaseBlock` | reservationId | Release a reservation from tempRequestBlock you did not use. |

## api.characters

| Method | Args | Description |
|---|---|---|
| async `list` | options? | List characters (paginated). Returns { data, total }. [characters] |
| async `get` | id | Get a character by ID. Returns null if not found. [characters] |
| async `getByName` | name | Find the first character whose name exactly matches (case-sensitive). Scans all pages. Returns null if no match. [characters] |
| async `create` | input | Create a new character. [characters] |
| async `setAvatar` | id, avatar | Replace a character's avatar image. `avatar` is { data: Uint8Array, filename?, mimeType? }. Useful for image-gen integrations or bulk avatar tooling. [characters] |
| async `update` | id, input | Update a character. [characters] |
| async `delete` | id | Delete a character. Returns true if deleted. [characters] |

## api.chats

| Method | Args | Description |
|---|---|---|
| async `list` | options? | List chat sessions (paginated). Options: characterId, limit, offset. [chats] |
| async `get` | id | Get a chat session by ID. [chats] |
| async `getActive` | — | Get the currently active chat session. [chats] |
| async `update` | id, input | Update a chat session name or metadata. [chats] |
| async `delete` | id | Delete a chat session and all its messages. [chats] |
| async `getMemories` | chatId?, options? | Retrieve long-term memory chunks via vector search. Falls back to active chat. [chats] |

## api.worldInfo

| Method | Args | Description |
|---|---|---|
| async `list` | options? | List world books (paginated). |
| async `get` | ref | Get a world book by ID or name. |
| async `create` | input | Create a world book. |
| async `update` | ref, input | Update a world book by ID or name. |
| async `delete` | ref | Delete a world book and all its entries. |
| async `entries.list` | ref, options? | List entries in a world book. |
| async `entries.get` | entryId | Get a single entry by ID. |
| async `entries.create` | ref, input | Create a new entry in a world book. |
| async `entries.update` | entryId, input | Update an entry by ID. |
| async `entries.delete` | entryId | Delete an entry by ID. |
| async `entries.listByAutomationIdPrefix` | prefix | Find all entries across all world books whose automationId starts with the given prefix. Useful for enumerating / cleaning up entries a script owns (e.g. "lumiscript:<scriptId>:" convention). Returns WorldInfoEntry[]; O(books × entries-per-book). |
| async `getCapturedActive` | chatId? | Get all entries that would activate for the current chat (full pipeline). |
| async `getGlobal` | — | Read the IDs of the user's globally-active world books — books applied to EVERY chat, independent of character/chat scope. Requires world_books permission. |
| async `setGlobal` | refs | Replace the set of globally-active world books. Refs accept a name or UUID (resolved like get/update/delete). Returns the applied ID list; refs that don't resolve to an existing book are dropped by the host. Requires world_books permission. |
| async `activateGlobal` | ref | Activate a single world book globally (name or UUID). Returns the updated global ID list. Throws if the book does not exist. Requires world_books permission. |
| async `deactivateGlobal` | ref | Deactivate a single globally-active world book (name or UUID). No-op if it was not active. Returns the updated global ID list. Requires world_books permission. |
| `registerInterceptor` | handler, options? | Register a handler that runs BEFORE world info activation. Returns disable / enable / force / mutate decisions for the candidate entries. Returns handle { id, remove }. Multiple handlers compose by priority; vote-off precedence on disabled. 2s soft timeout (configurable). Requires generation. [generation] |
| `listInterceptors` | — | Sync read of all currently-registered world-info interceptors. Diagnostic surface. Returns RegisteredWorldInfoInterceptorInfo[]. [generation] |

## api.databanks

> **Concepts:** Three ownership scopes — `global` (no owner key), `character` (owned by character UUID), `chat` (owned by chat UUID). Documents within a databank inherit their parent's scope. Document ingestion is **asynchronous**: `documents.create()` returns immediately with `status: 'pending'`; use `documents.waitUntilReady(docId)` to await chunking + vectorization. For input-bar actions or other UI surfaces that need ready-state confirmation, prefer `waitUntilReady` over manual polling.

**File-type constraint**: Lumiverse accepts text-oriented uploads only — `.txt`, `.md`, `.markdown`, `.csv`, `.tsv`, `.json`, `.xml`, `.html`, `.htm`, `.yaml`, `.yml`, `.log`, `.rst`, `.rtf`. PDFs, images, archives, audio, and other binary payloads are rejected at ingestion even though `DatabankDocumentCreateInput.data` is typed `string | Uint8Array`. For non-text persistence, use `api.files.*` (UTF-8 strings — base64-encode binary first) or `api.images.*` (raw image bytes). Max 10 MB per document.

| Method | Args | Description |
|---|---|---|
| async `list` | options? | List databanks (paginated). Options: limit, offset, scope, scopeId. Returns { data: DatabankInfo[], total }. Requires databanks permission. |
| async `get` | databankId | Get a databank by ID. Returns null if not found. Requires databanks permission. |
| async `findByName` | name, scope? | Find the first databank whose display name exactly matches (case-sensitive) within an optional scope. Convenience over list(). Returns null if no match. Requires databanks permission. |
| async `create` | input | Create a new databank. `input.scope` must be one of `'global' \| 'character' \| 'chat'` (DatabankScope) — `'script'` is NOT a valid scope. `scopeId` is REQUIRED for `'character'` and `'chat'` scopes; omit for `'global'`. Requires databanks permission. |
| async `update` | databankId, input | Update a databank (name / description / enabled). Scope cannot be changed after creation. Requires databanks permission. |
| async `delete` | databankId | Delete a databank and all its documents. Returns true if deleted. Requires databanks permission. |
| async `documents.list` | databankId, options? | List documents inside a databank (paginated). Returns { data: DatabankDocumentInfo[], total }. Requires databanks permission. |
| async `documents.get` | documentId | Get a document by ID. Returns null if not found. Requires databanks permission. |
| async `documents.findByName` | databankId, name | Find the first document whose display name exactly matches inside a databank. Returns null if no match. Requires databanks permission. |
| async `documents.create` | databankId, input | Upload a document. **Required input fields**: `data` (`string \| Uint8Array` — NOT `content`) and `filename` (string with extension, e.g. `'notes.md'`). **Optional**: `mimeType`, `name` (display override). Returns immediately with `status: 'pending'` — ingestion (chunking + vectorisation) runs async. Use `waitUntilReady()` or poll `get()` to await completion. Max size 10 MB; supported extensions in DatabankDocumentCreateInput. Requires databanks permission. |
| async `documents.update` | documentId, input | Update document display name (URL slug regenerates). Requires databanks permission. |
| async `documents.delete` | documentId | Delete a document. Returns true if deleted. Requires databanks permission. |
| async `documents.getContent` | documentId | Read the document's ingested text content. Returns `{ content: string }` on success, or `null` if the document does not exist OR has not finished processing — check `status === 'ready'` via `get()` first, or call `waitUntilReady()` to block. Requires databanks permission. |
| async `documents.reprocess` | documentId | Reset a document to `status: 'pending'`, drop its vectors, and re-queue for full reingestion. Useful after upstream content changes or when ingestion errored. Requires databanks permission. |
| async `documents.waitUntilReady` | documentId, options? | Poll until the document reaches `status: 'ready'`. Throws on error/timeout/deletion. Default 60s timeout, 500ms poll interval — override via DatabankWaitUntilReadyOptions. Use after `create()` or `reprocess()` to await ingestion. Requires databanks permission. |

## api.memories.cortex

> **Concepts:** `api.memories.*` bridges Lumiverse's **hybrid memory architecture**, under one `memories` permission. Two halves: the **Memory Cortex** (entity/relation graph + narrative-arc consolidations + salience + vaults + chat interlinks + fused retrieval) and **Long-Term Chat Memory** (the vectorized chunk store behind the `{{memories}}` macro — see `api.memories.chatMemory`). **Distinct from `api.databanks`** — databanks are explicitly-uploaded reference documents; the cortex is what Lumiverse automatically *remembers* about a chat. The active userId is folded in implicitly, and **every chat-scoped call is ownership-checked** host-side (other users' chats return `null` on reads / throw on writes). `cortex.query(CortexQueryDTO)` is the headline call: it fuses semantic search + salience + recency + reinforcement + emotional + entity components into a ranked `CortexResult` (the same shape the host uses during prompt assembly) — use it for grounding/RAG over what the chat remembers. Results are server-cached ~5 min per chat+query shape; `getCached` / `getCachedLinked` read that warm cache without re-querying (return `null` when empty/expired); `invalidateCache` drops it. `queryLinked` resolves every attached vault + interlink target in parallel. `getConfig` / `putConfig` read + patch the (permissive, host-owned) cortex configuration.

| Method | Args | Description |
|---|---|---|
| async `getConfig` | — | Get the Memory Cortex configuration (MemoryCortexConfig — permissive; advanced/host-added fields pass through). Requires memories permission. |
| async `putConfig` | patch | Patch the Memory Cortex configuration (deep merge; unspecified fields untouched). Returns the updated config. Requires memories permission. |
| async `query` | query | Fused-score retrieval (semantic + salience + recency + reinforcement + emotional + entity). query: CortexQuery { chatId (required), queryText (required), entityFilter?, timeRange?, emotionalContext?, generationType?, topK?, includeConsolidations?, includeRelationships?, excludeMessageIds? }. Returns CortexResult { memories, entityContext, activeRelationships, arcContext, stats }. Server-cached ~5 min per chat + query shape. Requires memories permission. |
| async `queryLinked` | chatId, options? | Resolve every attached vault + interlink target in parallel. options: { queryText? } — pass queryText to rank by relevance. Returns LinkedCortexResult { vaults, interlinks }. Requires memories permission. |
| async `getCached` | chatId | Read the warm cortex cache without re-running retrieval. Returns CortexResult or null (no/expired cache). Requires memories permission. |
| async `getCachedLinked` | chatId | Read the cached linked-cortex result. Returns LinkedCortexResult or null. Requires memories permission. |
| async `invalidateCache` | chatId | Drop the warm cortex cache for a chat. Requires memories permission. |
| async `invalidateLinkedCache` | chatId | Drop the warm linked-cortex cache for a chat. Requires memories permission. |

## api.memories.entities

> **Concepts:** The cortex **entity graph** — characters, locations, items, factions, concepts, events. `list` defaults to active-only, ordered by salience; `findByName` matches canonical name OR known aliases. `upsert` is a **smart merge** against the canonical name + aliases (not a blind insert) — pass `MemoryEntityUpsert { name, type, aliases?, confidence?, role?, provisional? }`; entities below the configured confidence threshold are dropped host-side. Mutators: `updateStatus` (active / inactive / deceased / destroyed / unknown), `addFacts` (deduplicated, keeps the most recent 20), `updateEmotionalValence` (replaces the running valence map). Use to build entity dashboards, replay your own NER into the graph, or correct what Lumiverse extracted.

| Method | Args | Description |
|---|---|---|
| async `list` | chatId, options? | List entities for a chat. options: { activeOnly? (default true), limit? }. Ordered by salience. Returns MemoryEntity[]. Requires memories permission. |
| async `get` | entityId | Get an entity by id, or null if not found / not owned. Requires memories permission. |
| async `findByName` | chatId, name | Find an entity by canonical name OR known alias, or null. Requires memories permission. |
| async `upsert` | chatId, entity, options? | Smart-merge upsert against canonical name + aliases. entity: MemoryEntityUpsert { name, type ("character"\|"location"\|"item"\|"faction"\|"concept"\|"event"), aliases?, confidence?, role?, provisional? }. options: { chunkId?, createdAt? } attribute the mention. Returns the merged MemoryEntity. Requires memories permission. |
| async `updateStatus` | entityId, patch | Update status. patch: { status ('active'\|'inactive'\|'deceased'\|'destroyed'\|'unknown'), statusChangedAt? }. Returns MemoryEntity. Requires memories permission. |
| async `addFacts` | entityId, facts | Append facts (string[]); deduplicated, keeps the most recent 20. Returns MemoryEntity. Requires memories permission. |
| async `getFacts` | entityId | Read an entity's facts (string[]; tagged branch facts stripped). Requires memories permission. |
| async `updateEmotionalValence` | entityId, valence | Replace the running emotional-valence map (Record<string, number>, e.g. { betrayal: 0.6, grief: 0.4 }). Returns MemoryEntity. Requires memories permission. |

## api.memories.relations

> **Concepts:** The typed **relation graph** between entities. `list` is active edges only; `listAll` includes superseded/merged (diagnostics); `forEntity` / `forEntities` filter by incident entity. **`upsert` uses entity NAMES, not ids** (`MemoryRelationUpsert { source, target, type, label, sentiment }`) — the host resolves canonical ids server-side. **Gotcha: both endpoints must already exist in the entity graph** — call `entities.upsert` for source + target first, or the relation is silently dropped and `upsert` returns `null` (not the row). Always null-check the return.

| Method | Args | Description |
|---|---|---|
| async `list` | chatId | Active relation edges for a chat (excludes superseded / merged). Returns MemoryRelation[]. Requires memories permission. |
| async `listAll` | chatId | Every relation edge including superseded / merged — for diagnostics. Requires memories permission. |
| async `forEntity` | chatId, entityId | Active edges incident to one entity. Requires memories permission. |
| async `forEntities` | chatId, entityIds, options? | Active edges across a set of entity ids. options: { limit? }. Requires memories permission. |
| async `upsert` | chatId, relation, options? | Upsert a relation by entity NAMES (not ids — the host resolves them). relation: MemoryRelationUpsert { source, target, type (RelationType), label, sentiment }. BOTH endpoints must already exist in the graph (call entities.upsert first) — returns the row or null if silently dropped. options: { chunkId? } attributes evidence. Requires memories permission. |

## api.memories.consolidations

> **Concepts:** Narrative-arc **consolidations** — compressed summaries across a tier of chunks (tier 1 = scene, 2 = chapter, …). `list` (optionally tier-filtered, most-recent-first) and `latestArc` read them; `run` triggers a background pass. **`run` is extractive / heuristic only — it does NOT call a sidecar LLM** (sidecar-driven consolidation is host-owned and runs automatically during ingestion). `run` is fire-and-forget: it returns immediately and new arcs surface via `list()` once the background job finishes.

| Method | Args | Description |
|---|---|---|
| async `list` | chatId, options? | List narrative-arc consolidations (compressed summaries). options: { tier? } (1 = scene, 2 = chapter, …). Ordered most-recent first. Returns MemoryConsolidation[]. Requires memories permission. |
| async `latestArc` | chatId | The most recent arc across all tiers, or null. Requires memories permission. |
| async `run` | chatId | Trigger a background EXTRACTIVE consolidation pass (heuristic only — no sidecar LLM). Fire-and-forget: returns immediately; new arcs surface via list() once the job completes. Requires memories permission. |

## api.memories.salience

| Method | Args | Description |
|---|---|---|
| async `list` | chatId, options? | Per-chunk salience records, ordered by scoredAt desc. options: { limit? (max 500/page), offset? }. Returns MemorySalience[] { chunkId, score, scoreSource, emotionalTags, narrativeFlags, statusChanges, hasDialogue/Action/InternalThought, wordCount, scoredAt }. Requires memories permission. |

## api.memories.vaults

> **Concepts:** **Vaults** are frozen cortex snapshots — capture a chat's entities + relations + chunk content for reuse. `create({ chatId, name, description? })` copies entities + relations **synchronously** but copies the LanceDB chunks **in the background** (the vault is queryable in structural-only mode until that finishes); `reindex` re-runs the chunk copy (e.g. after an embedding-model swap). `get` returns the vault + entities + relations; `getChunks` the chunk snapshot. Attach a vault to other chats as read-only knowledge via `api.memories.links`.

| Method | Args | Description |
|---|---|---|
| async `list` | — | All vaults owned by the active user (Vault[]). Requires memories permission. |
| async `get` | vaultId | A vault with its entities + relations (VaultWithContents), or null if not found / not owned. Requires memories permission. |
| async `getChunks` | vaultId | The chunk snapshot copied into the vault at creation (VaultChunk[]). Requires memories permission. |
| async `create` | input | Snapshot a chat into a new vault. input: VaultCreate { chatId, name, description? }. Entities + relations copy synchronously; LanceDB chunks copy in the background (queryable structural-only until done). Returns Vault. Requires memories permission. |
| async `rename` | vaultId, name | Rename a vault. Returns true if renamed. Requires memories permission. |
| async `delete` | vaultId | Delete a vault + its chunks + attached links. Returns true if deleted. Requires memories permission. |
| async `reindex` | vaultId | Re-run the LanceDB chunk copy from the source chat (e.g. after an embedding-model swap). Returns VaultReindexResult { mode, chunkCount }. Requires memories permission. |

## api.memories.links

> **Concepts:** Chat **links** — two kinds via `attach(ChatLinkAttach)`: a **vault attach** (`linkType: 'vault'`, `vaultId`) gives a chat read-only access to a frozen snapshot, and an **interlink** (`linkType: 'interlink'`, `targetChatId`) makes two chats see each other's live entities/relations (pass `bidirectional: true` to also create the reverse edge on the target). `list` enumerates a chat's links; `toggle` enables/disables without removing; `remove` deletes. Linked data surfaces through `cortex.queryLinked`.

| Method | Args | Description |
|---|---|---|
| async `list` | chatId | All links attached to a chat — vault attaches + interlinks (ChatLink[]). Requires memories permission. |
| async `attach` | input | Attach a vault as read-only knowledge, or interlink two chats. input: ChatLinkAttach { chatId, linkType ('vault'\|'interlink'), vaultId? (for vault), targetChatId? (for interlink), label?, bidirectional? (interlinks — also creates the reverse link) }. Returns the created ChatLink(s). Requires memories permission. |
| async `remove` | chatId, linkId | Remove a link. Returns true if removed. Requires memories permission. |
| async `toggle` | chatId, linkId, enabled | Enable / disable a link without removing it. Returns true if toggled. Requires memories permission. |

## api.memories.chatMemory

> **Concepts:** Long-Term Chat Memory — the vectorized **chunk store behind the `{{memories}}` macro**. `get(chatId, { topK })` runs the same top-K hybrid (vector + BM25) retrieval the macro uses and returns `ChatMemoryResult { chunks, formatted, count, … }` — **equivalent to `api.chats.getMemories()`** but under the `memories` permission (use that lighter alias if `memories` is overkill). `listChunks` inspects the raw index; `warm` rebuilds stale chunks + queues pending vectorizations (no-op `status: 'skipped'` when chat vectorization is disabled — check the user's embedding config first); `invalidate` drops the cached retrieval. **Distinct from `cortex`** — chatMemory is flat semantic chunk retrieval; cortex is the structured graph + fused scoring.

| Method | Args | Description |
|---|---|---|
| async `listChunks` | chatId | All vectorized chunks for a chat, oldest first (ChatChunk[]). The raw index behind the {{memories}} macro. Requires memories permission. |
| async `get` | chatId, options? | Top-K hybrid (vector + BM25) retrieval. options: { topK? }. Returns ChatMemoryResult { chunks, formatted, count, enabled, queryPreview, settingsSource, chunksAvailable, chunksPending } — same payload as the {{memories}} macro (equivalent to api.chats.getMemories but under the memories permission). Requires memories permission. |
| async `warm` | chatId, options? | Rebuild stale chunks + queue pending vectorizations. options: { force? }. Returns ChatMemoryWarmupResult { status, reason?, rebuilt?, vectorizationsQueued? }. No-op (status:'skipped') when chat vectorization is disabled. Requires memories permission. |
| async `invalidate` | chatId | Drop the cached {{memories}} retrieval result for a chat. Requires memories permission. |

## api.memories.stats

| Method | Args | Description |
|---|---|---|
| async `usage` | chatId | Entity / relation / consolidation / salience counts (CortexUsageStats; host gc fields pass through). Requires memories permission. |
| async `ingestionStatus` | chatId | Live ingestion phase + pending job count (CortexIngestionStatus), or null when the chat was never ingested. Requires memories permission. |
| async `ingestionTelemetry` | chatId | Last sample + per-phase averages over recent ingestions (CortexIngestionTelemetry). Requires memories permission. |

## api.personas

| Method | Args | Description |
|---|---|---|
| async `list` | options? | List personas (paginated). [personas] |
| async `get` | personaId | Get a persona by ID. [personas] |
| async `getDefault` | — | Get the default persona (isDefault = true). [personas] |
| async `getActive` | — | Get the currently active persona. [personas] |
| async `create` | input | Create a persona. [personas] |
| async `update` | personaId, input | Update a persona. [personas] |
| async `delete` | personaId | Delete a persona. [personas] |
| async `switchActive` | personaId | Switch the active persona. Pass `personaId: string` to activate a persona, or `null` to deactivate. [personas] |
| async `getWorldBook` | personaId | Get the world book attached to a persona. [personas] |
| async `addons.list` | options? | List global add-ons — named, sortable injectable content blocks (persona-adjacent). Paginated { data, total }. Requires personas permission. [personas] |
| async `addons.get` | addonId | Get a global add-on by ID — resolves an add-on-state ID (from a generation's personaAddonStates) to its label + content. Returns null if not found. Requires personas permission. [personas] |
| async `addons.update` | addonId, input | Update a global add-on (partial: label / content / sortOrder / metadata). Authoring + removal stay in the host UI. Requires personas permission. [personas] |

## api.presets

> **Concepts:** Generation preset CRUD. A preset is the complete prompt configuration: sampler/provider `parameters`, ordered `prompt_order` (prompt blocks with roles / positions / depth), `prompts` (behavior + completion settings), and `metadata` (description, model profiles, prompt-variable values). **Three sub-namespaces**: `api.presets.*` (preset CRUD), `api.presets.blocks.*` (prompt-block CRUD within a preset — block ops update the parent's `prompt_order` and trigger the normal preset update flow), `api.presets.categories.*` (host-derived category grouping view). **Categories are NOT separate records** — a category is a structural prompt block with `marker === 'category'`, and its children are the following non-category blocks until the next category marker. `categoryMode` is `'radio'` (one enabled child) or `'checkbox'` (many). Use `categories.list()` for the precomputed grouping; create / update / delete category headers via `blocks.*` with `marker: 'category'`. **Snake_case fields** (`prompt_order`, `created_at`, `updated_at`) are preserved from Spindle DTOs since they identify stored data. **Use cases**: rotate prompt blocks based on chat context, toggle radio-category options on character state changes, snapshot presets to JSON for backup, build ephemeral per-chat presets and clean up via `ls:teardown`. Requires `presets` permission.

| Method | Args | Description |
|---|---|---|
| async `list` | options? | List user presets (paginated). Options: `{ limit?, offset? }`. Defaults: limit 50, max 200. Returns `{ data: Preset[], total }`. Requires presets permission. [presets] |
| async `get` | presetId | Get a preset by ID. Returns `null` if not found. Requires presets permission. [presets] |
| async `create` | input | Create a new preset. `input.name` and `input.provider` are required (`provider` is typically `'loom'` for native Lumiverse presets). All other fields optional with host defaults (`engine: 'classic'`, empty parameters / prompt_order / prompts / metadata). Requires presets permission. [presets] |
| async `update` | presetId, input | Update a preset. All fields optional. When `prompt_order` or `metadata` is updated, Lumiverse prunes stale `metadata.promptVariables` entries that no longer correspond to a variable definition on a block. Requires presets permission. [presets] |
| async `delete` | presetId | Delete a preset. Returns `true` if deleted. Requires presets permission. [presets] |
| async `blocks.list` | presetId | Return the preset's ordered prompt blocks (`PromptBlock[]`), including structural category-marker blocks. Requires presets permission. [presets] |
| async `blocks.get` | presetId, blockId | Get a block by ID. Returns `null` if not found. Requires presets permission. [presets] |
| async `blocks.create` | presetId, input, options? | Create a prompt block. `options.index` inserts at a specific zero-based position within the preset's `prompt_order`; omitted appends to the end. Block ops update the parent preset's `prompt_order` array and trigger the normal preset update flow. Requires presets permission. [presets] |
| async `blocks.update` | presetId, blockId, input | Update a block. All fields except `id` are optional. Requires presets permission. [presets] |
| async `blocks.delete` | presetId, blockId | Delete a block. Returns `true` if deleted. Requires presets permission. [presets] |
| async `categories.list` | presetId | Return host-derived category groupings (`PromptBlockCategoryGroup[]`) for the preset's ordered blocks. Categories aren't separate records — they're structural prompt blocks with `marker === 'category'`, and a group's children are the following non-category blocks until the next category marker. The first group may have `categoryBlock: null` if normal blocks appear before any category marker. To create / update / delete a category, use `blocks.*` with `marker: 'category'`. Requires presets permission. [presets] |

## api.regexScripts

| Method | Args | Description |
|---|---|---|
| async `list` | options? | List regex find/replace scripts (paginated). Options: scope, scopeId (required for character/chat scope), target ('prompt'\|'response'\|'display'), limit (max 200), offset. Returns { data: RegexScriptInfo[], total }. [regex_scripts] |
| async `get` | scriptId | Get a single regex script by id. Returns null if not found. [regex_scripts] |
| async `findByName` | name, scope? | Find the first regex script whose name exactly matches. Convenience over list() — pages through. O(scripts) worst case. [regex_scripts] |
| async `getActive` | options | Resolve enabled rules that would actually fire for the given target + character/chat context, merged across global + character + chat scopes and ordered by scope tier then sortOrder. Mirrors Lumiverse's internal resolution. Required: target. Optional: characterId, chatId. [regex_scripts] |
| async `create` | input | Create a new regex script. name and findRegex are required; everything else gets host-side defaults (placement: ['ai_output'], scope: 'global', target: 'response', flags: 'gi', etc.). [regex_scripts] |
| async `update` | scriptId, input | Update a regex script. All fields optional; only provided fields are touched. Throws if the script is not found. [regex_scripts] |
| async `delete` | scriptId | Delete a regex script. Returns true if the row was deleted. [regex_scripts] |

## api.images

> **Concepts:** Thin wrapper over Lumiverse's image store. Use cases: persist generated / fetched / pasted images and obtain an `imageId` that can be passed to `api.theme.extractColors` for palette derivation, stored on a character avatar, or attached to a databank document. **Two upload paths**: `upload({data: Uint8Array, ...})` for raw bytes (sourceable from `api.utils.http.*` with `responseType: 'arraybuffer'`, `api.utils.image.dataUrlToBytes(...).data`, `api.files.*`, etc.); `uploadFromDataUrl(dataUrl, options?)` for `data:image/...;base64,...` URLs. Both return `ImageInfo` whose `id` is the persisted UUID. **Distinct from `api.utils.image.*`** — those are CHILD-side byte-manipulation helpers (mime sniff, dataUrl ↔ bytes conversion); `api.images.*` is HOST-side persistence. Requires `images` permission.

| Method | Args | Description |
|---|---|---|
| async `upload` | input | Upload raw image bytes to Lumiverse's image store. `input.data` is a Uint8Array (source via api.utils.http.* with responseType:'arraybuffer', api.utils.image.dataUrlToBytes, api.files.*, etc.). Optional: filename, mimeType, ownerCharacterId, ownerChatId. Returns the ImageInfo whose `id` can be passed to api.theme.extractColors or stored on a character avatar. Requires images permission. [images] |
| async `uploadFromDataUrl` | dataUrl, options? | Convenience: upload from a `data:image/...;base64,...` data URL. Optional options: originalFilename, ownerCharacterId, ownerChatId. Returns ImageInfo. Requires images permission. [images] |
| async `get` | imageId | Look up an image by id. Returns ImageInfo or null. Requires images permission. [images] |
| async `delete` | imageId | Delete an image by id. Returns `true` if a row was removed. Requires images permission. [images] |

## api.imageGen

> **Concepts:** Image-generation surface. `generate({prompt, ...})` fires against the user's configured connection profiles (the same profiles the Lumiverse UI uses for image generation) and returns `ImageGenResult { imageDataUrl, model, provider, imageId?, imageUrl? }`. **`imageId` is the integration seam** — pass to `api.images.get`, `api.theme.extractColors`, or `spindle.characters.setAvatar` to compose with the rest of the API. `imageUrl` is an auth-free public URL suitable for `api.ui.pushNotification(title, body, { image: result.imageUrl })` (positional signature, NOT object-form). **Provider/connection metadata** via `getProviders` (capability schemas — drive parameter UIs), `listConnections` / `getConnection` (connection picker UIs; API keys masked), `getModels` (model picker; dynamic providers fetch live from upstream). **Provider-specific parameters** flow opaquely through `input.parameters` — validate against the provider's `parameters` schema from `getProviders()` if your script accepts user input. **img2img / inpainting** via the `image_array` parameter type: pass arrays of `imageId` strings (`parameters: { input_images: [id1, id2] }`). **Distinct from `api.images.*`** — that one is raw-byte CRUD on already-stored images; this one creates new ones. Requires `image_gen` permission.

| Method | Args | Description |
|---|---|---|
| async `generate` | input | Generate an image. `input.prompt` required; optional: connectionId (default: user's default connection), negativePrompt, model, parameters (provider-specific — validate against the provider's `parameters` schema from getProviders() if your script accepts user input), ownerCharacterId, ownerChatId. Returns ImageGenResult { imageDataUrl, model, provider, imageId?, imageUrl? } — `imageId` is the canonical handle accepted by api.images.get / api.theme.extractColors / characters.setAvatar; `imageUrl` is an auth-free public URL suitable for api.ui.pushNotification(title, body, { image: result.imageUrl }) — pushNotification is positional, NOT object-form. For img2img / inpainting, pass `parameters: { input_images: [imageId, ...] }`. Requires image_gen permission. [image_gen] |
| async `getProviders` | — | List all image-generation providers available on this Lumiverse install along with their capability schemas. Each provider's `capabilities.parameters` describes the supported `parameters` for generate() calls against that provider's connections — use to drive dynamic parameter UIs. Requires image_gen permission. [image_gen] |
| async `listConnections` | — | List the user's image-gen connection profiles. API keys are never exposed — only `hasApiKey: boolean`. Use to populate a connection picker UI. Requires image_gen permission. [image_gen] |
| async `getConnection` | connectionId | Get a single image-gen connection profile by id. Returns ImageGenConnectionInfo or null. Requires image_gen permission. [image_gen] |
| async `getModels` | connectionId | List the models available on a connection profile. For dynamic-list providers, this fetches live from the upstream API (network round-trip). Static-list providers return their capabilities.staticModels directly. Returns Array<{id, label}>. Requires image_gen permission. [image_gen] |

## api.oauth

> **Concepts:** OAuth callback surface — the **only inbound-HTTP hook** Spindle exposes to extensions. Three primitives: `onCallback(handler)` registers a handler for this extension's OAuth redirect URL, `getCallbackUrl()` returns the URL path to use as `redirect_uri`, `createState()` mints a CSRF state nonce. **Single handler per extension** (host stores in a module-scope ref; last-wins). LumiScript adds a `spindle.log.warn` on cross-script or same-script-re-register collisions — non-terminating; the host's behavior is preserved, only the silent overwrite is surfaced. **Wrapper is intentionally thin** — everything beyond these primitives (constructing the authorize URL, exchanging the code for a token, persisting + refreshing the token) is the script's responsibility. Pair with `api.utils.http` (`cors_proxy` + `allowDangerous`) for token-endpoint POSTs and `api.enclave` for encrypted token persistence. PKCE cookbook recipe deferred to v1.0 docs pass. Requires `oauth` permission.

**Surfacing the authorize URL.** Scripts run server-side in the Bun subprocess — there is NO `window.open` and no programmatic browser-tab control. To prompt the user to visit the authorize URL, use one of: (a) `api.ui.showAdvancedModal({title:'Authorize', items:[{kind:'html', html:'<a href="..." target="_blank">Click to authorize</a>'}]})` (requires `app_manipulation`); (b) `api.ui.toast('Open this URL: '+authorizeUrl, 'info')` for a passive notice; (c) `api.ui.pushNotification('Authorize required', authorizeUrl, { url: authorizeUrl })` for an OS notification (positional: title, body, options; the `url` option opens-on-click) (requires `push_notification`); (d) inject a button into the host shell via `api.ui.dom.inject` (requires `app_manipulation`).

**Composing the full `redirect_uri`.** `getCallbackUrl()` returns a host-relative path (e.g. `/api/spindle-oauth/lumiscript/callback`); the OAuth provider needs the absolute URL. Scripts can't introspect the Lumiverse origin at runtime — pass it as a config constant in the script source, or store via `api.variables.global` from a one-time setup script.

| Method | Args | Description |
|---|---|---|
| `onCallback` | handler | Register a callback handler for this extension's OAuth redirect URL. Handler receives the URL query params as Record<string, string>; optional return { html } becomes the response body shown in the user's browser tab. **Single handler per extension** (host stores in a module-scope ref; last-wins). LumiScript emits a `spindle.log.warn` on cross-script or same-script-re-register collisions — non-terminating; the host's last-wins behavior is preserved. Returns a sync unsubscribe fn. Requires oauth permission. [oauth] |
| async `getCallbackUrl` | — | Get the host-relative callback URL path (e.g. `/api/spindle-oauth/lumiscript/callback`). Stable per-extension; use as the `redirect_uri` in your authorize URL construction. Async on the LumiScript side due to IPC boundary even though the host method is sync. Requires oauth permission. [oauth] |
| async `createState` | — | Mint a CSRF state nonce. Pass to your authorize URL as `state=...`; the host verifies the returned state at callback time and rejects mismatches before invoking your handler. Requires oauth permission. [oauth] |

## api.theme

> **Concepts:** Lumiverse theme manipulation surface. Three usage tiers, increasing in flexibility: **simple** — `applyPalette({accent: {h, s, l}})` and let Lumiverse generate the full coherent ~80+ CSS variable set; **mode-aware** — `apply({variablesByMode: {dark: {...}, light: {...}}})` and the host dispatches per-mode at apply time; **expert** — `generateVariables(config)` → tweak → `apply({variables: ...})` for full programmatic control. **Per-script attribution**: multiple LumiScript scripts can apply themes concurrently — LumiScript maintains a per-script override registry and merges before pushing to spindle. Conflict resolution: per-key last-applied-wins for variables, most-recent-script-wins for palette. Auto-cleared on script disable / delete (no manual `clear()` needed for normal disable flows). **Cookbook pattern for interactive UI scripts**: scripts that combine theme apply with interactive DOM should clear their theme in the close / dismiss handler symmetric to DOM removal — `clear()` drops just this script's contributions, other scripts' themes survive. `extractColors(imageId)` pairs cleanly with `applyPalette({accent: result.dominantHsl})` for image-driven theming (avatar-themed UI, dynamic mood theming, etc.).

**The `--lumiverse-*` design tokens** — target these from `apply({variables: {...}})` or `api.ui.dom.addStyle` CSS so your UI matches the host theme. Values shown are the dark-theme defaults; the user's active theme may override them, so prefer the token over a hard-coded colour. Grouped:
• **Accent** — `--lumiverse-primary` (rgba(147,112,219,.9)) + `-hover` / `-light` / `-muted` / `-text` (brighter, for accent TEXT on dark) / `-010` `-015` `-020` `-050` (opacity steps). The brand purple.
• **Text** (decreasing emphasis) — `--lumiverse-text` (white .9) / `-text-muted` (.65) / `-text-dim` (.4) / `-text-hint` (.3).
• **Backgrounds** (opaque surfaces) — `--lumiverse-bg` (dark purple-tinted) / `-bg-elevated` (+ `-bg-elevated-040`) / `-bg-hover` / `-bg-dark` / `-bg-darker` / `-bg-deep` / `-bg-040` `-050` `-070` (opacity steps); `--lumiverse-scene-text-scrim` (rgba(7,9,16,.46)) for legible text over scene / character art.
• **Fills** (translucent BLACK scrims that DARKEN what is under them — not surface colours) — `--lumiverse-fill-subtle` → `-fill` → `-fill-hover` / `-medium` / `-strong` / `-heavy` / `-deepest`.
• **Borders** — `--lumiverse-border` (purple-tinted .12) / `-border-hover` / `-border-light` / `-border-neutral` / `-border-neutral-hover` (neutral grey).
• **Status** — `--lumiverse-danger` (#ef4444) / `-success` (#22c55e) / `-warning` (#f59e0b), each plus `-015` `-020` `-050` tints; `-danger-hover`; `-error` aliases danger.
• **Radii** — `--lumiverse-radius-sm` (5px) / `-radius` (8) / `-radius-md` (10) / `-lg` (12) / `-xl` (16).
• **Shadows & sheen** — `--lumiverse-shadow` + `-sm` `-md` `-lg` `-xl`; `--lumiverse-highlight-inset` (+ `-md` `-lg`) for a top-edge highlight.
• **Transitions** — `--lumiverse-transition` (200ms ease) / `-transition-fast` (150ms ease).
• **Typography** — `--lumiverse-font-family` (system sans) / `-font-mono` / `-font-scale` / `-ui-scale`.
• **Icons** — `--lumiverse-icon` / `-icon-muted` / `-icon-dim`.
• **Cards & modals** — `--lumiverse-card-bg` (gradient) + `-card-bg-solid` `-top` `-bottom`; `--lumiverse-modal-backdrop`; `-gradient-modal`; `-swatch-border`.
• Chat-Sheld glass tokens (`--lcs-glass-bg` / `-glass-border` / `-glass-blur` / `--lcs-radius`, etc.) exist too but are host-internal — read them live via `getCurrent()` rather than hard-coding.

**`applyPalette` vs `apply`** — `applyPalette({accent})` is the high-level path (regenerates the whole coherent ~80-variable set from one accent colour); `apply({variables: {'--lumiverse-primary': '...'}})` is surgical (override specific tokens, leave the rest untouched). Don't blend the two mental models in a single call. **Fallbacks** — these tokens live on `:root`, so they cascade into script-injected DOM; if a token ever reads as unset (e.g. a `document.body` portal context), write it with an rgb fallback: `var(--lumiverse-text, rgba(255,255,255,.9))`. Requires `app_manipulation` permission.

| Method | Args | Description |
|---|---|---|
| async `apply` | overrides | Apply CSS variable overrides on top of the user's current theme. `overrides.variables` is a flat map applied regardless of mode; `overrides.variablesByMode.{dark,light}` is mode-selected at apply time by the host. LumiScript maintains per-script attribution — multiple scripts' apply calls merge with per-key last-applied-wins semantics. Requires app_manipulation permission. [app_manipulation] |
| async `applyPalette` | palette | Apply a palette-driven theme. Pass `palette: ThemePaletteConfig` where `palette.accent` is `{h, s, l}` and Lumiverse generates the full variable set coherently, preserving the user's glass/radius/font/UI-scale. Pass `null` to drop this script's palette contribution. Across LumiScript scripts: most-recent-script-wins. Requires app_manipulation permission. [app_manipulation] |
| async `clear` | — | Drop this script's contributions from the per-script override registry, re-merge, push the post-clear result to spindle.theme.{apply,applyPalette}. Auto-called on script disable / delete. Requires app_manipulation permission. [app_manipulation] |
| async `getCurrent` | — | Get a read-only snapshot of the user's current theme configuration (NOT including any extension overrides). Returns ThemeInfo with id, name, mode ('light' \| 'dark'), accent (HSL), enableGlass, radiusScale, fontScale, uiScale, characterAware. Requires app_manipulation permission. [app_manipulation] |
| async `extractColors` | imageId | Extract a color palette from an image stored in Lumiverse's image system. `imageId` is a host-side UUID (sources: `character.imageId`, `api.images.upload(...).id`). Returns ColorExtractionInfo with dominant + per-region RGB + flatness scores + isLight + dominantHsl (ready to pass to applyPalette). Throws if the id is unknown. Requires app_manipulation permission. [app_manipulation] |
| async `generateVariables` | config | Generate the full set of Lumiverse CSS variables from a theme config without applying them. Pass the result to apply({variables}) for a complete coherent override (or tweak individual keys before applying). config.accent + config.mode required; glass/radius/font/UI-scale/baseColors/statusColors optional. Requires app_manipulation permission. [app_manipulation] |

## api.council

| Method | Args | Description |
|---|---|---|
| async `getSettings` | — | Get the user's full Council settings: mode flag, members[], tool-execution settings (timeout, sidecar context window, etc.). Returns CouncilSettings verbatim. No permission required. |
| async `getMembers` | — | Get the user's currently-assigned Council members with full Lumia context (role + chance from the assignment, plus avatar / definition / personality / behavior from the source Lumia item). Returns CouncilMemberContext[]. Inside a tool handler, prefer the ctx.councilMember arg passed automatically — this method is for inspecting Council state OUTSIDE a tool execution cycle. |
| async `getAvailableLumiaItems` | — | Get all Lumia items available across the user's installed packs. Superset of getMembers() — includes items not currently assigned. Returns LumiItem[] (camelCase mapping of the upstream snake_case DTO). |

## api.tools

> **Concepts:** Tool-registration surface. All tools you register via `api.tools.register` are **extension tools** — they bypass any sidecar LLM and are invoked directly by the host with `(args, ctx?)`. On a Council invocation `args` is `{context, __deadlineMs}` only — your registration-schema parameters are NOT filled (no sidecar LLM runs to populate an extension tool's schema), so read `args.context` and decide the values yourself — and `ctx` carries `{councilMember?, contextMessages?, requestId?}`. The "two execution paths" framing applies to **Lumiverse built-in tools** (which DO go through a sidecar LLM with description-as-prompt — that's how the Council reasons about which to invoke); LumiScript scripts can't register sidecar-LLM-driven tools. Practical implication: do your own analysis inside the handler — `generateStructured` against a fast connection is the canonical pattern. One-line tool descriptions are sufficient for extension tools — the description doesn't prompt anything; it's purely a human label shown in the tool inspector. Set `council_eligible: true` on the definition to make the tool selectable for Council `chance` rolls (the tool then receives Council context on invocation); leave it false for tools only invoked via `api.tools.invoke` or `api.llm.generateWithTools`. To **observe** invocations of your registered tools (Council or direct) with their result + timing, subscribe to the `ls:tool:invoked` broadcast via `api.broadcast.on` (fires after each handler returns; payload `{ name, args, result, scriptId, callMs, councilMember? }`) — the host delivers only invocations of YOUR tools, so built-in / DLC tools (pure LLM prompts) and tools owned by other extensions are not visible.

| Method | Args | Description |
|---|---|---|
| `register` | name, def, handler | Register an LLM tool. Handler receives (args, api, ctx?) and must return a string. ctx is populated when the host invokes the tool (Council / direct LLM) — read ctx.councilMember to personalise output per Council member, ctx.requestId to correlate with host-side logging. [tools] |
| `unregister` | name | Unregister a tool registered by this script. No-op if not found. [tools] |
| `list` | — | List all currently registered tools across all scripts. [tools] |
| async `invoke` | name, args? | Invoke a registered tool handler directly (for use inside an agentic loop). [tools] |

## api.macros

> **Concepts:** Two registration modes. **Pull mode** (`register(name, handler)`): handler runs at macro-resolution time, can be sync or async (function-reference form; string-handler form is sync-only). **Push mode** (`register(name)` + `updateMacroValue(name, value)`): register once with no handler, push values whenever they change — avoids RPC latency at generation time. Pull is simpler but pays per-resolve cost; push is faster but requires upstream "value changed" knowledge. Pick based on whether macro resolution is hot.

| Method | Args | Description |
|---|---|---|
| `register` | name, def, handler? | Register a Lumiverse macro. Omit handler for push-mode (value set via updateValue); provide handler for pull-mode (computed at resolution). |
| `updateValue` | name, value | Push a new value for a push-mode macro. Throws if the macro was registered with a handler. |
| `unregister` | name | Unregister a macro owned by this script. No-op if not found or not owned. |
| `list` | — | List all currently registered macros across all scripts. |
| `registerInterceptor` | handler, options? | Register a handler that receives the RAW template before Lumiverse parses it; return a transformed template or void to pass through. Use for iteration-heavy templates ({{#each LARGE_LIST}}…{{my_macro}}…{{/each}}) where per-macro RPC cost dominates. Options: id, priority (default 100), phase filter (prompt/display/response/other), matchTemplate (string \| string[] \| RegExp), timeoutMs (default 2000). Returns handle { id, remove }. Requires macro_interceptor permission. [macro_interceptor] |
| `listInterceptors` | — | List all currently registered macro interceptors across all scripts. |

## api.broadcast

> **Concepts:** In-memory real-time pub/sub between scripts. Events are NOT persisted — handlers fire synchronously when an event is emitted, and there's no replay across script reloads. Subscriptions persist between trigger runs, so a "subscriber-only" script can watch events from a script it isn't co-triggered with. The owning script's next run clears its subscriptions at the START (then the body re-registers them) — so a re-firing script never stacks duplicate listeners, while a script that fires once (e.g. on `ls:startup`) keeps its subscriptions until it's disabled or deleted. The `ls:*` prefix is reserved for system events; scripts should namespace their own events with a project-specific prefix. **Distinct from `api.events`** — that one is for persistent event tracking; this one is for real-time messaging.

**Payload size cap + emit rate limit.** `api.broadcast.emit(event, payload)` synchronously throws if the JSON-serialised payload exceeds **1 MB** (matches the `api.scriptStorage` per-value ceiling), OR if the calling script has emitted more than **100 events/sec sustained** (token bucket with **1000-emit burst capacity**). Errors carry clear migration hints. The caps apply at the `api.broadcast.emit` proxy entry (NOT at the underlying bus, so internal `ls:*` events the engine emits are unaffected). For high-frequency data flow, push the data to `api.db.*` or `api.scriptStorage` and emit a small "data updated" notification on the bus instead.

| Method | Args | Description |
|---|---|---|
| `emit` | event, payload? | Fire a named event to all subscribed handlers across all scripts. |
| `on` | event, handler | Subscribe to a named event. Returns an unsubscribe function. |

## api.rpc

> **Concepts:** Cross-extension shared RPC pool. Wraps Spindle's `spindle.rpcPool` with two-tier namespacing: every endpoint is fully-qualified as `lumiscript.<scriptSlug>.<channel>` where `scriptSlug` auto-derives from the calling script's name (overridable via `options.as`). Use `sync(channel, value)` to publish a latest-value snapshot and `handle(channel, fn)` to register on-demand handlers — other LumiScript scripts AND other Lumiverse extensions can `read(endpoint)` from these channels. Free tier (no permission). Endpoints auto-unregister on script disable / delete / stale-after-re-run. **Permission delegation**: `options.policy` controls how owner permissions flow to readers. Omit for the legacy "requester inherits every owner permission" guard; pass `{ requires: [] }` for intentionally narrow / public endpoints; pass `{ requires: ['name'] }` to scope delegated permissions explicitly. Handlers receive `effectivePermissions` on the `RpcRequestContext` so they can branch on what's actually delegated to this call. **Distinct from `api.broadcast`** — broadcast is in-process pub/sub between LumiScript user-scripts; rpc is cross-extension, asks-the-pool RPC where the caller knows the target endpoint by name. Backend-console logs registrations for cross-extension exposure visibility.

| Method | Args | Description |
|---|---|---|
| async `sync` | channel, value, options? | Publish the latest value on a channel for cross-extension consumption. Endpoints are auto-namespaced as `lumiscript.<scriptSlug>.<channel>` — `scriptSlug` auto-derives from the calling script's name, overridable via `options.as`. `options.policy` controls cross-extension permission delegation: omit for legacy "requester must hold every gated permission the owner has" guard, `{ requires: [] }` for public/narrow endpoints, `{ requires: ['name'] }` to scope delegated permissions explicitly. Returns the fully-qualified endpoint string. Free tier. Endpoints auto-unregister on script disable / delete / stale-after-re-run. |
| async `handle` | channel, handler, options? | Register an on-demand handler for a channel. Handler receives `RpcRequestContext { endpoint, requesterExtensionId, effectivePermissions }` and returns the response value (sync or async). `effectivePermissions` lists the gated permissions available to THIS delegated call per the endpoint's `options.policy`. Same `lumiscript.<scriptSlug>.<channel>` namespacing + `options.policy` semantics as `sync`. Returns the fully-qualified endpoint string. Free tier. |
| async `read` | endpoint | Read a value from another extension's published endpoint. Pass the full `<extensionId>.<channel>` path. Throws on missing endpoint. For cross-extension data sharing — use `api.broadcast` for in-extension pub/sub instead. |
| async `unregister` | channel, options? | Remove a channel previously published by the calling script via `sync` or `handle`. Idempotent — no-op if the channel isn't registered. Pass the same `options.as` you used at registration time if any. |

## api.commands

| Method | Args | Description |
|---|---|---|
| `register` | commands[] | Register (or replace) command palette entries. Max 20 per extension. |
| `unregister` | commandIds? | Remove specific commands by ID, or all if no IDs given. |
| `onInvoked` | handler | Register a handler for when the user selects a command. Returns unsubscribe fn. |

## api.events

> **Concepts:** Persistent event tracking + replay. Events are durably stored and queryable across script reloads / extension restarts. Use cases: audit logs, state-resuming scripts (`getLatestState` for keys), custom analytics. **Distinct from `api.broadcast`** — that one is in-memory real-time pub/sub; this one is durable storage. Recording requires `event_tracking` permission.

| Method | Args | Description |
|---|---|---|
| async `track` | eventName, payload?, options? | Record a named event. Options: level, chatId, retentionDays. [event_tracking] |
| async `query` | filter? | Query events (newest-first). Filter by name, chat, date range, level, limit. [event_tracking] |
| async `replay` | filter? | Replay events (oldest-first). Same filter options as query. [event_tracking] |
| async `getLatestState` | keys[] | Retrieve latest known state for a set of keys. Useful for resuming after restarts. [event_tracking] |

## api.enclave

| Method | Args | Description |
|---|---|---|
| async `put` | key, value | Store or overwrite an AES-256-GCM encrypted secret. Requires allowDangerous. Key: alphanumeric + _ - . (max 128 chars); value: printable ASCII, max 64 KB. |
| async `get` | key | Retrieve a decrypted secret, or null if not found. Requires allowDangerous. |
| async `delete` | key | Delete a secret. Returns true if it existed. Requires allowDangerous. |
| async `has` | key | Check if a secret exists without decrypting it. Requires allowDangerous. |
| async `list` | — | List all secret keys for this user and extension. Requires allowDangerous. |

## api.tokens

| Method | Args | Description |
|---|---|---|
| async `countText` | text, options? | Server-side token count for an arbitrary string. Uses the provider's actual tokenizer (falls back to char/4 heuristic with `approximate: true`). Options: { model?, modelSource? } — `model` overrides `modelSource`. Returns { totalTokens, model, modelSource, tokenizerId, tokenizerName, approximate }. Free-tier. |
| async `countMessages` | messages, options? | Same as countText but for an array of LLMMessage-shaped items. Accepts the output of api.chat.getMessages directly (only role + content are used). Free-tier. |
| async `countChat` | chatId, options? | Count tokens for a live stored chat by ID. Convenient when you want to size a whole chat without fetching messages yourself. Free-tier. |

## api.db

> **Concepts:** Per-script schema-validated JSON collections. Each collection is a typed array of records persisted under the owning script's storage path; collections never leak across scripts. Optional Zod schema validates writes (insert + update). Built-in fields `id` / `createdAt` / `updatedAt` are reserved and auto-managed; Zod's `.strict()` / unknown-key stripping preserves them. Use for structured per-script data; for cross-script shared state see `api.variables.global`.

| Method | Args | Description |
|---|---|---|
| async `collection` | name, opts? | Open or create a collection. opts.scope = 'script' (default, per-scriptId) / 'character' (per-active-character) / 'chat' (per-active-chat). opts.schema (0.20.0+) attaches a ZodLike validator applied on every write. Path is baked into the handle at creation — throws if scope requires context (e.g. 'chat') that isn't present. Collection name: 1-64 chars, alphanumeric + _ - ., leading char must be alphanumeric. |
| async `list` | scope? | List collection names visible to this script in the given scope (default `script`). Owner-scoped — cross-script visibility is not supported. |
| async `exists` | name, scope? | Cheap existence check — true if the collection file exists, false otherwise. Does NOT load or parse. Ownership-safe: scope paths bake in the calling script id, so exists only sees this script's own collections. (0.20.0+) |
| async `drop` | name, scope? | Delete a collection entirely. No-op if the collection does not exist. Fires `ls:collection:dropped` with deletedCount. |
| `collection.insert` | record | Insert a record. Auto-assigns id (UUID v4), createdAt, updatedAt unless caller supplies them. Returns the persisted record. With schema: validates AFTER injection; reserved fields (id/createdAt/updatedAt) are preserved even when the schema strips unknown keys. |
| `collection.insertMany` | records | Batch-insert N records with a single file-write. All records share one timestamp (batch-commit semantic). Atomicity: validation + size guard run on the final array before persist — if any record fails, NOTHING lands. Fires one `ls:collection:inserted` per record in insertion order after the persist resolves. Empty array is a fast no-op. (0.20.0+) |
| `collection.find` | filter? | Find matching records. Filter: undefined = all, Partial<T> = literal match with dot-notation paths, (r) => boolean = caller predicate, or operator envelope { $gt, $in, $regex, ... } per-value (0.20.0+). Direct RegExp shorthand also works: { name: /alice/i }. |
| `collection.findOne` | filter | First matching record or null. |
| `collection.update` | filter, patch | Update all matching records. Returns count. Silently strips id/createdAt/updatedAt from patch — updatedAt is bumped to Date.now() on every match. With schema: validates the MERGED record against the full schema (not the patch alone); atomic (no records persist if any validation fails). |
| `collection.delete` | filter | Delete all matching records. Returns count. |
| `collection.count` | filter? | Count matching records (or all if filter omitted). |
| `collection.clear` | — | Remove all records, leaving an empty collection file. |
| `collection.query` | jsonQuery | Run a jsonquery string against the full collection. Escape hatch for aggregations / sorts / complex projections. Example: 'filter(.margin > 0) \| size()'. Throws SyntaxError on malformed queries. |

## api.scriptStorage

> **Concepts:** Per-script in-memory key/value store for session state. Closes the "where does my script keep its session state?" UX gap that was previously covered by the `globalThis.__lumiscript_script_<id>_*` convention (verbose, easy to forget the prefix). **Free tier — no permission required.**

**Picking the right storage primitive** — the LumiScript storage story now has three tiers, picked by intent:
  - `api.scriptStorage` — in-memory, session-scoped, free-tier. Best for "remember this for the session" flags (tracker rerun-inflight, current selection, transient cache).
  - `api.variables.*` — disk-persisted, scope-tiered (local/global/character/chat), free-tier. Best for "remember this across restarts" state.
  - `api.db.*` — disk-persisted, structured collections with schema + filters + queries. Best for record-shaped data you want to search / aggregate.

**Lifecycle**: in-memory only — values live in a parent-side `Map<scriptId, Map<key, value>>`, no disk write. Survives worker eviction / respawn (parent-side state, not in worker memory). Survives script edit / hot-reload (matches the `globalThis` convention — preserves dev iteration state). Cleared on script disable / delete via the `teardownDisabledScript` path. Lost on full backend restart.

**Size cap**: 1 MB per script on the JSON-serialised size of the full map. `set()` throws `"capacity exceeded"` cleanly when a write would cross the cap, with a migration hint pointing to `api.variables.*` / `api.db.*`. The cap is intentional — scriptStorage is a "small bag of session flags" surface, not bulk storage.

**Broadcasts**: every mutation fires an `ls:scriptStorage:*` event on the broadcast bus. `ls:scriptStorage:set` carries `{ scriptId, key, value }`; `ls:scriptStorage:delete` carries `{ scriptId, key }`; `ls:scriptStorage:clear` carries `{ scriptId }`. No-op `delete` / empty `clear` calls don't fire. The `ls:*` prefix avoids the eviction-pinning policy. Useful for debug / admin tooling; user scripts typically don't need to subscribe.

**Values must be JSON-serialisable.** Passing functions / symbols / DOM elements throws at the IPC boundary — same posture as `api.broadcast.emit` and `api.variables.*`.

**Cross-script isolation**: per-script via `scriptId`-keyed outer Map. Script A's writes never appear in Script B's reads. (Cross-script visibility for debug tooling is available via the broadcast events above.)

| Method | Args | Description |
|---|---|---|
| async `get` | key, defaultValue? | Read a value. Returns `defaultValue` (or `undefined` if not provided) when the key is missing. Generic type hint via `get<T>(...)` for IDE completion — the runtime doesn't enforce T. |
| async `set` | key, value | Write a value. Overwrites any prior value at the key. Fires `ls:scriptStorage:set` with `{ scriptId, key, value }`. Throws "capacity exceeded" if the JSON-serialised total would cross the 1 MB per-script cap (use `api.variables.*` or `api.db.*` for storage at this scale). Value must be JSON-serialisable. |
| async `delete` | key | Remove a key. Returns `true` if it existed (and fires `ls:scriptStorage:delete` with `{ scriptId, key }`), `false` if it didn't (no broadcast). |
| async `has` | key | Check whether a key exists. Returns true for keys with any value including 0 / false / null / "". |
| async `clear` | — | Remove every entry for this script. Fires `ls:scriptStorage:clear` with `{ scriptId }` if at least one entry existed; no broadcast for an already-empty storage. |
| async `keys` | — | List the current keys. Order is insertion-order (Map semantics). |

## script

| Method | Args | Description |
|---|---|---|
| `id` | (property) | This script's stable UUID. Immutable across enables, edits, renames. Use as owner key for any external state the script creates (world-book entries via automation_id, persistent storage paths, etc.). |
| `name` | (property) | This script's current human-readable name. Tracks the Script Manager — can change when the user renames. Useful for log lines; NOT stable for ownership (use script.id for that). |
| `type` | (property) | Script type: 'trigger' or 'library'. |
| async `require` | nameOrId | Load a library by name/ID, or a built-in library by ls: prefix (e.g. 'ls:components'). |

## Events

Lumiverse + LumiScript lifecycle events. Scripts react to these by being **wired in the editor UI** — open the script in the script editor and pick events in the event-wiring control. There is no script-source syntax for subscription; the `// @triggers` comment some scripts carry is informative-only and not parsed by the host (see the **Trigger model** section above). At handler time, the event name appears in `data.__event`; payload fields are listed below. The **Fires** column flags non-obvious firing semantics — read it before writing role-based or count-based filtering logic.

| Event | Payload | Fires |
|---|---|---|
| `ls:startup` | { __event: "ls:startup" } | Per-script when the script enters the active state: at LumiScript boot (extension enable / app start) AND after the user toggles the script from disabled→enabled. Symmetric partner to `ls:teardown`. Use for tool registration, cache pre-warm, broadcast subscription setup, and other init that should run whenever the script becomes runnable. On re-enable the case body re-runs in full — bottom-of-body `api.broadcast.on(...)` calls also re-execute, re-registering the subscriptions disable's cleanup wiped, so the case body itself can be empty if all you need is the body firing. **Use the `if (data.__event === "ls:startup")` branch for code that should run EXCLUSIVELY on cold boot — not on every Reload click.** Common cold-boot-only patterns: seeding a default config into `api.scriptStorage`, logging a boot timestamp, one-shot migrations on persistent data. To make init code ALSO re-run on hot reload, branch on `(data.__event === "ls:startup" \|\| data.__event === "ls:reload")`, or put it at the top of the body un-gated (the body re-runs on every fire including `ls:reload`). See `ls:reload` for the hot-reload counterpart. |
| `ls:teardown` | { reason: 'disabled' \| 'deleted', scriptId, scriptName } | Per-script when the script is disabled or deleted. Use for cleanup of external state (dynamic world-book entries, persistent storage migrations, etc.). **NOT fired on hot reload — `ls:reload` performs the per-script state wipe silently. ls:teardown is exclusive to disable / delete.** |
| `ls:reload` | { reason: 'autosave' \| 'manual', previousCodeHash, currentCodeHash, previousLength, currentLength, triggeredAt } | After a code edit IF the script opts in via the `// @ls:reload-on-edit` directive (~500ms debounce). Body re-runs in its existing worker so registered handlers refresh their closures. Also fires on click of the editor topbar Reload button (manual — bypasses the directive check). Branch on `data.__event === "ls:reload"` to detect. **Pairs with `ls:startup` as the lifecycle distinction**: ls:startup = cold boot (extension boot OR disable→enable transition), ls:reload = hot reload (autosave-with-directive OR manual Reload button click). **Reload performs a full per-script state wipe BEFORE the body re-runs** (DOM injections, modals, float widgets, drawer tabs, input-bar actions, handler closures, broadcast subs, tools, macros, interceptors, injections) while preserving `api.scriptStorage`, `api.theme.*` contributions, and the worker's `script.require()` cache. So the body re-running from `ls:reload` lands into a clean slate identical to a fresh `ls:startup` fire — the only difference is `data.__event` + the presence of `data.reason` (`"autosave" \| "manual"`). v1.0.0-rc.8+. **NOT in the trigger picker UI**: ls:reload is a deterministic side-effect of the Reload button OR the `@ls:reload-on-edit` directive, not a picker-checkbox subscription (parallel to the Run button, which also has no checkbox). |
| `MESSAGE_SENT` | { chatId, message: Message } | Fires once per **message creation** — the host emits it from the generic message-insert path, so it fires for the **user's send AND the assistant's empty streaming placeholder** created immediately after (`is_user: false`, empty `content`), plus auto-greetings and manually-added messages. **To react to the user's send only, guard with `if (!data.message.is_user) return;`** — `data.message.is_user` (boolean) is the discriminator; `GENERATION_ENDED` is the clean hook for *generated* assistant text. For a user send it fires **before prompt assembly** for the generation that follows — `api.chat.inject(...)` calls from the handler ARE picked up by the immediate-next generation (host emits via `queueMicrotask`; assembly runs a tick later in the macrotask queue, so the inject lands first). **`message` is the raw host `Message` record — it uses `is_user` (boolean), NOT `role`, and snake_case fields (`swipe_id`, `swipe_dates`); it is NOT the camelCase `ChatMessage` DTO from `getMessages()`.** Note: `message` does NOT carry the active character — resolve via `api.chats.get(chatId).then(c => c.characterId)` then `api.characters.get(characterId)`. |
| `MESSAGE_EDITED` | { chatId, message: Message } |  |
| `MESSAGE_DELETED` | { chatId, messageId } |  |
| `MESSAGE_SWIPED` | { chatId, message: Message, action, swipeId, previousSwipeId? } | Twice per swipe-with-regen (initiation + completion); once for swipe-without-regen. |
| `SWIPE_EDITED` | { chatId, message: Message, previousSwipeId } |  |
| `CHARACTER_MESSAGE_RENDERED` | { chatId, messageId } |  |
| `USER_MESSAGE_RENDERED` | { chatId, messageId } |  |
| `GENERATION_STARTED` | { generationId, chatId, model } | At the start of a new generation, **before prompt assembly + interceptor invocation** in the same generation pipeline. `api.chat.inject(...)` calls from this handler ARE picked up by this generation. Symmetric pre-assembly hook to `MESSAGE_SENT` — use whichever fits the script flow. |
| `GENERATION_ENDED` | { generationId, chatId, messageId, content } | Assistant-side message arrival — the clean hook for *generated* replies (whereas `MESSAGE_SENT` fires on every message creation, user sends included). Fires **after** the generation completes — `api.chat.inject(...)` calls from this handler are too late for the just-finished generation but WILL be picked up by the next one. Payload has no `swipeId` — look it up via `api.chat.getMessages` if needed. |
| `GENERATION_STOPPED` | { generationId, chatId, content } |  |
| `STREAM_TOKEN_RECEIVED` | { generationId, chatId, token } | Once per streamed token — **high-frequency**. Deliberately NOT offered in the editor event picker (wiring a whole script body per token is rarely intended); advanced use only. |
| `CHAT_CHANGED` | { chat, changedFields }  // chat = the full updated Chat; chatId is `chat.id`, not flat | Chat **metadata** mutations only (rename, etc.). The primary emission carries `{ chat, changedFields }` (read the id as `chat.id`). Does NOT fire on chat open/switch — use `CHAT_SWITCHED` for that. |
| `CHAT_SWITCHED` | { chatId: string \| null }  // null on return-to-home — NO characterId on the payload | Active chat opens, switches, or closes (chatId becomes null on return-to-home). **Important — Phase-1/Phase-2 character resolution**: triggers fire during Phase 1 (chatId set sync); characterId is resolved Phase-2 ~10–15 ms later via async lookup. So `data.characterId` does NOT exist on the payload, and reading the active-context characterId at trigger-fire time can see null/stale. **Pattern**: call `api.chats.getActive()` and read `chat.characterId` — that hits the host's live state which has it populated regardless of Phase-2 status. |
| `CHAT_FORKED` | { sourceChatId, forkedChatId, chat, branchId, forkedAtMessageId, forkedAtMessageIndex } | A chat was forked (branched) from a message into a new chat that shares the source character — messages up to and including the fork point are copied. `forkedChatId === chat.id`; the forked `chat.metadata` carries `branched_from` + `branch_at_message`. Use it to clone per-chat state into the fork, log lineage, or re-decorate the new chat. Emitted by the host branch-chat flow on newer Lumiverse builds. |
| `CHARACTER_CREATED` | { character: Character } | A character was created OR imported (the host emits it on card import too). `data.character` is the new character record (carries its `extensions` blob). NOTE: LumiScript itself subscribes to this internally for card-embedded-script detection; your trigger handler runs alongside that, independently. |
| `CHARACTER_EDITED` | { id, character: Character } |  |
| `CHARACTER_DELETED` | { id } |  |
| `CHARACTER_DUPLICATED` | { id, newId } |  |
| `PERSONA_CHANGED` | { persona: Persona } |  |
| `IMAGE_UPLOADED` | { image } | An image was saved to the user image library — manual upload OR generated/saved via api.imageGen / api.images. `data.image` is the full Image record (carries its `id`, usable with api.images.get / api.theme.extractColors / spindle.characters.setAvatar). NOTE: host docs list `{ imageId }` — that is stale; the real payload is `{ image }`. |
| `IMAGE_DELETED` | { id } | An image was deleted from the library. `data.id` is the removed image id. (Host docs list `{ imageId }` — stale; real payload is `{ id }`.) |
| `WORLD_INFO_ACTIVATED` | { entries: WorldInfoEntry[] } | World Info entries were activated during prompt assembly. |
| `WORLD_BOOK_CHANGED` | { id, worldBook: WorldInfo } | Coarse-grained: world book was created, updated, had its semantic-activation toggled, or had any of its entries mutated (entry create / update / delete / reorder / bulk-op / import). Fires alongside `WORLD_BOOK_ENTRY_CHANGED` on per-entry mutations — handlers subscribed to both see two events per change. Bulk imports suppress per-entry events and emit this once at the end. |
| `WORLD_BOOK_DELETED` | { id } | World book was deleted. |
| `WORLD_BOOK_ENTRY_CHANGED` | { id, worldBookId, entry: WorldInfoEntry } | Entry was created or updated. Does NOT fire during bulk imports — those emit a single `WORLD_BOOK_CHANGED` for the parent book instead. Subscribe to `WORLD_BOOK_CHANGED` in addition if you need to catch imported entries. |
| `WORLD_BOOK_ENTRY_DELETED` | { id, worldBookId } | Entry was deleted. |
| `SETTINGS_UPDATED` | { key, value } |  |
| `PRESET_CHANGED` | { presetId } |  |
| `CONNECTION_PROFILE_LOADED` | { connectionId } |  |
| `REGEX_SCRIPT_CHANGED` | { id, script: RegexScriptInfo }  // create / update / duplicate / reorder / enable / disable. Requires regex_scripts permission. |  |
| `REGEX_SCRIPT_DELETED` | { id }  // Requires regex_scripts permission. |  |
| `PERMISSION_CHANGED` | { extensionId, permission, granted, allGranted: string[] } | A Lumiverse permission was granted to or revoked from LumiScript. `granted` is the new boolean state of `permission`; `allGranted` is the full current permission list. Use to (re)initialize a feature when its permission is granted, or degrade gracefully when revoked. `extensionId` identifies which extension — filter if you only care about LumiScript's. |

## Broadcast events (script-to-script pub/sub)

Emitted on the broadcast bus (`api.broadcast.on(event, handler)`). Built-in events are listed below; scripts can also publish custom events (any non-`ls:` prefix).

| Event | Payload | Emitted by |
|---|---|---|
| `ls:tool:registered` | { name, scriptId } | api.tools.register() |
| `ls:tool:unregistered` | { name, scriptId } | api.tools.unregister() / auto-cleanup |
| `ls:tool:invoked` | { name, args, result, scriptId, callMs, councilMember? } | api.tools.invoke() + host tool dispatch (Council / direct LLM) |
| `ls:macro:registered` | { name, scriptId, mode: 'push' \| 'pull' } | api.macros.register() |
| `ls:macro:unregistered` | { name, scriptId } | api.macros.unregister() / auto-cleanup |
| `ls:collection:created` | { name, scope, scriptId, path } | api.db.collection() |
| `ls:collection:dropped` | { name, scope, scriptId, path, deletedCount } | api.db.drop() |
| `ls:collection:inserted` | { name, scope, scriptId, id, record } | collection.insert() |
| `ls:collection:updated` | { name, scope, scriptId, count, filterKind: 'all' \| 'object' \| 'fn' } | collection.update() (only when count > 0) |
| `ls:collection:deleted` | { name, scope, scriptId, count, filterKind } | collection.delete() / clear() (clear emits count=-1) |
| `ls:collection:size-warning` | { name, scope, scriptId, bytes } | auto — collection exceeds 10 MB soft threshold |
| `ls:scriptStorage:set` | { scriptId, key, value } | api.scriptStorage.set() |
| `ls:scriptStorage:delete` | { scriptId, key } | api.scriptStorage.delete() (only when the key existed) |
| `ls:scriptStorage:clear` | { scriptId } | api.scriptStorage.clear() / auto-cleanup on script disable / delete (only when the script HAD entries) |

## Runtime directives

LumiScript **runtime directives** are special comments that change how the runtime treats your script. They live anywhere at line start in the script source and follow the form `// @ls:<directive-name>`. The `@ls:` prefix distinguishes runtime-active directives from passive frontmatter tags like `@description`, `@author`, `@version`, `@tags` — those are read by humans and the pack import/export tooling but don't affect runtime behavior. Detection happens at `update_script` time (each code save); no persistence, no schema change.

| Directive | Applies to | What it does |
|---|---|---|
| `// @ls:reload-on-edit` | Enabled trigger scripts (libraries are loaded on-demand and ignore the directive). | Opts the script INTO automatic hot-reload after a code save. Without this directive, the script's closures stay stale until the next real trigger fire or until the user clicks the Reload button on the editor topbar. Add the directive to scripts whose module-scope code is idempotent and cheap (no expensive LLM calls, no duplicate DB writes, no leaked timers). The body re-runs end-to-end on each edit ~500ms after the autosave settles. |

## Macros

Built-in `{{macros}}` LumiScript registers with the Lumiverse macro engine. Available in chat templates, presets, and `api.utils.template.render()`.

### Presence

| Macro | Aliases | Returns | Description |
|---|---|---|---|
| `{{lumiScriptActive}}` | — | boolean | Push-model boolean. "true" when the LumiScript master toggle is on, "false" when off. Ideal for conditional preset blocks: {{if::{{lumiScriptActive}}}}…{{/if}} |

### Character Variables

| Macro | Aliases | Returns | Description |
|---|---|---|---|
| `{{getcvar::key}}` | {{getcharvar::key}} | string | Get a character-scoped variable. Returns "" if the key is not set or there is no active character. |
| `{{setcvar::key::value}}` | {{setcharvar::key::value}} | silent | Set a character-scoped variable to value. |
| `{{addcvar::key::n}}` | {{addcharvar::key::n}} | silent | Add the number n to a character-scoped variable (treated as 0 if unset or non-numeric). |
| `{{inccvar::key}}` | — | silent | Increment a character-scoped variable by 1. |
| `{{deccvar::key}}` | — | silent | Decrement a character-scoped variable by 1. |
| `{{hascvar::key}}` | {{hascharvar::key}} | boolean | Returns "true" if the variable exists in the active character's store, "false" otherwise. |
| `{{deletecvar::key}}` | {{deletecharvar::key}} | silent | Delete a character-scoped variable. |

## Key types

Public types referenced by `api.*` method signatures. **Each type is in the lookup table** — call `lookup_api("TypeName")` for the full field list with per-field type, optionality, and description.

- `ChatMessage` — Returned by api.chat.getMessages().
- `Message` — The RAW host message record delivered as `data.message` by chat events (MESSAGE_SENT, MESSAGE_EDITED, MESSAGE_SWIPED, SWIPE_EDITED). Snake_case — distinct from the camelCase `ChatMessage` DTO returned by api.chat.getMessages(). Most importantly it uses `is_user` (boolean), NOT `role`.
- `GetMessagesOptions` — Passed to api.chat.getMessages(options?).
- `SendMessageOptions` — Passed to api.chat.sendMessage(content, options?). HTML rendering note: a block-level element (<div>, <section>, <article>, etc.) whose content includes a <style> tag OR three or more inline style="..." attributes is auto-extracted into a Shadow DOM "island" by the host renderer. This isolates card-style rules from the chat UI and prevents markdown from corrupting interactive markup. To opt out (e.g. you need document-level click delegation, CSS cascade into surrounding DOM, or MutationObserver access from the message subtree), add data-no-island to the outer block element's opening tag. Opting out disables both style isolation AND the markdown-safety wrapper — scope your selectors with a unique class prefix and ensure markdown won't misinterpret your content. Standalone <style> blocks not inside a wrapper element are extracted together with subsequent sibling HTML; wrap them in <div data-no-island> if you need them inline.
- `ChatGenerationOptions` — Per-call generation overrides for api.chat.sendMessage(content, { triggerGeneration: true, generation: ... }). Mirrors the host's ChatAppendGenerationOptionsDTO 1:1 in camelCase. Each field is optional; omitted fields fall through to the active chat's resolved defaults (same as a manual UI generation). Use this when a tool script needs to deviate from the user's normal chat configuration for a single triggered generation.
- `MessagePatch` — Passed to api.chat.editMessage(id, patch) when using the richer object form. Only fields you provide are updated. Patches touching swipes / swipeId / swipeDates fire SWIPE_EDITED alongside MESSAGE_EDITED; plain content patches fire only MESSAGE_EDITED.
- `InjectOptions` — Passed to api.chat.inject(id, content, options?).
- `InjectionInfo` — Returned by api.chat.getInjections().
- `MessageContentProcessorOptions` — Passed to api.chat.registerContentProcessor(handler, options?).
- `MessageContentProcessorCtx` — Passed to a registerContentProcessor handler. All fields readonly. The host's chat_mutation permission gates this surface, but does NOT route api.chat.* mutations through the chain (loop safety).
- `MessageContentProcessorResult` — Return value of a registerContentProcessor handler. Return undefined / void to pass through, or a partial patch. content replaces the stored content. extra shallow-merges into existing — keys you omit are PRESERVED. extra is IGNORED on swipe origins (swipes share the parent message's extra) and on 'render' (no row to mutate). Return ONLY keys you mutated; pristine initial.extra keys are NOT round-tripped to avoid re-stamping unchanged keys on every write.
- `MacroInterceptorOptions` — Passed to api.macros.registerInterceptor(handler, options?). Pre-filters short-circuit before the handler runs.
- `MacroInterceptorCtx` — Passed to a registerInterceptor handler. All fields readonly. The handler receives the CURRENT raw template (already transformed by any earlier interceptors in the chain) and returns either a transformed template string or void to pass through.
- `ModalItem` — A single content item in an api.ui.showModal() items array. Five variants rendered in order using the system theme.
- `ShowModalOptions` — Options for api.ui.showModal(items, options).
- `ModalResult` — Dismissal payload inside ModalHandle.result.
- `ModalHandle` — Returned by api.ui.showModal(). Await handle.result for dismissal; call handle.close() to dismiss programmatically.
- `AdvancedModalOptions` — Options for api.ui.showAdvancedModal(options). Extension-owned body DOM.
- `AdvancedModalDismissReason` — Reason a modal was dismissed. Passed to onDismiss handlers.
- `AdvancedModalHandle` — Returned by api.ui.showAdvancedModal(). Body DOM is fully script-owned via the root DOMHandle. The modal element carries data-ls-script and data-ls-modal attributes, so api.ui.dom.addStyle() @scope rules match content inside the modal just like any other injected DOM.
- `ContextMenuItem` — A single entry in api.ui.showContextMenu()`s items array.
- `ShowContextMenuOptions` — Options for api.ui.showContextMenu().
- `InputBarActionOptions` — Options for api.ui.registerInputBarAction().
- `InputBarActionHandle` — Returned by api.ui.registerInputBarAction(). Actions appear in the chat input-bar Extras popover under a teal-badged extension header. Limits: 4 per script, 12 global.
- `FloatWidgetOptions` — Options for api.ui.createFloatWidget(). Widget is a small draggable overlay with script-owned body DOM.
- `FloatWidgetHandle` — Returned by api.ui.createFloatWidget(). Body DOM is script-owned via root (DOMHandle). The root element carries data-ls-script and data-ls-widget attributes, so api.ui.dom.addStyle() @scope rules match content inside the widget. getPosition() / isVisible() return backend-cached state — see docs for caching semantics around moveTo and drag-end echoes.
- `MountAppOptions` — Options for api.ui.mountApp(). All optional.
- `MountedAppHandle` — Returned by api.ui.mountApp(). A route-persistent full-bleed document.body portal. Body DOM is script-owned via root (DOMHandle); the root element carries data-ls-script + data-ls-mount so api.ui.dom.addStyle() @scope rules match content inside.
- `DrawerTabOptions` — Options for api.ui.registerDrawerTab(). Tab appears in the ViewportDrawer sidebar and is automatically searchable in the command palette.
- `DrawerTabHandle` — Returned by api.ui.registerDrawerTab(). Body DOM is script-owned via root (DOMHandle). The root element carries data-ls-script and data-ls-tab attributes, so api.ui.dom.addStyle() @scope rules match content inside the tab. LumiScript enforces 1 drawer tab per script; if all 4 of LumiScript's host-quota tabs are in use by other scripts, registration throws with a distinct 'quota exhausted' message.
- `UIDrawerTab` — A drawer tab discoverable via api.ui.getDrawerTabs() — built-in or extension-contributed. The id is what you pass to api.ui.openDrawerTab().
- `UISettingsTab` — A settings tab discoverable via api.ui.getSettingsTabs(). Role-restricted tabs are filtered out for users lacking the role. The id is what you pass to api.ui.openSettings().
- `PickFileOptions` — Options for api.ui.pickFile().
- `PickedFile` — A file returned by api.ui.pickFile(). bytes is the raw Uint8Array — decode text via new TextDecoder().decode(bytes), or pass to api.images.upload / api.files.
- `UIKeyboardState` — Snapshot from api.ui.events.getKeyboardState() / onKeyboardChange().
- `UIDrawerState` — Snapshot from api.ui.events.getDrawerState() / onDrawerChange().
- `UISettingsState` — Snapshot from api.ui.events.getSettingsState() / onSettingsChange().
- `DOMInjectOptions` — Options for api.ui.dom.inject(target, html, options?).
- `DOMMessageInjectOptions` — Options for api.ui.dom.injectAtMessage(messageId, html, options?).
- `DOMDelegateOptions` — Options for api.ui.dom.delegate(selector, event, handler, options?).
- `DOMDelegatedEventData` — Event data delivered to handlers registered via api.ui.dom.delegate(). Extends DOMEventData with a serialized snapshot of the matched element + modifier-key state + optional message context.
- `DOMHandle` — Returned by api.ui.dom.inject() and api.ui.dom.injectAtMessage(). Most methods are fire-and-forget; the exception is `read(options?)` which is async (it awaits a frontend roundtrip).
- `MountedComponentHandle` — Returned synchronously by api.ui.components.mountBadge / mountSpinner (display-only components). Methods are fire-and-forget.
- `MountedValueComponentHandle` — Generic `MountedValueComponentHandle<TOptions, TValue>`. Returned by interactive mounts (api.ui.components.mountSwitch → boolean, mountTextInput → string). Extends MountedComponentHandle with an async getValue().
- `SpindleBadgeOptions` — Options for api.ui.components.mountBadge().
- `SpindleSpinnerOptions` — Options for api.ui.components.mountSpinner().
- `SpindleSwitchOptions` — Options for api.ui.components.mountSwitch(). onChange fires into your script on every toggle.
- `SpindleTextInputOptions` — Options for api.ui.components.mountTextInput(). onChange fires on every user change.
- `SpindleTextAreaOptions` — Options for api.ui.components.mountTextArea(). Like SpindleTextInputOptions plus rows.
- `SpindleNumericInputOptions` — Options for api.ui.components.mountNumericInput(). Value is number | null (null = empty when allowEmpty).
- `SpindleNumberStepperOptions` — Options for api.ui.components.mountNumberStepper(). Like SpindleNumericInputOptions minus integer; step defaults to 1.
- `SpindleCheckboxOptions` — Options for api.ui.components.mountCheckbox().
- `SpindleRangeSliderOptions` — Options for api.ui.components.mountRangeSlider(). onCommit fires once when a drag/tap ends; onDragValue fires live during a drag.
- `SpindleSelectOption` — A single option in api.ui.components.mountSelect() / mountMultiSelect().
- `SpindleSelectOptions` — Options for api.ui.components.mountSelect() (single-select). Extends SpindleSelectOptionsBase (options, placeholder, searchPlaceholder, searchThreshold, emptyMessage, noResultsMessage, triggerLabel, triggerIcon, portal, align, maxHeight, minWidth, disabled, className).
- `SpindleMultiSelectOptions` — Options for api.ui.components.mountMultiSelect(). Same base as SpindleSelectOptions but value/onChange use string[].
- `SpindleFolderDropdownOptions` — Options for api.ui.components.mountFolderDropdown().
- `SpindleModelComboboxOptions` — Options for api.ui.components.mountModelCombobox(). Connection-bound mode (connection) is recommended; manual mode uses models + onRefresh.
- `SpindlePaginationOptions` — Options for api.ui.components.mountPagination(). Fully controlled — currentPage/totalPages/onPageChange are REQUIRED; call handle.update({currentPage}) after navigating.
- `SpindleCloseButtonOptions` — Options for api.ui.components.mountCloseButton().
- `SpindleCollapsibleSectionOptions` — Options for api.ui.components.mountCollapsibleSection(). title is REQUIRED.
- `MountedCollapsibleSectionHandle` — Returned by api.ui.components.mountCollapsibleSection(). The host owns the header chrome; your script owns the body.
- `DOMEventData` — Serialized event data passed to DOM event handlers. A safe subset of the browser Event object.
- `DOMListenOptions` — Options bag for DOMHandle.on(event, handler, options?).
- `DOMReadOptions` — Options bag for DOMHandle.read(options?). All fields optional — `read()` with no argument returns a baseline snapshot.
- `SerializedDOMElement` — Snapshot returned by DOMHandle.read(). Frontend-built serialization of the element bound to the handle.

Which element gets snapshotted depends on the shape of what the script injected: for the common single-root case the user's root element is returned directly (e.g. `inject('<button class="x">Hi</button>')` → `tag: 'button'`); for multi-root or text-only content the snapshot falls back to LumiScript's wrapper (`tag: 'div'`, accurate `childCount`). Either way, internal `data-ls-*` and `data-spindle-ext` wrapper attributes are stripped from the `attrs` map.

Deliberate omissions: computed styles, bounding rect, recursive child snapshots, property snapshots (`.value` / `.checked`). Form-control live values can be read via `delegate(selector, 'input', ...)` event handlers; for deep markup traversal, request `{ html: true }` and parse client-side.
- `ConditionalPreventDefault` — Predicate-based preventDefault rule for DOMDelegateOptions / DOMListenOptions. Fires event.preventDefault() only when the event matches all provided filters (AND semantics). Each filter is optional; empty {} = always match (equivalent to `preventDefault: true`). Filters are evaluated synchronously frontend-side at fire time. Common shapes: { onKeys: ['Enter'], whenModifiers: { exclude: ['shift'] } } (plain Enter, not Shift+Enter); { onKeys: ['s', 'S'], whenModifiers: { require: ['ctrl'] } } (Ctrl+S override); { onButtons: [2] } (right-click only).
- `LLMMessage` — A single message in the messages array passed to api.llm.generate / generateStructured / generateWithTools.
- `LlmMessagePart` — A single content part inside an LLMMessage. Discriminated union — switch on the `type` field. Mirrors the host's LlmMessagePartDTO.
- `LLMOptions` — Resolution order: connectionId → connectionName → provider + model → active user connection.
- `Connection` — Read-only view of an LLM connection profile (api.connections.*). snake_case, mirrors the host ConnectionProfileDTO. NEVER contains the API key — only has_api_key. id/name map to LLMOptions.connectionId/connectionName.
- `GenerationReasoningOverride` — Per-request reasoning control, passed as LLMOptions.reasoning (host 0.5.x+). Resolved by the 'source' discriminator; rides beside parameters on the request, not inside the sampler bag. Mirrors the host GenerationReasoningOverrideDTO.
- `ConnectionReasoningBindings` — Typed value of Connection.reasoning_bindings (api.connections.*). Read-only snapshot of the reasoning settings bound to a profile; these override the user's global reasoning settings on this connection. Mirrors the host ConnectionReasoningBindingsDTO.
- `WebSearchOptions` — Passed to api.webSearch.query().
- `WebSearchResponse` — Returned by api.webSearch.query(). documents/context are omitted when scrape:false.
- `WebSearchSettings` — Returned by api.webSearch.getSettings(). Safe view — NEVER the API key (only hasApiKey).
- `DryRunOptions` — Passed to api.llm.dryRun(options?). All fields are optional; defaults use the active context.
- `LLMRawResult` — Return type of api.llm.generateWithTools() without a schema. On intermediate steps tool_calls is set; on the final step content holds the text response.
- `LLMRawResultStructured` — Generic type `LLMRawResultStructured<T>`. Return type of `api.llm.generateWithTools(messages, tools, opts, schema)` — the structured-output overload. On intermediate steps only `tool_calls` is set. On the final step only `content` is set, typed as `T` (the schema-parsed result).
- `StreamChunk` — One chunk yielded by api.llm.generateStream. Discriminated union — switch on the `type` field. snake_case throughout, mirroring LLMRawResult and the upstream StreamChunkDTO.
- `ToolCall` — A single function call inside LLMRawResult.tool_calls or LLMRawResultStructured.tool_calls.
- `DryRunResult` — Return type of api.llm.dryRun(). Contains everything that would be sent to the LLM plus diagnostic data.
- `DryRunBlock` — A single prompt composition block inside DryRunResult.breakdown.
- `DryRunTokenCount` — Per-block token counts inside DryRunResult.tokenCount. Only present if a tokenizer is configured.
- `WorldInfoActivationStats` — World info activation statistics inside DryRunResult.worldInfoStats.
- `DryRunMemoryStats` — Long-term memory retrieval statistics inside DryRunResult.memoryStats.
- `HttpRequestOptions` — Passed to api.utils.http.get / post / put / delete / request. Requires allowDangerous + cors_proxy permission. Responses are capped at 25 MB by the Lumiverse cors_proxy; larger bodies are rejected upstream.
- `HttpResponse` — Returned by api.utils.http.* methods. Response body is capped at 25 MB by the Lumiverse cors_proxy — requests for larger payloads reject with an upstream error.
- `TempWriteOptions` — Passed to api.files.tempWrite / tempWriteBinary (path, data, options?).
- `FileStatResult` — Returned by api.files.sharedStat(path).
- `TempStatResult` — Returned by api.files.tempStat(path).
- `TempRequestBlockOptions` — Passed to api.files.tempRequestBlock(sizeBytes, options?).
- `TempReservation` — Returned by api.files.tempRequestBlock(). Pass reservationId to tempWrite/tempWriteBinary options, or to tempReleaseBlock.
- `TempPoolStatus` — Returned by api.files.tempGetPoolStatus(). Global = across all extensions; extension* = this extension only.
- `Character` — Returned by api.characters.get / create / update.
- `CharacterCreateInput` — Passed to api.characters.create(input). Only name is required.
- `CharacterUpdateInput` — Passed to api.characters.update(id, input). Same fields as CharacterCreateInput, all optional.
- `ChatSession` — Returned by api.chats.get / getActive / update.
- `ChatSessionUpdateInput` — Passed to api.chats.update(id, input).
- `ChatMemoryChunk` — A single memory chunk inside ChatMemoryResult.chunks.
- `ChatMemoryResult` — Returned by api.chats.getMemories().
- `WorldInfo` — A world book header. Returned by api.worldInfo.get / create / update.
- `WorldInfoCreateInput` — Passed to api.worldInfo.create(input).
- `WorldInfoUpdateInput` — Passed to api.worldInfo.update(ref, input). All fields optional.
- `WorldInfoEntry` — A lorebook entry. Returned by api.worldInfo.entries.get / create / update. Key fields listed; full set available in IntelliSense hover.
- `WorldInfoEntryInput` — Passed to api.worldInfo.entries.create / update. All fields optional on update; content and key recommended on create.
- `ActivatedWorldInfoEntry` — Returned by api.worldInfo.getCapturedActive(). Extends WorldInfoEntry with activation metadata.
- `WorldInfoInterceptorEntry` — Subset of WorldInfoEntry exposed to a registerInterceptor handler. Read-only — to mutate, return a result patch from the handler.
- `WorldInfoInterceptorMessage` — One chat message exposed to a registerInterceptor handler.
- `WorldInfoInterceptorCtx` — Passed to a registerInterceptor handler. All fields readonly. Persist cross-turn state via api.chats.update(chatId, { metadata: ... }) — chatMetadata here is a snapshot.
- `WorldInfoInterceptorResult` — Return value of a registerInterceptor handler. Return undefined / void / omit all four arrays for full pass-through. Vote-off precedence: once any handler in the chain votes disabled for an id, no later enabled or forced vote can revive it. mutated is last-write-wins per id.
- `WorldInfoInterceptorOptions` — Passed to api.worldInfo.registerInterceptor(handler, options?).
- `RegisteredWorldInfoInterceptorInfo` — Returned by api.worldInfo.listInterceptors(). Diagnostic surface — un-gated.
- `RegexScriptInfo` — Snapshot of a regex find/replace script. Returned by api.regexScripts.list / get / findByName / getActive / create / update. Field names are camelCase translations of the underlying snake_case host DTO.
- `RegexScriptListOptions` — Filter options for api.regexScripts.list().
- `RegexScriptActiveOptions` — Required + optional fields for api.regexScripts.getActive(). Mirrors the resolution Lumiverse uses internally during a generation: only enabled rules, only rules whose target matches, only rules whose scope applies.
- `RegexScriptCreateInput` — Passed to api.regexScripts.create(input). Only name and findRegex are required; everything else gets host-side defaults.
- `RegexScriptUpdateInput` — Passed to api.regexScripts.update(scriptId, input). Same shape as RegexScriptCreateInput but ALL fields optional.
- `Persona` — Returned by api.personas.get / getDefault / getActive / create / update.
- `PersonaCreateInput` — Passed to api.personas.create(input). Only name is required.
- `PersonaUpdateInput` — Passed to api.personas.update(personaId, input). All fields optional — only the fields provided are updated; omitted fields are left unchanged.
- `CouncilSettings` — Returned by api.council.getSettings(). The user's top-level Council configuration object. All fields camelCase — no DTO transform on the LumiScript side.
- `CouncilMember` — A single Council member assignment — the binding row stored in CouncilSettings.members. Includes role + chance + tool assignment list. getMembers() returns the same data enriched with full Lumia source fields as CouncilMemberContext[].
- `CouncilMemberContext` — Returned by api.council.getMembers() AND delivered as the second arg to api.tools.register handlers when invoked via the Council execution path. Merges a member's assignment (role + chance) with the source Lumia item's full definition (avatar / definition / personality / behavior). When you're inside a tool handler, prefer reading ctx.councilMember directly rather than calling getMembers() — it's faster and tied to the active invocation.
- `CouncilToolsSettings` — Settings governing Council tool execution. Nested inside CouncilSettings.toolsSettings.
- `LumiaItem` — Returned by api.council.getAvailableLumiaItems(). LumiScript-shaped (camelCase) mapping of upstream LumiaItemDTO. The full pool of Lumia items the user has across all installed packs — superset of what's currently assigned to Council members.
- `ToolDefinition` — Passed to api.tools.register(name, def, handler).
- `ToolInvocationArgs` — Parameter passed to tool handler callbacks registered via api.tools.register(). Always carries the well-known Lumiverse fields (context / __userId / __deadlineMs) on a host invocation; your registration-schema parameters are filled only on the DIRECT paths (api.tools.invoke + api.llm.generateWithTools), NOT on the Council path — see the [key] field.
- `ToolInvocationContext` — Optional third parameter passed to tool handlers. Populated when the host invokes the tool (Council / direct-LLM function-call); undefined when invoked via api.tools.invoke() (script-to-script). The Council-path fields (requestId, councilMember, contextMessages) are populated together; the script-to-script path leaves them all undefined.
- `RegisteredToolInfo` — Returned by api.tools.list(). A serialisable snapshot of a registered tool.
- `MacroDefinition` — Passed to api.macros.register(name, def, handler?).
- `MacroContext` — Parameter passed to a pull-mode macro handler at resolution time. Mirrors Lumiverse's MacroExecContext. Per convention, `args` is on ctx — not a top-level variable.
- `RegisteredMacroInfo` — Returned by api.macros.list(). Visible across scripts — any script can see push-values set by any other script (matches the already-world-readable nature of macros).
- `DbScope` — Scope of an api.db collection — determines the storage path and resolution requirements. Baked into the collection handle at api.db.collection() time.
- `CollectionOpts` — Options for api.db.collection(name, opts). Generic over the record type so schema (if provided) infers field shapes.
- `DbRecord` — Record shape produced by api.db.*. Every inserted record carries id + createdAt + updatedAt alongside user-supplied fields. id and createdAt are immutable (update() silently strips them from the patch). updatedAt is bumped on every successful update.
- `DbFilter` — Filter shapes accepted by find() / findOne() / update() / delete() / count(). The store picks a matching strategy based on the runtime type. Operator envelopes (LumiScript 0.20.0+) unlock Mongo-style comparisons without dropping to a function filter.
- `EventTrackOptions` — Options for api.events.track().
- `EventQueryFilter` — Filter for api.events.query() and api.events.replay().
- `EventRecord` — Returned by api.events.query() and api.events.replay().
- `MacrosResolveOptions` — Options for api.utils.macros.resolve(template, options?).
- `MacrosResolveResult` — Returned by api.utils.macros.resolve().
- `TokenCountOptions` — Options for api.tokens.count* methods.
- `TokenCountResult` — Returned by api.tokens.count* methods.
- `CharacterAvatarUpload` — Payload for api.characters.setAvatar(id, avatar).
- `DatabankScope` — **Enum**: `'global' | 'character' | 'chat'`. Activation scope for a databank. There are EXACTLY THREE values — there is no `'script'` scope. `'global'` is unscoped (available everywhere); `'character'` is keyed by a character UUID via `scopeId`; `'chat'` is keyed by a chat UUID via `scopeId`. `scopeId` is REQUIRED for `'character'` and `'chat'`, omitted (or null) for `'global'`. Scope cannot be changed after creation — pick the right one up front.
- `DatabankDocumentStatus` — **Enum**: `'pending' | 'processing' | 'ready' | 'error'`. Ingestion lifecycle of an uploaded document. New uploads land as `'pending'` and progress through `'processing'` to `'ready'` (success) or `'error'` (terminal failure). `documents.getContent()` returns null for anything other than `'ready'`; `documents.waitUntilReady()` polls until ready and throws on `'error'` or timeout.
- `DatabankInfo` — Returned by api.databanks.get / findByName / create / update; entries inside list().
- `DatabankCreateInput` — Passed to api.databanks.create(input). Validates host-side — invalid scope or missing scopeId rejects the call.
- `DatabankUpdateInput` — Passed to api.databanks.update(databankId, input). Scope cannot be changed after creation — there are no scope/scopeId fields here on purpose.
- `DatabankDocumentInfo` — Returned by api.databanks.documents.get / findByName / create / update / waitUntilReady; entries inside documents.list().
- `DatabankDocumentCreateInput` — Passed to api.databanks.documents.create(databankId, input). Upload returns immediately with status='pending' — use waitUntilReady() to await ingestion. Max size 10 MB.
- `DatabankDocumentUpdateInput` — Passed to api.databanks.documents.update(documentId, input). The URL-safe slug regenerates automatically from the new name.
- `DatabankWaitUntilReadyOptions` — Optional polling parameters for api.databanks.documents.waitUntilReady(documentId, options?). Throws on timeout, error status, or document deletion.
- `CortexQuery` — Input to api.memories.cortex.query(). chatId + queryText are required; the active userId is folded in by LumiScript.
- `CortexResult` — Returned by api.memories.cortex.query() / getCached(). The same shape the host uses internally during prompt assembly.
- `LinkedCortexResult` — Returned by api.memories.cortex.queryLinked() / getCachedLinked(). Attached vaults + interlink targets.
- `MemoryCortexConfig` — Returned by api.memories.cortex.getConfig(); patch with putConfig(). Permissive — only top-level toggles are typed; advanced + host-added fields pass through the index signature.
- `ChatChunk` — Returned by api.memories.chatMemory.listChunks(). A vectorized chat chunk — the {{memories}} retrieval unit.
- `ChatMemoryWarmupResult` — Returned by api.memories.chatMemory.warm().
- `CortexUsageStats` — Returned by api.memories.stats.usage(). Host gc fields (mention counts, last GC, etc.) pass through.
- `CortexIngestionStatus` — Returned by api.memories.stats.ingestionStatus(), or null when never ingested.
- `CortexIngestionTelemetry` — Returned by api.memories.stats.ingestionTelemetry().
- `MemoryEntity` — A tracked entity in the cortex graph. Returned by api.memories.entities.* and inside CortexResult.entityContext (as the lighter EntitySnapshot).
- `MemoryEntityUpsert` — Input to api.memories.entities.upsert(). Matches the extractor shape so a script can replay its own NER results.
- `MemoryRelation` — A typed relation edge. Returned by api.memories.relations.*.
- `MemoryRelationUpsert` — Input to api.memories.relations.upsert(). Uses entity NAMES — both endpoints must already exist in the graph or the edge is silently dropped.
- `MemoryConsolidation` — A narrative-arc consolidation. Returned by api.memories.consolidations.list / latestArc.
- `MemorySalience` — A per-chunk salience record. Returned by api.memories.salience.list.
- `Vault` — A frozen cortex snapshot. Returned by api.memories.vaults.list / create; inside VaultWithContents.vault.
- `VaultCreate` — Input to api.memories.vaults.create().
- `ChatLink` — A vault attach or chat interlink. Returned by api.memories.links.list / attach.
- `ChatLinkAttach` — Input to api.memories.links.attach(). Provide vaultId for a vault attach, or targetChatId for an interlink.
- `ImageInfo` — Returned by api.images.upload / uploadFromDataUrl / get. Camel-case mirror of ImageDTO from Spindle.
- `ImageUploadInput` — Passed to api.images.upload(input).
- `ImageUploadFromDataUrlOptions` — Passed to api.images.uploadFromDataUrl(dataUrl, options?). The data URL itself carries the bytes + MIME; these options only set ownership / display metadata.
- `ImageGenInput` — Passed to api.imageGen.generate(input). Mirrors ImageGenRequestDTO with camel-case field names on the LumiScript surface.
- `ImageGenResult` — Returned by api.imageGen.generate(input). The `imageId` is the integration seam — pass to api.images.get / api.theme.extractColors / spindle.characters.setAvatar. Use `imageDataUrl` for inline rendering (no auth needed) or `imageUrl` for push-notification image fields.
- `ImageGenProviderInfo` — Returned by api.imageGen.getProviders(). Each provider declares its capability schema; drive dynamic parameter UIs from `capabilities.parameters`.
- `ImageGenConnectionInfo` — Returned by api.imageGen.listConnections() / getConnection(). API keys are NEVER exposed — only `hasApiKey: boolean` indicates presence.
- `ImageGenParameterSchema` — One parameter's contract within an ImageGenProviderInfo.capabilities.parameters record. Use to drive dynamic parameter UIs or validate user-supplied args before calling generate().
- `ColorRGB` — RGB color value, 0–255 per channel. Used in ColorExtractionInfo.dominant / regions.* / average.
- `ColorHSL` — HSL color value. Used in ColorExtractionInfo.dominantHsl + ThemePaletteConfig.accent + ThemeInfo.accent. Drop-in compatible across all three — the typical pipeline is `extractColors(imageId).then(p => applyPalette({accent: p.dominantHsl}))`.
- `ColorExtractionInfo` — Returned by api.theme.extractColors(imageId). `dominantHsl` is the ready-to-pass accent for api.theme.applyPalette({accent: ...}).
- `ThemeOverride` — Passed to api.theme.apply(overrides). Two-axis: `variables` applies regardless of mode, `variablesByMode` applies per dark/light at apply time. LumiScript maintains per-script attribution — multiple scripts' apply() calls merge with per-key last-applied-wins semantics.
- `ThemePaletteConfig` — Passed to api.theme.applyPalette(palette | null). Lumiverse generates the full coherent variable set from the accent — preserves the user's glass / radius / font / UI-scale settings. Across LumiScript scripts: most-recent-script-wins. Pass `null` to drop this script's palette contribution.
- `ThemeInfo` — Returned by api.theme.getCurrent(). Read-only snapshot of the user's current theme configuration (NOT including any extension overrides).
- `ThemeVariablesConfig` — Passed to api.theme.generateVariables(config). Mirrors the inputs that Lumiverse's theme engine uses to produce the full set of ~80+ CSS variables. The result can be passed to apply({variables}) for a complete coherent override, or tweaked individually before applying.

## Built-in libraries

Load via `script.require('ls:<name>')`. **Each entry below is in the lookup table** — call `lookup_api("ls:<name>.<method>")` for the full record.

### ls:components

| Method | Args | Description |
|---|---|---|
| `messageFooter` | messageId, html, options? | Attach a styled footer below a message bubble. Returns DOMHandle, or CollapsibleDOMHandle when options.collapsible is true. Options: { id?, className?, collapsible?, title?, defaultCollapsed? }. |
| `messageHeader` | messageId, html, options? | Attach a styled header above message content. Returns DOMHandle, or CollapsibleDOMHandle when options.collapsible is true. Options: { id?, className?, collapsible?, title?, defaultCollapsed? }. |
| `progressBar` | target, options? | Inject a progress bar with live setValue(). Returns ProgressBarHandle. Options: { value?, label?, color?, showPercent?, height?, id?, className? }. |
| `floatingButton` | label, options? | Fixed-position action button. Returns DOMHandle. Options: { position?, icon?, variant?, size?, draggable?, id?, className? }. Pass `draggable: true` to make the button user-repositionable; the new position persists per-script via api.scriptStorage. |
| `badgeHtml` | text, options? | Returns badge/pill HTML string for composing inside other injections. |
| `statBarHtml` | label, value, options? | Returns labeled stat bar HTML string. Options: { max?, color?, showValue?, height?, className? }. |
| `keyValueHtml` | label, value, options? | Returns label-value pair HTML string. Options: { muted?, className? }. |
| `multiSelect` | options | Open an advanced modal with a checkbox list + Confirm / Cancel. Resolves Promise<string[] \| null> — selected keys on confirm, null on cancel / dismiss / teardown. Options: { title, items, confirmLabel?, cancelLabel?, minSelect?, maxSelect?, width?, maxHeight? }. Keys returned in input item order. Requires app_manipulation (transitively via showAdvancedModal). |

### ls:council-prompt

| Method | Args | Description |
|---|---|---|
| `buildCouncilMessages` | options | Build the full LLMMessage[] array for a Council-voice tool invocation — identity + role + tool spec + flattened context + closing directive. Returns [system, system?, user]. Throws if options.councilMember is missing. |
| `buildCouncilSystemPrompt` | options | Build just the system-prompt string used by buildCouncilMessages. Useful when composing your own message structure. |
| `buildCouncilIdentity` | councilMember | Member-identity block: "You are a council member named ..." plus WHO YOU ARE / INSTRUCTION sections when personality fields are present. |
| `roleNote` | role | Role-aware directive block. Returns "" when role is empty; otherwise prepends "\n". |
| `brevityNote` | maxWords | Word-budget directive. Returns "" when maxWords ≤ 0; otherwise prepends "\n\n" to attach as a paragraph. |
| `userControlNote` | allow | User-character guidance block. Permissive variant when allow=true, restrictive variant when false. Always non-empty (prepended with "\n\n"). |
| `debug.formatMember` | councilMember | Pretty-printed snapshot of all CouncilMemberContext fields — identifiers, identity strings, chance, gender label, avatar URL, personality strings (truncated for long values). Returns a framed string ready to console.log. |
| `debug.formatIdentity` | councilMember | Framed wrapper around buildCouncilIdentity output with the member name in the header. For "what does the identity prefix look like for this member" inspection. |
| `debug.formatSystemPrompt` | options | Framed wrapper around buildCouncilSystemPrompt output with character count in the header. Shows exactly what goes to the LLM as the system message. |
| `debug.formatMessages` | options | Framed rendering of the full LLMMessage[] array with per-message headers (index, role, char count). Reveals the context system message that isn't visible from the system-prompt view alone. |
| `debug.formatReport` | options | Comprehensive one-call dump: member snapshot + identity + system prompt + all messages, stitched together. What you reach for when you want the whole picture in one console.log. |

### ls:icons

| Method | Args | Description |
|---|---|---|
| `svg` | Record<IconName, string> | Map of icon name to inline SVG string. Direct-property access is sync and typed — e.g. svg.heart returns a 24×24 <svg>…</svg> string with stroke: currentColor and fill: none. Drop straight into iconSvg options (except input-bar actions — see forInputBar) or DOM-injected HTML templates. |
| `sized` | name, pixels | Return svg[name] with width/height attributes overridden to pixels. viewBox is preserved so the icon scales cleanly. Throws on unknown name or non-positive pixels. All other default attrs (fill, stroke, stroke-width, stroke-linecap, stroke-linejoin) pass through unchanged. |
| `forInputBar` | name | Equivalent to sized(name, 14). Use for api.ui.registerInputBarAction iconSvg — the host renders input-bar icons in a 14×14 slot, and the default 24×24 svg[name] overflows and misaligns with the label. For drawer tabs / float widgets / DOM injections, the default size is usually fine. |
| `names` | () | All available icon names — returns a FRESH array each call, so scripts can safely mutate it (e.g. filter in place for a picker UI) without corrupting the canonical list or affecting subsequent callers. |

### Built-in types

- `MessageFooterOptions / MessageHeaderOptions` — Options for messageFooter() and messageHeader().
- `CollapsibleDOMHandle` — Extends DOMHandle. Returned by messageHeader() / messageFooter() when collapsible is true.
- `BadgeHtmlOptions` — Options for badgeHtml().
- `StatBarHtmlOptions` — Options for statBarHtml().
- `ProgressBarOptions` — Options for progressBar(). Returns ProgressBarHandle (extends DOMHandle + setValue).
- `MultiSelectItem` — A single selectable row in a multiSelect() items array.
- `MultiSelectOptions` — Options for multiSelect(). Built on api.ui.showAdvancedModal — inherits the 2-per-script stack limit.
- `FloatingButtonOptions` — Options for floatingButton().
- `CouncilSystemPromptOptions` — Options for buildCouncilSystemPrompt() from ls:council-prompt. Three Council settings the host doesn't forward to extension tools (tool.prompt, maxWordsPerTool, allowUserControl) are supplied here — published tools probably want deterministic behavior regardless of local user preferences.
- `CouncilMessagesOptions` — Extends CouncilSystemPromptOptions. Passed to buildCouncilMessages() — adds context-source fields so the helper can include chat history in the output message array. When both contextMessages and args.context are present, contextMessages takes priority (preserves role boundaries); args.context is the fallback path for older Lumiverse hosts.
