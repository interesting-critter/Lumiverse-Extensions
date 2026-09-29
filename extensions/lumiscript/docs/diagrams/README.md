# Diagrams

Typst + Fletcher sources + their generated SVGs for the diagrams referenced by the markdown docs in `docs/concepts/` and `docs/guides/`.

## Workflow

1. Edit the `.typ` source file for whichever diagram you're changing.
2. Regenerate the matching `.svg`:
   - Single diagram: `typst compile --format svg docs/diagrams/<name>.typ docs/diagrams/<name>.svg`
   - All diagrams:   `bash scripts/build-diagrams.sh`
3. Commit BOTH the `.typ` source and the regenerated `.svg`. The markdown files reference the `.svg` directly via `![alt](path)`, so the SVG needs to be committed for the docs to render anywhere.

## Install Typst

Typst is a single binary. Pick whichever channel you prefer:

- **Binary**: download from <https://github.com/typst/typst/releases>
- **winget**: `winget install Typst.Typst`
- **Homebrew**: `brew install typst`
- **Cargo**: `cargo install --locked typst-cli`

The Fletcher diagram package (`@preview/fletcher:0.5.8`) is auto-fetched from Typst Universe on first compile and cached locally — no extra install step needed.

## Style convention

Every diagram in this folder follows the same palette so they share a visual identity:

| Element              | Color     | Source                                                          |
|----------------------|-----------|-----------------------------------------------------------------|
| Borders + edge lines | `#9370DB` | Canonical Lumiverse purple (rgb 147, 112, 219)                  |
| Box + label fills    | `#F0EAF7` | Light purple wash — legible on light AND dark page backgrounds  |
| All text             | `#2D1B4E` | Dark purple — strong contrast against the fill                  |

The palette is borrowed from Lumiverse's default theme (see `frontend/src/theme/variables.css` in the host repo for the canonical token definitions).

Boilerplate to crib for a new `.typ`:

```typst
#import "@preview/fletcher:0.5.8" as fletcher: diagram, node, edge, shapes

#set page(width: auto, height: auto, margin: 0.4cm, fill: none)

#let lumi-purple = rgb("#9370DB")
#let lumi-fill   = rgb("#F0EAF7")
#let lumi-text   = rgb("#2D1B4E")

#set text(
  font: ("Inter", "Segoe UI", "Helvetica", "Arial"),
  size: 10pt,
  fill: lumi-text,
)

#diagram(
  spacing: (2cm, 2.5cm),
  node-stroke: 1.5pt + lumi-purple,
  node-fill:   lumi-fill,
  node-corner-radius: 4pt,
  node-inset:  10pt,

  edge-stroke: 1.5pt + lumi-purple,

  // ... nodes + edges go here ...
)
```

`#set page(fill: none)` makes the SVG background transparent (lets the markdown viewer's background show through, blending visually with both light and dark themes).

## Current diagrams

| Source                | Rendered in                      | Description                                |
|-----------------------|----------------------------------|--------------------------------------------|
| `storage-model.typ`   | `docs/concepts/storage-model.md` | Decision tree for picking a storage tier   |
| `llm.typ`             | `docs/guides/llm.md`             | LLM API primitive selection                |
| `tools.typ`           | `docs/guides/tools.md`           | Tool dispatch paths converging on handler  |

## Why Typst + Fletcher (not D2 or Mermaid)?

The diagrams went through Mermaid → D2 → Typst+Fletcher as the docs grew. Each step was driven by a specific limitation:

- **Mermaid (dagre auto-layout)** — failed on asymmetric trees + heavy edge labels. Labels overlapped when one branch was deep and a sibling was a single leaf, with no override available.
- **D2 (ELK auto-layout)** — much better on asymmetric trees, but D2 doesn't expose `edgeLabels.placement` overrides. When ELK *still* overlapped labels at a convergence point, no escape hatch.
- **Typst + Fletcher** — explicit grid coordinates. You write the positions. No auto-layout to fight; more code, but you can make any diagram look right.

Worth reaching for Typst+Fletcher when the diagram has any of:

- Three or more sources converging on a single sink,
- Multi-line edge labels that need to sit in specific places,
- A mix of solid + dotted edges that shouldn't visually collide,
- One node with many children (the "fan out" case — see the comb pattern in `storage-model.typ`).

For straight-line flowcharts with short labels, D2 (or even Mermaid) is still fine — manual coordinates are overkill.

## Comparison cheatsheet

Mapping D2 concepts → Typst + Fletcher concepts, for anyone who used the previous-generation `.d2` sources:

| Concern                       | D2                                  | Typst + Fletcher                            |
|-------------------------------|-------------------------------------|---------------------------------------------|
| Layout                        | Auto (ELK)                          | Manual coordinates                          |
| Node positioning              | Inferred                            | `(col, row)` tuples                         |
| Edge routing                  | Inferred                            | Waypoints (explicit or `"d,r,u,l"` shorthand) |
| Label positioning             | Inferred (no override)              | `label-pos: (seg, %)` + `label-side`        |
| Palette                       | `style: { fill: ..., stroke: ... }` | `node-fill`, `node-stroke`, `edge-stroke`   |
| Page background               | `style.fill: transparent`           | `#set page(fill: none)`                     |
| Multi-line node label         | `\n` in quoted string               | Content block `[a\\ b]`                     |
| Embedded code in label        | Plain string                        | `` `monospace` `` raw inline                |
| Regen command                 | `d2 --layout elk in.d2 out.svg`     | `typst compile --format svg in.typ out.svg` |
| Source extension              | `.d2`                               | `.typ`                                      |

## Gotchas

- **Pin Fletcher to `0.5.8`.** The 0.5.9 PDF manual is downloadable, but the release isn't actually published to Typst Universe yet — `#import "@preview/fletcher:0.5.9"` errors with "package not found".
- **Relative-shorthand creates per-step vertices.** `edge(<a>, "d,d,r", "-|>")` creates three single-cell steps — not a two-cell step then a one-cell step. If you put a label at one of those intermediate vertices, Fletcher renders a faint joint mark there and the joint's tip leaks past the `label-fill` mask, making the post-label line appear to resume from inside the label. Workaround: route long straight runs as one segment via an explicit coordinate waypoint (`edge(<a>, (0, 2), <b>, "-|>")` with `label-pos: (0, 50%)`). See `tools.typ` for the worked fix.
- **Markdown-embedded SVGs don't propagate `prefers-color-scheme`.** SVGs loaded via `<img src="...">` (what `![alt](path)` generates) render in a security sandbox where the parent page's color-scheme media query never fires. The Lumiverse-purple + lavender-fill palette is the workaround — chosen to read on both light and dark page backgrounds without theme detection.
- **Font warnings are noise.** Listing fonts not installed locally (Helvetica on Windows, Inter on a fresh Linux box) emits `warning: unknown font family: <name>`. The next family in the fallback list takes over and the SVG renders fine. Don't trim the list — the next developer may be on a different OS.
- **Build verification: don't pipe through `tail`.** `typst compile ... | tail -N` masks the real exit code (tail always returns 0, so compile failures silently look like successes). Use `typst compile ... 2>&1; echo "EXIT=$?"` to capture the exit code explicitly.
