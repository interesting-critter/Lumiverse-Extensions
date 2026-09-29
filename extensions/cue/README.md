<!-- Mirror of upstream documentation. Everything below the metadata table is the original file, unmodified. -->

| Field | Value |
| --- | --- |
| Extension | Cue |
| Source repository | https://github.com/japolino/cue-living-novel |
| Original link (from Lumiverse-Extensions README) | https://github.com/japolino/cue-living-novel |
| Upstream path | `README.md` |
| Retrieved | 2026-09-29 @ `main` (`aee7774`) |
| Note | Originally listed as `japolino/cue-visual-novel`, which no longer exists (HTTP 404). Resolved to the renamed canonical repository `japolino/cue-living-novel`; the source list link has been updated. |

---

# Cue — Living Novel

Cue is a Lumiverse extension that turns a chat into a living visual novel. It replaces the native message and input surfaces only while VN mode is active. Canonical chat messages remain the source of truth.

> The project was previously named *Visual Novel Preview*. Its install identifier remains `visual_novel_preview` so existing installations upgrade in place.

The preview includes:

NovelAI profiles automatically use model-aware prompt formatting. V4+ receives separate scene and character captions and numerical emphasis; V3 keeps a combined prompt with bracket emphasis. The saved profile model is used unless Cue overrides it. Negative numerical emphasis requires V4.5 or newer.

Under Advanced, NovelAI has toggles for model-specific quality tags and default negative tags. Custom positive prefixes and edited or empty negatives are preserved. Automatic quality tags are written into the request so they can be inspected; V4.5 Curated includes `rating:general` and reduced feet emphasis. These options do not affect other providers. The Lumiverse NovelAI adapter must also include the accompanying change that honors `qualityToggle: false` and empty negatives.

- paragraph-by-paragraph reveal with a final acknowledgement gate
- Previous button and Left Arrow to reread earlier paragraphs in the current reply. Going back pauses Auto and Skip, keeps your response draft, and does not rewind the chat. Skip still uses your selected mode.
- Standard typed-response and CYOA choice modes
- a single native image pipeline — planner → asset scheduler — with per-provider bounded concurrency and progressive delivery
- previous-image retention and decode-before-swap so a late image never blanks the stage
- exactly one centered protagonist per frame: no second character, crowd, or bystander
- a frozen per-chat character-identity/tag block, migrated automatically from the legacy visual-profile record
- a closed pose/expression catalogue, with deterministic selection by default
- fixed 16:9 camera scene planning and explicit scene-boundary checks
- swipe, edit, delete, duplicate-submit, and stale-image reconciliation
- view-gated generation: Cue only plans turns and generates images for chats whose Cue view is open; closing the view (or leaving the chat) aborts the in-flight image batch and marks it cancelled, opening the view plans the latest reply only when nothing current is stored, and cancelled images wait for a manual Retry instead of resuming on their own
- per-user and per-chat persisted continuity
- a settings tab, a user-selectable scene-image fit (Cover / Contain / Stretch / Original size / Scale down), five built-in theme presets (Lumiverse, Golden hour, Boxed console, Paper novel, Midnight noir), and a shadow-DOM custom CSS contract that is always the final styling layer
- an always-accessible **Back to chat** control that restores native Lumiverse

It targets Lumiverse staging `1.1.6`, audited at commit `33dfa9ee62999fa3e2567066ed5cdadf61635323`, and `lumiverse-spindle-types` `0.6.23`.

## Optional System One decisions

Advanced settings configure Cue's own System One URL and model. Save the API key in Cue's settings; Lumiverse encrypts it in Cue's private enclave. Cue sends Jev's typed `state` and `questions` request through the Spindle HTTP proxy. This requires the `cors_proxy` permission. If no key is saved or the request fails, Cue uses its existing story reader.

**Compare** logs decision latency, input tokens, and speaker/expression agreement without changing the turn. **Use for presentation and familiar scenes** applies confident speaker, expression, and audio decisions. For replies of up to seven short paragraphs, Cue can reuse a known scene without calling the prose planner when Jev finds no scene or visual change. Longer replies use the story reader and split Jev's presentation questions into requests that fit the API limits. This setting is off by default.

## Image pipeline and identity

When System One is On and returns confident expression decisions, the picture limit applies to new renders after cache lookup. Cue checks expressions across the classified paragraphs, reuses compatible images without spending generation slots, and generates distinct missing images in paragraph order up to the limit. Repeated requests for the same missing image share its render. Other misses keep the current image; uncertain continuation paragraphs and repeated expressions do not force a swap. Jev currently classifies the first 24 paragraphs. Off, Compare, unavailable classifiers, and native card art retain their existing image-budget behavior.

Generated scene images come from the one built-in path: `src/backend/runtime/planner.ts`, then `src/backend/runtime/images.ts`, then the per-provider `AssetScheduler`. The sidecar planner proposes only scene boundaries, environments, and paragraph cues. It never supplies free-form pose, expression, or a prompt delta; those fields are emitted empty and ignored by the prompt compiler.

Every image shows exactly one centered protagonist composed from a stable identity/tag block and a pose suffix from a finite catalogue. The tag block is frozen per chat and is never auto-updated by a later turn. By default, pose is selected by a pure function of paragraph index and text; optional System One decisions can choose another pose from the same catalogue. The camera is fixed to a centered, eye-level, medium-wide 16:9 composition with the lower quarter clear for the dialogue surface.

The pipeline never mutates canonical chat messages. It writes only extension-owned projections: per-turn records, per-chat state, and the single-character visual-state identity record.

As with any generative system, image-provider speed and visual identity quality depend on the selected model, provider, and prompt settings. Cue does not promise a specific provider result.

RisuAI card greetings that pack several alternative scenes into one message are macro-resolved before planning, so only the selected scene is shown. Limitation: no host event fires when a scene picker in the chat is clicked, so a changed selection is picked up only when Cue is reopened, when state is requested again, through the Try again control on the waiting card (`vn_refresh`), or through Try again on a visible turn (`vn_retry_turn` re-resolves first and replans when the selection changed). Volatile macros such as `{{random}}`, `{{roll}}` or `{{time}}` in a message re-resolve on every check without counting as a new scene; a message whose host resolution fails keeps its stored turn.

## Scene image reuse (temporary cache)

Cue keeps a small, in-memory cache of the scene images it generated so an
exact-compatible cue can reuse an image instead of sending a new provider
request. The cache is reuse-first and generate-on-miss; it never generates
ahead of time and never relaxes a match.

**What "exact compatible" means.** A cached image is reused only when all of
these are identical: the durable character id and subject class (never just
the display name), the resolved appearance tags and wardrobe, the effective
environment (location, time and weather, lighting, description, persistent
elements), the closed-catalogue pose/expression, the bounded action/prop, the
camera framing, and the exact provider request (positive and negative prompt,
connection and workflow, provider, model, your image parameters, prompt
syntax, and the reference-anchoring toggle). Scene ids, cue ids, paragraph
numbers, the turn key and free-form
`promptDelta` are not part of the match, so alternating speakers (Mira, Alex,
Mira) in the same room still reuse. Composition and camera locks are not
rendered into prompts and are ignored. Cues whose planner run did not persist
a character id, resolved appearance and subject class never use the cache.

**Lifetime.**

- Memory only, per backend worker. A Lumiverse restart starts empty. Nothing
  is written to storage and no image bytes are held: entries are pointers
  (image id, URL, provenance). Bounded to 256 entries / 1 MiB of metadata,
  least-recently-used first.
- Scoped to one user and one chat. Switching chats releases the previous
  chat's entries. The backend learns the active chat from the host
  `CHAT_SWITCHED` event and from the frontend's `vn_get_state` request (sent
  on every chat switch); a chat that only ever produced generation events
  without either signal is released when its message is deleted or the
  worker restarts.
- Scoped to one physical scene episode. The planner keeps `priorSceneId`
  across speaker switches and changes it only on a real boundary (location
  change, major time jump, environment replacement, forced). Entries from an
  earlier episode are retired, so leaving a room and returning to an identical
  room does not reuse the earlier visit. Deleting the active message (chat
  scene lineage restarts) releases the whole chat scope.
- Late results are never admitted. Every new generation, swipe, edit,
  regeneration, cancel and asset batch start advances the chat's admission
  epoch; a provider result that arrives afterwards is kept for its own turn
  record but not cached.
- Forced regeneration ("Regenerate reply" / "Try again"): unfinished jobs are
  re-made without a lookup and any image they showed is dropped from the
  cache. Finished images are kept, as before.
- Missing assets: before a reuse the extension checks `images.get`; a deleted
  or unverifiable image is dropped and normal generation runs. `IMAGE_DELETED`
  host events drop entries too.
- Eviction or invalidation never deletes an image and never rewrites a stored
  turn; historical turns keep their image ids.

**Extra swaps beyond the image cap.** The planner still generates at most
`maxImagesPerTurn` images. Cues beyond the cap are kept as reuse-only
candidates (`cacheCues`, up to 16). A candidate becomes an extra image only
when an exact-compatible image is already cached: at planning time (it then
appears in the first turn view) or later in the same turn when a budgeted image
lands (it then arrives as a normal asset update). A candidate with no cached
match produces nothing: no job, no request, no error, no retry. Such assets
are marked `source: "cache"` and are excluded from generation progress and
from the "Try again" wording.

**Concurrent requests.** If two compatible cues are in flight at once (for
example a superseded batch still running), only one owns the provider
request; the other waits and shares the result. An aborted owner releases the
waiter immediately; a failed owner fails the waiter once (no retry cascade).

**Reference anchoring first.** The portrait decision runs before any reuse.
A render that must capture a character's portrait (anchoring on, no
compatible portrait yet) is always generated, never served from the cache; it
is still stored for later cues. Toggling reference anchoring on or off changes
compatibility, so images made under the other setting are not reused.

**Limitations.** Exact match only. The captured portrait itself is not part
of the match (only the toggle is), so once a portrait exists, renders anchored
to it and the capture render of the same appearance count as compatible; your
own reference or source settings in image parameters do count. A swipe that re-enters a room
starts a new episode. The cache does not survive a restart. Reuse means the
same Lumiverse image id is shown in several turns; deleting it from the
gallery affects all of them, exactly as today.

**Debugging.** With **Debug logging** on, `[VN] scene-cache ...` lines report
each hit (with the source message and an estimate of the generation time
avoided), each miss with its reason (`absent`, `evicted`, `invalidated`,
`episode_retired`, `bypass`, `portrait_capture`, `asset_missing`,
`asset_unverifiable`, `identity_unresolved`), each store and rejection (`stale_admission`,
`aborted`, `no_image_id`), waits for in-flight owners, candidate resolutions,
scope releases, and a per-batch summary (hits, shared, avoided, stores,
misses, rejects, waits, evictions, entries, bytes).

## Run the standalone preview

```powershell
bun install --frozen-lockfile
bun run serve:demo
```

Open `http://localhost:4173` for CYOA mode or `http://localhost:4173/?mode=standard` for typed input.

## Install into a local Lumiverse staging checkout

Build the bundles:

```powershell
bun run verify
```

Copy this directory, including `dist`, into:

```text
data\extensions\visual_novel_preview\repo
```

Enable the extension and grant its requested permissions in Lumiverse. Enter from **Visual novel** in the chat header, or **Composer Extras → Open visual novel**. The settings tab controls connections, generation limits, planning context, prompts, scene-image fit, the visual-novel theme preset, and the custom CSS layer.

Opening an existing chat bootstraps its latest assistant reply automatically. A new empty chat needs its first normal assistant reply before there is anything to present.

The overlay feature-detects the staging component override API. If that API is absent, it leaves native chat intact and reports that the build is unsupported.

## Development

### Pinned status cards and inline HTML

Open **Panels** at the top left of VN. Newly planned replies expose complete HTML/SVG blocks when their paragraph is reached. Choose **Pin this card** for a saved snapshot, or **Keep it updated** for a uniquely identified status source. Drag the title, drag the lower-right resize handle, or use arrow keys on the title; Shift + arrows resizes. Positions and pins are stored per chat in this browser. **Reset positions** recovers misplaced cards; **Unpin all** removes saved cards.

SimTracker is optional. Panels shows available cards from the current reply first; **Advanced tools** holds SimTracker capture, pasted HTML, and status template rules. Opening or closing advanced tools does not disable saved rules or pinned updates.

Settings starts with presentation, AI connections, reading, and audio. Expand **Advanced settings** for prompt editing, context controls, model overrides, JSON parameters, text filtering, and custom CSS. These controls keep their saved values while hidden.

For a regex status template, expand **Add status template rule**, enter the pattern without slash delimiters, flags, and the multiline HTML replacement. Captures through `$36` and named captures are supported and escaped. Host macros in replacements are resolved through Lumiverse without committing variable changes. Use **Refresh live sources** after changing language variables. Rules operate on the first 200,000 source characters and reveal at the end of the turn. Multiple matches offer snapshots only, to avoid following the wrong character.

Rules do not change filtering. **Ignored tags** hides whole blocks from dialogue and image planning; **Display regex rules** affects dialogue only. Recognized ignored status tags remain available as plain-text cards. Complete HTML blocks are extracted separately before narrative planning. Previously cached plans may not contain panel sources.

Frames allow CSS layouts, SVG drawings, and CSS-only checkbox/radio interactions, but no scripts or navigation. Remote images/fonts are blocked unless explicitly enabled for that frame. A content update rebuilds that card's frame; other cards retain their state. Followed values from previous turns are hidden until the source appears in the current turn. Snapshot pins deliberately remain visible.

For stock SimTracker, let its card render in normal chat, open VN, then choose **Capture SimTracker snapshot**. This captures markup and checkbox state, not JavaScript behavior. Live updates require the companion adapter in [integrations/simtracker-vn-bridge.ts](./integrations/simtracker-vn-bridge.ts), wired to SimTracker's own template output. It is not installed into SimTracker automatically. The adapter must return only the requested chat/message/swipe's cards, call `refresh()` after updates, and `destroy()` on teardown. No DOM nodes are moved between extensions.

Browser regression checks:

```powershell
bunx playwright install chromium
bun run test:panels
```

```powershell
bun run test
bun run typecheck
bun run build
bun run build:demo
```

See [SPEECH.md](./SPEECH.md) for the default-off paragraph speech (TTS) feature, its contracts, and its truthful limitations.

See [ARCHITECTURE.md](./ARCHITECTURE.md) for the viability decision, staging host contract, image pipeline, deterministic single-character identity, data ownership, state machines, and remaining preview limits.
