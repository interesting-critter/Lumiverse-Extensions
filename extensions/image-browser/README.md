<!-- Mirror of upstream documentation. Everything below the metadata table is the original file, unmodified. -->

| Field | Value |
| --- | --- |
| Extension | Image Browser |
| Source repository | https://github.com/CreativityPod/lumiverse-image-browser |
| Original link (from Lumiverse-Extensions README) | https://github.com/CreativityPod/lumiverse-image-browser |
| Upstream path | `README.md` |
| Retrieved | 2026-09-29 @ `main` (`0ecd162`) |

---

# Lumiverse Image Browser

Browse, preview, select, and safely remove unreferenced images stored by Lumiverse without opening a shell in the Docker container.

## Features

- Wide, responsive thumbnail browser in a native Lumiverse modal
- Drawer launcher, command-palette entry, and chat Extras action
- Draggable floating image icon that opens the browser directly and remembers its position
- Custom violet-to-blue stacked-photo SVG: 25 px in the drawer and 32 px in the floating widget
- **Show Widget** switch in the drawer; visibility changes immediately and is saved per account
- Paginated browsing with All, Generated, and Non-generated views and correctly filtered totals
- Remembers the last successfully loaded page for the signed-in Lumiverse account
- Remembers the selected image-type view and recently confirmed reference badges
- Full-resolution, contained image previews; stored videos use native controls
- Videos have a **▶ Video** badge on their thumbnails, including when a still poster is shown
- Missing dimensions are read from the original media when its preview loads and appear on its card for the current browser session
- Multi-selection across loaded pages
- Safe bulk cleanup through Lumiverse's existing `unused=true` deletion path
- Referenced images remain stored and are marked as protected after a cleanup attempt

## Safety boundary

The extension requests the privileged `images` permission for image access and `ui_panels` for the optional floating widget. It never calls the force-delete Spindle methods. A cleanup request is sent separately for each selected asset, and Lumiverse's own `deleteImageIfUnreferenced()` check decides whether the original, thumbnails, and database row may be removed.

Reference status is not currently exposed as a non-destructive Spindle query. Images therefore begin with unknown status; an image that Lumiverse refuses to delete is marked **Referenced** and that observation is cached for the signed-in account for up to 90 days (at most 2,000 records). The badge tooltip explains that the cached status may have changed and the image can be checked again.

## Account storage

The last page, image filter, widget visibility, and reference cache are persisted through `spindle.userStorage` in `state.json` under `{DATA_DIR}/users/{userId}/extensions/image_browser/`. The backend uses the authenticated sender's user ID for every read and write, including globally installed extensions. Each account has separate state, available on its other browsers and devices when Image Browser is reopened. Updates are serialized per account and reference changes are merged so concurrent tabs do not overwrite unrelated entries; the last saved value for each preference wins.

Image Browser does not directly read or write browser localStorage. Lumiverse manages the floating widget's position with a stable geometry key. Existing browser preferences are not migrated. If storage is unavailable, browsing and safe deletion remain available with a visible warning; reopen the browser to retry loading preferences.

## Install

Install the extension from:

```text
https://github.com/CreativityPod/lumiverse-image-browser
```

Then enable it and grant the **Images** permission. Grant **UI panels** to use the floating widget, which is shown by default. Click its image icon to open Image Browser, drag it to reposition it, or turn off **Show Widget** in the drawer to hide it. The drawer launcher remains available if UI panels is not granted.

The extension targets Lumiverse **1.1.6 or newer**, matching the host APIs used and verified during development.

For local development, place or clone the repository under Lumiverse's `data/extensions/` directory and use **Extensions → Add Extension → Import Local**.

## Development

The project is dependency-free and uses Node.js 20 or later.

```bash
npm run check
```

`npm run build` emits a self-contained frontend bundle, the backend entry, and a standalone `image-browser.svg` into `dist/`. The icon's editable vector source is `src/icon.js`; each UI surface uses its own size and gradient ID. Commit the built files before publishing because Lumiverse installs the repository contents directly.

## Known boundaries

- Accurate reference badges cannot be displayed before a safe deletion attempt with the current public Spindle API.
- Search filters the currently loaded page; use the page controls to browse the rest of the collection.
- The extension uses Lumiverse's authenticated image URLs and must run inside the signed-in Lumiverse application.
