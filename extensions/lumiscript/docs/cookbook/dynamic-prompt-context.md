# Inject dynamic prompt context per turn

Compute something fresh each turn — the time of day, how deep the scene is, a running state value — and slip it into the prompt the model sees, without touching the visible chat. This is the **script-owned-timing** channel for feeding the model context: *you* decide what to inject and exactly when.

## What you'll use

- `api.chat.inject(id, content, options?)` — register a prompt injection under a stable `id`. Re-injecting with the same `id` **overwrites** the previous content — that's what keeps it fresh. Requires the `interceptor` permission (declared in LumiScript's manifest by default). ([Permissions](../concepts/permissions.md))
- `MESSAGE_SENT` as the trigger — it fires when a message is created (guard on `data.message.is_user` for *your* sends), a moment *before* the prompt for the reply is assembled. ([Trigger model](../concepts/trigger-model.md))
- `globalThis` — a synchronous per-process counter, here only to make "this changes every turn" visible. ([Storage model](../concepts/storage-model.md))

The idea up front: **an injection is keyed by its `id` and lives until you remove it.** "Dynamic per turn" isn't a special mode — it's calling `inject` again each turn with the same `id` and freshly-computed content. The newest call wins, so the model always reads current values.

## The script

```js
// @triggers MESSAGE_SENT
// Inject a fresh "scene context" note before each model reply.
// Permission: interceptor.
//
// MESSAGE_SENT fires for every message created — including the assistant's
// empty placeholder — so guard on is_user to run only on YOUR sends (otherwise
// the counter below climbs twice per turn). On a user send it fires a moment
// before the prompt for the reply is assembled, so build the note and inject it
// SYNCHRONOUSLY (no awaits before the inject) so it lands in THIS turn's
// generation — see "How it works".
if (!data.message.is_user) return;

// 1. Compute the note synchronously — values that change from turn to turn.
const now  = new Date();
const hour = now.getHours();
const band =
  hour < 6  ? 'the dead of night' :
  hour < 12 ? 'morning'           :
  hour < 18 ? 'afternoon'         :
              'evening';

// A per-process exchange counter — synchronous, so it never delays the inject.
// (Resets when the script-runner restarts; fine for a scene counter.)
globalThis.lsSceneContext_turn = (globalThis.lsSceneContext_turn ?? 0) + 1;

// 2. Inject under a STABLE id, so each turn OVERWRITES the previous note
//    instead of stacking a new one.
const note =
  `Out-of-character context (do not quote verbatim): it is currently ${band} ` +
  `(${now.toTimeString().slice(0, 5)} server time), exchange #${globalThis.lsSceneContext_turn} this session.`;

api.chat.inject('scene-context', note, { role: 'system', depth: 0 });
```

Enable it, open a chat, and send a few messages. The note never appears in the transcript, but the model reads a live version every turn — the time advances and the counter climbs. Peek at it from any other script (or a dry run) with `api.chat.getInjections()`: the `scene-context` entry's content changes on each send.

## How it works

**Re-inject to refresh; the `id` is the identity.** `inject('scene-context', …)` registers — or replaces — one injection slot named `scene-context`. There is no separate "update" call; overwriting *is* the update. Because the body re-runs on every send, the slot always holds this turn's note. Use a random `id` each time instead and you'd stack a fresh note every turn rather than replacing the old one.

**Why `MESSAGE_SENT`, and why inject synchronously.** On a user send, `MESSAGE_SENT` fires *before* the prompt for the reply is assembled — so an injection registered in its handler is read by the very next generation. The margin is thin: the host emits the event a microtask ahead of assembly, so a **synchronous** inject (like this one) lands in time, but `await`-ing slow work first can slip you past assembly and into the *following* turn (see the gotcha). `MESSAGE_SENT` also fires for the assistant's empty placeholder, so the `is_user` guard keeps this body to user turns — exactly when fresh context is wanted; the finished reply arrives on the separate `GENERATION_ENDED` event.

**`mode`, `role`, and `depth`.** The default `mode: 'intercept'` splices your content into the fully-assembled message array right before it reaches the model — reliable, no host cooperation required. (There's also `mode: 'context'`, which enriches the assembler *before* assembly, but whether it's consumed depends on host support — prefer `intercept`.) `role` is the message role the content rides as — `'system'` by default, which is right for out-of-character notes. `depth` counts from the **end** of the array: `0` appends after everything (most recent, most salient), `1` inserts before the last message, and so on. End-position is usually what you want for "right now" notes — it sits closest to the generation.

**It never touches the transcript.** The injected note shapes the prompt but is not a chat message — it won't show in history, won't persist to the chat, and won't survive into the next assembly unless you re-inject. That's the whole appeal over `api.chat.sendMessage`: context for the model's eyes only.

## Make it yours

- **One-shot injections.** Pass `{ ephemeral: true }` and the injection auto-removes after the next generation — present for exactly one reply, then gone, no manual `removeInjection`. Reach for it when "this turn only" is the literal requirement.
- **Inject real signal, not the clock.** Swap the time-of-day stub for anything you can read: a relationship score from `api.variables.character`, the last dice outcome, an active scene flag. If the read is `await`ed, mind the timing gotcha below — compute it ahead of the inject, or accept a one-turn lag.
- **Summarize recent history.** Pull the last few messages with `api.chat.getMessages({ last: 6 })` (needs `chat_mutation`), condense them — heuristically or via `api.llm.generateStructured` — and inject the recap. This is the awaited-work case: keep a stable `id` so the model always has *something*, even if a turn stale.
- **Pick a different channel.** `inject` is the right tool when *your script* owns the timing. If instead you want the user to place the value wherever they like in their preset, expose it as a `{{macro}}` ([Custom macros](../guides/macros.md)); if you want it keyword-activated from a world book, drive a dynamic entry ([World info](../guides/world-info.md)). Same goal — script-computed context — three different control models.

## Gotchas

- **Awaited work can miss the current turn.** `intercept` mode reads whatever is in the slot *at assembly time*, and assembly runs barely a tick after `MESSAGE_SENT`. A synchronous compute (like the clock above) lands well before it. But if you `await` something slow before injecting — an LLM call, a network fetch, even a storage read — generation may already be assembling, and your content lands in the *next* turn's prompt instead. Keep the inject synchronous; or keep the slot populated under a stable `id` and accept a one-turn lag (often fine); or move the expensive work to the prior turn's `GENERATION_ENDED`.
- **`MESSAGE_SENT` fires for every message creation, not just yours.** The host emits it for the assistant's empty placeholder (and greetings, manual inserts) too — guard on `data.message.is_user` (as the script does) to react to user sends only. If you want to react *after* the model speaks rather than before, wire to `GENERATION_ENDED` instead; its payload is `{ generationId, chatId, messageId, content }`, with no `message` object.
- **`interceptor`, not `allowDangerous`.** `api.chat.inject` is gated by the `interceptor` permission and nothing else. The one member of this family that additionally needs `allowDangerous` is `clearAllInjections`, which wipes *every* script's injections — `clearInjections()` (just yours) does not.
- **`depth` counts from the end, not the start.** `depth: 0` is the end of the array (after the latest message), not the beginning. If a note is landing somewhere surprising, you've probably got the direction inverted.

## See also

- [Trigger model](../concepts/trigger-model.md) — `MESSAGE_SENT` (fires on every message creation — guard `is_user`; pre-assembly) vs `GENERATION_ENDED` (generated assistant text), and how to read each one's payload.
- [Permissions](../concepts/permissions.md) — what `interceptor` gates and how denial degrades.
- [Custom macros](../guides/macros.md) and [World info](../guides/world-info.md) — the other two channels for getting script-computed content into a prompt.
- [Storage model](../concepts/storage-model.md) — where the state you inject should actually live between turns.
