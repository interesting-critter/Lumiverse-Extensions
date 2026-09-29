# Tab Management

## Overview

Canvas manages tabs across two drawers: the main (Lumiverse-owned) and the secondary (Canvas-owned). The tab system handles button visibility, assignment tracking, active-tab state, context menus, and activation handoff when tabs move between drawers.

## Tab ID Format

Lumiverse assigns tab IDs in the format: `spindle:{extensionId}:tab:{tabName}:{counter}`

The `counter` is a session-variant suffix (`:1`, `:2`, `:3`) that changes across sessions. Canvas handles this via suffix-drift fallback: strip the trailing `:N` and match by the prefix.

## Assignment System (`tabs/assignment.ts`)

Since Task 10.6, `assignment.ts` is a **thin facade over the owned model** (`LayoutModel` in `core/model.ts`), not a private map:

- `getTabAssignments()` returns a fresh `Map` **derived from the model, keyed by TabKey** (`builtin:regex`, `ext:foo/Bar`) — **not** by liveId (`regex`, `spindle:foo:tab:bar:0`).
- Writes (`setTabAssignment`, `deleteTabAssignment`, mutating the returned Map) are **no-ops whenever the model is active** — placement state changes go through `dispatch({ t: 'move' | 'activate' | ... })`. The legacy in-memory map exists only pre-bootstrap and in tests.
- Reading the facade by liveId **always misses** — this caused three separate regressions (observed world reverting moves, restore placement failing, `placeTab` early-returns always wrong). See [pitfalls.md](pitfalls.md) §1 for the conversion rules (`host.findKey` / `liveIdForFacadeKey`).

### Key Functions

- `getTabAssignments()` — read the facade (model-derived, TabKey-keyed)
- `hasTabAssignment(tabId)` — check if assigned
- `setTabAssignment(tabId, panel)` — legacy write; no-op with an active model
- `deleteTabAssignment(tabId)` — legacy write; no-op with an active model
- `clearTabAssignments()` — reset all (called on teardown)
- `getTabSidebar(tabId)` — returns 'primary' or 'secondary'

### Moving tabs today: `placementFirstMoveByLiveId` (`recon/dispatch.ts`)

Right-click "Move to second drawer" no longer dispatches a model intent first. The flow is **placement-first**:

1. Capture the taskbar chrome decision **before** placement (moved tab's button still visible): if the moved tab is the mirror's active, capture the nearest visible neighbor (`findNeighborHostButtonFor`); else capture the active id for content re-assert.
2. `assignToSecondary(liveId)` — DOM placement now (button + root reparent + hide main button).
3. Chrome: explicit drawer open; neighbor handoff (`adoptMainMirrorNeighbor`) or active-content re-assert (host panel drift — see [pitfalls.md](pitfalls.md) §6).
4. Model: `dispatch({ t: 'move', ... })` **only when the model doesn't already have the tab in the target** ("model already in target" is the common case for restored tabs — the chrome steps above must not be skipped by an early return, see [pitfalls.md](pitfalls.md) §5).
5. Neighbor convergence: `dispatch({ t: 'activate', ... })` for the handoff target so the model's active doesn't lag the mirror key.

Built-in placement: `requestTabLocation` to the container is an allowlist silent no-op for most built-ins in this runtime, and `store.moveTabTo` is missing — the `via=dom` fallback (registry root reparent) is the real path; `via=bridge` works for allowlisted tabs.

## Button Management (`tabs/buttons.ts`)

### Main Sidebar

- `hideMainTabButton(tabId)` — sets `display: none` on the button
- `showMainTabButton(tabId)` — clears `display: none`
- `findMainTabButton(tabId)` — lookup by:
  1. `data-tab-id` attribute (fast path, set by tag-buttons)
  2. Direct `title` attribute match (LumiScript fallback)
  3. Store-based title match (pre-tag window)

### Secondary Sidebar

- `addSecondaryTabButton(tab)` — creates a button in `.sidebar-ux-tab-list` with icon, label, click handler (opens drawer + shows tab), and right-click handler (shows context menu)
- `removeSecondaryTabButton(tabId)` — removes the button
- `showSecondaryTab(tabId)` — activates a tab by setting `data-canvas-active` on the matching root, updating header title, toggling `sidebar-ux-tab-active` class on buttons
- `updateDrawerTabVisibility()` — shows/hides the drawer tab button based on whether any tabs are assigned

### Settings Button Detection

`isSettingsButton(btn)` — heuristic to exclude the Settings tab from move operations. Checks class, aria-label, and title for "settings"/"preferences".

## Active Tab Tracking (`tabs/active-tab.ts`)

### Main Drawer

`getActiveTabId()` returns a discriminated union:
- `{ state: 'closed' }` — drawer is closed
- `{ state: 'active', id: string }` — the active tab's ID
- `{ state: 'other', id: string }` — some other tab is active
- `{ state: 'unknown' }` — can't determine

`resolvePrimaryActiveTabId()` — single user-visible primary active id (mirror exclusive key when pin on; else host DOM, then store).

`isTabActiveInMainDrawer(tabId)` — boolean over `resolvePrimaryActiveTabId` (shared by rClick, DnD, Configure).

### Secondary Drawer

`getActiveSecondaryTabId()` / `setActiveSecondaryTabId(tabId)` — in-memory tracking.

## Activation Handoff

The legacy `tabs/activation-handoff.ts` orchestration was **deleted** (Task 10.5; the file is now a stub kept only for `assignment.ts` compile parity). Neighbor handling lives in two layers:

### Model layer (`core/reduce.ts` — `applyMove`)

When the moved key was the source side's active, `applyMove` adopts `activeAfterRemoval(model, from, key)` — the nearest **visible** neighbor: the tab immediately above the moved slot, else the first selectable below (skipping hidden), else `null`. This is the durable truth of "Part A + Part B".

### Chrome layer (taskbar mode, `sidebar/main-tab-pin.ts` + `recon/dispatch.ts`)

The mirror key, header title, and host content must follow the model's replacement — `placementFirstMoveByLiveId` applies `adoptMainMirrorNeighbor(btn, title)` after a move of the mirror's active tab:

- Sets the mirror key to the neighbor (`id__<id>`), clicks the neighbor's host button (content settle), updates the header.
- May override a user-picked key (the user's own move drove it) but keeps `userPicked: true` so a later restore activation still can't clobber it.
- `findNeighborHostButtonFor(tabId)` resolves the neighbor from the **host drawer's visible button order** (above, else below, skipping Settings) — captured **before** placement because the moved button is hidden afterward.
- The model converges via a follow-up `activate` intent when its active lags the mirror key (mirror clicks don't reliably produce host-syncs).

### Mirror active-key rules (taskbar mode)

- The Canvas exclusive key (`_state.activeKey`) drives highlight, header, and toggle-close — host `tabBtnActive` is **not** authoritative (stale after restore, first-tab reset after moves).
- `userPicked` (set on mirror clicks and neighbor adoption; cleared by heal/adopt/restore) blocks `activateMainMirrorFromRestore` from clobbering a user selection mid-session — this was the "moving a tab activates Databank" regression.
- The heal (`reconcileMainMirror`) keeps the exclusive key while its host button is merely `display:none` (mid-move), and only heals from the host when the key's button is truly gone.

See [pitfalls.md](pitfalls.md) §3–§5 for the full failure modes.

## Dual Mode × Configure Tabs (`tabs/configure-*.ts` + `layout/mode-profiles.ts`)

Canvas intercepts "Configure tabs" from Lumiverse's context menu regardless of whether the second drawer is enabled (always-on intercept, started in `setup.ts`). The modal presentation depends on the second-drawer state:

- **Second drawer enabled**: Configure Tabs opens in dual-column mode. The user can reorder tabs across columns, change visibility, swap drawer sides, and apply the change.
- **Second drawer disabled**: Configure Tabs still opens (Canvas modal, not vanilla Lumiverse). The footer toggle lets the user enable the second drawer directly from within the modal.

The host `drawerSettings` are patched in one atomic-like write via `commitConfigureDraft` (see `tabs/configure-commit.ts`).

### Three dual-mode state paths

Canvas maintains three distinct state paths for the second drawer lifecycle:

| Path | When | Source | Behaviour |
|------|------|--------|-----------|
| **First-enable seed** | First time user enables second drawer (no prior dual tabs anywhere) | Live `snapshotLayout()` | Seeds `lastLoaded` with live primary, secondary closed/empty. Runs **before** `setSettings`. |
| **Re-enable dual restore** | User re-enables after a prior dual session | `dualLayout` slot (persisted, hydrated at boot) | `restoreSingleModeLayout` bootstraps the owned model from the slot. |
| **Single-layout restore** | On disable (off path) | `singleLayout` slot (persisted) → live host | `restoreSingleModeLayout` swaps the owned model to the single layout (reconcile writes host settings) + restores main open/active. |

**Mode layout profiles (2026-08-16, consolidated v2):** each mode keeps its
own saved layout so switching never destroys the other — see
[persistence.md](persistence.md) "Mode Layout Profiles". The session-only
vanilla baseline and dual session profile are **retired**: the slots are the
only mode state. `detachedTabs` writers (`getLiveIdAssignmentEntries`) emit
`tabId` = current live id and `tabTitle` = the model TabKey (authoritative
for restore).

The first-enable seed is implemented in `layout/persist.ts` (`seedDualLayoutFromLive`). It is guarded by `hasDetachedTabs()` which checks `lastLoaded` and the dual slot. If either has detached tabs, the seed is skipped — this prevents overwriting real dual tabs on re-enable.

### Conflict rule: slot wins on disable

The **singleLayout slot** (the durable baseline — captured at enable from the same pre-dual state the retired vanilla baseline used to capture) is restored on disable:

| Configure Tabs choice during dual session | Effect on slot | Effect on disable |
|------------------------------------------|----------------|-------------------|
| **Apply** | Slot unchanged | Restore overwrites the Apply with the saved single state |
| **Discard** | Slot unchanged | Restore applies the saved single state (the Discard never wrote anyway) |
| **Cancel** | Slot unchanged | No effect (the user stays in dual) |
| (no dirty draft) | Slot unchanged | Restore applies the saved single state (no dual changes to overwrite) |

**Repeated Apply** in the dual session: each Apply patches the host `drawerSettings` (active dual session only) but does not modify the single slot — the owned-model persist path only writes the slot while the model is single. On disable, the saved single state is restored.

### Teardown + restore ordering

`finishDisable()` (in `settings/second-drawer-mode.ts`) runs the following sequence:

1. Resolve any dirty Configure Apply/Discard/Cancel dialog.
2. Save the `dualLayout` slot from the live owned model (what re-enable restores).
3. Determine the single layout: `singleLayout` slot (freshest) → live host.
4. `setSettings({ secondSidebarEnabled: false })` — feature.apply tears down the secondary sidebar.
5. **`restoreSingleModeLayout(slot)`** — bootstrap the owned model from the slot (reconcile writes host side/tabOrder/hiddenTabIds) + restore main open/active via `restoreMainDrawerFromDom`. Done AFTER teardown so the host tabs are back in main-drawer before the restored primary tab is clicked.
6. Modal refresh from live (existing) — now shows the restored single layout.
7. **Clear baseline on success** — `clearVanillaBaseline()`. On failure (NO-GO / partial), retain the baseline for retry.

### Why baseline wins

### Auto-save behavior (2026-07-12)

Configure Tabs now auto-saves individual edits immediately rather than
waiting for Done. The draft model is retained but commits happen
automatically on toggle hide, swap side, and drag-end (not mid-drag).

| Action | Behavior |
|--------|----------|
| Toggle tab hidden (eye icon) | Draft commits immediately (no Done needed). |
| Swap drawer side radio | Draft commits immediately (no Done needed). |
| Drag-end within or between columns | Draft commits immediately (no Done needed). |
| Mid-drag | No draft commit; the previous committed state is unchanged. If drag-tab and drop-tab change order, only the final drop commits. |
| Close modal (X / Escape) | No special behavior — regular draft-based close dialog applies. |
| Enable/disable second drawer | No special behavior — full mode-switch dialog. |
