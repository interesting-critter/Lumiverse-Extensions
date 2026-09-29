# Reactive UI state

`api.ui.events.*` lets a script read and react to Lumiverse's live UI state — the virtual keyboard, the side drawer, and the settings modal. Each surface has a **snapshot getter** (the state right now) and a **change subscription** (fires on every change). Free tier — no permission required.

The headline use case is **mobile-safe positioning**: nudge a float widget or injected DOM up when the on-screen keyboard opens, using the keyboard's `insetBottom`.

## Snapshots

```js
const kb = await api.ui.events.getKeyboardState();   // { visible, insetBottom, viewportWidth, viewportHeight }
const dr = await api.ui.events.getDrawerState();      // { open, tabId }
const st = await api.ui.events.getSettingsState();    // { open, view }
```

The getters are **async** — they cross the worker boundary like `DOMHandle.read()` — but they resolve from a cache the frontend keeps fresh, so there's no per-call round-trip. Before the frontend has reported in (the first moments after connect), they resolve with sensible "off" defaults (`visible: false`, `open: false`, etc.).

## Subscriptions

Each `on*Change(handler)` fires with the new state on every change and returns an unsubscribe function:

```js
const unsub = api.ui.events.onKeyboardChange((state) => {
  if (state.visible) console.log(`keyboard up, safe inset ${state.insetBottom}px`);
});

// later, when you're done:
unsub();
```

The state shapes match the snapshots:

- **Keyboard** `{ visible, insetBottom, viewportWidth, viewportHeight }` — `insetBottom` is the CSS-pixel safe-area to keep content above the keyboard.
- **Drawer** `{ open, tabId }` — `tabId` is the active drawer tab (or `null`).
- **Settings** `{ open, view }` — `view` is the active settings view id.

## Worked example: keep a float widget above the keyboard

```js
const widget = api.ui.createFloatWidget({ /* … */ });   // needs ui_panels

api.ui.events.onKeyboardChange((kb) => {
  // Lift the widget by the keyboard's safe inset when it opens; drop it back when it closes.
  const pos = widget.getPosition();
  widget.moveTo(pos.x, kb.visible ? pos.y - kb.insetBottom : pos.y);
});
```

## Worked example: react when the user opens your drawer tab

```js
api.ui.events.onDrawerChange(({ open, tabId }) => {
  if (open && tabId === 'my-script-tab') {
    refreshMyTabContents();   // lazy-load only when the user actually looks
  }
});
```

## Lifecycle

A live subscription **keeps the script pinned** — it won't be evicted from the worker pool while a handler is registered, so background subscriptions stay responsive. Two ways they end:

- **Call the returned unsub** when you no longer need the subscription — releases the pin.
- **Script disable / reload** tears down every subscription automatically (you don't have to track them for teardown).

So for a fire-and-forget background reaction you can subscribe and never unsubscribe; for a subscription tied to a transient UI (a modal you opened), unsub in the dismiss path to release the pin promptly.

## Notes

- **Free tier**, no user id — scoped to the active user automatically.
- **Keyboard events are mostly mobile / PWA.** On desktop the drawer and settings channels are the active ones; the keyboard channel typically stays `{ visible: false }`.
- Handlers fire in your script's sandbox — make them `async` and `await` everything inside (the host keeps the per-fire run alive across the await chain, same as component callbacks / DOM event handlers).
- See the in-app **Reference tab** for the `UIKeyboardState` / `UIDrawerState` / `UISettingsState` field lists.
