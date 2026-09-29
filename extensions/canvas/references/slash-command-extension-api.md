# Canvas Slash-Command Extension API

> **Audience:** Authors of Spindle extensions that want to contribute slash
> commands to Canvas's runtime. Chronicle uses this to register `/summarize`.

## Overview

Canvas ships a slash-command runtime that intercepts text in Lumiverse's main
chat input, parses `/command args`, and dispatches to registered handlers.
Any extension running in the same `window` context can register a command
by dispatching a `CustomEvent` from its `setup()` function.

This works without a Spindle API change — extensions are already loaded as
separate `<script type="module">`s in the same `window`, so `CustomEvent`
is the established inter-extension bus.

## Scope (v1.1.0)

The v1.1.0 runtime ships two built-in commands:

- `/help` — lists every registered command (name + usage).
- `/select` — supports `range` (`/select 5-10`, `/select 5,7,9`),
  `all`, and `clear` sub-forms.

The following `/select` sub-forms are **deferred to v1.2.0** and are not
implemented in the runtime today:

- `/select hide`
- `/select unhide`
- `/select delete`

Extension authors who register a command at one of these names should be
aware that Canvas's runtime will not implement the behavior — the handler
they supply is the one that runs.

## The contract

```typescript
interface SlashCommandDef {
  name: string                          // 'summarize', 'select', etc.
  description: string                   // shown in /help
  usage?: string                        // '/summarize <range>' — shown in /help
  argsSchema?: {                        // for suggestion preview (future)
    name: string
    type: 'string' | 'number' | 'enum'
    values?: string[]
    optional?: boolean
  }[]
  handler: (args: Record<string, string>, ctx: SlashContext) => Promise<void> | void
  owner: string                         // your extension identifier
  category?: 'select' | 'layout' | 'theme' | 'lore' | 'chat' | 'meta'
}

interface SlashContext {
  chatId: string
  setText: (text: string) => void        // write back to textarea (input event)
  toast: (kind: 'info'|'error'|'success', text: string) => void
}
```

## How to register

In your extension's `setup(ctx)`:

```typescript
export function setup(ctx: SpindleFrontendContext) {
  window.dispatchEvent(new CustomEvent('canvas:slash-register', {
    detail: {
      command: {
        name: 'summarize',
        description: 'Summarize the selected messages',
        usage: '/summarize [range]',
        owner: 'chronicle',
        category: 'lore',
        handler: async (args, slashCtx) => {
          // Your logic. Use slashCtx.toast for user feedback.
          // Use slashCtx.setText to write back to the input.
        },
      },
    },
  }))

  // Return your teardown
  return () => {
    // Unregister this extension's command on teardown. The runtime
    // implements `canvas:slash-unregister` and looks up the cleanup
    // function it captured at registration time.
    window.dispatchEvent(new CustomEvent('canvas:slash-unregister', {
      detail: { name: 'summarize' }
    }))
  }
}
```

## Lifecycle

- **Registration:** dispatch the event once during `setup()`. The runtime
  registers the command in its `CommandRegistry` and tracks the cleanup
  function returned by `registry.register()` in a `Map<name, () => void>`
  keyed by command name. Re-dispatching the same name replaces the prior
  registration — the prior cleanup is invoked first, so the old entry
  cannot leak in the Map.
- **Dispatch:** the runtime invokes your `handler(args, ctx)` when the user
  types `/summarize ...` and presses Enter or taps the send button.
- **Unregistration:** dispatch `canvas:slash-unregister` with
  `{ name: 'summarize' }` to remove a command mid-session. The runtime
  invokes the tracked cleanup, drops it from the `Map`, and the command
  no longer resolves. The runtime also auto-cleans the entire `Map` on its
  own teardown, so unloaded extensions leave no orphans.

## Pitfalls

1. **The `args` object is a flat `Record<string, string>`.** In v1.1.0 the
   runtime only ever passes `{ _raw: '<full args string>' }` — per-command
   schemas and named keys are not produced yet. The `argsSchema` field on
   `SlashCommandDef` is documentation-only for v1.1.0; per-command
   schemas land in v1.2.0. Until then, parse `args._raw` yourself in the
   handler (split on whitespace, handle quoted segments, etc.).
2. **`ctx.toast` is a CustomEvent dispatcher, not a Spindle API call.**
   It dispatches `canvas:slash-toast` and Canvas's runtime renders it.
   You don't need to do anything special.
3. **`ctx.setText(text)` triggers a controlled-component input event.**
   Use it to write back to the chat input. The runtime handles the rAF.
4. **Long-running handlers** (LLM calls, etc.) should `await` in the
   handler. The runtime dispatches in a microtask; your handler can take
   as long as it needs. Toasts for "in progress" + "done" are the user's
   feedback.
5. **`SlashContext` does not include `userId` in v1.1.0.** If your handler
   needs the current user, fall back to a Spindle API call from your
   extension's `ctx` (the `SpindleFrontendContext` you receive in
   `setup()`) — that one is closed over your extension and has access
   to whatever Spindle exposes. Do not assume `userId` will be there
   on the slash context; it is not, and v1.2.0 may or may not add it.
6. **Registration events fired before Canvas's runtime attaches are
   dropped.** The runtime only listens for `canvas:slash-register` while
   it is attached. If your extension's `setup()` runs before Canvas's,
   dispatch again after `DOMContentLoaded` (or after a short delay) to
   be safe. In practice all extensions load in the same `<script>` burst
   on the page, but ordering is not guaranteed across reloads.
