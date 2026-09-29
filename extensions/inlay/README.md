<!-- Mirror of upstream documentation. Everything below the metadata table is the original file, unmodified. -->

| Field | Value |
| --- | --- |
| Extension | Inlay |
| Source repository | https://github.com/japolino/inlay-illustrator |
| Original link (from Lumiverse-Extensions README) | https://github.com/japolino/inlay-illustrator |
| Upstream path | `README.md` |
| Retrieved | 2026-09-29 @ `stable` (`68cc000`) |

---

# Inlay Illustrator

Lumiverse extension for persistent, context-aware character image generation.

## Main features

- Persistent character image generation across a chat
- Dynamic visual memory for returning characters, including base attire
- One-time, first-generation avatar vision enrichment when the selected parser model accepts image input; observed details complement but never rewrite card-derived memory
- Image prompts generated from current and recent chat context
- Optional cinematic cover/key-visual prompt generated for the whole message and rendered above the prose, with its own display width and max-height settings
- Progressive illustration slots that fill as each image finishes without changing paragraph order
- Per-chat FIFO scheduling, scoped progress, duplicate suppression, and cooperative cancellation
- Manual Creative concept exploration, visual-novel-style Static, action-focused Dynamic, or Original-style single-character Asset generation
- Optional Adaptive Mode that selects Creative, Static, or Dynamic independently for each illustration; Asset remains manual-only
- Identity-safe Creative candidate slates for objects, environments, shadows, silhouettes, spatial details, and non-identifying fragments
- Atomic Anima composition, structured environments, and ComfyUI-friendly prompt output
- Image lightbox with prompt metadata, fresh-seed rerolls, and per-image sidecar reruns
- Optional activated-lorebook context with macro resolution, compact first-pass references, and full-context retry
- Configurable maximum parser token budget with automatic allowances for reasoning-heavy sidecars

## Install from source

Clone or download this repository into your Lumiverse extension data folder:

```powershell
data\extensions\inlay_illustrator\repo
```

The built extension files are included in `dist/`, so no build step is required for normal installation.

## Setup

1. In **Parser and context**, select a parser connection. Leave the model field empty to use that connection's default model.
2. Configure the image provider in Lumiverse's image-generation settings. When no extension-specific connection is saved, Inlay uses the account default or first available image connection.
3. **Auto generate** is enabled by default after setup. Disable it for manual-only use with **Generate latest**. **Cancel** cooperatively stops queued or running work.

If generation does not start, check the panel status first. A setup message means the parser connection is missing. A missing-connection error means a previously selected image connection was deleted or disconnected. Invalid parser parameters must be corrected to a JSON object. Enable **Debug logging** for detailed `[Inlay:stage]` entries.

## Development

```powershell
$env:BUN_INSTALL_CACHE_DIR = "$PWD\.cache\bun"
bun install --frozen-lockfile
bun run verify
bun run build
```

`verify` runs the Bun test suite and strict TypeScript checking. `build` type-checks
the runtime sources, then bundles the backend and frontend entrypoints into `dist/`.
