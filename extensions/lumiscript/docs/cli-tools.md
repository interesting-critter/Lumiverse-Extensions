# CLI Tools

LumiScript ships two command-line tools under `scripts/` for working with packs outside of the app:

- **`pack2js`** — expands a `.lumiscript.zip` pack (or a bare `pack.json`) into a directory of individual `.js` files plus a `manifest.json`.
- **`js2pack`** — the inverse: builds a `.lumiscript.zip` pack from a directory of `.js` files.

Both are Bun scripts. They run from the repo root and share validation code with the in-app importer (same Zod schemas), so output produced by one tool is always accepted by the other.

## Why these exist

The in-app Script Manager is the primary authoring surface and is sufficient for most work. The CLI tools exist for the cases it doesn't cover well:

- **Editing in an external editor.** Monaco (the in-app editor) is fine for small edits; for larger refactors, multi-file search/replace, or just a preferred editor setup, you want files on disk.
- **Version control.** A directory of `.js` files plus a manifest gives meaningful diffs in git — one script per file, code changes separate from metadata changes.
- **Grep and static tooling.** Ripgrep, ESLint, Prettier, and similar tools operate on files. Compact packs hide everything behind one JSON blob.
- **Hand-authoring packs.** Write scripts in a directory, let `js2pack` assemble the distributable archive.

## Round-trip workflow

### Export-edit-import

1. In the Script Manager, use the `↓` button to export your scripts as a `.lumiscript.zip`.
2. Run `pack2js` on the archive:
   ```sh
   bun scripts/pack2js.ts my-pack.lumiscript.zip ./work-dir/
   ```
3. Edit the `.js` files in `./work-dir/` using your preferred editor. Optionally edit `manifest.json` to change names, triggers, bindings, folders, or metadata.
4. Run `js2pack` to rebuild the pack:
   ```sh
   bun scripts/js2pack.ts ./work-dir/ --output ./my-pack.lumiscript.zip
   ```
5. In the Script Manager, use the `↑` button to import the rebuilt archive. Imported scripts land with `enabled: false` and `allowDangerous: false` — review before enabling.

When `manifest.json` is present, `js2pack` uses it as the source of truth and the round-trip is lossless.

### Author from scratch

1. Create a directory with `.js` files. Each file can carry optional `// @...` frontmatter directives at the top (see the reference below).
2. Run `js2pack` to assemble:
   ```sh
   bun scripts/js2pack.ts ./my-pack/
   ```
3. Import the resulting `.lumiscript.zip` via the Script Manager.

This path (no `manifest.json`) is simpler for new work but cannot express `bindings` — see [Limitations](#limitations).

---

## `pack2js` — pack to directory

### Usage

```
bun scripts/pack2js.ts <input> <output-dir> [--force]
bun run pack2js       <input> <output-dir> [--force]
```

Both forms are equivalent — `bun run pack2js` is a `package.json` alias.

### Arguments

- **`<input>`** — Path to a `.lumiscript.zip` archive or a bare `pack.json` file. Format is auto-detected by extension.
- **`<output-dir>`** — Directory to write the expanded contents into. Refuses to overwrite an existing directory unless `--force` is passed.

### Options

- **`--force`** — Overwrite an existing output directory. Pre-existing files not produced by the current run are left in place and listed as a warning (capped at 10 examples).

### Output structure

```
<output-dir>/
├── manifest.json       Lossless metadata. See format reference below.
└── <slug>.js           One file per script. Code copied verbatim — no
                        added frontmatter, no mutation of your comments.
```

Filenames are slugs of the script's name: lowercase, diacritics stripped, non-alphanumerics collapsed to `-`. Collisions get `-2`, `-3`, … suffixes. Empty-slug names (all punctuation, etc.) fall back to `script`.

### Examples

```sh
# Expand a zipped pack
bun scripts/pack2js.ts ./my-pack.lumiscript.zip ./work-dir/

# Expand a bare pack.json
bun scripts/pack2js.ts ./pack.json ./work-dir/

# Overwrite an existing directory
bun scripts/pack2js.ts ./my-pack.lumiscript.zip ./work-dir/ --force
```

---

## `js2pack` — directory to pack

### Usage

```
bun scripts/js2pack.ts <directory> [--name <pack-name>] [--output <path>]
bun run js2pack        <directory> [--name <pack-name>] [--output <path>]
```

Both forms are equivalent — `bun run js2pack` is a `package.json` alias.

### Arguments

- **`<directory>`** — Path to a folder containing `.js` script files. May optionally contain a `manifest.json` alongside them (see modes below).

### Options

- **`--name <name>`** — Pack name used in the default output filename. Default: the directory's base name.
- **`--output <path>`** — Output path for the `.lumiscript.zip`. Default: `./<name>.lumiscript.zip` in the current working directory.

### Two modes

`js2pack` auto-detects its input mode based on directory contents. The first line of its output announces which mode it's using.

#### Manifest mode (preferred — lossless)

Triggered when `manifest.json` exists in the input directory. The manifest is the source of truth for all metadata; each entry's `file` field points at the `.js` file containing the code. Any `// @...` frontmatter in the `.js` files is ignored in this mode — the manifest wins entirely.

This is the mode produced by `pack2js` output, and the only way to round-trip `bindings` (which can't be expressed as frontmatter directives).

`js2pack` warns about any `.js` files in the directory that are not referenced by the manifest — these are orphans and end up excluded from the built pack.

#### Frontmatter mode (hand-authoring fallback)

Used when no `manifest.json` is present. Each `.js` file's leading `// @...` comment block supplies metadata. Missing directives fall back to defaults:

- `name` — derived from filename (`foo.js` → `"foo"`).
- `type` — `"trigger"`.
- `triggers`, `folder`, `metadata.*` — absent.
- `bindings` — always absent (not expressible as frontmatter).

This mode is useful when hand-authoring a pack; if you need bindings, start from a manifest-mode directory (e.g., export one script via `pack2js` as a template).

### Frontmatter directive reference

Frontmatter directives are `//` line comments at the very top of a `.js` file. Parsing stops at the first non-comment, non-blank line.

| Directive      | Type       | Notes                                                                                      |
|---             |---         |---                                                                                          |
| `@name`        | string     | Display name. Defaults to the filename (without `.js`).                                    |
| `@type`        | `trigger`/`library` | Script type. Default `trigger`.                                                    |
| `@triggers`    | comma-separated list | Lumiverse event names (`MESSAGE_SENT`, `ls:startup`, etc.).                       |
| `@folder`      | string     | Sidebar folder label.                                                                      |
| `@description` | string     | Shown in the Script Manager metadata panel.                                                |
| `@author`      | string     | Metadata author field.                                                                     |
| `@version`     | string     | Semantic version string.                                                                   |
| `@tags`        | comma-separated list | Searchable tags.                                                                 |

Frontmatter lines are regular JS comments and remain in the script after import — they don't need to be stripped before use.

### Examples

```sh
# Build from a directory with manifest.json (manifest mode)
bun scripts/js2pack.ts ./work-dir/

# Build from a hand-authored directory (frontmatter mode)
bun scripts/js2pack.ts ./my-pack/

# Override the pack name and output path
bun scripts/js2pack.ts ./my-pack/ --name shared-utilities --output ./dist/shared-utilities.lumiscript.zip
```

---

## `manifest.json` format reference

Produced by `pack2js`; consumed by `js2pack` in manifest mode. Hand-authorable if you know the shape. Validated by `ManifestSchema` in `src/utils/pack-schema.ts` — invalid manifests are rejected with a Zod error before any file writes happen.

```jsonc
{
  "format": "lumiscript-manifest-v1",
  // Provenance fields — optional, for traceability.
  "sourcePack":   "my-pack.lumiscript.zip",
  "sourceFormat": "lumiscript-pack-v1",
  "exportedAt":   "2026-04-17T00:00:00.000Z",
  "convertedAt":  "2026-04-17T06:05:50.954Z",

  // 1..100 script entries.
  "scripts": [
    {
      "name":     "Dice Roller",
      "file":     "dice-roller.js",           // relative to the manifest's directory
      "type":     "trigger",                  // "trigger" | "library"
      "triggers": ["MESSAGE_SENT", "ls:startup"],
      "bindings": [
        { "type": "character", "characterId": "abc-123", "displayName": "Aphrodite" },
        { "type": "chat",      "chatId":      "def-456", "displayName": "Greeting 1/2" }
      ],
      "folder":   "Demos",
      "metadata": {
        "description": "Rolls dice",
        "author":      "mindbound",
        "version":     "1.0.0",
        "tags":        ["demo", "dice"]
      }
    }
    // ...
  ]
}
```

All fields under `scripts[n]` except `name`, `file`, and `type` are optional and omitted from the emitted manifest when empty or absent.

## Limitations

- **Frontmatter mode can't express `bindings`.** If you author packs by hand in frontmatter mode, any bindings set in the Script Manager before export — or that you intend to add — need to be either hand-written into `manifest.json` or set up via the in-app bindings editor after import.
- **Maximum 100 scripts per pack** (enforced by schema). Matches the in-app import limit.
- **Non-recursive directory scan.** `js2pack` reads `.js` files from the top level of the input directory only; subdirectories are ignored. Matches how `pack2js` emits a flat layout.
- **No JavaScript execution by either tool.** Scripts are treated as opaque text; they aren't parsed, linted, or run. Syntax errors in a `.js` file will reach the user's Script Manager intact.
- **Timestamps are regenerated.** Every `js2pack` run writes a fresh `exportedAt`. The timestamp isn't considered part of the pack's semantic identity.

## Troubleshooting

| Symptom                                                          | Likely cause                                                                                              |
|---                                                               |---                                                                                                        |
| `Error: output path already exists`                              | `pack2js` won't overwrite by default. Pass `--force` or choose a different path.                          |
| `Error: manifest entry "…" references missing file: …`           | The manifest's `file` field points at a `.js` that isn't in the directory. Rename, restore, or edit the manifest. |
| `Error: manifest.json failed schema validation`                  | Check the Zod error detail. Usually a typo in `type` (must be `"trigger"` or `"library"`) or a malformed binding entry. |
| `No .js files found in <dir>`                                    | `js2pack` in frontmatter mode with an empty directory. Add at least one `.js` file, or a `manifest.json` pointing at one. |
| Scripts re-imported with `enabled: false` and `allowDangerous: false` | Expected — the in-app importer resets both flags on every import for safety. Re-enable manually after review. |
