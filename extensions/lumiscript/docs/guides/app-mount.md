# Mounting a full-bleed app surface

`api.ui.mountApp()` mounts a **route-persistent, full-bleed portal** into the Lumiverse app shell — a script-owned region for full-screen overlays or persistent chrome that goes beyond what dock panels, drawer tabs, or float widgets offer. Requires the `app_manipulation` permission.

```js
const mount = api.ui.mountApp({ position: 'app-overlay' });
mount.root.update('<div class="my-overlay"><h1>Focus mode</h1></div>');
// …later
mount.destroy();
```

## Where it fits

LumiScript gives you a ladder of script-owned UI regions, in increasing scope:

| Surface | Use it for |
|---|---|
| `showAdvancedModal` | a transient modal dialog |
| `createFloatWidget` | a small draggable overlay (`ui_panels`) |
| `registerDrawerTab` | a tab in the sidebar drawer |
| **`mountApp`** | a full-bleed, route-persistent portal — full-screen overlays / persistent chrome |

Reach for `mountApp` when none of the bounded surfaces fit — e.g. a full-screen reading mode, a custom landing surface, or chrome that should persist across route changes.

## The handle

```ts
mountApp(options?: {
  className?: string;
  position?: 'start' | 'end' | 'app-overlay';  // before / after the main view, or covering it
}): MountedAppHandle
```

`MountedAppHandle` is `{ mountId, root, setVisible(visible), destroy() }`:

- **`root`** — a `DOMHandle` bound to the mount's content container. Render and wire it with the **same `api.ui.dom.*` pipeline** you'd use for any injected element: `root.update(html)`, `root.on(event, handler)`, `root.injectChild(...)`, `api.ui.dom.addStyle(css)` (the mount carries `data-ls-script`, so `@scope` rules match content inside). Calls are buffered until the frontend has created the mount.
- **`setVisible(visible)`** — show / hide without destroying.
- **`destroy()`** — remove the mount. Idempotent; subsequent handle calls are silent no-ops.

## Worked example

```js
// A dismissible full-screen overlay.
const mount = api.ui.mountApp({ position: 'app-overlay', className: 'reading-mode' });

api.ui.dom.addStyle(`
  .reading-mode-body { padding: 2rem; max-width: 60ch; margin: 0 auto; }
`);
mount.root.update(`
  <div class="reading-mode-body">
    <button data-act="close">Close</button>
    <article>…</article>
  </div>
`);
mount.root.on('click', (ev) => {
  if (ev.target?.dataset?.act === 'close') mount.destroy();
});
```

## Lifecycle

Like float widgets and drawer tabs, a live mount **pins the script** against worker eviction (so its `.root` event handlers keep working), and every mount the script created is **destroyed automatically** when the script is disabled or reloaded — you don't have to track them for teardown. Call `destroy()` yourself when the overlay's job is done.

## Notes

- **Requires `app_manipulation`** (same gate as `showAdvancedModal` and `api.ui.dom.*`).
- The `.root` is buffered: ops you fire synchronously right after `mountApp()` are applied once the frontend has created the mount (the call resolves the moment the host confirms the mount + binds `.root`).
- `position` defaults are host-defined; pass `'app-overlay'` for a covering layer.

See the in-app **Reference tab** for the `MountAppOptions` / `MountedAppHandle` field lists, and [DOM injection](dom-injection.md) for the `.root` content API.
