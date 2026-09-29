# Pinned status and inline panels

Status: local implementation added. See README for usage. The completed image compiler and generation lifecycle were not changed; narrative preparation now extracts panel artifacts before planning.

Implemented: isolated draggable/resizable panels, chat-scoped saved pins, snapshot and unique-source following, HTML/SVG extraction, multiline filtering, worker-bounded regex templates, non-committing host macro resolution, stock SimTracker snapshot capture, and a versioned live receiver plus companion adapter source.

Remaining integration work: the companion adapter must be wired into SimTracker itself for live exports. This repository does not modify or install the third-party extension. Existing host regex rules are not imported automatically. Old cached turn plans may lack panel sources. Live card updates rebuild that card's iframe; local control state survives unrelated cursor updates but not replacement markup. Followed values are hidden on turn changes until a current-turn source is available, rather than carrying potentially future state into historical turns.

Validation: the project verification suite passed, plus type checks and production build. Headless Chromium checks covered reveal timing, CSS checkbox interactions, blocked scripts/network requests, keyboard movement, mobile bounds, chat/swipe isolation, saved snapshots, stale bridge revisions/removals, and regex worker timeout.

The sections below record the original design and acceptance targets, including behavior that requires the companion integration.

## Recommendation

Add a panel layer independent of dialogue, image generation, and the current scene image. A pin chooses what content belongs in a panel. Dragging its header chooses where it appears. The panel must not require a wide generated image or depend on the image's dimensions.

Use two pin modes:

- **Follow updates** for a named tracker or status rule. Keep the position and replace the displayed state when the next valid update is reached.
- **Keep this card** for an individual inline artifact, such as a letter, map, SVG, or World Voice announcement. Do not silently replace it with the next unrelated HTML block.

Do not infer these modes from HTML class names or the order of blocks. Store the source rule/provider identity explicitly. Offer both modes where a stable source identity exists; arbitrary inline HTML starts as a snapshot.

## What the current code does

- `src/frontend/host/controller.ts` replaces BubbleMessage, MinimalMessage, and InputArea with null while VN is active. SimTracker's message-attached cards depend on a mounted message element. A DOM-only integration cannot reliably keep receiving cards under this arrangement.
- `src/backend/core/paragraphs.ts` splits on blank lines before removing ignored tags. A tagged block spanning those boundaries loses its wrapper but leaks its contents.
- Square-bracket ignored tags use a single-line match. Multiline World Voice content survives.
- `src/frontend/stage/rich-text.ts` applies display regex rules, escapes HTML, then restores a small allowlist of formatting tags. It is not a general HTML-template renderer.
- The installed host type declarations expose tag interception and sandboxed message widgets. Their display resolver API registers a replacement resolver; it is not a documented API for asking another extension to export its rendered card.

### Read-only probes run

Ran the current paragraph and dialogue functions through Bun without editing production code.

| Input | Observed result |
| --- | --- |
| Ignored `<status>` containing HP and MP separated by a blank line | HP and MP both remain as narrative paragraphs |
| Ignored multiline `[WORLD_VOICE: first\nsecond]` | Entire block remains |
| Ignored single-line `<Hero: Jay ...>` | Block is removed |
| Display regex replacing a status marker with a styled `<div>` | The div markup is escaped, not rendered as a panel |

These are behavior probes, not browser security tests or a completed compatibility suite.

## SimTracker integration

Inspected https://github.com/prolix-oc/Lumiverse-SimTracker and its current `src/frontend.ts` on 2026-09-05.

SimTracker parses structured tracker data, renders its selected template, and mounts it either inside a message or in a sidebar. Its message wrapper carries a message ID. Its templates include CSS-only form controls, and some sidebar tabs receive JavaScript handlers. Copying `innerHTML` does not copy those handlers or live form state.

Preferred integration: an optional, versioned card-export bridge in SimTracker, consumed by VN. SimTracker remains responsible for parsing its JSON/YAML, choosing its template, and producing the card. VN owns placement and its own isolated display copy. Do not move SimTracker-owned DOM nodes into VN.

An exported update needs a provider ID, stable card ID, chat ID, message ID, swipe/source revision, template revision, title, rendered markup, and update/removal status. VN validates size and identity, rejects stale updates, and clears chat-scoped state on switches. This is a proposed contract, not an API that already exists.

Without a companion change, an explicitly labelled snapshot import from a currently mounted card is possible. It is not equivalent to full live SimTracker support. Virtualization, message replacement, and late secondary-model updates prevent a DOM scraper from being a reliable source of truth.

## Regex templates and inline HTML

The Hero and STATUS examples are source text plus regex replacement templates. Capture the source match before suppression and use the host's resolved replacement result where an appropriate integration is available. Do not implement a second partial version of Lumiverse's macro language. Unresolved `when`, `getvar`, and language branches should produce a visible unsupported-template notice, not three language versions stacked together.

Keep source offsets through extraction so a card has a reveal point and adjacent prose retains its order. Use a token-aware block parser, not a regex matching the first opening and closing div. Support fenced HTML/SVG and complete explicit block roots first. Associate adjacent style blocks with the artifact inside its isolated document. Preserve input/label relationships within that document.

Avoid automatically capturing ordinary inline formatting such as spans, emphasis, or links. For ambiguous or malformed output, preserve the source and offer a manual preview selection. Do not consume the remainder of the response to guess where a broken block ends.

Extraction must happen before narrative splitting and stripping. Both the panel and cleaned narrative should derive from that extraction result. This avoids removing content first and trying to reconstruct it later.

## Filtering controls

Separate the controls and explain their effects:

- **Hide from dialogue** changes what the reader sees.
- **Exclude from image planning** changes what the image planner receives.
- **Offer as a panel** makes a matched artifact available to pin.

Pinning does not change the original chat message or automatically change the first two controls. World Voice may deserve both dialogue and a panel; an inventory usually does not. Show a sample match and preview, with invalid rules and unmatched captures reported rather than silently ignored.

Do not run user regexes with unrestricted input on the UI thread. Use bounded inputs and a cancellable worker with a time budget. Support multiline replacement text in a real rule editor instead of overloading the current one-rule-per-line field. Replacement groups must support the Hero example's `$36` and distinguish missing captures from literal dollars. Interpolated text needs context-appropriate escaping, with final markup sanitization as a second layer.

## Rendering boundary

Use a separate sandboxed document per panel, not direct HTML inside the stage. Sanitize markup and apply a restrictive CSP. Keep scripts, event attributes, navigation, form submission, nested frames, and access to the host document disabled. HTML and SVG both need this boundary; SVG can carry active content too.

Allow declarative CSS layouts, SVG drawings, details/summary, and checkbox/radio interactions. Duplicate IDs in different cards then remain isolated. Retain compatible local interaction state across a data update rather than rebuilding the frame on every streaming token.

Block remote assets by default. Offer explicit remote-image/font permission with clear disclosure because both examples load third-party resources. Missing assets must have local font and background fallbacks. Do not silently enable scripts to repair a template's tabs. Script-dependent interactions need a separately designed, narrowly scoped adapter.

## Placement and reading behavior

- Default new pins to an available edge; users may drag anywhere within the VN window. Do not assume black bars always exist.
- Keep a small accessible header with title, pin/unpin, collapse, and reset-position actions. Drag only from the header so scrolling a card cannot move it.
- Store normalized positions and constrained dimensions per chat and stable panel ID. Clamp after resizing, zooming, fullscreen changes, and keyboard appearance. Keep Exit and panel recovery controls reachable.
- Provide keyboard movement and resizing, visible focus, touch-sized controls, and a collapsed rail on narrow screens. Long logs scroll within the card.
- Panel clicks, typing, wheel events, and arrow keys must not advance dialogue or open the VN backlog.
- Reveal inline artifacts at their source position. A follow-up tracker at the end of a response must not spoil that response's outcome at paragraph zero.
- Keep the last valid tracker while a new one is incomplete, visibly marked as updating. Clearing, deletion, and invalidation must be distinct from an ordinary message that contains no tracker.
- A historical swipe selects its own panel state. Never apply a later message's stats to an earlier turn. A chat switch must never flash the previous chat's cards.

## Acceptance cases

1. Hero replacement with 36 captures, percentage widths, a long log, and its CSS checkbox.
2. STATUS with all language branches, changed language variables, Unicode, punctuation, and missing fields.
3. Multiline World Voice with preserved line breaks and optional dialogue retention.
4. Nested HTML, multiple SVGs, fenced artifacts, adjacent styles, and prose on both sides.
5. Truncated streaming HTML, malformed closures, escaped code examples, and tag-only responses with no narrative.
6. Two trackers, repeated announcements, duplicate IDs, and changed template order without pin reassignment.
7. Late SimTracker updates, same-message edits, swipes, deletion, reload, chat fork/switch, and virtualized messages.
8. Script/event/SVG payloads, external requests, form navigation, oversized content, and pathological regexes.
9. Portrait screens, no letterboxing, browser zoom, touch dragging, keyboard operation, and offscreen saved positions.
10. Updating one card does not reset another card's scroll, tabs, or collapsed state; none of its interactions advance dialogue.

## Delivery sequence

First build the isolated panel layer and explicit snapshot pinning, with extraction and filtering regression tests. Next add stable rule-driven updates and host-resolved template support. Add full SimTracker following through the companion export bridge, not by claiming a DOM snapshot adapter provides live compatibility. Each stage should declare unsupported behavior in the UI.
