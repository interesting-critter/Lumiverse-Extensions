<!-- Mirror of upstream documentation. Everything below the metadata table is the original file, unmodified. -->

| Field | Value |
| --- | --- |
| Extension | Auto Close |
| Source repository | https://github.com/kittyafterdark/Lumi-AutoClose |
| Original link (from Lumiverse-Extensions README) | https://github.com/kittyafterdark/Lumi-AutoClose |
| Upstream path | `README.md` |
| Retrieved | 2026-09-29 @ `main` (`b988d20`) |

---

# Lumi Auto-Close

A tiny frontend Lumiverse/Spindle extension that auto-closes common dialogue, Markdown, and bracket markers across Lumiverse text fields.

## Where it works

Auto-Close applies to visible, editable:

- Chat composers
- Character editor fields and modals
- Lorebook metadata and entry content
- Prompt and preset editors
- Expanded text editors
- Standard single-line text fields
- Compatible `contenteditable` editing surfaces

Password fields, disabled/read-only fields, hidden backing inputs, and elements marked with `data-lumi-autoclose="off"` are ignored.

## Included pairs

- `""` straight dialogue quotes
- `“”` curly dialogue quotes
- `**` action / emphasis markers
- `` `` `` inline-code markers
- `()` parentheses
- `[]` square brackets
- `{}` braces

Typing an opening marker inserts both characters and leaves the caret between them. Typing the closer when it is already under the caret moves across it instead of duplicating it. Pressing Backspace between an empty pair removes both characters. Selecting text and typing an opening marker wraps the selection.

## Global controls and widget

Version 1.2 adds a small draggable curly-quote widget that can toggle Auto-Close from anywhere in Lumiverse. The widget is **hidden by default**.

The chat input bar's **Extras** popover contains two independent commands:

- **Auto-Close: On/Off** toggles the feature directly, whether or not the widget is visible.
- **Auto-Close widget: On/Off** only shows or hides the floating widget.

Clicking the floating curly quote toggles Auto-Close globally through the same state path as the first command, so its label updates immediately. The bright/accent button means Auto-Close is on; the muted button means it is off. The widget follows Lumiverse route changes and can be dragged to a convenient screen edge.

## Permission

The extension requests `ui_panels` only so it can use Lumiverse's official Float Widget API. It does not request character, lorebook, prompt, generation, network, or backend access.

## Install

Install via the Lumiverse Extensions tab, pasting this repository's link in the installation field:
```https://github.com/kittyafterdark/Lumi-AutoClose```

## Customize pairs

Edit the `PAIRS` object at the top of `src/frontend.ts`, then rebuild. Avoid adding straight apostrophes (`'`) unless you enjoy contractions fighting back.

## Compatibility notes

The extension uses one delegated `beforeinput` listener, so fields mounted later by React portals or lazy-loaded editor screens are handled automatically. Native value setters and bubbling input events keep controlled React fields synchronized.

`contenteditable` support is intentionally conservative and plain-text oriented. Native Lumiverse textareas and text inputs are the primary supported path.
