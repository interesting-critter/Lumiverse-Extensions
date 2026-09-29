# Canvas Architecture Overview

## What is Canvas?

Canvas is a **Spindle extension** for **Lumiverse** (an AI chat frontend) that adds UI enhancements: a second sidebar drawer for organizing extension tabs, chat column reflow, drag-to-resize, slash commands, and more. It is a pure-frontend extension — no server-side logic — running as a bundled JavaScript module in the browser.

Canvas is authored by "Creature" and lives at `https://github.com/j-dandelion/Lumiverse-Canvas`. It targets Lumiverse v0.1.0+ and uses the Spindle extension loader.

## Build System

- **Runtime**: Bun (v1.3.14)
- **Language**: TypeScript (strict)
- **Framework**: Preact (for the toast surface only; everything else is vanilla DOM)
- **Bundle targets**:
  - `dist/frontend.js` — built from `src/frontend.ts`, targets `browser`, ESM format
  - `dist/backend.js` — built from `src/backend.ts`, targets `bun`, ESM format
- **Version injection**: `build.sh` uses `sed` to replace the `CANVAS_VERSION` stub in `src/layout/persist.ts` with the real version from `package.json` before bundling, then restores it.
- **Deploy**: `build.sh` copies bundles + `spindle.json` to `$HOME/Lumiverse/data/extensions/canvas/repo/`

## Entry Points

### Frontend (`src/frontend.ts` → `src/setup.ts`)

The Spindle loader calls `setup(ctx: SpindleFrontendContext)` — this is the single entry point. It returns a teardown function that the host calls when the extension is disabled.

**Setup lifecycle (order matters):**

1. `setBackendCtx(ctx)` — wire IPC context before any layout call
2. Register `pagehide`/`beforeunload`/`visibilitychange` flush handlers
3. Register style cleanup teardowns
4. `mountSettingsPanel(ctx)` — attach to `[data-spindle-mount="settings_extensions"]`
5. Register always-on teardowns (toast surface, applyLayout interval, slash runtime)
6. `loadSavedLayout()` — single IPC roundtrip, hydrates settings
7. After layout loads:
   - Version mismatch warning
   - `setDebug(getSettings().debugMode)` — sync debug flag
   - `setLastLoadedLayout(layout)` — cache for re-apply
   - `hydrateSettings(layout.settings)` — merge saved settings with defaults
   - `refreshSettingsPanel()` — re-render toggles with loaded values
   - `installDebugEscapeHatch()` — if debugMode is on, install `window.__canvasDebug()`
   - `feature.init()` hooks — one-time setup (inject disable-CSS)
   - `feature.mount()` hooks — conditionally mount each feature
   - `startSideChangeWatcher()` — detect main drawer side changes
   - `startMainDrawerPersistence()` — watch main drawer open/close/resize
   - `startMobileExclusion()` — viewport-cross detection
   - `drawerObserver.start()` — tab registration/unregistration watcher
   - `initSecondaryDrawer()` — secondary drawer state machine
   - `startContextMenuListener()` — right-click menu injection
   - `applyMainDrawer(layout)` — restore main drawer state
   - `applyLayout(layout)` — restore secondary sidebar + tab assignments

### Backend (`src/backend.ts`)

Runs in Bun (server-side). Handles persistence via `spindle.storage.*`:

- `SAVE_LAYOUT` — writes JSON to `layout.json` (atomic write via temp key + `storage.move`)
- `LOAD_LAYOUT` — reads and parses `layout.json`
- `SET_DEBUG` — toggles verbose server-side logging

The backend is stateless between messages — no long-lived state.

## Module Graph (High-Level)

```
frontend.ts → setup.ts (orchestrator)
  ├── settings/panel.ts + state.ts + render.ts   (settings UI + state)
  ├── features/registry.ts                        (feature lifecycle)
  ├── layout/persist.ts + apply.ts               (save/load/restore)
  ├── core/ + recon/ + host/                      (owned tab model — single source of truth)
  │   ├── core/model.ts + reduce.ts + select.ts   (LayoutModel, intents, activeAfterRemoval)
  │   ├── recon/dispatch.ts + reconcile.ts        (dispatch queue, placement-first moves, reconcile/persist)
  │   └── host/lumiverse/implementation.ts        (HostPort against live Lumiverse: observe/place/setOrder/activate)
  ├── sidebar/                                    (secondary drawer subsystem)
  │   ├── secondary-drawer.ts                     (state machine)
  │   ├── secondary.tsx                           (DOM construction)
  │   ├── drawer-sync.ts                          (cross-drawer visual sync)
  │   ├── drawer-observer.ts                      (tab registration watcher)
  │   ├── main-persist.ts                         (main drawer persistence)
  │   ├── mobile-exclusion.ts                     (mobile mutual exclusion)
  │   ├── tab-position.ts                         (flex-direction toggle)
  │   ├── panel-header-sync.ts                    (header CSS mirroring)
  │   ├── animation.ts                            (wrapper open/close anim)
  │   ├── styles.ts                               (CSS injection)
  │   ├── cleanup.ts                              (teardown registry)
  │   └── persist-polling.ts                      (DOM polling for hard refresh)
  ├── tabs/                                       (tab button management)
  │   ├── assignment.ts                           (owned-model facade, TabKey-keyed)
  │   ├── buttons.ts                              (hide/show/find/create buttons)
  │   ├── active-tab.ts                           (active tab tracking)
  │   ├── activation-handoff.ts                   (STUB — deleted Task 10.5; neighbor handoff lives in core/reduce.ts applyMove + sidebar/main-tab-pin.ts)
  │   ├── visibility-observer.ts                  (display transition watcher)
  │   └── tab-context-menu.ts                     (secondary sidebar menu)
  ├── chat/                                       (chat column)
  │   ├── reflow.ts                               (margin reflow + MutationObserver)
  │   └── tag-buttons.ts                          (data-tab-id tagging)
  ├── slash/                                      (slash command system)
  │   ├── runtime.ts                              (attach/detach)
  │   ├── registry.ts                             (command registry)
  │   ├── intercept.ts                            (keydown/click intercept)
  │   ├── suggest.ts                              (popup UI)
  │   ├── dispatch.ts                             (command dispatch)
  │   ├── parse.ts                                (pure command parser)
  │   ├── intent.ts                               (committed-command state)
  │   ├── dom-utils.ts                            (autocomplete helpers)
  │   ├── positioning.ts                          (popup positioning)
  │   ├── microtask.ts                            (defer via MessageChannel)
  │   ├── builtin-help.ts                         (/help command)
  │   ├── toast.tsx                               (Preact toast surface)
  │   └── commands/select/                        (/select command)
  ├── resize/handles.ts                           (drag-to-resize)
  ├── drawerTabPosition/                          (drawer tab drag)
  ├── context-menu/index.ts                       (main sidebar context menu injection)
  ├── store/index.ts                              (Zustand store fiber walk)
  ├── dom/                                        (DOM helpers)
  │   ├── lumiverse.ts                            (element queries)
  │   ├── host-bridge.ts                          (spindle API wrapper)
  │   ├── fiber.ts                                (React fiber access)
  │   ├── wait-for.ts                             (rAF polling)
  │   ├── selectors.ts                            (stable selectors)
  │   └── clamp.ts                                (width clamp)
  └── debug/                                      (debug utilities)
      ├── log.ts                                  (dlog/dwarn)
      ├── fiber-scan.ts                           (window.__canvasDebug)
      └── styles.ts                               (injectStyles helper)
```

**Note on `secondary.tsx`**: The file `src/sidebar/secondary.tsx` contains the DOM construction for the secondary sidebar wrapper. It is a `.tsx` file (not `.ts`), which is why it doesn't appear in `*.ts` glob patterns. It exports `mountSecondarySidebar`, `tearDownSecondarySidebar`, `getSecondaryWrapper`, `isSecondarySidebarOpen`, `openSecondarySidebar`, `closeSecondarySidebar`, `getClosedTransformPx`, `getSecondaryDrawer`, `getSecondaryTabList`, `getSecondaryPanel`, `injectDrawerTabStyles`, and `PUZZLE_ICON_SVG`.

## Key Design Patterns

### Feature Registry Pattern

Every user-togglable behavior is a `CanvasFeature` with optional `init()`, `mount()`, and `apply()` hooks. The orchestrator iterates the registry — adding a new feature is a one-liner. Features are idempotent; mount/apply can be called multiple times safely.

### DOM-First State Reading

Canvas prefers reading from the live DOM over cached store snapshots for up-to-the-moment accuracy. The Zustand store is accessed via fiber-tree walking (not a direct API), cached for 3s, and used as a fallback when DOM queries fail.

### Cleanup Chain

All teardowns are registered in a single `cleanupAll()` chain. The setup function returns this chain, and the host calls it on extension disable. Cleanup functions are idempotent and swallow errors.

### IPC via CustomEvent

The frontend-backend communication uses `spindle.sendToBackend()` / `spindle.onFrontendMessage()` — a message-passing IPC, not direct function calls.

### Reactive Observers over Polling

Canvas heavily uses `MutationObserver` and `ResizeObserver` for reactive updates instead of `setInterval` polling. The 3s store cache TTL and rAF coalescing prevent redundant work.
