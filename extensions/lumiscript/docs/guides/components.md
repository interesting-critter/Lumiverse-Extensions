# Mounting shared host components

`api.ui.components.*` mounts Lumiverse's own first-party, themed React components — switches, text inputs, badges, spinners — into containers your script owns. They inherit the active Lumiverse theme automatically (accent, glass, dark/light, density), so they look native with **zero CSS to ship**. Requires the `app_manipulation` permission.

> **The full catalog is bridged (v1.0.0-rc.9).** All 16 host components are available: `mountBadge`, `mountSpinner`, `mountSwitch`, `mountTextInput`, `mountTextArea`, `mountNumericInput`, `mountNumberStepper`, `mountCheckbox`, `mountRangeSlider`, `mountSelect`, `mountMultiSelect`, `mountFolderDropdown`, `mountModelCombobox`, `mountPagination`, `mountCloseButton`, and `mountCollapsibleSection`. See the **Reference tab** for each component's full option fields. Two convenience handle-methods are a deferred follow-up: `focus()`/`blur()` (text inputs), `open()`/`close()` (selects), and `refresh()` (model combobox manual mode) aren't bridged yet — the components are fully functional without them (connection-bound combobox auto-manages its list).

## The mounting model

Unlike `api.ui.dom.inject` (which takes a CSS selector and HTML), components mount into a **container you already own**. Inject an empty slot first, then mount into it:

```js
const slot   = api.ui.dom.inject('body', '<div></div>');   // returns a DOMHandle (sync)
const toggle = api.ui.components.mountSwitch(slot, {        // returns a handle (sync)
  checked: true,
  onChange: (on) => console.log('toggled →', on),
});
```

`mountX(target, options?)` returns a handle **synchronously** (the underlying mount is dispatched fire-and-forget, exactly like `inject`). The `target` is the `DOMHandle` you injected.

Every handle has:

- `update(patch)` — merge a partial of the mount options into the live component. Fire-and-forget.
- `destroy()` — unmount and release. Idempotent. The container is left in place.

Interactive components (`mountSwitch`, `mountTextInput`) additionally have:

- `getValue()` — read the current value. **Returns a `Promise`** (see below).

## Two things that differ from the host API

1. **`getValue()` is async here.** The host's `getValue()` is synchronous; across LumiScript's worker boundary, reading the live value is a round-trip to the frontend — so it returns a `Promise<T>` (same reason `DOMHandle.read()` is async). You don't need to mirror state into your own variables; the host owns it.

   ```js
   const input = api.ui.components.mountTextInput(slot, { placeholder: 'Name…' });
   // …later, e.g. in a button handler:
   const text = await input.getValue();
   ```

2. **The mount target is a `DOMHandle`, not a raw selector.** This keeps mounts scoped to DOM your script owns (a raw selector could reach into host UI). Inject the slot, mount into it.

## Callbacks fire into your script

Option callbacks (`onChange`, …) run in your script's sandbox, not on the frontend. When the user interacts with the component, the callback is invoked with the new value:

```js
const sw = api.ui.components.mountSwitch(slot, {
  checked: false,
  onChange: async (on) => {
    // Runs in your script. `await` works — the host keeps the run alive.
    await api.variables.global.set('featureEnabled', on);
  },
});
```

There's a small amount of latency (the value crosses frontend → backend → sandbox), so callbacks are effectively async. Make them `async` and `await` everything inside — the host keeps the per-fire run alive across the await chain (a sync callback that kicks off un-awaited async work can have its `api.*` calls land after the run closed).

## Lifecycle

- `destroy()` unmounts a single component and drops its callbacks.
- When your script is **disabled or reloaded**, every component it mounted is destroyed automatically (the host removes the containers and releases the React trees) — you don't need to track them for teardown.
- Re-running a script that mounts into a fresh slot each time will accumulate components; mount into a stable injected slot (or `destroy()` the previous handle) if you re-run.

## A taste of the catalog

```js
// Badge — inline status/label
const badge = api.ui.components.mountBadge(slot, { text: 'Beta', color: 'warning', size: 'pill' });
badge.update({ text: 'Ready', color: 'success' });

// Switch — boolean toggle (onChange + async getValue)
const sw = api.ui.components.mountSwitch(slot, { checked: true, onChange: (on) => {/* … */} });
const isOn = await sw.getValue();

// Range slider — two callbacks: live drag + committed value
const opacity = api.ui.components.mountRangeSlider(slot, {
  min: 0, max: 100, value: 35, label: 'Opacity', format: { suffix: '%' },
  onDragValue: (v) => {/* live preview */},
  onCommit:    (v) => {/* persist */},
});

// Select — searchable single-select
const tone = api.ui.components.mountSelect(slot, {
  options: [{ value: 'casual', label: 'Casual' }, { value: 'formal', label: 'Formal' }],
  onChange: (v) => {/* … */},
});

// Model combobox — connection-bound (host manages the model list)
api.ui.components.mountModelCombobox(slot, { connection: { kind: 'llm' }, onChange: (m) => {/* … */} });
```

See the **Reference tab** for the full option fields of each component and the handle shapes.

## Collapsible sections — the body-slot pattern

`mountCollapsibleSection` is the one component that works differently: the host renders the **header chrome** (title, chevron, badge), and hands your script back a **`body` element** to fill. The handle's `.body` is a regular {@link DOMHandle} — inject and wire it exactly like any element you injected yourself:

```js
const slot    = api.ui.dom.inject('body', '<div></div>');
const section = api.ui.components.mountCollapsibleSection(slot, {
  title: 'Advanced',
  badge: '3',
  defaultExpanded: false,
  onToggle: (open) => console.log('section is now', open ? 'open' : 'closed'),
});

// Fill the body — it's a DOMHandle, so inject/update/on all work
section.body.update('<p>Body content owned by your script.</p>');
const btn = section.body.injectChild('p', '<button data-act="go">Go</button>', { position: 'afterend' });
btn.on('click', (ev) => { if (ev.target?.dataset?.act === 'go') doThing(); });

// Imperative controls
section.expand();
section.collapse();
section.toggle();
const open = await section.isExpanded();   // async — frontend round-trip
```

`title` is required (note it's a *required* `options` argument — `mountCollapsibleSection(slot, { title })`, not optional). `expand`/`collapse`/`toggle` are fire-and-forget; `isExpanded()` is async like `getValue()`.
