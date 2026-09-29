# Palette floating widget

The widget is a Zen-inspired bridge that remains accessible even when a modal, popover, drawer, or temporary surface blocks the normal Spindle drawer.

## Current prototype

The widget and the drawer share one picker, one selection, one guide controller, one project store, and one Palette UI instance.

Compact widget:
- current project and selected target in readable human labels
- compact Boost palette/type status with direct color Shuffle
- one-shot Pick (persistent multi-pick belongs to the main editor)
- Guides toggle
- Zap selector popup that remains open across one-shot target picks
- Code selector popup
- Float full editor

**Zap** does not destructively remove DOM. It creates an editable `Visibility → Gone` semantic packet for the chosen scope. “Related” uses the next useful reusable scope when available.

**Code** mirrors the useful part of Zen's selector helper: choose the current or related selector and Palette inserts a ready-to-edit rule into Custom CSS, then opens Code in the floating editor.

**Float** physically reparents the same live Palette mount into a high-z draggable editor window. It does not clone the editor. This matters because users can pick and edit modal content without closing the modal just to reach the Spindle drawer. **Return** moves that exact mount back to its original Spindle sidebar host. The word Dock is reserved for Lumiverse’s persistent dock UI.

The picker treats both the compact widget and floating editor as Palette-owned UI, so neither can accidentally become the selected page target. The compact widget is a persistent `document.body` portal and, where supported, a non-modal manual popover in the browser top layer so ordinary mobile drawer/tab stacking cannot cover it. Palette owns its fixed-position drag/persistence, opts this body-direct utility out of Lumiverse's global body-child UI scale, and keeps geometry in rendered viewport space so controls and pointer hitboxes stay aligned. Native reference catalogs stay hidden while the full editor is floated; they remain available in the normal drawer.

The collapsed mini widget has an explicit visibility menu: **right-click on desktop** or **long-press on touch** to open it, then choose **Hide mini widget**. The square widget control in the Design workbar is a real visibility toggle in both directions, so it restores a hidden widget and can hide a visible one again. Visibility is remembered locally.

## Direction

Keep the compact widget a bridge rather than a second government-form inspector. Good future additions are tiny palette handles, recent colors, guide/state indicators, and edge snapping. Full packet editing, project management, resource browsers, and selector diagnostics belong in the real editor that the widget can float.

On mobile, the compact panel stays edge-safe and the floated editor becomes a draggable bottom working sheet with three snap heights: peek, work, and full. It is the same live editor at every height, not three UIs. The drag gesture is delta-based from the sheet’s current rendered height, so grabbing the handle never teleports the sheet before it starts following the finger.
