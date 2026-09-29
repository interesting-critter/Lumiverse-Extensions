# Chat Reflow &amp; Taskbar Mode Strip Gutters

Two separate systems own different surfaces. Do not merge them.

| System | Setting | What it does | Consumers |
|--------|---------|--------------|-----------|
| **Strip gutters** | `taskbarMode` (+ outer-edge) | Permanent **Welcome/Landing** bounds = **pin-strip width only** (56px). Open drawers **overlay** Welcome. | `[data-component="LandingPage"]` via static CSS |
| **Chat reflow** | `chatReflow` | Open-drawer (and taskbar closed-strip) margins on the **chat column** only (with transition). | `[class*="_chatColumn_"]` only |

## Policy matrix

| taskbarMode (desktop) | chatReflow | Strip gutters | Chat reflow |
|------------------------------|------------|---------------|-------------|
| OFF | OFF | none | none |
| OFF | ON | none | Classic host open-drawer widths on chat column |
| ON | OFF | strip on Landing pin edges | none (chat may sit under pin strips) |
| ON | ON | strip on Landing only | **Active**: mirror open width / closed strip reserve; secondary open or strip |

Mobile (≤600px): both clear / no-op.

---

## Strip gutters (`src/sidebar/strip-gutter.ts`)

Owned by **taskbar mode**, not by chat reflow.

### Behavior

When taskbar mode is effective on desktop:

1. Main edge → reserve `TAB_LIST_WIDTH_PX` (56)
2. Opposite edge → 56 only if a secondary tab list exists
3. Map to left/right from main drawer side
4. Dock composition: `extra = max(0, stripBase - dockInset)` per side (overlap, not sum)
5. **Never** use open-drawer / mirror open width (Welcome only)

### CSS

```css
html.sidebar-ux-strip-gutters [data-component="LandingPage"] {
  margin-left: var(--sidebar-ux-strip-l, 0px) !important;
  margin-right: var(--sidebar-ux-strip-r, 0px) !important;
  /* no transition — stable chrome, not reflow lag */
}
```

Vars live on `document.documentElement`. Chat is **not** a strip-gutter consumer so reflow CSS is not overridden by the more-specific strip selector.

### When updated

- taskbar mode mount / apply / teardown  
- secondary list create/destroy  
- main side change (pin reconcile)  
- dock style changes on `[data-app-root]`  
- viewport cross 600px  

Not observed: drawer/mirror open width.

---

## Chat reflow (`src/chat/reflow.ts`)

### CSS injection

```css
[class*="_chatColumn_"] {
  margin-left: var(--sidebar-ux-chat-ml, 0px) !important;
  margin-right: var(--sidebar-ux-chat-mr, 0px) !important;
  transition: margin 0.35s cubic-bezier(0.4, 0, 0.2, 1) !important;
}
```

Welcome/Landing is **not** a reflow consumer.

### Margin calculation (`updateChatReflow`)

1. Mobile → clear + return  
2. Main width:  
   - **Main mirror active** (taskbar mode): open → `MAIN_MIRROR_WIDTH_VAR` (fallback 420); closed → `TAB_LIST_WIDTH_PX`  
   - **Else**: host `isMainDrawerOpen` ? live width : 0; if taskbar mode effective and still 0 → strip reserve  
3. Secondary: open → `--sidebar-ux-secondary-w` (fallback 420); if closed + taskbar mode + secondary list → strip reserve  
4. Subtract dock insets per side  
5. Write `--sidebar-ux-chat-ml/mr` on the **chat column element**

### Observers

`startReflowObserver()` (gated on `chatReflow`):

1. Main wrapper class/style → open/close  
2. App style → dock insets  
3. App childList → when chat column appears (SPA navigate into chat)  
4. matchMedia 600px → clear on cross-down, recompute on cross-up  
5. Button tagger (co-located lifecycle)

Also: secondary open/close, main-mirror `bumpReflow()`, taskbar mode apply, and feature toggle call `updateChatReflow()` directly.

`scheduleReflow()` coalesces via `requestAnimationFrame`.

### Mobile

- Early-return + clear  
- Injected CSS zeros margins at ≤600px  
- Cross-down clears leftover chat vars  

### Button tagging

Co-located with reflow (same `chatReflow` lifecycle). `tagMainSidebarButtons()` / `startTagObserver()` tag extension tab buttons with `data-tab-id`.
