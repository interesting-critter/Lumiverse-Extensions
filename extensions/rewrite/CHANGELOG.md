# Changelog

All notable changes to this extension are documented here. The format is based on
[Keep a Changelog](https://keepachangelog.com/), and the project aims to follow
[Semantic Versioning](https://semver.org/).

## [1.2.0] — 2026-08-05

### Added
- **Configurable request timeout** — `Options → Behaviour → Timeout (s)` sets how long a
  model call may run before it is aborted (10–600s, default 120s, was hard-coded). The
  frontend watchdog follows it automatically, and the value is included in settings
  export/import.

### Fixed
- **Timeouts no longer masquerade as cancels** — a timed-out generation reported the same
  "Cancelled." as pressing Cancel, since both surface as an `AbortError`. It now reads
  "Timed out after Ns — raise Timeout in Options." across rewrite, multi-rewrite, refine,
  architect, and auto-profile.

## [1.1.0] — 2026-07-12

Formatting-preserving rewrites, and user-configurable generation parameters.

### Added
- **Generation sampler controls** — a new **Sampling** group in the Options tab exposes
  temperature, top-P, top-K, max tokens, and frequency/presence penalties. Any field left blank
  inherits from the selected connection's preset; the temperature/top-P defaults (0.7 / 0.9)
  reproduce the previously hard-coded values, so existing users see no change until they touch
  them. A **"Use these for AI helper calls too"** toggle extends the same values to the Refine /
  AI-generate-style / per-chat-voice calls (off by default — those keep their task-tuned
  temperatures otherwise). All values persist and round-trip through export/import.

### Fixed
- **Rewrites stripped `<font>` tags and markdown.** The model was fed the *rendered*
  (formatting-free) selection and told to emit no markdown, and its plain output then overwrote
  the raw span that held the markup — so whole-message and font-wrapped rewrites lost all
  formatting. The rewrite now feeds the model the **raw markdown slice** of the selection (located
  through the same render→raw alignment the splice uses), the prompt instructs preserving HTML
  tags and markdown exactly (and never inventing a balancing tag/mark for a fragment), and output
  quote-stripping is skipped when the input was itself quote-wrapped. Inline markup outside the
  selected span was already safe; this closes the in-span and whole-message cases.
- **Length target inflated by markup.** The target word count is now measured from the rendered
  selection, so tags and markdown carried in the raw slice no longer skew it.

## [1.0.3] — 2026-07-05

Post-1.0.2 polish — fixes surfaced from real-world use.

### Fixed
- **Full-message rewrites failed on very large messages.** A full-content selection has nothing to bracket outside `[0, n)`, so the windowed anchor path always returned null. The LCS isn't needed here — the whole rendered content maps to the whole raw content by definition. Shortcut added; oversized full-message rewrites now splice in under a millisecond regardless of length.
- **"Couldn't locate the selection" on multi-paragraph messages.** The `alignExact` cell cap was 4M (~2000×2000 chars), which realistic 10-paragraph messages tripped. Cap bumped to 16M (~4000×4000, ~64MB dp allocation — modern browsers handle it fine). For anything above 16M, the windowed anchor retry loop now keeps the first-found anchor per side and only overwrites empty slots — a message uniquely anchoring left at LEN=60 but right at LEN=40 no longer falls to null.
- **"Couldn't locate" on markdown-link and HTML-tag spans.** The stale-capture guard required strict equality after normalization, which broke any selection whose raw form carried URL letters or attribute values. Now uses a subsequence check — sel's characters must appear in order in rawSpan — which still rejects the mid-word substring-bypass class while accepting real markdown content.
- **Rewrite output flashing then disappearing.** Single-capture `doCapture` wasn't idempotent — any stray `selectionchange` producing the same Capture cleared `outputEl` and wiped the just-rendered result. Now short-circuits identical captures, mirroring the multi-capture sig check.
- **Watch mode not picking up new selections after a rewrite.** The `resultPending` freeze in `onSelectionChange` was too aggressive — reselecting after a result is the user moving on. Now only freezes while a rewrite is actually in flight.
- **Hide-built-ins checkbox stealing focus mid-toggle.** The sig-cache only skipped renders on unrelated config echoes; profile mutations now pre-sync the sig cache so the incoming echo matches and the re-render is skipped entirely.

## [1.0.2] — 2026-07-04

Comb-driven correctness + isolation pass — 35 verified findings, 5 batches.

### Fixed
- **Splice guard bypass.** The stale-capture check used `sel.includes(rawSpan) || rawSpan.includes(sel)`, which admitted mid-word substring matches — selecting three words that overlapped two remaining words in an edited message could silently apply the rewrite to only the two. Now requires equality after normalization.
- **Surrogate-pair splice boundaries** are rejected so an emoji or CJK codepoint can't be split into orphan half-surrogates.
- **Single-char bold/italic tokens** ('I', 'a', a name initial) no longer bail the splicer via island-demotion pathology.
- **`apply_multi` history loss.** A mid-loop `updateMessage` throw used to leave the successfully-applied writes un-undoable. History is now saved before reporting.
- **Watch-mode wrong-target apply.** A stray selection between Run and result-arrival could swap the capture — auto-apply then landed on the wrong message. `onSelectionChange` and `Alt+R` are now gated on `running` too.
- **Failed apply left the panel stuck.** `apply_error` now clears `resultPending`, `multiCapture`, and `outputEl.readOnly`, mirroring the success paths.
- **Multi-apply skipped the frontend stale-check** the single path enforces; every segment now gets `liveRenderedFor` verified before dispatch.
- **`update_config` silently dropped disk writes.** Persist failures now surface via `persisted: false` and show a "settings not saved" status.
- **`reset_config` and mid-multi-batch cancels** now report partial state instead of failing silently.
- **Focus theft** on the Hide-built-ins toggles — the config echo only rebuilds the checkboxes when profile sets actually changed.

### Changed
- **Per-user isolation** on shared multi-user Lumiverse hosts: `cancel` only aborts the caller's in-flight rewrites; undo/redo only pop entries the caller owns; `get_debug` filters the ring buffer to the caller's entries.
- **Cancellability** extends to `refine_prompt`, `architect_style`, and `gen_autoprofile` (they used to run to completion regardless of Cancel).
- **`undo` / `redo` gate on `chat_mutation`** with the same friendly error the apply paths use.
- **AI-action buttons (Refine / AI generate style / Style this chat)** now have watchdogs so a dropped backend reply can't leave them disabled forever.
- **Diff view** now refreshes when the input textarea is edited (was only listening to output edits).
- **CJK word-count fallback** only trips when a no-space script is actually detected, so long-average-word English prose no longer produces nonsense counts.
- **Export/import parity** — `historyDepth` and `showDiff` now round-trip.
- **`sanitizeImport`** rejects empty-string profile entries a crafted file could inject.

### Removed
- Redundant `outputEl.readOnly = true` in `rewrite_multi_result`, pre-`get_history` `redoBtn.disabled` flash, throwaway setup renders, and `?? default` fallbacks on non-optional config fields.

## [1.0.1] — 2026-06-26

### Fixed
- "Couldn't locate the selection" on multi-paragraph rewrites: the selection-end offset was
  computed from `Selection.toString()` (which inserts paragraph newlines) in `Range.toString()`
  coordinates, so it overshot `R.length` on messages with paragraph breaks and the splice was
  rejected. The end is now probed with a `Range`, in consistent coordinates.
- Watch mode could swap the apply target out from under a finished-but-unapplied result; the
  capture is now frozen while a result is shown, so a stray selection can't hijack it.

### Changed
- Type now scales with the host text-size setting and transitions use the host motion tokens,
  matching the native Lumiverse panels.
- Denser style chips — two columns at every panel width, compact height.

### Added
- Empty state for the saved-styles list, and a "Rewriting…" status while a rewrite is in flight.

## [1.0.0] — 2026-06-26

Initial release. A native Lumiverse Spindle port of the Marinara Engine "Rewrite-Assistant"
browser extension, with full feature parity plus a redesigned, accessible panel.

### Added

**Rewriting**
- In-place rewrite of a highlighted message span via corruption-safe render→raw LCS alignment.
- Watch mode (auto-capture on selection) and `Alt+R` on-demand capture.
- Multi-message rewrite across several bubbles in one pass.
- Editable output, word-level diff preview, and live word-count delta before applying.
- Multi-level undo/redo with configurable history depth; optional auto-apply.
- Cancel in-flight generations.

**Styles**
- Built-in style profiles, plus user-authored custom styles (create / edit / delete / hide).
- Architect tools: AI-refine a prompt, or generate a full style from a description.
- Per-chat "voice" auto-profiles generated from the character card.
- One-off custom instructions with a recent-prompts history.

**Context**
- Toggleable, gracefully-degrading context injection: previous messages, character card,
  user persona, speaker-awareness, memory/cortex, and world-book/lore.

**Cost & control**
- Live token-cost estimation with a per-source breakdown.
- Length control as a target word-count range; CJK-aware word counting.
- Per-rewrite connection picker (rewrite with a different model than the active chat).
- Concise-prompt mode, automatic quote-stripping.
- Settings export / import, reset to defaults, and debug-log export.

**Interface**
- Three-tab panel (Rewrite / Architect / Options) with a collapsible style chip-grid.
- Accessibility: ARIA tablist with arrow-key navigation, visible focus rings, `aria-live`
  status regions, container-query responsive layout, and `prefers-reduced-motion` support.
- Single-source design tokens keyed off the host theme; minified backend and frontend bundles.

[1.1.0]: https://github.com/Beeopo/Lumiverse-Rewrite/releases/tag/v1.1.0
[1.0.3]: https://github.com/Beeopo/Lumiverse-Rewrite/releases/tag/v1.0.3
[1.0.2]: https://github.com/Beeopo/Lumiverse-Rewrite/releases/tag/v1.0.2
[1.0.1]: https://github.com/Beeopo/Lumiverse-Rewrite/releases/tag/v1.0.1
[1.0.0]: https://github.com/Beeopo/Lumiverse-Rewrite/releases/tag/v1.0.0
