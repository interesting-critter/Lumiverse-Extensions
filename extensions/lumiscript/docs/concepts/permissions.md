# Permissions

What does and doesn't gate the `api.*` surface, why `allowDangerous` confuses everyone the first time, and how to write scripts that fail gracefully when a permission isn't granted.

## The core idea: permissions are extension-level, not per-script

Permissions in LumiScript come from **one** place: the `permissions` array in LumiScript's `spindle.json` manifest. The user grants them all at once when they enable the LumiScript extension, the same way they'd grant permissions to any other Lumiverse extension. Once granted, **every script inside LumiScript shares the same grant set** — there's no per-script permission declaration, no way for one script to opt out, no way for another to ask for more.

If you've used a similar scripting system that put permissions in the script source (SillyTavern's `@permissions` header is the canonical example), unlearn it. LumiScript doesn't work that way and the editor doesn't read script-header permission directives. What's declared in `spindle.json` is the floor and the ceiling.

Two consequences worth keeping in front of you:

- **You — the script author — are usually also the user.** That means *you* are the one who said "yes" to all the permissions when you installed LumiScript. If you ever need to revoke or re-grant, that's done from Lumiverse's Extensions panel, not from the script editor.
- **Permissions gate `api.*` method calls, not the trigger system.** Reading `data.message.content` from a `MESSAGE_SENT` body does NOT require `chat_mutation` — the host already routed the event payload to your script for free. The permission kicks in only when your script reaches back through the API (`api.chat.getMessages`, `api.chat.editMessage`, etc.).

## The two gates

LumiScript surfaces are protected by up to two gates:

1. **The Spindle permission** — granted at extension-enable time, in the user's Extensions panel. Examples: `chat_mutation` for `api.chat.*` writes, `app_manipulation` for `api.ui.dom.*` and `api.theme.*`, `image_gen` for `api.imageGen.*`. This is the gate scripts share.
2. **`allowDangerous`** — a per-script UI toggle that lives in the LumiScript editor's script-list row (look for the "dangerous" badge next to enabled / disabled). It's NOT a Spindle permission and it's NOT declared in `spindle.json`. The script author sets it themselves, per script, in the editor.

Most surfaces use just one gate. Some surfaces use both — the in-app Reference's **Permission Matrix** annotates these with a muted `+ allowDangerous` note next to the permission chip, meaning both gates must be on for the method to work. The canonical example is outbound HTTP: `api.utils.http.*` requires `cors_proxy` (granted at the extension level) **AND** `allowDangerous` (toggled per-script).

## What `allowDangerous` does — and doesn't — cover

This is the single most common point of confusion in LumiScript, so it's worth being precise. `allowDangerous` gates a **fixed set of four surfaces** and nothing else:

- `api.utils.http.*` — outbound HTTP via the host's CORS proxy.
- `api.enclave.*` — AES-256-GCM encrypted, per-user-per-extension secret storage.
- `api.files.*` — file I/O (per-user / shared / temp tiers).
- `api.chat.clearAllInjections` — the nuclear-option chat-injection wipe.

Everything else routes through its own Spindle permission and does NOT need `allowDangerous`:

| Surface | Gated by |
|---|---|
| DOM injection (`api.ui.dom.inject`, `addStyle`, `delegate`, etc.) | `app_manipulation` only |
| Theme manipulation (`api.theme.apply`, `applyPalette`, etc.) | `app_manipulation` only |
| Advanced modals (`api.ui.showAdvancedModal`) | `app_manipulation` only |
| Image upload / fetch (`api.images.*`) | `images` only |
| Image generation (`api.imageGen.*`) | `image_gen` only |
| Character avatar upload (`api.characters.setAvatar`) | `characters` only |
| Character mutations (`api.characters.*`) | `characters` only |
| OAuth callbacks (`api.oauth.*`) | `oauth` only |
| Chat injection (`api.chat.inject`) | `interceptor` only |
| Databanks (`api.databanks.*`) | `databanks` only |
| Presets (`api.presets.*`) | `presets` only |
| World books (`api.worldInfo.*`) | `world_books` only |

The mental model that works: `allowDangerous` means *"this script is trusted to reach the outside world or local disk"* — network, filesystem, encrypted secrets. It is **not** a master unlock. If you find yourself reaching for `allowDangerous` to "unlock" a DOM / theme / image / character / OAuth surface, stop — that surface is gated by its own permission instead.

## There's no `// @permissions` directive

LumiScript doesn't parse `@permissions` or `@permission` script-header directives. Writing one is a no-op — it looks like it grants permissions but does nothing, and your script will then fail at runtime when it calls a gated method.

```js
// @permissions chat_mutation, generation    ← THIS IS A LIE

// The above comment is a documentary placeholder at best. The actual
// permissions came from the LumiScript spindle.json manifest, granted
// once at extension-enable time. This script can do whatever the
// extension grant set allows; this comment changes nothing.
```

If you're tempted to write one as documentation, consider an honest variant — `// Requires: chat_mutation, generation` — so readers know what the script *expects* without implying any runtime effect.

## Handling permission denial gracefully

There are two complementary approaches.

**Proactive — check before you call (preferred).** As of **v2.0**, `api.permissions` gives scripts introspection into what's granted, with no permission of its own required:

```js
// Is a specific permission granted?
if (await api.permissions.has('images')) {
  await api.images.upload(bytes);
}

// …or read the whole granted set:
const granted = await api.permissions.getGranted(); // e.g. ['chat_mutation', 'generation', …]
```

`script.allowDangerous` is *not* exposed to user code, so the surfaces it gates (outbound HTTP, encrypted enclave, file I/O) can't be preflighted this way — probe those reactively.

**Reactive — call and catch.** Call the gated method, catch the thrown error, and branch on it. The error messages are stable enough to match by prefix:

- A missing Spindle permission throws an `Error` whose `.message` starts with `PERMISSION_DENIED:<permission-name>`. Most denials use the standard format `PERMISSION_DENIED:<name> — grant this permission to use this API` and emit a `[LumiScript] ...` warning to the backend log before throwing. One known exception: `cors_proxy` throws `PERMISSION_DENIED:cors_proxy — grant this permission to use api.utils.http` with no backend warn (the check lives outside the shared `assertPerm` helper). Match by prefix (`startsWith('PERMISSION_DENIED:<name>')`), not by the full message, to stay forward-compatible.
- A missing `allowDangerous` throws an `Error` whose `.message` contains the literal phrase `must have "Allow Dangerous" enabled` (formatted with the script's name). This path also emits a `[LumiScript] ...` warning to the backend log.

The basic pattern:

```js
try {
  await api.chat.editMessage(messageId, { content: newContent });
} catch (err) {
  if (err.message.startsWith('PERMISSION_DENIED:chat_mutation')) {
    console.warn('[my-script] chat_mutation not granted; skipping edit');
    return;
  }
  throw err; // re-throw anything else — don't swallow unrelated errors
}
```

For surfaces gated by both a permission AND `allowDangerous` (HTTP, enclave, files), match either error shape:

```js
try {
  const resp = await api.utils.http.get('https://example.com/data.json');
  return resp.body;
} catch (err) {
  if (err.message.startsWith('PERMISSION_DENIED:cors_proxy')
      || err.message.includes('must have "Allow Dangerous" enabled')) {
    console.warn('[my-script] http unavailable; falling back to cached data');
    return await api.scriptStorage.get('cached:data', null);
  }
  throw err;
}
```

For long-lived handlers — broadcast subscribers, tool callbacks, registered macros — the same pattern matters more, because an uncaught denial in a handler surfaces as an unhandled rejection in the editor console every time the handler fires:

```js
api.broadcast.on('something:happened', async (payload) => {
  try {
    await api.images.upload({ data: payload.bytes, mimeType: 'image/png' });
  } catch (err) {
    if (err.message.startsWith('PERMISSION_DENIED:images')) {
      console.warn('[my-script] images not granted; ignoring upload request');
      return;
    }
    throw err;
  }
});
```

A pragmatic note: in personal-use deployments where the script author is also the user, denial handling is mostly defensive — if a method denies, you go grant the permission and move on. The patterns above matter more for scripts you intend to share or ship as packs, where the recipient's grant set may differ from yours.

## Permission quick reference

The in-app Reference's **Permission Matrix** is the authoritative method-to-permission map — open it when you're not sure which permission a given `api.foo.bar()` call needs. For offline use, here's the plain-English description of each permission LumiScript declares:

- **`chat_mutation`** — read / send / edit / delete chat messages. Required for most `api.chat.*` operations.
- **`chats`** — chat session metadata + CRUD on the chat list. Distinct from message content (`chat_mutation`).
- **`characters`** — CRUD on characters via `api.characters.*`, including `setAvatar`.
- **`personas`** — CRUD on personas via `api.personas.*`.
- **`presets`** — CRUD on generation presets + their prompt blocks via `api.presets.*`.
- **`world_books`** — CRUD on world books and entries via `api.worldInfo.*`.
- **`regex_scripts`** — CRUD on regex find/replace scripts.
- **`generation`** — call LLM providers via `api.llm.*`. Also required to register world-info interceptors.
- **`interceptor`** — register prompt injections via `api.chat.inject` that modify the assembled prompt mid-flight. (Content processors and world-info interceptors have their own gates — see `macro_interceptor` / `generation`.)
- **`context_handler`** — host-side plumbing for the context-handler stage that `api.chat.inject(..., { mode: 'context' })` injections flow through. LumiScript declares this in `spindle.json` alongside `interceptor`, but it isn't checked at the LumiScript level — only `interceptor` is enforced in LumiScript's `api.chat.inject` code path. Listed here for completeness because users see both on the extension grant prompt.
- **`macro_interceptor`** — register macro-resolution interceptors. Gated separately from `interceptor` because it's performance-sensitive.
- **`cors_proxy`** — outbound HTTP via `api.utils.http.*`. Paired with `allowDangerous` (both required).
- **`ui_panels`** — float widgets / dock panels — surfaces that hold their own persistent UI region in the app shell.
- **`app_manipulation`** — DOM injection (`api.ui.dom.*`), advanced modals (`api.ui.showAdvancedModal`), theme manipulation (`api.theme.*`). The "script-owns-its-own-shell-pixels" gate. Notably does NOT gate `api.ui.showContextMenu` (system-themed, free-tier) or simpler UI primitives (`toast`, `confirm`, `prompt`, `showModal`).
- **`push_notification`** — OS-level push notifications via `api.ui.pushNotification`.
- **`ephemeral_storage`** — TTL-bound `api.files.temp*` storage with auto-expiry.
- **`tools`** — register Council-eligible LLM tools via `api.tools.*`.
- **`event_tracking`** — record + query persistent events via `api.events.*` (the recording API, not the trigger system).
- **`databanks`** — CRUD on databanks + their documents via `api.databanks.*`.
- **`images`** — persist + retrieve images in Lumiverse's image store via `api.images.*`.
- **`image_gen`** — generate images via `api.imageGen.*` against the user's configured connection profiles.
- **`oauth`** — register an OAuth callback handler via `api.oauth.*`. The only inbound-HTTP hook Spindle exposes.
- **`web_search`** — run web searches via `api.webSearch.*`.
- **`memories`** — Memory Cortex + chat-memory access via `api.memories.*`.

## What's next

1. **[Storage model](storage-model.md)** — picking between `api.variables.*`, `api.db.*`, `api.scriptStorage`, `api.enclave.*`, and `api.files.*`. Several of these are permission-gated; this doc and that one cross-reference each other.
2. **[Handler lifetime](handler-lifetime.md)** — the closure-persistence story for `broadcast.on`, `commands.onInvoked`, `macros.register`, `tools.register`. The permission-denial-in-a-long-lived-handler pattern from above is the practical motivator.
3. **The in-app Reference's Permission Matrix** — the canonical method-to-permission table. Always current.
