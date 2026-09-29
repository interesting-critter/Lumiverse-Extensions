# DOM Layer

## Overview

Canvas interacts with Lumiverse's DOM through a layer of query helpers, fiber-tree access, and a typed host-bridge wrapper. This module isolates all DOM-dependent code from feature logic.

## Lumiverse Element Queries (`dom/lumiverse.ts`)

All queries use attribute/substring selectors that survive CSS-module hash changes:

| Function | Selector | Returns |
|----------|----------|---------|
| `getMainSidebar()` | `[data-spindle-mount="sidebar"]` | The main sidebar mount node |
| `getMainDrawer()` | `sidebar.parentElement` | The drawer div (contains sidebar + panel) |
| `getMainPanel()` | `[class*="_panel_"]` | The panel containing the active tab's content |
| `getMainPanelContent()` | `[class*="_panelContent_"]` | The content area inside the panel |
| `getMainPanelHeader()` | `[class*="_panelHeader_"]` or first non-content child | The panel header |
| `getMainWrapper()` | `sidebar.closest('[class*="_wrapper_"]')` | The wrapper (carries `wrapperOpen` class) |
| `getChatColumn()` | `[class*="_chatColumn_"]` or fallback by structure | The chat content column |
| `getMainDrawerWidth()` | `drawer.getBoundingClientRect().width` | Current drawer width in px |

**Key insight**: CSS-module class names are hashed in production. Canvas matches by prefix (`[class*="_panelContent_"]`), not full hash, so the code is stable across Lumiverse rebuilds.

## React/Preact Fiber Access (`dom/fiber.ts`)

Canvas reads Lumiverse's Zustand store by walking the React fiber tree (the store is not directly accessible from extensions).

```typescript
findFiberKey(el: Element): string | null       // finds __reactFiber$* or __preact* key
getFiberFromElement(el: Element): Record<string, unknown> | null  // returns fiber object
```

Fiber key prefixes: `__reactFiber$` (React 16+) and `__preact` (Preact).

## Zustand Store Access (`store/index.ts`)

Canvas cannot import Lumiverse's Zustand store directly. Instead, it walks the fiber tree to find the store snapshot:

### Walk Strategy

1. Start from the main sidebar element's fiber
2. Walk UP to root ancestor
3. Scan DOWN from top ancestors (max depth 30)
4. Look for:
   - `drawerTabs` array: objects with `id`, `title`, `root`, `badge` fields
   - Store snapshot: objects with `drawerOpen` or `drawerTabs` keys
5. Cache results for 3s (`CACHE_TTL_MS`)

### Public API

- `findStoreData(force?)` — trigger fiber walk
- `getDrawerTabs()` — array of `{ id, extensionId, title, shortName, iconSvg, iconUrl, root }`
- `getStoreSnapshot()` — raw store snapshot
- `isMainDrawerOpen()` — DOM-first, store fallback
- `getMainDrawerSide()` — DOM-first, store fallback

**DOM-first pattern**: For `isMainDrawerOpen()` and `getMainDrawerSide()`, the wrapper's className is checked first (updates synchronously). The store snapshot is a fallback for early code paths before the wrapper element exists.

## Host Bridge (`dom/host-bridge.ts`)

Typed wrapper for the Spindle frontend context. **Primary source is the
`setup(ctx)` argument** (stored via `setHostBridgeContext`); `window.spindle`
is only a test/fallback — the host loader never assigns that global.

```typescript
interface HostBridge {
  ui: {
    getBuiltInTabRoot?(tabId: string): HTMLElement | undefined
    getBuiltInTabTitle?(tabId: string): string | undefined
    requestTabLocation?(tabId: string, loc: SpindleTabLocation): void
    getTabLocation?(tabId: string): SpindleTabLocation | null  // undocumented
  }
  containers: {
    registerContainer?(entry: { id: string; side: 'left' | 'right' | 'top' | 'bottom'; element: HTMLElement }): void
    unregisterContainer?(id: string): void
  }
}
```

Returns null when no setup ctx and `window.spindle` is undefined.
Built-in moves also require the `ui_panels` permission (declared in
`spindle.json`; `ensureUiPanelsPermission()` requests it at setup).

## Stable Selectors (`dom/selectors.ts`)

Selectors that don't depend on CSS-module hashes:

- `SELECTOR_TEXTAREA = 'textarea[name="chat-message"]'` — the chat input
- `SELECTOR_SEND_BTN = 'button[class*="sendBtn"]'` — the send button

## Width Clamp (`dom/clamp.ts`)

```typescript
clampSidebarWidth(px: number): number
// Min: 200px, Max: 80% of viewport width
```

Used by resize handles, `applyLayout`, `createSecondarySidebar`, and `restoreMainDrawerFromDom`.

## Element Polling (`dom/wait-for.ts`)

```typescript
waitForElement<T>(getElement: () => T | null, label: string, maxFrames?: number): Promise<T | null>
```

Polls via `requestAnimationFrame` until the element appears or ~5s elapses (300 frames at 60fps). Used for late-mounting elements (main wrapper, sidebar).
