# Changelog

## 1.2.1

- Split the Extras controls into two independent commands: **Auto-Close: On/Off** and **Auto-Close widget: On/Off**.
- Kept the Auto-Close command synchronized when the floating widget toggles the feature.
- Centered the curly-quote icon and removed the redundant status dot.
- Kept the floating widget hidden by default.

## 1.2.0

- Added an optional draggable global Auto-Close toggle using Lumiverse's official Float Widget API.
- Kept the floating widget hidden by default.
- Changed the Extras popover action to show or hide the widget and report the current On/Off state.
- Added a curly-quote button with accent-colored enabled and muted disabled states.
- Added a direct-toggle fallback when Float Widgets are unavailable.
- Added the required `ui_panels` permission; no content, generation, network, or backend permissions are requested.

## 1.1.0

- Expanded auto-close from the chat composer to all visible editable textareas and supported text inputs.
- Added support for character editors, lorebook fields, prompt/preset editors, expanded text editors, and React-mounted modal fields.
- Added conservative support for compatible `contenteditable` surfaces.
- Preserved controlled React field synchronization through native value setters and bubbling input events.
- Added explicit exclusions for password, disabled, read-only, hidden, inert, and opted-out fields.
- Added `data-lumi-autoclose="off"` as a per-element or ancestor opt-out.
- Kept the existing global session toggle in the chat input-bar Extras menu.
