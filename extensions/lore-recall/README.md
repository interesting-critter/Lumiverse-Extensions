<!-- Mirror of upstream documentation. Everything below the metadata table is the original file, unmodified. -->

| Field | Value |
| --- | --- |
| Extension | Lore Recall |
| Source repository | https://github.com/archkr/Lumiverse-LoreRecall |
| Original link (from Lumiverse-Extensions README) | https://github.com/archkr/Lumiverse-LoreRecall |
| Upstream path | `README.md` |
| Retrieved | 2026-09-29 @ `main` (`61d93e7`) |

---

<div align="center">

# Lore Recall

**Tree-aware retrieval for Lumiverse worldbooks.**

[![Version](https://img.shields.io/badge/version-0.1.6-6b8ff0)](./spindle.json)
[![Lumiverse](https://img.shields.io/badge/Lumiverse-%E2%89%A5%200.9.0-d4a35a)](https://lumiverse.chat)
[![Status](https://img.shields.io/badge/status-stable-7fb380)](https://github.com/archkr/Lumiverse-LoreRecall)
[![License: GPL-3.0](https://img.shields.io/badge/License-GPL--3.0-green.svg)](LICENSE)

*A curator for narrative knowledge.*

</div>

Lore Recall turns flat lorebook entries into a navigable, character-scoped knowledge tree, then retrieves only the slices that matter for the current scene through either a fast collapsed lookup or a controller-guided traversal.

Build a structured index of your world once. Let the right entries surface at the right moment instead of brute-keyword-matching every turn.

> Inspired by [TunnelVision](https://github.com/Coneja-Chibi/TunnelVision)'s philosophy of giving the AI an actual map of what it knows. Lore Recall is a Lumiverse-native take on the same idea.

---

## Table of contents

1. [At a glance](#at-a-glance)
2. [Why Lore Recall](#why-lore-recall)
3. [How it works](#how-it-works)
4. [Compatibility](#compatibility)
5. [Installation](#installation)
6. [Quick start](#quick-start)
7. [Concepts](#concepts)
8. [Settings reference](#settings-reference)
9. [Tips and best practices](#tips-and-best-practices)
10. [Troubleshooting](#troubleshooting)
11. [Architecture](#architecture)
12. [Credits and inspiration](#credits-and-inspiration)
13. [License](#license)

---

## At a glance

| | |
|---|---|
| **Character-scoped sources** | Each character has its own managed lorebook list. Retrieval pulls only from those. |
| **Tree organization** | Hierarchical index per book, built from metadata or by an LLM. |
| **Two retrieval modes** | `Collapsed` for one-pass scope picks. `Traversal` for exploratory navigate/search/retrieve passes. |
| **Reserved constants** | Native `constant` entries always inject. Dynamic budget fills the rest. |
| **Selective retrieval** | Controller picks the final injected entry IDs from the accumulated candidate pool. |
| **Live retrieval feed** | Scope, search, manifest, pulled, injected, and issue events stream as they happen. |
| **Editable tree workspace** | Move, rename, retag, regenerate summaries, bulk-flag whole branches. |
| **Per-book permissions** | `Read + write`, `Read only`, and `Write only` modes. |
| **Snapshots** | One-click export and import of full Lore Recall state. |
| **Diagnostics** | Surfaces problems and prunes stale references automatically. |

---

## Why Lore Recall

Default worldbook injection fires on keywords. Type "Yuki" and entries with "Yuki" as a key fire, even if the scene isn't about her. Don't type "Yuki" and her backstory never lands, even if the scene clearly *is* about her. Keywords are brittle.

The alternatives:

- **Vector RAG** is closer, but still pattern-matching on embeddings. It finds text that *looks similar* to the query. It has no concept of *what the scene needs*.
- **Manual constants** force entries to always inject. Fine for a few core entries; useless when you have hundreds of scoped lore items.

Lore Recall's bet: if you give the controller LLM a structured tree of your world, it can *reason* about which scopes are relevant to the current scene, and inject the entries that actually matter for the next reply. The tree is built once. Retrieval is contextual. The user maintains the world; the controller chooses what to surface.

---

## How it works

```
Your worldbook                Lore Recall tree              Retrieval
                            
[entry] [entry] [entry]     ┌─ Characters                   ┌─ scope picked
[entry] [entry] [entry]     │  ├─ Main party                ├─ manifest narrowed
[entry] [entry] [entry]  →  │  │  ├─ Aria                →  ├─ entries pulled
[entry] [entry] [entry]     │  │  └─ Ren                    └─ entries injected
[entry] [entry] [entry]     │  └─ NPCs                          into the prompt
   (flat, keyword-fired)    └─ Locations                          (only what matters)
                                ├─ Thornfield
                                └─ Underground
```

1. **Build** a tree for each managed book (metadata-only or LLM-assisted).
2. **Configure** retrieval per character (mode, depth, limits, reranking).
3. **Send a message.** The interceptor runs the retrieval pipeline before the main LLM call.
4. **Watch the live feed** in the drawer to see exactly what was picked and why.
5. **Maintain** the tree over time as your story grows: edit summaries, move entries, regenerate as needed.

---

## Compatibility

| Requirement | Value |
|---|---|
| Lumiverse version | `0.9.0` or newer |
| Spindle permissions | `world_books`, `characters`, `chats`, `chat_mutation`, `generation`, `interceptor` |
| Controller connection | Strongly recommended (required for LLM build, summary regeneration, and traversal-mode retrieval) |

The metadata-build path and collapsed-mode retrieval can run without a controller, but most of Lore Recall's intelligence comes from the controller-driven flows.

---

## Installation

### From GitHub inside Lumiverse

```
1. Copy:    https://github.com/archkr/Lumiverse-LoreRecall
2. Open:    Lumiverse  →  Extensions  →  Install
3. Paste:   the URL into the repo field
4. Press:   Install
5. Enable:  Lore Recall and grant the requested permissions
6. Verify:  a "Lore Recall" tab appears in the Extensions drawer
```

### Manual / local checkout (development)

Clone the repo, then either:

- Point Lumiverse at the local folder, or
- Run `bun run build` from the repo root to regenerate `dist/backend.js` and `dist/frontend.js`, then reload the extension

---

## Quick start

> **Heads up:** Lore Recall settings are per-character. Most actions are no-ops without an active character chat open.

| Step | Action | Where |
|---|---|---|
| 1 | Open a character chat | Lumiverse |
| 2 | Open Lore Recall | Extensions drawer |
| 3 | Manage one or more lorebooks (refresh if a new book doesn't appear) | Workspace, Sources |
| 4 | Build a tree (`Build from metadata` is free and instant) | Workspace, Build |
| 5 | Enable retrieval for the active character | Workspace, Retrieval |
| 6 | Pick a retrieval mode (`Collapsed` is the fast default) | Workspace, Retrieval |
| 7 | Pick a controller connection | Workspace, Maintenance, Advanced |
| 8 | Send a message and watch the live feed | Drawer |

---

## Concepts

<details>
<summary><b>Sources</b> &mdash; pick which lorebooks the active character can retrieve from</summary>

- **Managed books** are the actual retrieval set. Only managed books participate.
- **Suggested books** are auto-detected from your global pattern (default `*recall*`).
- **Attached** indicates a book is also natively wired to the character via Lumiverse's normal worldbook attachment. Native attachment is independent of Lore Recall management. Usually you want them detached so Lore Recall is the sole retrieval path.
- **Per-book settings** include a `description` (helps the controller during multi-book retrieval), permission mode, and an enable toggle.

</details>

<details>
<summary><b>Build</b> &mdash; create or rebuild trees for managed books</summary>

- **Build from metadata** uses each entry's existing keys, tags, and group hints. Instant, free, no LLM calls. Works well when your lorebook already has decent metadata.
- **Build with LLM** uses your controller connection to categorize entries by content. Slower and costs tokens, but produces much better trees for messy or sparsely-tagged books.

**Build tuning:**

| Knob | Effect |
|---|---|
| Build detail | How much of each entry the controller sees (`Names`, `Lite`, `Full`) |
| Tree granularity | `Auto` scales with book size; manual presets target different category counts |
| LLM chunk size | Characters per categorization call. Larger chunks mean fewer calls |
| Dedup mode | `None`, `Lexical`, or `LLM` deduplication during build |

> **Common gotcha:** a brand-new book with no entries cannot be built. There's nothing to organize. Add entries first.

</details>

<details>
<summary><b>Retrieval</b> &mdash; configure per-character behavior</summary>

| Setting | What it does |
|---|---|
| `Search mode` | `Collapsed` or `Traversal` |
| `Multi-book mode` | `Unified` merges all managed books; `Per book` lets the controller pick |
| `Collapsed depth` | Tree depth shown to the controller in collapsed mode |
| `Traversal depth` | Max tree depth the controller can drill into |
| `Traversal step limit` | Max controller drill calls per turn |
| `Scope pick limit` | Max scopes the controller can choose in one step |
| `Pull limit` | Max pooled candidates exposed to final manifest selection |
| `Inject limit` | Max entries written into the prompt |
| `Context messages` | How many recent chat messages become retrieval context |
| `Rerank top candidates` | Reorder pooled candidates before final manifest selection |
| `Selective retrieval` | Controller picks exact entry IDs from the candidate pool |

In `Traversal` mode, retrieval is additive. The controller can navigate into a branch, retrieve its entries into a temporary pool, run a global search, retrieve more entries, and then finish with one final manifest selection. A valid sparse selection is respected: if the controller chooses one dynamic entry, Lore Recall injects one dynamic entry; if it chooses none, only constants are injected.

</details>

<details>
<summary><b>Book</b> &mdash; inspect the selected lorebook and access its tree</summary>

- See managed/attached/tree-built status, last build source, entry/category/unassigned counts
- Edit per-book settings (enable, permission, description)
- Open the **tree workspace** to navigate and edit the actual tree

</details>

<details>
<summary><b>Tree workspace</b> &mdash; the modal editor for a managed book's tree</summary>

- **Tree sidebar** with collapsible categories, an Unassigned section, and Collapse all / Expand all controls
- **Editor pane** with breadcrumbs, label and parent selectors, summary and collapsed-text fields, native flags (`disabled`, `constant`, `selective`), aliases, and tags
- **Bulk entry-flag actions** at the category level: set/clear `constant`, enable/disable all descendants, set/clear `selective`
- **Regenerate summary** for a category, an entry, or a whole book
- **Move/delete categories** with safe targets for orphaned entries
- **Read-only books** disable destructive controls; the editor is browse-only

</details>

<details>
<summary><b>Retrieval feed</b> &mdash; what Lore Recall is doing in real time</summary>

Each session card shows:

- Mode (`Collapsed` / `Traversal`), status, controller-vs-deterministic path, elapsed time
- A flow strip: `scope → manifest → pulled → injected` counts
- The top injected entry preview
- An expandable thread of every event: scope choices, search calls, manifest selections, reservations, pulls, injections, and any issue events
- Filter chips to show only one event kind

Live sessions get a subtle pulse and an amber outer edge so you can see retrieval is happening *now*.

</details>

<details>
<summary><b>Maintenance</b> &mdash; the wider extension state</summary>

- **Diagnostics** &mdash; warnings about missing trees, write-only conflicts, attached-but-unmanaged books, metadata gaps, missing controller connections, and information about auto-cleaned stale references
- **Backup & restore** &mdash; snapshot export (downloads a JSON) and import (uploads one)
- **Advanced** &mdash; global settings: master enable, auto-detect pattern, controller connection, controller temperature and max tokens, build detail, tree granularity, chunk size, dedup mode

</details>

---

## Settings reference

<details>
<summary><b>Per-character (Retrieval panel)</b></summary>

| Setting | Default | Notes |
|---|---|---|
| `Enable retrieval for this character` | Off | Master toggle for the character |
| `Search mode` | `Collapsed` | `Collapsed` is faster; `Traversal` is more exploratory |
| `Multi-book mode` | `Unified` | `Unified` merges all managed books; `Per book` lets the controller pick |
| `Collapsed depth` | 2 | Tree depth shown to the controller in collapsed mode |
| `Pull limit` | 6 | Max pooled candidates exposed to final manifest selection |
| `Traversal depth` | 3 | Max tree depth in traversal mode |
| `Traversal step limit` | 5 | Max controller drill-down calls per turn |
| `Scope pick limit` | 5 | Max scopes the controller can choose in one step |
| `Inject limit` | 6 | Max entries injected into the prompt |
| `Context messages` | 10 | Recent chat messages used as retrieval context |
| `Rerank top candidates` | Off | Reorder candidates before final manifest selection |
| `Selective retrieval` | On | Controller picks exact entry IDs from the accumulated candidate pool |

</details>

<details>
<summary><b>Per-book</b></summary>

| Setting | Default | Notes |
|---|---|---|
| `Enable this managed source` | On | Per-book disable switch |
| `Permission` | `Read + write` | `Read only` blocks all rebuild/edit; `Write only` is for write-oriented workflows |
| `Description` | (empty) | Helps the controller route between books in multi-book retrieval |

</details>

<details>
<summary><b>Global (Advanced panel)</b></summary>

| Setting | Default | Notes |
|---|---|---|
| `Master enable` | On | Global kill switch |
| `Auto-detect pattern` | `*recall*` | Books matching this glob get suggested |
| `Controller connection` | (default) | Which connection profile to use for LLM flows |
| `Controller temperature` | 0.2 | Sampling for controller calls |
| `Controller max tokens` | 8192 | Output cap for controller calls |
| `LLM chunk size` | 30,000 | Characters per categorization call during LLM build |
| `Build detail` | `Lite` | `Names` / `Lite` / `Full` &mdash; how much entry content the LLM sees |
| `Tree granularity` | `Auto` | Auto scales with book size; manual presets target different category counts |
| `Dedup mode` | `None` | `None` / `Lexical` / `LLM` |

</details>

---

## Tips and best practices

> **Detach managed books from native attachment.** Once a book is managed by Lore Recall, you usually want it *not* natively attached to the character. Native attachment fires keyword triggers in parallel with Lore Recall's retrieval, leading to double-injection and confusing prompt breakdowns. The `Attached` tag on the book panel is a heads-up, not an error.

> **Use metadata build first.** It's instant and lets you confirm the tree pipeline works end-to-end before paying for an LLM build.

> **Constants are always-on.** Native `constant`-flagged entries are injected separately from the dynamic retrieval cap. Use them for must-always-inject anchors (current location tracker, party stats, season-of-the-story flags). Don't use constants for general lore. That defeats the point of dynamic retrieval.

> **Per-book descriptions matter for multi-book retrieval.** When the controller has to choose which book to consult, the description is its main signal. Write descriptions that explain what kind of content lives in each book, not just what the book is called.

> **Watch the retrieval feed during the first few turns.** It tells you whether the controller is making sensible scope choices. Consistently bad picks usually mean the tree summaries are too vague. Regenerate them.

> **Read-only managed books are a real workflow.** Use `Read only` permission for community lorebooks you want to retrieve from but never edit. Lore Recall blocks rebuild and rewrite operations cleanly.

> **Selective retrieval on is the better default.** It lets the controller make the final call about *which* entries inject from the accumulated traversal pool. Sparse valid selections are intentional, not automatically backfilled.

---

## Troubleshooting

<details>
<summary><b>"I clicked Build and nothing happens"</b></summary>

The most common cause is an empty book. Lore Recall builds the tree out of the existing entries, so a book with zero entries has nothing to organize. Add a few entries to the book in Lumiverse, then build.

If the book has entries and the button is still disabled, check the inline reason directly under the buttons. It will tell you whether the issue is a missing controller connection, a read-only book, or no active character.

</details>

<details>
<summary><b>"I made a new book in Lumiverse and Lore Recall doesn't see it"</b></summary>

Click the refresh button in the Sources panel toolbar. Lore Recall caches the world-book list for 5 seconds; the refresh button busts that cache and re-pulls the full list.

</details>

<details>
<summary><b>"A book I deleted in Lumiverse still shows as managed"</b></summary>

This auto-cleans on the next state refresh. You'll see an info diagnostic in Maintenance reading *"Removed N stale managed-book reference(s)"*. If you want to force it immediately, open Maintenance and run diagnostics.

</details>

<details>
<summary><b>"Retrieval feed is empty"</b></summary>

Make sure:

- The character has retrieval enabled (Retrieval panel)
- At least one managed book has a built tree (Sources / Build panels)
- You actually sent a message (the feed only populates during a real generation; opening Lore Recall doesn't trigger retrieval)

</details>

<details>
<summary><b>"Controller fell back to deterministic"</b></summary>

This is logged in the feed as a fallback event. Common causes:

- The controller connection is missing or invalid (check Advanced settings)
- The controller returned malformed output (some smaller models do this; try a bigger model)
- The controller exceeded the interceptor timeout (default 180s; bump in `spindle.json` if your provider is slow)

</details>

<details>
<summary><b>"It works, but the choices are bad"</b></summary>

Almost always a tree quality problem. Try in this order:

1. Regenerate summaries on the worst-offending categories (Tree workspace, Regenerate summary)
2. Switch from metadata build to LLM build for those books
3. Bump build detail to `Full` and rebuild
4. Tighten `Tree granularity` so categories are more specific

</details>

---

## Architecture

<details>
<summary><b>Source tree</b></summary>

```
src/
  backend.ts            entrypoint: registers messages, interceptor, lifecycle
  backend/
    index.ts            state assembly, message dispatch, interceptor wiring
    operations.ts       all long-running operations (build, summarize, snapshots)
    retrieval.ts        the actual retrieval pipeline (collapsed + traversal)
    runtime.ts          shared runtime state, send helpers, storage paths
    storage.ts          per-extension storage layer (configs, trees, caches)
    contracts.ts        backend-only DTO helpers
    controller-json.ts  resilient JSON parsing for controller responses
  frontend.ts           entrypoint: re-exports the UI setup
  ui/
    app.ts              all rendering: drawer, settings workspace, tree modal, feed
    helpers.ts          formatting and small DOM utilities
    styles.ts           the Codex CSS template (loaded via spindle.dom.addStyle)
  shared.ts             types and normalizers used by both sides
  types.ts              wire-format DTOs

dist/
  backend.js            built backend bundle
  frontend.js           built frontend bundle

spindle.json            extension manifest
```

</details>

<details>
<summary><b>Build commands</b></summary>

```sh
bun run build              # backend + frontend
bun run build:backend      # just the backend
bun run build:frontend     # just the frontend
bun run typecheck          # tsc --noEmit
```

After source changes, run `bun run build` and reload the extension in Lumiverse.

</details>

---

## Credits and inspiration

Lore Recall is meaningfully inspired by [TunnelVision](https://github.com/Coneja-Chibi/TunnelVision), in particular:

- TunnelVision's AI-directed retrieval philosophy, where the model actively helps decide what context it needs
- TunnelVision's activity-feed style UX for making retrieval behavior visible and debuggable

TunnelVision is a separate project with its own codebase and license. This README is crediting influence and inspiration; Lore Recall is a Lumiverse-native rebuild of similar ideas, not a code port or shared-code derivative.

| Project | License |
|---|---|
| Lore Recall | AGPL-3.0 |
| TunnelVision | AGPL-3.0 |

Each license applies to its own repository independently.

---

## License

See [`LICENSE`](./LICENSE).
