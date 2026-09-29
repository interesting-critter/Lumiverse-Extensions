# Getting started

A 10-minute first script. By the end you'll have something running, you'll know where the editor + console live, and you'll have a working mental model for what "a LumiScript script" is.

## Prerequisites

- A running Lumiverse install with the LumiScript extension enabled. (Settings → Extensions; toggle it on if it isn't already.)
- At least one chat open. Some scripts fire only when a chat is active; for our first script we'll trigger on `MESSAGE_SENT`, which means we need a chat to send a message in.
- That's it. No CLI, no `npm install`, no build step.

## The script editor

Open the LumiScript dock panel (rightmost edge by default — configurable in settings). The panel has three tabs:

- **Manage** — script list and editor. This is where you'll spend 95% of your time.
- **Status** — runtime status: which scripts are running, their last execution result, any registered macros / tools / DOM injections.
- **Storage** — admin view of script-owned data (`api.variables`, `api.db` collections, `api.scriptStorage`).

Click **Manage**, then the **+** button to create a new script. You'll see two fields up top — a script **name** (whatever you want) and a **type** dropdown. Leave type as `trigger` for now (the other option, `library`, is for scripts that other scripts import — see the **Built-in Libraries** section in the in-app Reference for the pattern and built-in `ls:*` libraries). Below the name, there's an **events** multi-select — this is where you wire your script to Lumiverse events.

## Your first script

Pick the event **`MESSAGE_SENT`** from the events selector. This makes Lumiverse fire your script every time you (the user) send a message in the active chat.

In the script editor body, paste this:

```js
// First LumiScript script.
// Triggered: MESSAGE_SENT — fires whenever a message is created in the chat.

// MESSAGE_SENT fires for EVERY message the chat creates — including the
// assistant's empty placeholder. Skip non-user messages so we react only to
// what *you* send (without this guard the script logs twice, the second time
// with empty content).
if (!data.message.is_user) return;

console.log('Hello from LumiScript!');
console.log('You just sent:', data.message.content);
```

Save the script (Ctrl/Cmd+S, or the floppy icon). Make sure the script is **enabled** — there's a toggle in the script-list row.

Now go to your chat and send any message. Watch the **editor console** below the script body. You should see:

```
[10:23:45] LOG: Hello from LumiScript!
[10:23:45] LOG: You just sent: <whatever you typed>
```

If it didn't fire:

- The script needs to be **enabled** (toggle in the row).
- The events selector needs to include `MESSAGE_SENT`.
- The script needs to be **saved** (the editor shows an unsaved-changes indicator if not).
- You need to actually send a message in a chat. (`MESSAGE_SENT` fires whenever *any* message is created — the `is_user` guard above keeps us to your sends; the assistant's finished reply arrives on `GENERATION_ENDED`.)

## What just happened

Three things to internalise from this:

**1. The script body IS the handler.** There's no `api.on('MESSAGE_SENT', handler)` or `addEventListener` call. When the wired event fires, Lumiverse wraps your script body in a fresh `AsyncFunction` and runs it top-to-bottom. The wiring lives in the editor UI (the events multi-select you clicked), not in the script source. (That's the default AsyncFunction engine; the opt-in QuickJS isolate runs a fresh invocation per fire too — the fresh-scope lesson below holds either way. See [Execution engine](concepts/engine.md).)

**2. The `data` global carries the event payload.** When `MESSAGE_SENT` fires, `data` looks roughly like:

```js
{
  __event:  'MESSAGE_SENT',
  chatId:   '<chat UUID>',
  message:  { id, content, is_user, name, send_date, ... },
  // a couple more fields per event type
}
```

Each event has its own payload shape. The full per-event shape is in the in-app **Reference** tab under "Lumiverse Events".

**3. `api.*` is your gateway to everything else.** We didn't use it in this first script, but the `api` global gives you `api.chat.*` (read / write messages), `api.llm.*` (call the LLM), `api.variables.*` (persistent storage), `api.ui.*` (toasts, modals, UI surfaces), and 15+ more namespaces. Coming up next.

## Adding a UI affordance

Let's make the script do something visible outside the editor console. Replace the body with:

```js
const text = data.message.content;
const words = text.trim().split(/\s+/).filter(Boolean).length;

api.ui.toast(`You sent ${words} word(s)`, 'info');
```

Save, then send a message. A small toast pops up in the corner of the app with the word count. `api.ui.toast(message, level?)` is free-tier — no permission needed. Levels are `'info'`, `'success'`, `'warning'`, `'error'`.

A handful of other `api.ui.*` primitives that need no permission:

- `api.ui.prompt(message, defaultValue?, options?)` — themed text input. Returns `Promise<string | null>` (the entered text, or `null` on cancel).
- `api.ui.confirm(message, title?, options?)` — yes/no dialog. Returns `Promise<boolean>`.
- `api.ui.showModal(items, options)` — multi-section modal (`options` is required and must carry a `title`); returns a `ModalHandle` (await `handle.result`).

For DOM injection, theming, drawer tabs, float widgets, and other host-shell-modifying surfaces, your script needs the `app_manipulation` permission (covered in `concepts/permissions.md`).

## Two things to know early

These bite first-timers. Worth knowing up front:

**Every fire is a fresh function scope.**

```js
let counter = 0;
counter += 1;
console.log(counter);
```

This logs `1`, every time. The `let counter` declaration is local to the AsyncFunction wrapping the body — it doesn't survive to the next fire. If you want state that persists across fires, you have several storage tiers (`globalThis`, variables, db, scriptStorage, enclave, files) — covered in [`concepts/storage-model.md`](concepts/storage-model.md). For a quick in-memory hack:

```js
globalThis.myCounter ??= 0;
globalThis.myCounter += 1;
console.log(globalThis.myCounter);
```

`globalThis.*` persists for the lifetime of the script-runner subprocess (i.e. until the extension reloads). For durable state, use `api.variables.global` / `.local` / `.character` / `.chat`.

**A few things that look like normal JavaScript don't work in scripts.**

The sandbox rejects these patterns at dispatch — your script won't run at all, and you'll see a `[security]` entry in the editor console:

- `import('module-name')` — use `script.require('library-name')` for inter-script dependencies.
- `require('module-name')` — same; bare `require()` is rejected, `script.require(...)` is the LumiScript API.
- `new Function('code')` / `Function('code')` — define functions normally; the constructor is blocked.
- `.constructor.constructor` — the prototype-chain backdoor to the Function constructor, blocked for the same reason.
- `globalThis.Bun.X` / `globalThis.process.X` — the host runtime API isn't exposed to scripts.

Don't worry about memorising these — when you hit one the error message tells you the replacement. The full list and rationale is in the in-app Reference under **Sandbox hardening** (with a narrative summary in [`concepts/trigger-model.md`](concepts/trigger-model.md#the-sandbox-boundary)).

## What's next

You have enough to write something useful. Suggested reading order from here:

1. **[Trigger model](concepts/trigger-model.md)** — A deeper look at how events, the `data` global, `ls:startup` / `ls:teardown`, and `@ls:reload-on-edit` work together. The 5-minute deep dive on the mental model.
2. **[Permissions](concepts/permissions.md)** — Which methods need which permissions, what `allowDangerous` is, and how to handle permission denial gracefully.
3. **[Storage model](concepts/storage-model.md)** — Picking the right tier from variables / db / scriptStorage / enclave / files.
4. **[DOM injection](guides/dom-injection.md)** — Building UI inside the host app shell. The single highest-leverage capability if you want LumiScript to feel like a first-class part of Lumiverse.
5. **[Calling the LLM](guides/llm.md)** — `api.llm.*` — text, validated structured output, tool-using generation, dry-runs, cancellation. Almost every non-trivial script reaches for this.
6. **[Custom macros](guides/macros.md)** — `api.macros.*` — push vs pull mode, the `{{name}}` integration point for injecting computed values into prompts, plus interceptors for template-level rewrites.
7. **[Registering tools](guides/tools.md)** — `api.tools.*` — publishing tools the LLM can invoke (the inversion of `generateWithTools`). Council integration, `ls:council-prompt` patterns, cross-script invocation.
8. **The in-app Reference tab** — for everything else. Open it once and skim the structure so you know where things live.

If something doesn't make sense or you hit an edge case these docs don't cover, that's a doc gap — flag it (we're co-writing these, missing-coverage flags are welcome).
