<!-- Mirror of upstream documentation. Everything below the metadata table is the original file, unmodified. -->

| Field | Value |
| --- | --- |
| Extension | LumiStage |
| Source repository | https://github.com/Archkr/Lumiverse-LumiStage |
| Original link (from Lumiverse-Extensions README) | https://github.com/Archkr/Lumiverse-LumiStage |
| Upstream path | `README.md` |
| Retrieved | 2026-09-29 @ `main` (`022f1b3`) |

---

<div align="center">

# LumiStage

**An independent expression studio and cinematic sprite stage for Lumiverse.**

[![Version](https://img.shields.io/badge/version-1.0.0-8b7cf6)](./spindle.json)
[![Lumiverse](https://img.shields.io/badge/Lumiverse-%E2%89%A5%201.1.0-d4a35a)](https://github.com/prolix-oc/Lumiverse)
[![Status](https://img.shields.io/badge/status-active-6f9f78)](https://github.com/Archkr/Lumiverse-LumiStage)
[![License](https://img.shields.io/badge/license-Lumiverse%20Community%202.0-6f9f78)](./LICENSE.md)

*Give every character outfits, expressions, exact sprite variants, and a live place on stage.*

</div>

LumiStage manages character sprite libraries and directs them after completed roleplay replies. Each character owns outfit folders; each outfit owns named expression slots; and each expression can contain multiple exact image or video variants.

It supports solo characters, group scenes, automatic LLM direction, manual locks, portable archives, and a floating stage that can be positioned per chat.

> **Independence note:** LumiStage owns its profiles, decisions, timelines, archives, and media references. It does not inspect, synchronize with, migrate from, or modify Lumiverse’s built-in expression feature.

---

## Table of contents

1. [At a glance](#at-a-glance)
2. [Why LumiStage](#why-lumistage)
3. [How it works](#how-it-works)
4. [Library model](#library-model)
5. [Compatibility](#compatibility)
6. [Installation](#installation)
7. [Quick start](#quick-start)
8. [Studio guide](#studio-guide)
9. [Importing media](#importing-media)
10. [Automatic direction](#automatic-direction)
11. [Manual direction and locks](#manual-direction-and-locks)
12. [Floating stage](#floating-stage)
13. [Settings reference](#settings-reference)
14. [Archives and recovery](#archives-and-recovery)
15. [Permissions](#permissions)
16. [Storage, ownership, and reliability](#storage-ownership-and-reliability)
17. [Troubleshooting](#troubleshooting)
18. [Development](#development)
19. [License](#license)

---

## At a glance

| | |
|---|---|
| **Editable sprite libraries** | Organize every character into outfit folders, expression slots, and exact media variants. |
| **Safe additive imports** | Import individual files or folder trees into the current Studio draft without replacing unrelated outfits. |
| **Exact manual control** | Apply a sprite once, lock an outfit while expressions change, or lock one exact state. |
| **Completed-reply automation** | Runs only after a successful assistant message finishes; streaming updates do not trigger expression changes. |
| **One-call direction** | Sends the active outfit and expression catalog to one structured detector call, then randomly chooses an eligible variant locally. |
| **Confidence-safe state** | A malformed or low-confidence result preserves the complete prior stage, including focus. |
| **Solo and ensemble staging** | Displays one character at full strength or composes focused and supporting characters in group scenes. |
| **Timeline replay** | Reconciles deletions, swipes, and regenerations against message and content fingerprints. |
| **Portable archives** | Exports and explicitly restores a complete character profile and its referenced media. |
| **Revision-safe editing** | Uses revision checks for saves, imports, restores, deletion, settings, and chat layouts. |
| **Diagnostics** | Reports configuration, revision, catalog, media, cleanup, and detector health without story text. |

---

## Why LumiStage

A flat folder of expression images works until a character has multiple wardrobes, alternate poses, animated variants, or different looks for the same emotion.

LumiStage separates those concerns:

- an **outfit** selects the character’s wardrobe or visual context;
- an **expression** describes the visible state inside that outfit;
- a **variant** selects the exact image or video to display;
- a **timeline decision** records why that state belongs to a particular completed assistant message;
- a **manual override** lets the user take control without rewriting the library.

That structure prevents filename conventions from becoming the runtime identity. Display names can change while stable IDs continue to address the same outfit, expression, and sprite.

---

## How it works

```text
Completed assistant reply
          │
          ▼
One structured detector call
          │
          ├── character ID
          ├── outfit name
          ├── expression name
          ├── expression selection
          ├── focus
          └── confidence
          │
          ▼
Revisioned chat timeline
          │
          ▼
Live Stage + floating stage
```

1. Build a sprite library for each character.
2. Open a chat containing one or more configured characters.
3. After a successful assistant generation commits its final message, LumiStage reads the configured recent context.
4. The detector receives active character, outfit, and expression names plus the current stage state in one call. Variant IDs and filenames remain private to LumiStage.
5. LumiStage validates every returned character and expression, randomly chooses one eligible variant locally, resolves the selection to stable IDs, and applies the configured confidence threshold to the complete decision.
6. The accepted state is saved to the chat timeline and rendered on every LumiStage surface.
7. Edits, swipes, deletions, and regenerations reconcile the timeline instead of leaving stale sprites behind.

Manual direction uses the same catalog and timeline but does not need an LLM call.

---

## Library model

```text
Character profile
└── Outfit folder
    └── Expression slot
        ├── Variant A.png
        ├── Variant B.webp
        └── Variant C.webm
```

### Character profile

One revisioned profile belongs to one Lumiverse character. The profile stores its default outfit and the complete outfit library.

### Outfit folder

An outfit is a selectable visual context such as `Default`, `Formal`, `Armor`, or `Rain Coat`. Each outfit has its own expression slots and default expression.

### Expression slot

An expression is a named state such as `Neutral`, `Happy`, `Concerned`, or `Battle Ready`. Names are user-facing; stable IDs are used for saves, imports, and detector results.

### Sprite variant

A variant is an exact media file within an expression. LumiStage stores its stable variant ID, image ownership ID, original filename, content hash, media type, and order.

The same uploaded media may be referenced from multiple destinations. LumiStage uploads identical content once and creates the requested references without silently moving it back to `Default`.

---

## Compatibility

| Requirement | Value |
|---|---|
| Lumiverse | `1.1.0` or newer |
| LumiStage | `1.0.0` |
| Runtime | Spindle extension backend and frontend |
| Automatic direction | `generation`, `chats`, and `chat_mutation` permissions |
| Character libraries | `characters` permission |
| Media import and deletion | `images` permission |
| Studio placements | `ui_panels` permission |
| Production entrypoints | `dist/backend.js` and `dist/frontend.js` |

LumiStage uses Lumiverse-native drawers, modals, character-editor tabs, input-bar actions, floating widgets, connections, models, and media storage.

---

## Installation

### Install from GitHub

```text
1. Copy:    https://github.com/Archkr/Lumiverse-LumiStage
2. Open:    Lumiverse → Extensions → Install
3. Paste:   the URL into the repository field
4. Press:   Install
5. Enable:  LumiStage and grant the requested permissions
6. Verify:  LumiStage appears in the extension drawer
```

No local build is required for a normal installation because release-ready bundles are committed in `dist/`.

### Update

Use the update action on LumiStage’s entry in Lumiverse’s Extensions panel. Reload the extension when Lumiverse asks you to so the new frontend bundle is mounted.

---

## Quick start

| Step | Action | Where |
|---|---|---|
| 1 | Open a character or group chat | Lumiverse |
| 2 | Open **LumiStage** | Extension drawer |
| 3 | Select **Open Studio** | LumiStage drawer |
| 4 | Create or select an outfit | Library |
| 5 | Create an expression or import a folder tree | Library |
| 6 | Choose a detector connection and model | Settings → Detection |
| 7 | Send a message and wait for the assistant reply to finish | Chat |
| 8 | Show or position the floating stage | Live Stage or Settings → Stage |

For immediate control without automatic detection, use the input-bar **LumiStage** action and choose **Apply once**.

---

## Studio guide

The large Studio has three primary views.

### Library

Library is the full character sprite editor.

- Switch characters without allowing one character’s response to overwrite another profile.
- Add, rename, reorder, and delete outfit folders.
- Add, rename, reorder, and delete expression slots.
- Choose default outfits and default expressions.
- Inspect every exact variant in an expression.
- Reorder variants, open full-size previews, import more media, or delete variants.
- Search the current outfit.
- Open Lumiverse's native Find & Replace editor to rename every expression in the current outfit as an undoable draft change.
- Select all filtered expressions and move, copy, or delete them in one contextual operation.
- Undo and redo local library edits before saving.

Blank names, duplicate normalized names, duplicate IDs, missing defaults, and invalid references are surfaced as blocking validation instead of being silently normalized away.

If backend state changes while the Studio is dirty, LumiStage preserves the draft. **Save changes** applies that preserved draft against the latest backend revision; **Reload profile** discards it.

### Live Stage

Live Stage shows the resolved state for the active chat:

- exact character, outfit, expression, and variant;
- detector confidence;
- focused versus ensemble status;
- active outfit or state locks;
- floating-stage visibility and captions;
- global or chat-specific layout controls.

**Analyze now** forces a fresh detector call and bypasses the normal decision cache.

The drawer’s always-open **Detector activity** panel keeps an in-memory transcript for the active chat. Every session attempt is rendered in the scrollbox without lazy loading, with host-parsed reasoning collapsed by default and the model’s response text or tool selection shown without internal parsed IDs or JSON scaffolding. **Copy all** exports only each run’s thinking and output. Transcripts are not persisted and disappear when the extension backend reloads.

### Settings

Settings is divided into:

- **Detection** — automatic direction, connection, model, context, confidence, and sampling.
- **Stage** — visibility, transitions, opacity, scaling, captions, overlap, size, and media preloading.
- **Data & health** — export, restore, permissions, and diagnostics.

### Character editor

When the `characters` and `ui_panels` permissions are available, the character editor receives a compact **LumiStage** tab for managing that character without opening the full Studio.

### Direct LumiStage

The input-bar action and stage sparkle button open the quick selector:

1. Choose a character.
2. Choose an outfit.
3. Search or scroll through the expression catalog.
4. Choose an expression.
5. Choose the exact variant.
6. Apply once or create a lock.

The expression catalog and exact-variant pane are independent scroll regions, so large libraries remain usable without compressing cards into unreadable strips.

---

## Importing media

LumiStage accepts:

- PNG
- JPEG
- WebP
- GIF
- muted looping WebM
- muted looping MP4

Individual images are limited to 25 MiB and videos to 100 MiB.

### Direct import

When importing into a selected expression, every chosen media file becomes a variant in that exact stable outfit and expression ID. The expression keeps its existing name; filenames do not rename it.

### Folder mapping

Folder-relative paths are preserved from the browser picker.

| Path | Result |
|---|---|
| `Happy.png` | Expression `Happy` in the selected/default outfit |
| `Formal/Happy.png` | Outfit `Formal` → expression `Happy` |
| `Formal/Happy/soft.png` | Outfit `Formal` → expression `Happy` → variant `soft.png` |

The import preview shows the resolved mapping before upload. Ambiguous paths are rejected instead of being guessed.

### Additive behavior

Media import is additive:

- the current unsaved Studio draft is sent with the import request;
- draft edits and imported variants commit atomically;
- unrelated outfits and expressions are retained;
- explicit destinations resolve by stable ID;
- identical content uploads once but may be referenced in multiple destinations;
- failed commits clean newly uploaded orphan media when possible.

`.lumistage.zip` files are not accepted as ordinary media. Use **Restore archive** for that destructive operation.

---

## Automatic direction

Automatic direction is enabled by default and can be disabled under **Settings → Detection**.

### Completion timing

LumiStage starts tracking when Lumiverse emits `GENERATION_STARTED`. While that generation is active, message-edit and swipe notifications cannot queue automatic analysis. A detector pass is scheduled only after a successful `GENERATION_ENDED` includes the committed assistant message ID.

Stopped or failed generations do not change the automatic expression.

Edits and swipes of already committed messages can still reconcile the saved timeline. **Analyze now** remains an explicit manual force action.

LumiStage reads persisted rows from `spindle.chat.getMessages`, marks them internally as `__isChatHistory: true`, and filters context through that marker. Only each selected row’s `role` and `content` enter the detector request. Lumiverse preset prompt blocks are not part of this history payload. The quiet call still inherits the selected connection’s generation parameters, but not its assembled preset messages.

### Detector contract

The detector:

- receives every active character, outfit, and expression name without receiving variant IDs or filenames;
- leaves exact variant selection to LumiStage, which randomly chooses among the selected expression's eligible variants;
- receives current stage state and active manual locks;
- waits for the exact completed assistant message and coalesces duplicate host/manual triggers into one structured generation call;
- sends an empty quiet-call model override for “connection default,” preventing legacy preset model fields from replacing the connection’s resolved model;
- must copy exact outfit and expression names from the catalog rather than inventing labels;
- may switch outfits whenever the completed scene supports it unless a manual outfit/state lock constrains the choice;
- inherits the selected connection’s reasoning configuration and exposes any host-parsed reasoning in the session-only drawer debugger; enabled reasoning may increase latency and token usage;
- has a 60-second abort;
- does not send `max_tokens`, leaving the selected controller and model to manage their own output limit;
- fails clearly before generation when estimated input exceeds 24,000 tokens;
- rejects unknown or ambiguous outfit/expression selections and duplicate character decisions.

### Confidence behavior

Confidence applies to the complete proposed visual state. If any accepted state is malformed or below the configured threshold, LumiStage preserves the complete prior snapshot—including character focus—instead of partially changing the stage.

### Decision cache

Cached decisions are fingerprinted with:

- the active catalog;
- detector settings;
- relevant context;
- the completed message ID, swipe, and content;
- current manual overrides;
- the selected/default connection profile, preset, model, and update timestamp.

Changing any of those inputs invalidates the cache. Forced analysis always bypasses it.

---

## Manual direction and locks

| Action | Behavior |
|---|---|
| **Apply once** | Applies the selected exact state now and allows the next completed assistant reply to direct it again. |
| **Lock outfit** | Keeps the selected outfit while automation may choose valid expressions and variants inside it. |
| **Lock state** | Keeps the exact outfit, expression, and variant until cleared. |
| **Clear current lock** | Removes the character’s active manual override. |

Manual overrides are validated against the active chat catalog. Stale or cross-character IDs are rejected.

---

## Floating stage

The floating stage is:

- draggable and resizable;
- edge-snapping;
- fullscreen-capable;
- optionally chromeless;
- caption-aware;
- responsive to solo and group scenes;
- configurable globally or per chat.

A lone character—or an ensemble with no explicit focus—renders at full visual strength. Unfocused opacity is applied only when another character has explicit focus, so a missing focus decision cannot dim the entire stage.

Supported transition styles are:

- `Crossfade`
- `Lift`
- `Cut`

Failed or unavailable media is cleared instead of leaving a stale sprite visible.

---

## Settings reference

### Detection

| Setting | Default | Notes |
|---|---:|---|
| Automatic direction | On | Runs after successful completed assistant replies. |
| Connection | Default Lumiverse connection | May use a dedicated configured LLM connection. |
| Model | Connection default | Blank explicitly uses the resolved connection model; legacy model fields in its preset are ignored. |
| Context messages | 5 | Recent messages sent to the detector. |
| Temperature | 0.1 | Low by default for stable structured selection. |
| Confidence threshold | 0.6 | Applies to the complete accepted decision. |

### Stage

| Setting | Default | Notes |
|---|---:|---|
| Transition | Crossfade | Crossfade, lift, or cut. |
| Transition duration | 280 ms | Clamped from 0 to 2,000 ms. |
| Stage opacity | 100% | Overall floating-stage opacity. |
| Focused scale | 1.035 | Scale applied to explicitly focused characters. |
| Unfocused opacity | 46% | Used only when another visible character is focused. |
| Captions | On | Shows character, outfit, and expression labels. |
| Chrome | On | Shows the stage header and controls. |
| Ensemble overlap | 34% | Controls multi-character overlap. |
| Size | 320 × 420 | Resizable from the stage itself. |
| Preload adjacent media | 3 | Clamped from 0 to 12 variants. |

### Chat layout overrides

The active chat can save its own complete stage layout. **Use global layout** removes that override and returns to the global appearance settings.

---

## Archives and recovery

### Export

**Export archive** creates a `.lumistage.zip` containing:

- a versioned manifest;
- the active character profile;
- outfit, expression, and variant IDs;
- every referenced media asset.

Export fails if a referenced media URL cannot be resolved, preventing silently incomplete backups.

### Restore

Restore is intentionally separate from import:

1. Choose exactly one `.lumistage.zip`.
2. LumiStage previews its profile and media counts.
3. Confirm the destructive replacement.
4. The backend validates manifest, path, variant, and media-reference consistency.
5. The profile is replaced under revision control.

Mixed archive/media selections and malformed manifests are rejected.

Archive limits:

- compressed archive: 250 MiB;
- expanded content: 1 GiB;
- supported entries: 5,000;
- individual image: 25 MiB;
- individual video: 100 MiB.

---

## Permissions

| Permission | Why LumiStage uses it |
|---|---|
| `generation` | Run the structured automatic detector. |
| `chats` | Resolve the active character or group catalog. |
| `chat_mutation` | Read committed messages, swipes, and content hashes for timeline replay. |
| `characters` | Load character identity and mount the character-editor tab. |
| `images` | Upload, resolve, export, and safely delete LumiStage-owned media. |
| `ui_panels` | Register Studio, quick direction, and the floating stage. |

LumiStage checks capability availability and reports missing permissions in Diagnostics.

---

## Storage, ownership, and reliability

LumiStage stores V2 records under its private extension storage:

```text
settings.v2.json
profiles/<character-id>.v2.json
chats/<chat-id>.v2.json
```

Existing LumiStage V1 data migrates automatically. Stable media references are retained, the obsolete internal owner layer is removed, and incompatible detector cache records become cache misses.

### Revision safety

All persistent writes use monotonic repository revisions and compare-and-swap checks. This covers profiles, settings, imports, restores, deletion, timelines, and chat layouts.

The frontend ignores lower-revision state and keeps character-scoped profile data isolated. Dirty drafts are preserved when newer backend state arrives.

### Media ownership

Media is stored through Lumiverse’s authenticated image service. Before permanent deletion, LumiStage verifies that:

1. no saved LumiStage profile still references the image; and
2. Lumiverse reports `owner_extension_identifier === "lumi_stage"`.

Persistence success is reported immediately. Permanent cleanup runs as best-effort background work, so cleanup failure cannot turn a successful save into a false timeout or failure.

### Request completion

Every correlated frontend operation receives exactly one terminal completion or error response. Control operations use a 60-second timeout; media operations use a 10-minute timeout.

---

## Troubleshooting

<details>
<summary><b>Save says the backend changed while I was editing</b></summary>

Your draft is preserved and **Save changes** remains enabled. Saving rebases the preserved draft onto the latest backend revision. Choose **Reload profile** only when you want to discard the draft and accept backend state.

</details>

<details>
<summary><b>An import went to Default or renamed my expression</b></summary>

Update LumiStage and reload the extension. Current imports send the active draft and stable destination IDs; filenames cannot rename an existing expression. Folder imports preserve `webkitRelativePath` and show the destination mapping before upload.

</details>

<details>
<summary><b>The Direct LumiStage cards are thin strips</b></summary>

Update and reload LumiStage. Current builds give the expression catalog fixed-height grid rows inside a dedicated vertical scrollbox, with a separate exact-variant scroll area.

</details>

<details>
<summary><b>The stage sprite looks dim</b></summary>

A solo sprite and an ensemble without explicit focus render at full strength. **Unfocused character opacity** applies only when another visible character is explicitly focused. Also check **Settings → Stage → Stage opacity** if the entire floating stage was intentionally lowered.

</details>

<details>
<summary><b>The expression changes while the assistant is still writing</b></summary>

Update and reload LumiStage. Automatic analysis is blocked from `GENERATION_STARTED` until a successful `GENERATION_ENDED` provides the committed assistant message ID. Failed or stopped generations do not trigger it.

</details>

<details>
<summary><b>Automatic direction does not run</b></summary>

Check:

- Automatic direction is enabled.
- A completed assistant reply exists.
- At least one active character has imported media.
- Generation, Chats, and Chat History permissions are granted.
- The selected connection is usable.
- The catalog stays under the 24,000 estimated input-token limit.

Open **Settings → Data & health → Diagnostics** for the precise status.

</details>

<details>
<summary><b>Dropdowns appear behind the Studio</b></summary>

Current LumiStage builds install a scoped compatibility guard that raises Lumiverse select portals above extension modals. Reload the extension after updating so the frontend stylesheet remounts.

</details>

<details>
<summary><b>An archive will not import with normal media</b></summary>

That is intentional. Ordinary imports are additive; archives are destructive profile restores. Use **Settings → Data & health → Restore archive** and confirm the preview.

</details>

---

## Development

### Source layout

```text
src/
  backend.ts          backend entrypoint, events, requests, detector orchestration
  detector.ts         one-call structured detector contract
  importer.ts         media mapping, archive validation, profile merge
  model.ts            normalization, catalog, snapshots, batch operations
  ownership.ts        safe image deletion checks
  storage.ts          revisioned repository and migration
  timeline.ts         cache, replay, edit/swipe reconciliation
  types.ts            shared V2 records and wire messages
  frontend.tsx        host placements and floating-widget lifecycle
  ui/
    client.ts         correlated frontend requests and state ordering
    modals.tsx        import, restore, lightbox, and Direct LumiStage
    stage.tsx         floating-stage renderer
    studio.tsx        drawer, Library, Live Stage, Settings, character tab
    styles.ts         scoped Lumiverse-native UI styles

dist/
  backend.js          committed production backend bundle
  frontend.js         committed production frontend bundle
```

### Commands

```powershell
npm install
npm run typecheck
npm test
npm run build
npm run contract
npm run safety
npm run check
```

`npm run check` runs strict TypeScript, the complete Vitest suite, the production build, the independent-data source contract, and the extension safety scan.

Regression coverage includes:

- completed-generation timing and streaming suppression;
- request completion and timeout cleanup;
- revision conflicts and dirty-draft saves;
- unsaved outfit and expression imports;
- folder-relative mapping;
- duplicate-content reuse;
- archive validation and restore;
- ownership-safe deletion;
- whole-state confidence preservation;
- private random variant resolution and detector-driven outfit/expression changes;
- manual locks and forced cache bypass;
- multi-character isolation and group pruning;
- Direct Stage scroll rows;
- undimmed solo-stage rendering;
- responsive controls and host component cleanup;
- 2,000-sprite catalogs and eight-character stages.

After source changes, rebuild `dist/` and reload LumiStage in Lumiverse.

---

## License

LumiStage is provided under the [Lumiverse Community License 2.0](./LICENSE.md).
