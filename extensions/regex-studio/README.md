<!-- Mirror of upstream documentation. Everything below the metadata table is the original file, unmodified. -->

| Field | Value |
| --- | --- |
| Extension | Regex Studio |
| Source repository | https://github.com/interesting-critter/Regex-Studio |
| Original link (from Lumiverse-Extensions README) | https://github.com/interesting-critter/Regex-Studio |
| Upstream path | `README.md` |
| Retrieved | 2026-09-29 @ `main` (`3db6f31`) |

---

# Regex Studio

**Regex Studio** is a text transformation tool for Lumiverse. It lets you find, replace, and clean up text across single character cards, multiple cards at once, world books, or a custom scratchpad using regex patterns and reusable multi-step pipelines.

---

## What It Does

* **Edit Across Multiple Sources**: Work on a single character card, batch-edit dozens of cards at once, update lorebook entries, or any other text in a blank scratchpad.
* **Tag & Field Filtering**: Filter characters by include/exclude tags, and choose exactly which fields to load (greetings, descriptions, dialogue, system prompts, etc.).
* **Find & Replace**: Regex or plain-text search with full flag toggles (`g`, `i`, `m`, `s`), match navigation (`◀` / `▶`), and automatic scroll-to-match highlighting.
* **Pipeline Presets**: Chain multiple regex steps together into a sequential pipeline. Save and manage presets to re-run complex formatting passes with one click.
* **Diff Preview Before Saving**: Review exact word-level deletions and additions before applying changes, with checkboxes to exclude individual fields from the update.
* **Safe Editing**: Full undo/redo history, copy-all to clipboard, and instant reset back to original text. Nothing is truly edited until you hit save.

---

## Quick Start

1. Pick a Source (Character, Batch, Lorebook, or Custom).
2. Choose your fields and use tag filters to narrow down cards.
3. Set your pattern under **Find / Replace**, or switch to Presets to run a saved pipeline.
4. Hit Replace All (or step through single matches with Replace).
5. Inspect the changes in the Diff Preview and click Save Changes when you're satisfied.
