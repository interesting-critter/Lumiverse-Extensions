<!-- Mirror of upstream documentation. Everything below the metadata table is the original file, unmodified. -->

| Field | Value |
| --- | --- |
| Extension | Super Deduper |
| Source repository | https://github.com/ajrc0re/Lumiverse-SuperDeduper |
| Original link (from Lumiverse-Extensions README) | https://github.com/ajrc0re/Lumiverse-SuperDeduper |
| Upstream path | `README.md` |
| Retrieved | 2026-09-29 @ `main` (`0cc94c0`) |

---

# Lumiverse SuperDeduper

Find, compare, and safely remove duplicate character cards in Lumiverse.

## Features

- Match cards by normalized name, exact card contents, or adjustable content similarity.
- Compare greetings, token counts, lorebooks and entries, character-scoped scripts, expressions, image galleries, timestamps, and other embedded extension data.
- Get a transparent payload-first keeper recommendation that uses recency as its first tie-breaker.
- Protect a selected keeper and delete other cards only after an explicit Lumiverse confirmation.
- Keep optional data visibly marked as unavailable when its permission is not granted.

## Install

Install this repository URL from Lumiverse's Extensions panel:

```text
https://github.com/ajrc0re/Lumiverse-SuperDeduper
```

The extension requires the `characters` permission. The `world_books`, `images`, and `regex_scripts` permissions add comparison details; scans still work without them, but recommendations are marked provisional.

Open the **SuperDeduper** drawer tab, select a match mode, and choose **Scan characters**. Scans run only when requested. If a card or associated payload changes afterward, the existing results are marked stale.

## How it works

SuperDeduper first loads the character cards in the selected scan scope, then compares them using one of three modes:

- **Names match** normalizes each name with Unicode NFKC normalization, converts it to lowercase, trims it, and collapses repeated whitespace. Cards with the same normalized name are grouped together.
- **Exact card contents** normalizes and compares the ordinary card fields listed below. Cards are considered exact matches when the resulting canonical content has the same SHA-256 fingerprint.
- **Similar card contents** compares the same fields separately using Sørensen–Dice similarity over sets of normalized character trigrams (overlapping three-character fragments). A field's contribution is weighted by the length of the longer value, fields empty on both cards are ignored, and the weighted field scores are combined into one value. A pair qualifies when that value meets the selected threshold.

The content fields used by exact and similar matching are description, personality, scenario, primary greeting, example dialogue, creator notes, system prompt, post-history instructions, creator, and tags. Normalization applies Unicode NFKC normalization, lowercase conversion, trimming, and whitespace collapsing. Tags are also deduplicated and sorted.

Names, IDs, timestamps, avatar IDs, alternate greetings, attached lorebooks, images, scripts, and extension payloads are excluded from content matching. This allows renamed imports and cards with different attached payloads to match. Those excluded resources are inspected later when duplicate candidates are compared and a keeper is recommended.

Similar-mode groups are connected components of qualifying pairs. For example, if A matches B and B matches C, all three cards appear in one group even if A does not directly match C. The percentage displayed on a card is its highest qualifying similarity to any other card in that group; it is not an average for the group or necessarily a comparison with the recommended keeper. Expand the comparison table to see every qualifying pair.

Character-trigram sets measure textual overlap, not meaning. Repeated fragments count only once, so long cards that share a highly consistent template, vocabulary, and formatting can receive unexpectedly high scores despite describing different characters. Raising the threshold reduces but does not eliminate this possibility. Similar-content results should therefore be reviewed before deletion, especially for large groups from one creator.

After matching, SuperDeduper enriches only the cards in duplicate groups. It counts accessible card and payload text, greetings, lorebooks, scripts, expressions, and images, then recommends a keeper using payload coverage, payload item count, update time, accessible text count, creation time, and finally character ID. Lumiverse's Chub expression-check timestamp is import bookkeeping rather than an expression payload, so SuperDeduper ignores it while retaining real expression data. The selected keeper remains protected, and every deletion requires confirmation and a fresh timestamp check.

## Development

```bash
bun install
bun run typecheck
bun test
bun run build
```

Lumiverse loads `dist/backend.js` and `dist/frontend.js`. If `dist/` is absent during installation, Lumiverse can build the TypeScript entry points from `src/`.

See [PLAN.md](PLAN.md) for the complete behavior and acceptance criteria.
