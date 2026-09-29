<!-- Mirror of upstream documentation. Everything below the metadata table is the original file, unmodified. -->

| Field | Value |
| --- | --- |
| Extension | Palette |
| Source repository | https://github.com/kittyafterdark/Palette |
| Original link (from Lumiverse-Extensions README) | https://github.com/kittyafterdark/Palette.git |
| Upstream path | `README.md` |
| Retrieved | 2026-09-29 @ `main` (`a0db704`) |

---

# Palette

**Visual theme authoring for Lumiverse.** Pick the thing you mean, describe the visual intent, and Palette turns it into scoped, reusable CSS.

> Palette is an independent, unofficial community extension for Lumiverse. It is not affiliated with, endorsed by, or supported by the Lumiverse project or its maintainers.

Palette is built around one idea: **Palette knows the CSS; you express the visual intent.** A normal user can resize, recolor, rearrange, crop, fade, group, and reuse real Lumiverse UI without learning selector syntax. Advanced users can still inspect the generated CSS or add a separate Custom CSS layer.

**Public release:** 1.0.5  
**Project schema:** v43  
**Minimum Lumiverse version:** 1.1.6

## Highlights

- **Pick real UI** — persistent element picker, semantic target ladder, Edit Part, Browse Inside, Assistant/User/Both message facets, and stable scoped selectors.
- **Visual style packets** — Background, Image, Mask, Ink, SVG assets, Typography, Border, Corners, Spacing, Shadow, Glass, Size, Layout, Quick Align, Media Flow, Position, Transform, Visibility, pseudo surfaces, and more.
- **Base + Mobile** — author responsive deltas without hand-writing media queries.
- **States** — Normal, Hover, Active, Focus, and Disabled styling.
- **Smart Guides** — box model, dimensions, layout geometry, positioning, crop/focal guides, and group member overlays.
- **Layout Groups** — arrange real siblings as Row / Column / Grid without reparenting React-owned DOM.
- **Read Style / Read Page** — inspect existing presentation without capturing it; only edited values become Palette-owned CSS.
- **Quick Looks + Style Library** — browse the current target or individual packs inline, then move the full library between fullscreen and Lumiverse’s native left dock.
- **My Styles** — save Targets, Components, or multi-component Bundles and reuse them across Palette projects.
- **Full App Boost** — faithful Primary/Secondary accents plus a Lumiverse-aware material graph for canvas, surfaces, raised/card states, Chat Shell glass, resting controls, hover, and quiet/interactive borders. **Recolor** preserves the source light/dark balance; **Invert brightness** reverses it while keeping the same palette/material relationships. Shuffle colors changes only Primary/Secondary and preserves the selected mode and transform tuning. Native depth survives extreme Brightness/Contrast edits instead of flattening into one slab; paired accent foregrounds stay contrast-safe without becoming accent paint, while neutral/text/status routing, backdrop treatment, and global Typography remain independent.
- **Theme projects** — persistent swatch-based Theme Stash with duplicate/rename/delete workflows.
- **Native handoff** — inspect generated CSS, keep Custom CSS separate, work with native assets, and hand compatible themes back through Lumiverse's public theme bridge.
- **Float / Dock / mobile** — keep Palette reachable while editing modals, drawers, and temporary surfaces.

## Installation

1. Open **Lumiverse → Extensions**.
2. Choose **Install Extension**.
3. Paste the GitHub repository URL for Palette.
4. Review and grant the requested `app_manipulation` and `ui_panels` permissions.
5. Open **Palette** from the Lumiverse sidebar/drawer.

The repository intentionally commits `dist/`; Lumiverse installs the built Spindle runtime directly from the repository.

To update, use Lumiverse's extension update flow after a new Palette release is published.

## First five minutes

1. Open **Design** and turn on **Pick**.
2. Click something in Lumiverse.
3. Use the breadcrumb, **Edit Part**, or **Browse Inside** to reach the visual part you actually mean.
4. Add a style packet. Try **Background**, **Typography**, **Size**, or **Image** first.
5. Switch to **Mobile ≤ 720px** if the phone layout needs a different value.
6. Open **Code** whenever you want to inspect the generated CSS.
7. Open Palette's built-in **Guide** for the full one-page manual and CSS field guide.

## How Palette thinks

Palette stores semantic styling intent, not a blob of generated CSS. The project model is canonical; CSS is deterministic compiler output.

That enables a few important behaviors:

- Reset removes Palette ownership and reveals the underlying cascade.
- Read Style can show observed values without immediately copying them into the stylesheet.
- Packs and Quick Looks remain editable through ordinary Design packets after application.
- Recipe reset can peel only recipe-owned layers while preserving unrelated manual styling.
- My Styles can merge semantic packet types into another project instead of replacing the destination target wholesale.

## Main workspaces

| Workspace | Purpose |
| --- | --- |
| **Design** | Pick mounted UI and author semantic styling. |
| **Themes** | Theme projects, Boost, Typography, Quick Looks, and pack workbenches. |
| **Style Library** | Browse reusable looks and cross-project My Styles. |
| **Code** | Inspect generated CSS, add Custom CSS, and use native theme handoff. |

## Built-in packs

Palette currently ships curated message/composer directions including **Manga**, **Editorial**, **Journal**, and **Visual Novel**. Pack recipes use the same semantic targets, packets, responsive states, groups, and provenance system as normal Design work—there is no separate hidden pack-CSS engine.

## Compatibility notes

Palette deliberately prefers public Lumiverse semantic boundaries (`data-component`, drawer surfaces, composer action identities, native theme APIs) and stable DOM families over generated CSS-module hashes. Some deeper mounted parts still require DOM-scoped fallbacks because Lumiverse does not expose a public semantic ID for every internal wrapper.

That means upstream DOM changes can occasionally require a Palette selector update even when the underlying feature still exists. The built-in Guide's debugging section explains how to distinguish a selector regression from ordinary CSS/layout behavior.

## Known limitations

- **My Styles Bundles do not save Layout Groups yet.** Group collision/provenance semantics need an explicit policy before cross-project groups are safe to merge.
- Some very deep/conditional Lumiverse parts are available through **Browse Inside** rather than a permanent semantic Edit Part entry.
- A target that Lumiverse itself removes with `display:none` may require Palette to explicitly own `display` before other visibility/position styling can matter.
- Pack previews teach the composition but are not a promise of pixel-identical mounted geometry for every custom theme/font/viewport.

## Development

Palette is TypeScript and ships compiled ES2022 bundles in `dist/`.

```bash
bun install
bun run verify
```

Useful scripts:

| Command | What it does |
| --- | --- |
| `bun run build` | Build frontend/backend bundles into `dist/`. |
| `bun run typecheck` | TypeScript check without emitting. |
| `bun test` | Run the test suite. |
| `bun run verify` | Typecheck + tests + build. |

Development receipts and pre-public-release archaeology live under [`docs/development/`](docs/development/) instead of the public README.

## Repository layout

```text
Palette/
├─ spindle.json
├─ dist/                 built installable runtime (committed)
├─ src/                  TypeScript source
├─ tests/                compiler/runtime/persistence tests
├─ GUIDE.md              source for the built-in one-page manual
├─ CHANGELOG.md          public release history
└─ docs/development/     implementation receipts / historical notes
```

## Permissions

Palette currently requests:

| Permission | Why |
| --- | --- |
| `app_manipulation` | Apply scoped visual presentation and interact with the public native theme authoring surface. |
| `ui_panels` | Move the Style Library into Lumiverse’s native resizable left dock without covering the live chat. |

## Privacy and data

Palette stores its projects and reusable styles through Lumiverse's extension storage for the current user. It does not require an external account or a Palette-hosted service.

Image source quality controls only promote recognized Lumiverse image routes when requested by the authored visual treatment. Imported SVGs are sanitized before being stored as project assets.

## License

Palette source code is released under the [MIT License](LICENSE.md), except for third-party material listed in [THIRD_PARTY_NOTICES.md](THIRD_PARTY_NOTICES.md).

Distribution and use alongside Lumiverse are also subject to Lumiverse's own license and community-extension requirements.

## Credits

- Palette drawer icon: **paintbrush by Alum Design**, CC BY 3.0. See [THIRD_PARTY_NOTICES.md](THIRD_PARTY_NOTICES.md).
- Built for the Lumiverse Spindle extension ecosystem using public extension APIs.
