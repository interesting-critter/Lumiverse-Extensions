# Mobile Support

## Overview

Canvas provides mobile-specific behavior for viewports <= 600px. The primary constraint: only one sidebar can be open at a time.

## Viewport Detection

`isMobileViewport()` — `window.matchMedia('(max-width: 600px)').matches`

**Distinct from**: `isPointerResizeActive()` in `resize/handles.ts` which uses `matchMedia('(pointer: coarse)')` for resize-handle suppression. Mobile viewport detection is for layout decisions; pointer detection is for interaction decisions.

`isHostMobileDrawerViewport()` — `isMobileViewport() || (pointer: coarse)`. Matches Lumiverse's `useIsMobile` hook. Used **only** for host main drawer full-width forcing — NOT for layout/exclusion/main-mirror decisions (which use the stricter `isMobileViewport()`).

## Mutual Exclusion (`sidebar/mobile-exclusion.ts`)

When a sidebar opens on mobile, the other must close:

- **Primary opens**: Close secondary silently (`closeSecondarySidebar({ silent: true })` — preserves `secondary.open = true` in layout.json)
- **Secondary opens**: Close primary by clicking its toggle button (host-owned, no API)

### Body CSS Classes

- `canvas-ux-mobile-primary-open` — set when primary is open on mobile
- `canvas-ux-mobile-secondary-open` — set when secondary is open on mobile

CSS rules in `styles.ts` use these classes to:
- Hide the inactive sidebar's drawer tab button
- Show a backdrop overlay behind the open drawer

### Backdrop

```css
body.canvas-ux-mobile-secondary-open::before {
  content: '';
  position: fixed;
  inset: 0;
  background: var(--lumiverse-fill-heavy);
  z-index: 9989;
  pointer-events: none;
}
```

Purely visual — touch interactions pass through.

## Scaled Viewport Width (Host Alignment)

Lumiverse applies `zoom: var(--lumiverse-ui-scale)` to top-level body children. On mobile the host drawer uses:

```css
var(--app-scaled-viewport-width, calc(100vw / var(--lumiverse-ui-scale, 1)))
```

Canvas previously used raw `window.innerWidth` px for the mobile drawer width, which diverges from the host's CSS-resolved width under zoom ≠ 1. Now Canvas uses the **same CSS expression** for the drawer's inline `width`, so both drawers agree on the rendered size regardless of zoom.

`syncCssVarToDrawerWidth()` reads `drawer.offsetWidth` (the actual rendered size after the CSS expression resolves) and stores it in `--sidebar-ux-secondary-w` so JS transform computation stays in sync. When the drawer isn't mounted yet, it falls back to `Math.round(window.innerWidth / uiScale)`.

## CSS Variable Sync

On mobile, the CSS variable `--sidebar-ux-secondary-w` is overwritten to match the **measured** drawer width (via `drawer.offsetWidth`), not `window.innerWidth`. On desktop, the saved value is restored.

`syncCssVarToDrawerWidth()` handles save/restore with `_desktopCssVarValue` cache.

## Viewport Crossing

`startMobileExclusion()` registers a `matchMedia` change listener:

- **Cross-down** (desktop → mobile): Close secondary silently, update body classes
- **Cross-up** (mobile → desktop): Clear body classes, restore CSS variable

Additionally, a `resize` listener keeps the CSS variable and wrapper transform in sync on mobile when the user drags the viewport (matchMedia only fires once per boundary crossing).

## Drawer Width on Mobile

- `_updateDrawerWidth()`: On mobile, forces `drawer.style.width = 'calc(var(--app-scaled-viewport-width, calc(100vw / var(--lumiverse-ui-scale, 1))) + 1px)'`. On desktop, restores `var(--sidebar-ux-secondary-w, 420px)`.
- Cancels any in-flight wrapper animation before updating.
- Updates the wrapper's `translateX` to match the new CSS var.

## Host Main Drawer Full-Bleed on Larger Mobile

The host main drawer is forced to full viewport width when `isHostMobileDrawerViewport()` returns true:

- **≤600px**: Canvas clears any `!important` width override from `restoreMainDrawerFromDom` (set by previous desktop sessions) so the host CSS media query enforcing `--drawer-panel-w: calc(... + 1px)` takes effect.
- **>600px with coarse pointer** (tablets, phone landscape): The host treats the viewport as mobile (backdrop) but its `@media (max-width: 600px)` CSS does not fire. Canvas JS forces `--drawer-panel-w` to the scaled full-viewport `+1px` expression on the wrapper with `!important`.

This is synchronized on viewport-cross (`startMobileExclusion` matchMedia `change` handler), resize events, and one-shot mount reconciliation via `syncHostMainDrawerToMobileWidth()` in `mobile-exclusion.ts`.

Key guard points:
- `restoreMainDrawerFromDom` — three width-stamp paths gated with `!isHostMobileDrawerViewport()`. When mobile, the saved desktop `clampedWidth` is not stamped and any existing inline width/`--drawer-panel-w` is cleared.
- `teardownMainMirror` — the `--drawer-panel-w` stamp from mirror width is gated with `!isHostMobileDrawerViewport()` so cross-down to mobile does not underfill.
- `resize/handles.ts` — handles never mount on touch devices (`isPointerResizeActive()`), so they don't set width on mobile.
- Resize listener in `startMobileExclusion` calls `syncHostMainDrawerToMobileWidth()` on every resize frame.

## Mobile Full-Bleed Width: +1px Oversize

On mobile, both drawers (Canvas secondary and host main) use the **scaled viewport width + 1px** to fill the visual viewport. Under fractional zoom/AA, `--app-scaled-viewport-width` can resolve ~1px short, leaving a 1px underfill gap when the drawer is open (`translateX(0)`).

The +1px is applied via `calc(... + 1px)` at three sites:

1. **Canvas secondary drawer** — `drawer-shell.ts` `fullViewportWidth` branch sets `width: calc(var(--app-scaled-viewport-width, ...) + 1px)`.
2. **Canvas secondary drawer (viewport cross)** — `mobile-exclusion.ts` `_updateDrawerWidth()` sets `drawer.style.width` to the same `calc(... + 1px)` expression.
3. **Host main drawer** — CSS rule injected via `styles.ts` `SECONDARY_MOBILE_CSS` sets `--drawer-panel-w` on `[class*="wrapperLeft"], [class*="wrapperRight"]` to the `calc(... + 1px)` expression with `!important`.

The closed path already measures `offsetWidth` and adds a +1 transform overshoot, so the wider drawer still fully hides when closed — no change needed there.

## Shadow Suppression When Closed

When the secondary drawer is closed (`data-drawer-open="false"` on the wrapper), CSS forces `box-shadow: none !important` on the drawer element. Without this, the shadow spread from `var(--lumiverse-shadow-xl)` could bleed 4–24px into the viewport past the closed edge. The data attribute is toggled by `openSecondarySidebar` / `closeSecondarySidebar`.

Default `drawerShadowsMobile` is `false` but users can enable it — shadow suppression applies regardless of the setting when the drawer is off-screen.

## Mobile Tab List

CSS in `styles.ts` restructures the secondary tab list on mobile:
- `flex-direction: row` (horizontal)
- `overflow-x: auto` (scrollable)
- Tab buttons: 52x48px uniform size
- Active indicator: bottom underline instead of left border
- Bottom border on the tab list

## Mobile-Specific Behavior in Other Modules

- `assignToSecondary`: does not auto-open secondary drawer on mobile
- `assignTab` (built-in path): does not auto-open secondary on mobile
- `placementFirstMoveByLiveId` (right-click "Move to …", both context menus): does not auto-open the destination drawer on mobile — desktop keeps the open-so-the-move-is-visible behavior
- `activation-handoff`: Part C (destination activation) is skipped on mobile
- `applyTabListPosition`: no-op on mobile (CSS forces layout)
- `resize/handles`: no handles on mobile
- `chat/reflow`: complete no-op on mobile
- `main-persist/restoreMainDrawerFromDom`: skips width override on mobile
