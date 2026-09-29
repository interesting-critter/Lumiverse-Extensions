<!-- Mirror of upstream documentation. Everything below the metadata table is the original file, unmodified. -->

| Field | Value |
| --- | --- |
| Extension | Luminote |
| Source repository | https://github.com/prolix-oc/Luminote |
| Original link (from Lumiverse-Extensions README) | https://github.com/prolix-oc/Luminote |
| Upstream path | `README.md` |
| Retrieved | 2026-09-29 @ `main` (`e6ff86b`) |

---

# Luminote

Luminote is an Obsidian-style vault workspace extension for Lumiverse. It was created by **Coffee** from the Lumiverse Discord server and is published by Prolix OCs with her authorship preserved.

## Original author

Coffee created Luminote, shaped its original product and implementation, and made it available to the Lumiverse community despite being too shy and stubborn to publish it herself — **2 shy to upload it to GitHub, or perhaps even to have a GitHub account.** Every Luminote release must retain this attribution:

> Original extension by Coffee (Lumiverse Discord). Published by Prolix OCs.

See [CREDITS.md](CREDITS.md) for the complete release attribution.

## Development

Install dependencies with `bun install`, then validate a release with:

```sh
bun run release:check
```

This type-checks the source, produces `dist/backend.js` and `dist/frontend.js`, and confirms the Lumiverse manifest is complete, version-aligned, and references the built entry points.

## Releasing

1. Update the shared version in `package.json` and `spindle.json`.
2. Run `bun run release:check`.
3. Publish the project directory with `spindle.json` and the generated `dist/` directory included. Do not include `node_modules/`.
4. Preserve Coffee's original-author attribution in the release listing and any accompanying announcement.

Luminote requests `app_manipulation`, `ui_panels`, `images`, `media`, and `ephemeral_storage` permissions. Its dynamic-code-execution capability declaration is required by bundled dependencies that can trigger Lumiverse's static scanner.

## Floating widget on Lumiverse Desktop

If you pop out the Luminote launcher as a native floating widget, use **Return to page** in the widget's host title bar to dismiss the pop-out. The desktop tray's **Floating Widgets → Return Luminote · Widget 1 to Page** action is another way back. The floating *workspace* in the main Lumiverse page has its own minimize, maximize, restore, and close buttons.

Spindle's floating-widget API does not expose native pop-out minimize, maximize, or close controls. Those require support in Lumiverse Desktop.
