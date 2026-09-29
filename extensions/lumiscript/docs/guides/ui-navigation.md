# UI navigation

`api.ui` has a small set of **navigation** methods that drive the user to surfaces that already exist — the drawer, the settings modal, the command palette. They're the same primitives the built-in Command Palette uses, exposed so a script can guide the user somewhere: an onboarding nudge, an agent-driven walkthrough, a "fix it" deep link. All free tier — no permission required.

## Navigate vs. contribute

This is the distinction to keep straight:

- **Contribute** — `api.ui.registerDrawerTab(...)` adds a *new* tab that your script owns and renders (see [DOM injection](dom-injection.md) for filling it).
- **Navigate** — `openDrawerTab(id)` / `openSettings(viewId?)` / `openCommandPalette()` send the user to surfaces that *already exist* — built-in ones, or tabs any extension (including yours) has contributed.

This guide is about the second kind.

## Enumerate, then navigate

`getDrawerTabs()` and `getSettingsTabs()` list what's reachable so you can pick a target (or build your own jump-to picker):

```js
const tabs = await api.ui.getDrawerTabs();
for (const t of tabs) {
  console.log(t.id, '·', t.tabName, '·', t.source);   // 'builtin' | 'extension'
}

await api.ui.openDrawerTab('connections');   // deep-link by id
```

A `UIDrawerTab` carries `{ id, shortName, tabName, tabDescription, keywords, source, extensionId? }`. `source` is `'builtin'` or `'extension'`; `extensionId` is set for extension-contributed tabs. Because navigation forwards the id verbatim, you can deep-link into **another extension's** tab if you know its id.

`getSettingsTabs()` returns `UISettingsTab` (`{ id, shortName, tabName, tabDescription, keywords, role? }`); role-restricted tabs (`role: 'admin' | 'owner'`) are filtered out for users who lack the role.

## The navigation calls

```js
await api.ui.openDrawerTab('characters');   // open the drawer to a tab id
await api.ui.closeDrawer();                  // close the drawer if open

await api.ui.openSettings('connections');    // open settings to a view id…
await api.ui.openSettings();                  // …or omit to land on 'display'
await api.ui.closeSettings();

await api.ui.openCommandPalette();            // the Ctrl+K overlay
await api.ui.closeCommandPalette();
```

Each `open*` / `close*` call **resolves once the host has dispatched the navigation** — the frontend then applies it asynchronously, so awaiting the promise means "the request was accepted", not "the animation finished".

## A worked example: onboarding nudge

Walk a first-run user to the Connections settings so they can set up a provider:

```js
// @triggers ls:startup
const seen = await api.variables.global.get('onboarded');
if (seen) return;

const ok = await api.ui.confirm(
  'Set up an LLM connection now?',
  'Welcome to this script',
  { confirmLabel: 'Open settings', cancelLabel: 'Later' },
);
if (ok) await api.ui.openSettings('connections');

await api.variables.global.set('onboarded', true);
```

## A worked example: jump-to picker

Surface a searchable list of every drawer tab and open the chosen one. `showContextMenu` returns the selected item's `key`:

```js
const tabs = await api.ui.getDrawerTabs();
const chosen = await api.ui.showContextMenu({
  position: { x: 200, y: 200 },
  items: tabs.map((t) => ({ key: t.id, label: t.tabName })),
});
if (chosen) await api.ui.openDrawerTab(chosen);
```

## Notes

- **Free tier.** The navigation methods need no permission. (Other `api.ui.*` methods carry their own — `showAdvancedModal` / DOM / components need `app_manipulation`, `pushNotification` needs `push_notification`, etc.)
- **Ids come from the getters.** Don't hardcode ids you haven't seen in `getDrawerTabs()` / `getSettingsTabs()` — built-in ids are stable, but enumerating is the safe way to discover extension-contributed ones.
- **The active user is implicit** — you never pass a user id.

See the in-app **Reference tab** for the full `UIDrawerTab` / `UISettingsTab` field lists and the rest of the `api.ui` surface.
