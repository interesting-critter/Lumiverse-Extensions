<!-- Mirror of upstream documentation. Everything below the metadata table is the original file, unmodified. -->

| Field | Value |
| --- | --- |
| Extension | LumiScript |
| Source repository | https://github.com/mindbound/lumiscript |
| Original link (from Lumiverse-Extensions README) | https://github.com/mindbound/lumiscript |
| Upstream path | `README.md` |
| Retrieved | 2026-09-29 @ `master` (`66ac526`) |

---

![LumiScript](https://raw.githubusercontent.com/mindbound/mindbound.github.io/refs/heads/master/ls/lumiscript.png)

# LumiScript

A JavaScript scripting platform for [Lumiverse](https://github.com/prolix-oc/Lumiverse). Write scripts that react to chat events, automate behaviour, inject prompt context, and build interactive UIs — without touching Lumiverse's core code.

## Install

In Lumiverse, open **Settings → Extensions** and install from URL:

```
https://github.com/mindbound/lumiscript
```

Grant the requested Spindle permissions when prompted. The exact set depends on which APIs your scripts use — granting all of them up front is fine and easy to tighten later.

## Hello, world

Open the **LumiScript** panel from Lumiverse's dock, click **+ New Script**, and paste this in:

```javascript
// Wire the script to the MESSAGE_SENT trigger in the editor.

const messages = await api.chat.getMessages();
const latest   = messages[messages.length - 1];

console.log(`You said: ${latest.content}`);
api.ui.toast(`Got a message — ${messages.length} total in this chat`, 'success');
```

Save, send a message in any chat, and watch the toast appear. Console output shows up in the script's row in the LumiScript panel.

## What you can do

LumiScript exposes a broad surface area through the `api.*` object available to every script:

**Chat, characters, world info**
- Read, send, edit, and delete messages
- Inject content into prompt assembly with first-class visibility in the Prompt Breakdown
- Full CRUD on characters, chats, personas, world books, databanks, presets, and regex scripts
- Resolve macros (`{{user}}`, `{{char}}`, custom) with or without committing side effects

**LLM and generation**
- Call any configured connection via `api.llm.generate` / `generateStructured` / `generateWithTools`, or stream tokens with `generateStream`
- Register tools usable by Lumiverse Council and inline function-calling
- Server-side token counting with the actual provider tokenizer

**UI and DOM**
- Toasts, prompts, confirms, modals, context menus
- Sanitised DOM injection with event handlers, draggable widgets, scoped CSS
- Drawer tabs, input-bar actions, floating widgets, edge-docked panels
- Theme manipulation (CSS variables, palette-driven theming, palette extraction from images)

**Storage**
- Four-tier variable scopes (local / global / character / chat)
- Per-script JSON micro-DB with filter predicates, jsonquery, and optional Zod schema validation
- Per-script in-memory session store with admin inspector
- AES-256-GCM encrypted secret storage, plus tiered file I/O (user / shared / ephemeral)

**Images, generation, OAuth**
- Image-store CRUD and image generation against configured providers (incl. img2img / inpainting)
- OAuth callback hook — the only inbound-HTTP surface — pair with HTTP and the encrypted enclave for full token flows

**Inter-script and inter-extension**
- Real-time pub/sub broadcast bus between scripts
- Cross-extension shared RPC pool via `api.rpc.*`

**Other**
- Long-term Memory Cortex + chat-memory access (`api.memories.*`) and web search (`api.webSearch.*`)
- Persistent event tracking with replay and latest-state lookup
- HTTP requests (text or binary) through Lumiverse's CORS proxy
- Handlebars templates, UUIDs, image MIME sniffing, base64 helpers

## Where to learn more

LumiScript ships with extensive documentation — both in the repo and in-app:

- **The [`docs/`](docs/index.md) tree** — hand-written concept docs, a per-API guide for every namespace, and an end-to-end **cookbook** of runnable recipes
- **In-app Reference** — the full API surface, method signatures, type definitions, event-payload shapes, and example snippets, always current with your installed version
- **Lisa** (Settings → Assistant) — in-app code assistant that knows the entire LumiScript API. Ask her to write a script, explain an error, or look up a method
- **Trigger model** concept doc — covers the script execution model in detail; worth reading before writing your first non-trivial script

## Sandbox model

Scripts execute server-side inside a supervised Bun subprocess. DOM access is proxied through messaging — scripts can manipulate the page but can't directly touch globals like `window` or `document`. Permissions are granted at the extension level when you install LumiScript; individual risky surfaces (outbound HTTP, encrypted secrets, file I/O) additionally require a per-script `allowDangerous` toggle. As of v2.0, outbound requests to **localhost and private-network** addresses are also blocked by default — if a script talks to a local model server (Ollama, ComfyUI, LM Studio) or a LAN device, allowlist that host under Settings → Network (see [`docs/guides/network.md`](docs/guides/network.md)).

A runaway user script can't take down Lumiverse. The supervisor SIGKILLs the script-runner subprocess via a heartbeat watchdog and respawns it automatically.

**Execution engine (new in v2.0).** By default, scripts run in `AsyncFunction` sandboxes — the long-term-support engine, unchanged from earlier versions. v2.0 adds an opt-in **QuickJS-WASM isolate** engine (choose it in the LumiScript settings): user code runs in a WebAssembly VM where host globals like `Bun` and `process` don't exist, with per-script memory and CPU limits, optional per-script context isolation, and graceful teardown. AsyncFunction remains the default and is fully supported — switch engines only when you want stronger structural isolation.

## Requirements

- Lumiverse `v1.0.4` or later

## Building from source

```bash
bun install
bun run build
```

Lumiverse must be restarted (or the extension toggled off/on) to pick up new builds.

## License

[MIT](LICENSE)

## Author

[mindbound](https://github.com/mindbound)
