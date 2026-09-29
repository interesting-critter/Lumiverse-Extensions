# DOM injection

`api.ui.dom.*` is how a LumiScript script puts pixels into the Lumiverse app shell — buttons next to messages, side panels, floating widgets, themed banners, drag-and-drop overlays. It's the single highest-leverage capability if you want LumiScript to feel like a first-class part of Lumiverse rather than a background automation tool.

All of `api.ui.dom.*` requires the `app_manipulation` permission. Every method asserts it up front; without the grant they throw `Error: PERMISSION_DENIED:app_manipulation — grant this permission to use this API`. (See [`concepts/permissions.md`](../concepts/permissions.md) for the broader permission story.)

The shape of the API mirrors what you'd reach for from raw DOM but routed through an IPC boundary — your script lives in a Bun subprocess, the DOM lives in the user's browser, and a `DOMHandle` is a small reference object that lets you act on the element across that boundary.

## Your first injection

Wire a script to `MESSAGE_SENT`, paste this body, save:

```js
const banner = api.ui.dom.inject(
  '#chat-container',
  '<div class="ls-demo">Hello from LumiScript</div>',
  { position: 'beforebegin' },
);

api.ui.dom.addStyle(`
  .ls-demo {
    background: var(--lumiverse-accent, rgb(147, 112, 219));
    color: var(--lumiverse-accent-fg, #fff);
    padding: 6px 10px;
    border-radius: var(--lumiverse-radius, 6px);
    font-size: 13px;
  }
`);

// Remove the banner after 3s.
setTimeout(() => banner.remove(), 3000);
```

Send a message — a purple banner appears above the chat container and disappears three seconds later.

Three primitives doing the work:

- **`api.ui.dom.inject(target, html, opts?)`** — find the element matching the CSS selector and inject your HTML at a position relative to it.
- **`api.ui.dom.addStyle(css, opts?)`** — attach a `<style>` element scoped to your script so its rules don't bleed into other scripts.
- **`handle.remove()`** — clean up the element when you're done.

Everything else in this guide is variations on those three.

## Targeting the page

There are three ways to land HTML in the page, depending on what you're anchoring to.

### `inject(target, html, opts?)` — any CSS selector

```js
const sidebar = api.ui.dom.inject(
  'body',
  '<aside class="ls-sidebar">...</aside>',
  { position: 'beforeend', id: 'sidebar' },
);
```

`target` is any CSS selector. `opts.position` accepts `'beforebegin'`, `'afterbegin'`, `'beforeend'`, `'afterend'` — the four positions [`insertAdjacentHTML`](https://developer.mozilla.org/en-US/docs/Web/API/Element/insertAdjacentHTML) takes. Default is `'beforeend'`.

`opts.id` is your **stable identifier**: the next time you call `inject` with the same `opts.id`, LumiScript clears the old listeners, updates the HTML in place, and returns the *same* handle. Use it whenever a fire might re-inject something the script already injected — see [Idempotent re-injection](#idempotent-re-injection) below.

### `injectAtMessage(messageId, html, opts?)` — anchored to a chat message

```js
const footer = api.ui.dom.injectAtMessage(
  data.message.id,
  '<div class="ls-msg-footer">word count: 42</div>',
  { position: 'footer', id: 'word-count' },
);
```

`position` here is `'header'` or `'footer'` — chat-message-specific anchors. `id` (the field name in `DOMMessageInjectOptions`) is the idempotency key — the dedup is per-script-per-id.

**Trap to know up front.** The id-key is `(scriptId, id)` *only* — `messageId` is **not** part of the key. If you call `injectAtMessage(msgA, html, { id: 'foo' })` and later `injectAtMessage(msgB, html2, { id: 'foo' })`, the second call updates the element attached to `msgA` and ignores `msgB`. Always namespace the id per-message:

```js
{ id: `footer-${data.message.id}` }
```

### `injectChild(target, html, opts?)` — descendant of an existing handle

When the parent element only exists once your script's outer container has mounted (drawer tabs, modal bodies, float widgets), reach the descendant via the parent handle, not via `document.querySelector`:

```js
const panel = api.ui.dom.inject('body', `
  <div class="ls-panel">
    <header class="title-bar">Stats</header>
    <div data-grid></div>
  </div>
`, { id: 'stats-panel' });

// Resolves relative to the panel — works even when the panel was just mounted.
const grid = panel.injectChild('[data-grid]', '<table>...</table>', { id: 'grid' });

// Updating just the grid without rebuilding the header:
grid.update('<table>... refreshed ...</table>');
```

`injectChild` resolves its selector through the host's element-map, so it works regardless of whether the parent has propagated to `document.querySelector` yet. The returned handle is a normal `DOMHandle` — `update`, `remove`, `on`, etc. all work.

## The handle lifecycle

Every injection method returns a `DOMHandle`:

| Method | What it does |
|---|---|
| `handle.id` | Read-only string — the internal element ID. |
| `handle.update(html)` | Replace the element's `innerHTML`. Listeners attached via `handle.on(...)` survive an update. |
| `handle.remove()` | Remove the element and detach all listeners. Idempotent. |
| `handle.on(event, handler, opts?)` | Attach a DOM event listener. Returns an unsub function. |
| `handle.makeDraggable(handleSelector?)` | Make the element user-draggable. See [Drag](#drag). |
| `handle.injectChild(target, html, opts?)` | Descendant-scoped injection (above). |
| `handle.read(opts?)` | Async — snapshot the element's current state from the frontend. |

All methods are fire-and-forget across the IPC boundary except `read` (which awaits a frontend round-trip).

### Idempotent re-injection

Two patterns where the same id matters:

**`inject` with `opts.id`** — listeners are cleared, HTML updated in place, same handle returned. The element node itself is *not* replaced — only its inner HTML.

**`addStyle` with `opts.id`** — prior stylesheet is *removed*, new one is injected. Replace-by-id (not in-place update). Useful during dev iteration:

```js
// Every fire reinjects the latest CSS — no zombie stylesheets from prior fires.
api.ui.dom.addStyle(myCss, { id: 'styles' });
```

Without `opts.id`, every `addStyle` call appends a fresh `<style>` element. That's the right default when you intentionally want layered stylesheets, and a footgun if you forgot.

**Caveat for inline `<style>` blocks inside `inject()` HTML.** The dedup path uses an internal IPC (`dom_update`) that **replaces the wrapper's `innerHTML` in place**. Browsers don't reliably reactivate `<style>` elements added that way — the tag is in the DOM, the rules are not in the active stylesheet list. First fire works; second fire silently loses every CSS rule from the inline `<style>`. The script's own UI loses its styling, host-targeting rules (`button[aria-label="..."]` etc.) stop applying, the script appears broken.

For scripts that combine **inline `<style>` blocks** (the host-styling pattern above) with **re-firing triggers** (`ls:startup` + `CHAT_SWITCHED` + manual Run-button presses), don't rely on `opts.id`-dedup. Call `api.ui.dom.cleanup()` at the top of the body instead:

```js
// @triggers ls:startup, CHAT_SWITCHED
api.ui.dom.cleanup();   // wipe every prior inject + style for this script

api.ui.dom.inject('body', `
  <style>${STYLES}</style>
  <label class="my-toggle">…</label>
`, { position: 'beforeend' });
// No `id` — explicit cleanup makes dedup unnecessary AND avoids the
// dom_update inline-style-inert path. Every fire is a fresh inject.
```

`cleanup()` only removes THIS script's injections + styles + delegations — other scripts are untouched. The brief flicker between cleanup and re-inject (one frame at most) is imperceptible for the typical `ls:startup` / `CHAT_SWITCHED` cadence. If you're firing on something high-frequency like `STREAM_TOKEN_RECEIVED`, the cleanup pattern is wrong for you — use plain `addStyle` (no inline `<style>`), accept the script-scope limit, and find a different way to reach host elements.

## Event handling

There are two routes for DOM events. They behave differently and serve different purposes.

### `handle.on(event, handler, opts?)` — scoped to this element

This is the everyday path. Attaches an `addEventListener` to the injected element. The handler receives a serialised `DOMEventData`:

```ts
{
  type: string;                                // 'click', 'input', 'keydown', ...
  targetId?: string;                           // event.target.id if present
  targetValue?: string;                        // value of form elements
  targetChecked?: boolean;                     // checked state of form elements
  dataset?: Record<string, string>;            // event.target.dataset
  detail?: unknown;                            // CustomEvent.detail (JSON-only)
  clientX?: number;                            // pointer position
  clientY?: number;
  key?: string;                                // KeyboardEvent.key
  code?: string;                               // KeyboardEvent.code
}
```

**Crucial mechanic.** The handler reads `event.target.dataset` *directly* — it does **not** walk up the DOM tree with `closest()`. This matters when the visible content of a button is a nested SVG, `<img>`, or styled `<span>`:

```js
const panel = api.ui.dom.inject('body', `
  <div class="ls-panel">
    <button data-action="refresh">
      <svg><!-- icon --></svg> Refresh
    </button>
    <button data-action="close">×</button>
  </div>
`);

panel.on('click', (ev) => {
  if (ev.dataset?.action === 'refresh') reload();
  if (ev.dataset?.action === 'close')   panel.remove();
});
```

If the user clicks the icon center, `event.target` is the inner `<svg>`. The svg has no `data-action` attribute, so `ev.dataset` is empty and the guard silently no-ops. Clicking the button's padding works; clicking the icon doesn't.

**Fix**: treat decorative inner content as transparent to pointer events.

```css
.ls-panel button > svg,
.ls-panel button > img,
.ls-panel button > span.icon { pointer-events: none; }
```

Now clicks pass through the inner content to the button itself, and `event.target` is the button (which carries `data-action`). Default convention: every icon button's inner decorations get `pointer-events: none`.

### `api.ui.dom.delegate(selector, event, handler, opts?)` — document-wide, selector-matched

Use this when you want one listener to handle clicks on many message-anchored elements, or when the target isn't injected by you (it's host-rendered):

```js
api.ui.dom.delegate(
  '[data-msg-action]',
  'click',
  (ev) => {
    console.log('matched:', ev.matched.tagName, ev.matched.dataset.msgAction);
    console.log('in message:', ev.message?.id);
  },
  { root: 'chat' },
);
```

`opts.root` is `'chat'` (only fires inside tracked chat messages) or `'document'` (anywhere on the page). The payload is a richer `DOMDelegatedEventData` that adds:

- `matched` — the actual matched element's snapshot (`tagName`, `id`, `classList`, `dataset`, `attributes`, `textContent`, plus form-specific fields where relevant).
- `modifiers` — `{ ctrl, shift, alt, meta, button? }`.
- `message` (chat-rooted only) — `{ id, role, swipeId }`.

This path **does** use `closest()` to find the matching ancestor of `event.target`, so clicks on nested decorative content still match (the opposite of `handle.on`). You're trading scope (document-wide) for that selector-aware behavior.

### `preventDefault` and `stopPropagation` are opt-in at registration

The handler is async (it dispatches across the IPC boundary), which is too late to prevent the browser's default action by the time your code runs. To suppress defaults, declare the intent at registration time:

```js
input.on('keydown', (ev) => {
  if (ev.key === 'Enter') submit();
}, { preventDefault: { onKeys: ['Enter'] } });
```

`preventDefault` accepts `true` (always prevent) or a `ConditionalPreventDefault` object: `{ onKeys?, onCodes?, onButtons?, whenModifiers?: { require?, exclude? } }`. The frontend evaluates the rule synchronously against the raw event before dispatching to your handler.

`api.ui.dom.delegate` accepts the same `preventDefault` shape plus a `stopPropagation: boolean` flag.

## Styling

```js
api.ui.dom.addStyle(`
  .ls-mystuff {
    background: var(--lumiverse-fill);
    color: var(--lumiverse-text);
    border: 1px solid var(--lumiverse-border);
    border-radius: var(--lumiverse-radius);
    transition: background var(--lumiverse-transition-fast);
  }
  .ls-mystuff:hover {
    background: var(--lumiverse-fill-hover);
  }
`);
```

Two things to know:

**Use `--lumiverse-*` CSS variables for theme tokens.** They follow the user's current theme — light/dark mode, accent colour, glass strength, font scale, etc. Hard-coded colours look out of place when the user switches themes.

**Stylesheets are *script-scoped* via CSS `@scope`** — Lumiverse wraps your CSS in `@scope ([data-ls-script="<id>"]) { ... }` so its rules apply only to elements injected by your script. Two scripts can both define `.ls-card` without colliding.

**One gotcha for portal modals.** Anything rendered via `createPortal(x, document.body)` (most modals) sits outside the host's `--lumiverse-*`-populated scope. The variables fall through to `unset` and your accent-tinted button vanishes. Always provide hard-coded RGB fallbacks for `--lumiverse-*` tokens in portal-modal CSS — the canonical accent purple is `rgb(147, 112, 219)`:

```css
.my-modal-btn {
  background: var(--lumiverse-accent, rgb(147, 112, 219));
  color:      var(--lumiverse-accent-fg, #fff);
}
```

### Styling host UI — escaping the `@scope` box

The `@scope` wrap is a feature for isolation between scripts, but it also means **`addStyle` rules can't reach host elements**. The host's chat input bar, message bubbles, action toolbar buttons, the `<body>` itself — none of them are inside any `[data-ls-script="<id>"]` ancestor, so none of them match your `addStyle` rules.

To style host UI — hide a host button, reposition a host element, reskin Lumiverse's own controls — inject an inline `<style>` block as part of an `inject()` call instead:

```js
const STYLES = `
  /* These rules ARE global — they apply wherever they match, not just
     inside [data-ls-script]. */
  button[aria-label="Attach file"] { display: none; }
  body:has(.my-toggle:checked) [class*="_actionBar_"] { display: none; }

  /* Your own button's styling can live here too — same lifecycle. */
  .my-toggle {
    display: inline-flex;
    align-items: center;
    color: var(--lumiverse-text, #ccc);
  }
`;

api.ui.dom.inject('body', `
  <style>${STYLES}</style>
  <label class="my-toggle">…</label>
`, { position: 'beforeend' });
```

Why this works: CSS rules inside a `<style>` element are document-global regardless of where the tag sits in the DOM. The host's `@scope` wrap is only applied to `addStyle` payloads — inline `<style>` blocks reaching the DOM via `inject()` bypass it entirely. The wrapper (`<div data-ls-script="<id>">`) still carries cleanup attribution, so on script disable the wrapper goes away, the inline `<style>` goes with it, and host UI returns to baseline. Same lifecycle as `addStyle`; different reach.

**Don't pair this pattern with `opts.id`-based dedup.** This combination has a non-obvious failure mode — see the caveat in [Idempotent re-injection](#idempotent-re-injection) above.

## Drag

```js
const panel = api.ui.dom.inject('body', `
  <div class="ls-floating">
    <header class="title-bar">Drag me</header>
    <div class="body">...</div>
  </div>
`);

panel.makeDraggable('.title-bar');
```

`handleSelector` is optional. When provided, only that child initiates drag; the entire root element moves. When omitted, the whole element is both handle and move target.

Drag is frontend-only — the new position is not auto-persisted, AND `DOMHandle` does not expose a `'dragend'` event for user listeners. The makeDraggable implementation uses internal `pointerdown` / `pointermove` / `pointerup` / `pointercancel` listeners; it does not surface drag-completion to script-side handlers. If you need post-drag persistence on a script-injected element, you have three options:

- **Use the built-in `floatingButton`** — `script.require('ls:components').floatingButton(...)` auto-persists position to `api.scriptStorage`. This is the path of least surprise.
- **Use `api.ui.createFloatWidget(...)` instead of a DOM-handle drag** — the FloatWidgetHandle DOES expose `onDragEnd(handler)` for explicit position-change events.
- **Sync position from scriptStorage at injection time** — re-injecting with the stored coordinates after each fire. Crude but works for static positions you set once.

For most drag use-cases the `floatingButton` built-in is the right answer.

## Reading element state

`handle.read()` is the one async method on `DOMHandle`:

```js
const snapshot = await panel.read();
if (snapshot) {
  console.log(snapshot.tag);          // 'div'
  console.log(snapshot.attrs);        // { class: 'ls-panel', ... }
  console.log(snapshot.text);         // concatenated textContent
  console.log(snapshot.childCount);   // direct element children
}
```

Pass `{ html: true }` to also include the current `innerHTML` — useful when you need to inspect descendant markup but the default omission keeps the IPC payload small for the common "verify attrs / check text" case.

Returns `null` if the element no longer exists (handle was removed, parent torn down, etc.). Useful inside polling-style code that needs to confirm an injection survived.

## Permissions + sanitization

### `app_manipulation` gates the entire namespace

Every `api.ui.dom.*` method asserts `app_manipulation` up front. So do all `DOMHandle` methods (`update`, `remove`, `on`, `makeDraggable`, `injectChild`, `read`). If the user hasn't granted the permission, the call throws synchronously:

```js
try {
  api.ui.dom.inject('body', '...');
} catch (err) {
  if (String(err.message).startsWith('PERMISSION_DENIED:app_manipulation')) {
    // Graceful degradation — script still works, just without the UI.
    return;
  }
  throw err;
}
```

The `PERMISSION_DENIED:<name>` prefix is stable; pattern-match on it. See [`concepts/permissions.md`](../concepts/permissions.md) for the broader denial-handling story.

### DOMPurify sanitizes everything injected

Every HTML payload passing through `inject` / `injectAtMessage` / `update` / `injectChild` is run through DOMPurify on the frontend before insertion. The host's FORBID_TAGS set strips:

- `<iframe>`, `<frame>`, `<object>`, `<embed>`, `<form>` — the form-action / framing surfaces that historically enabled cross-site exploits.

These tags are blocked by **three independent layers** (DOMPurify, the page's CSP `frame-src 'none'; child-src 'none'; object-src 'none'`, and the server's `X-Frame-Options: DENY`). There is no opt-out. If you need an embed-like experience, use a native LumiScript surface (`api.ui.showAdvancedModal`, a drawer tab, a float widget) — not an iframe.

DOMPurify's defaults additionally strip every inline `on*` event-handler attribute (`onclick`, `onerror`, `onload`, `onsubmit`, `onmouseover`, etc.) and the `formaction` attribute. So this **does not work**:

```html
<!-- onclick is stripped — the button renders but the handler is gone -->
<button onclick="alert('hi')">Click</button>
```

Use event delegation via `handle.on('click', fn)` with a `data-*` attribute on the trigger:

```js
panel.update('<button data-action="hello">Click</button>');
panel.on('click', (ev) => {
  if (ev.dataset?.action === 'hello') alert('hi');
});
```

`javascript:` URLs and `srcdoc` attributes are also stripped, same rationale.

## Common pitfalls

**`injectAtMessage` id persists across messageIds** — covered above; always namespace the `id` option per-message (`'footer-' + messageId`). (Common mistake: passing the option as `stableId` instead of `id` — the field name is `id` in `DOMMessageInjectOptions`; `stableId` is silently ignored.)

**CSS is bundled into JS** — stylesheets added via `addStyle` are part of your script source. Editing the source and saving doesn't reload an active stylesheet on its own; for CSS changes to take effect, the script body needs to re-fire (any wired event triggers it). For active dev iteration, use `opts.id` so re-fires replace cleanly, or toggle the extension off/on to force a full reload.

**Two React roots — no shared state.** The LumiScript panel (dock) and the settings panel are mounted as two independent `createRoot` trees with no common React parent. If you're writing a script that touches both — e.g. a script-list action that opens a modal in the settings root — pass state through a `window` CustomEvent bus, not through React props.

**Cleanup on script disable is automatic for what you registered.** When the user disables or deletes your script, LumiScript walks the per-script registries and removes everything: DOM elements, listeners, stylesheets, delegations, drag-handles. You don't have to call `api.ui.dom.cleanup()` manually — it's an *additional* explicit-cleanup escape hatch for mid-life sweeps, not a teardown requirement. (See [`concepts/handler-lifetime.md`](../concepts/handler-lifetime.md) for the broader handler-cleanup story.)

**Don't rely on `closest()` for `handle.on`** — that's `api.ui.dom.delegate`'s mechanic. `handle.on` reads `event.target.dataset` directly. The fix is `pointer-events: none` on decorative inner content (covered above).

## See also

- **In-app Reference, "API Functions → api.ui.dom" section** — auto-generated method list with full signatures, permission tags, and per-method notes. Source of truth for the surface.
- **`script.require('ls:components')`** — built-in library of higher-level widgets (`messageFooter`, `messageHeader`, `progressBar`, `floatingButton` with persistence, `multiSelect` modal, etc.) built on top of `api.ui.dom.*`. See the Reference's "Built-in Libraries" section.
- **[`concepts/trigger-model.md`](../concepts/trigger-model.md)** — how the script body becomes the handler, what `data` carries, the sandbox boundary.
- **[`concepts/handler-lifetime.md`](../concepts/handler-lifetime.md)** — when DOM injections + listeners are torn down, how that interacts with the eviction-pinning system.
- **[`concepts/permissions.md`](../concepts/permissions.md)** — `app_manipulation` in context, graceful denial handling.
