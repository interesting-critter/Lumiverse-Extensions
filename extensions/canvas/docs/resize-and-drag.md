# Resize Handles & Drawer Tab Drag

## Resize Handles (`resize/handles.ts`)

Drag-to-resize handles on both the main and secondary drawers.

### Handle Structure

Each handle is an 8px-wide `div` positioned at the drawer's inner edge (facing the content area). The settings panel UI says "4px grab handle" but the actual rendered width is 8px:
- `position: absolute` within the drawer
- `cursor: col-resize`
- `z-index: 99999`
- `touch-action: none`

### Drag Behavior

1. `pointerdown`: record start position and drawer width
2. `pointermove`: compute delta based on direction, apply `clampSidebarWidth(startWidth + delta)`
3. `pointerup`: persist layout

**Direction encoding**:
- `'right'` = expand on rightward drag (drawer is on left)
- `'left'` = expand on leftward drag (drawer is on right)

**Iframe blocking**: A transparent overlay is placed over the content area during drag to prevent iframes from intercepting pointer events.

**Side-aware positioning**: The handle uses CSS variables (`--drawer-panel-w`) so it tracks the correct edge when tab strip position changes.

### Mobile

Resize handles are suppressed on mobile (`isPointerResizeActive()` checks `matchMedia('(pointer: coarse)')`).

### `refreshResizeHandles()`

Idempotent — mount handles if setting is on and handles are missing; remove if setting is off.

## Drawer Tab Drag (`drawerTabPosition/`)

Vertical drag repositioning for drawer tabs (main + secondary).

### Drag Handler (`drag.ts`)

`installDrawerTabDrag(el, role, onCommit, onLiveUpdate?)`:

1. `pointerdown`: record start Y and current vh offset
2. `pointermove`: convert pixel delta to vh via `pxToClampedVh()`, write to `el.style.marginTop`
3. `pointerup`: call `onCommit(finalVh)` to persist

**Threshold**: 10px dead zone before drag activates (filters tap jitter).

**Click suppression**: Capture-phase click listener installed on first drag threshold crossing, removed on drag-end (deferred to next macrotask to catch synthesized click).

**Live update**: The `onLiveUpdate` callback propagates drag to a mirror element (bidirectional mirror when `mirrorCompactPosition` is on).

### Feature Integration (`features/drawer-tab-position.ts`)

- `init()`: installs drag on the main drawer tab (MutationObserver watches for its appearance)
- `mount()`: installs drag on the secondary drawer tab
- `apply()`: re-applies overrides from settings on diff

**Bidirectional mirror**: When `mirrorCompactPosition` is on, dragging the secondary also moves the main via `onLiveUpdate`. The style observer on the main fires and writes back to the secondary (idempotent).

### Utility Functions

- `pxToClampedVh(deltaPx, viewportHeight, currentVh, min, max)` — convert pixel delta to clamped vh
- `parseVhFromStyle(s)` — parse "12vh", "12.5vh", "12px", or bare number
- `applyDrawerTabPosition(settings, mainTab, secondaryTab)` — apply overrides from settings
