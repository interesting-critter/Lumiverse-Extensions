# Build a draggable info panel

Pin a small floating panel over the chat that the user can drag around by its title bar, refresh, and close — the foundation for any always-visible HUD, status readout, or control surface your script owns.

## What you'll use

- [`api.ui.dom.*`](../guides/dom-injection.md) — `inject` to add the panel, the returned handle's `makeDraggable` / `on` / `update` / `remove`, and `cleanup` to reset on re-run. Needs the `app_manipulation` permission. ([Permissions](../concepts/permissions.md))
- `addStyle` for a scoped stylesheet.
- `ls:startup` as the trigger so the panel appears when the script is enabled. ([Trigger model](../concepts/trigger-model.md))

## The script

```js
// @triggers ls:startup
// A draggable clock panel pinned over the chat. Re-running replaces it.
// Permission: app_manipulation.

// Re-rendered on every refresh. The header carries the drag handle + close button;
// data-action attributes let one click handler route everything (see below).
function renderContent() {
  return `
    <div class="ls-panel-header">
      <span class="ls-panel-title">🕒 Clock</span>
      <span class="ls-panel-close" data-action="close" title="Close">×</span>
    </div>
    <div class="ls-panel-time">${new Date().toLocaleTimeString()}</div>
    <button class="ls-panel-btn" data-action="refresh">Refresh</button>
  `;
}

// Clear any previous instance of THIS script's DOM so a re-run (or hot reload)
// replaces the panel instead of stacking a second one.
api.ui.dom.cleanup();

api.ui.dom.addStyle(`
  #ls-clock {
    position: fixed; top: 80px; right: 24px; z-index: 9998;
    width: 180px; padding: 12px 14px;
    background: var(--lumiverse-fill, rgba(20, 20, 28, 0.96));
    color: var(--lumiverse-text, #e8e8ea);
    border: 1px solid var(--lumiverse-border, rgba(255, 255, 255, 0.12));
    border-radius: 10px; box-shadow: 0 12px 40px rgba(0, 0, 0, 0.5);
    font-family: system-ui, sans-serif; font-size: 0.85rem;
  }
  .ls-panel-header { display: flex; align-items: center; justify-content: space-between; cursor: move; user-select: none; }
  .ls-panel-title  { font-weight: 600; pointer-events: none; }
  .ls-panel-close  { cursor: pointer; padding: 0 6px; border-radius: 4px; }
  .ls-panel-close:hover { background: rgba(255, 255, 255, 0.12); }
  .ls-panel-time   { margin: 10px 0; font-size: 1.4rem; font-variant-numeric: tabular-nums; }
  .ls-panel-btn    { font: inherit; cursor: pointer; padding: 3px 10px; border-radius: 5px;
                     border: 1px solid var(--lumiverse-border, rgba(255,255,255,0.15));
                     background: transparent; color: inherit; }
`);

// 1. The floating shell — a fixed-position container, draggable later.
const shell = api.ui.dom.inject('body', `<div id="ls-clock"></div>`, { id: 'clock-shell' });

// 2. Its contents, with their own handle so we can re-render in place via update().
const content = api.ui.dom.inject('#ls-clock', renderContent(), { id: 'clock-content' });

// 3. Drag the whole panel by its header bar.
shell.makeDraggable('.ls-panel-header');

// 4. One click handler routes by data-action (the events bubble up to the shell).
shell.on('click', (e) => {
  const action = e.dataset?.action;
  if (action === 'close') {
    // Symmetric teardown: remove the DOM (and its listeners) we created.
    content.remove();
    shell.remove();
    return;
  }
  if (action === 'refresh') {
    content.update(renderContent());
  }
});
```

Enable the script: a clock panel appears top-right. Drag it by the title bar, hit **Refresh** to re-read the time, **×** to dismiss.

## How it works

**Shell + content, two handles.** The `shell` is a fixed-position empty container injected into `body`; the `content` is injected *into* the shell (`'#ls-clock'`) and keeps its own handle. That split is the load-bearing idea: the shell stays put as the draggable frame while `content.update(html)` swaps the inner markup in place — so a refresh re-renders the body without disturbing the panel's position or the drag binding.

**`makeDraggable(selector)`** turns the handle's element into a drag target grabbed by the child matching `selector` — here `.ls-panel-header`, so the user drags from the title bar, not the whole panel (which would hijack text selection and button clicks). The selector is resolved against the injected subtree.

**One handler, `data-action` routing.** `shell.on('click', …)` catches clicks bubbling up from anywhere inside the panel. The callback reads `e.dataset?.action` — the `data-action` of the **element that was clicked** — and routes on it. Adding a new button is just a new `data-action` value + a branch; you never wire per-element listeners.

**`cleanup()` makes re-runs idempotent.** `api.ui.dom.cleanup()` removes everything this script previously injected. Calling it at the top means enabling the script twice, or hot-reloading it (`// @ls:reload-on-edit`), replaces the panel instead of stacking duplicates. On disable/delete the host clears your DOM for you; the explicit `content.remove()` + `shell.remove()` in the close path is the *user-initiated* teardown.

## Make it yours

- **Show live data.** Swap `renderContent()` to read real state — `await api.chat.getMessages()` for a message count (needs `chat_mutation`), `api.variables.*`, `api.scriptStorage.*`, etc. Keep `renderContent` pure (returns a string) and call `content.update(renderContent())` whenever the data changes.
- **React to events.** Drop the `ls:startup`-only trigger and also wire `MESSAGE_SENT` / `GENERATION_ENDED`; in the body, `if (data.__event === 'MESSAGE_SENT') content.update(renderContent())` to refresh on each turn. (Inject the panel only once — guard on whether `#ls-clock` already exists, or keep the `cleanup()` + re-inject approach.)
- **Update from another script.** Have a sibling script `api.broadcast.emit('myapp:state-changed', …)` and subscribe here to refresh — see [Share state between two scripts](share-state-broadcast.md). If you do, **capture the unsubscribe** `on()` returns and call it in the close branch, or the subscription (and the worker pinned by it) outlives the panel.
- **Persist position.** `makeDraggable` doesn't remember where the user left it; store the final coordinates in `api.scriptStorage` from a drag-end handler and apply them as inline `top`/`left` on next inject.

## Gotchas

- **Put `data-action` on the element the handler reads — and `pointer-events: none` on inner icons.** `handle.on` reads the `dataset` of the *clicked* element, with no `closest()` walk up the tree. A text `×` works (the click lands on the element carrying `data-action`). But if your button wraps an `<svg>`/`<img>`/`<span>` icon, the click can land on that inner node — which has no `data-action` — and the route misses. Give inner icon elements `pointer-events: none` (as `.ls-panel-title` has above) so the click registers on the parent.
- **Drag from a handle, not the whole panel.** `makeDraggable('#ls-clock')` (the whole panel) would make buttons and text-selection fight the drag. Scope the handle to the title bar.
- **Re-running stacks panels without `cleanup()`.** The very first line of the bring-up is `api.ui.dom.cleanup()` for exactly this reason — without it, every enable/reload leaves the old panel behind.
- **`position: fixed` + a high `z-index`** keep the panel above the chat. Anchor to a viewport corner; the host chat UI owns the normal flow.

## See also

- [DOM injection](../guides/dom-injection.md) — the full `api.ui.dom` surface: `inject`, `addStyle`, the handle methods (`update` / `remove` / `on` / `makeDraggable` / `read`), and `cleanup`.
- [Handler lifetime](../concepts/handler-lifetime.md) — why DOM listeners and broadcast subscriptions need explicit teardown, and what the host cleans up for you.
- [Share state between two scripts](share-state-broadcast.md) — feed the panel from a separate worker script.
