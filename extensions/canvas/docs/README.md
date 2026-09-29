# Canvas Documentation

Documentation for the Canvas extension codebase, optimized for coding agents. Start here to understand the architecture, then dive into specific subsystems.

## Reading Order

1. **[architecture.md](architecture.md)** — High-level overview: what Canvas is, build system, entry points, module graph, key design patterns
2. **[features.md](features.md)** — Feature registry, settings system, settings panel, live-apply dispatch
3. **[sidebar.md](sidebar.md)** — Secondary sidebar: DOM construction, state machine, DrawerObserver, cross-drawer sync, side-change detection, mobile support, animation
4. **[custom-css.md](custom-css.md)** — Theming: stable Canvas selectors (`.sidebar-ux-shell`), Lumiverse tokens, dual-mode host+Canvas recipes, pin-host pitfalls
5. **[tabs.md](tabs.md)** — Tab management: assignment system, button management, active-tab tracking, activation handoff, context menus, button tagging
6. **[persistence.md](persistence.md)** — Layout persistence: storage format, IPC, backend, frontend save/load, main drawer persistence, layout restore
7. **[slash-commands.md](slash-commands.md)** — Slash command system: runtime, registry, intercept, suggest popup, dispatch, intent, DOM utilities, built-in commands, extension API, toast surface
8. **[chat-reflow.md](chat-reflow.md)** — Chat column reflow: margin calculation, observer architecture, button tagging
9. **[dom-layer.md](dom-layer.md)** — DOM helpers: Lumiverse element queries, React fiber access, Zustand store walk, host bridge, selectors, width clamp
10. **[resize-and-drag.md](resize-and-drag.md)** — Resize handles and drawer tab drag: handle structure, drag behavior, drawer tab vertical positioning
11. **[mobile.md](mobile.md)** — Mobile support: viewport detection, mutual exclusion, CSS variable sync, viewport crossing, mobile-specific behaviors

**[pitfalls.md](pitfalls.md)** — Cross-cutting traps: TabKey vs liveId dual-keying, mirror active-key rules, placement-first flow, boot restore placement, host NO-GOs. **Read this before touching tab moves, the main-mirror, or restore.**

## Quick Reference

### Entry Points
- `src/frontend.ts` → `src/setup.ts` — Spindle loader calls `setup(ctx: SpindleFrontendContext)`
- `src/backend.ts` — Bun backend for `layout.json` persistence

### Key Types
- `LayoutState` — persisted drawer state (`types.ts`)
- `CanvasSettings` — all user-togglable settings (`types.ts`)
- `FullCanvasSettings` — `Required<CanvasSettings>` with all fields non-optional (`settings/state.ts`)
- `CanvasFeature` — feature lifecycle hooks (`features/registry.ts`)
- `DrawerTab` — store's tab entry with `id`, `title`, `root`, `iconSvg` (`store/index.ts`)
- `ObservedTab` — DrawerObserver's tab entry (`sidebar/drawer-observer.ts`)
- `SlashCommandDef` — slash command definition (`slash/types.ts`)

### Key Files
- `src/setup.ts` — orchestrator, lifecycle management
- `src/core/model.ts` + `src/core/reduce.ts` — owned layout model: single source of truth for tab placement, order, active, drawers
- `src/recon/dispatch.ts` — dispatch queue, `placementFirstMoveByLiveId` (the move path), `bootstrapFromLayout` (restore + boot placement)
- `src/host/lumiverse/implementation.ts` — `LumiverseHost` (HostPort): observe/place/setOrder/activate against live Lumiverse
- `src/features/registry.ts` — feature registry (add new features here)
- `src/sidebar/secondary.tsx` — secondary sidebar DOM construction + `reassignSecondaryTabsFromModel`
- `src/sidebar/secondary-drawer.ts` — secondary drawer state machine
- `src/sidebar/main-tab-pin.ts` — main-mirror pin: exclusive active key, `userPicked` guard, neighbor handoff
- `src/tabs/assignment.ts` — owned-model facade (TabKey-keyed)
- `src/slash/runtime.ts` — slash command runtime wiring
- `src/layout/persist.ts` — layout persistence + IPC

### State Flow
```
User toggles setting in panel
  → setSettings(patch)                    [settings/state.ts]
    → applySettings(prev, next)           [settings/panel.ts]
      → feature.apply(prev, next, ctx)    [features/registry.ts]
    → refreshSettingsPanel()              [settings/state.ts]
    → persistSettings()                   [settings/state.ts] (100ms debounce)
      → sendToBackend({ type: 'SAVE_LAYOUT', layout })  [layout/persist.ts]
```

### Extension Points
- **Slash commands**: register via `canvas:slash-register` CustomEvent
- **Features**: add `CanvasFeature` to `FEATURES` array in `features/registry.ts`
- **Settings**: add field to `CanvasSettings` interface in `types.ts`, add default in `DEFAULT_CANVAS_SETTINGS`, add toggle in `settings/panel.ts`
- **Custom CSS / themes**: prefer Lumiverse tokens + `.sidebar-ux-shell` / pin hosts — see [custom-css.md](custom-css.md)
