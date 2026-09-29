# Cue — Living Novel architecture

## Viability decision

The overlay is viable on Lumiverse staging. The decisive API is `ctx.ui.registerComponentOverride`, combined with an `app-overlay` mount. While VN mode is active, the extension replaces `BubbleMessage`, `MinimalMessage`, and `InputArea` with null components and renders its own full-screen stage. Destroying those override handles restores native chat immediately.

This is a staging contract, not a portable main-branch contract yet. The extension therefore feature-detects it at activation time. It never edits Lumiverse source or hides native UI permanently.

There are two host limitations:

1. `InputArea` exposes no send or stop callbacks. VN submissions cross the frontend/backend bridge, then use `spindle.chat.appendMessage(..., { triggerGeneration: true })`.
2. Spindle exposes no public stop method for a normal generation started that way. The overlay keeps an Exit control outside user CSS so the user can always return to the native composer and stop there.

## System map

```text
Lumiverse staging host
  |
  +-- frontend context
  |     +-- input-bar action
  |     +-- settings tab
  |     +-- app-overlay mount
  |     +-- component overrides
  |     +-- backend message bridge
  |              |
  |              v
  |        VN frontend controller
  |              |
  |              +-- VnStage
  |              |     +-- reveal/input state store
  |              |     +-- nested theme shadow root
  |              |     +-- image preload and decode
  |              |
  |              +-- safety shadow root
  |                    +-- permanent Exit control
  |
  +-- backend Spindle worker
        +-- generation and message events
        +-- request router
        +-- per-chat planning queue
        +-- context assembly
        +-- scene and cue planner
        +-- upper-body single-subject prompt compiler (vendored Inlay)
        +-- per-provider image scheduler
        +-- reference portrait store
        +-- audio catalog and import
        +-- chat mutation
        +-- per-user storage
```

## Source layout

| Area | Responsibility |
|---|---|
| `src/frontend/host` | staging API feature detection, activation, overrides, chat switching, frontend/backend routing |
| `src/frontend/stage` | framework-free full-screen VN renderer, accessibility, keyboard input, image swaps |
| `src/frontend/store` | deterministic paragraph, acknowledgement, choice, and input gating |
| `src/frontend/settings` | Lumiverse settings surface |
| `src/frontend/theme` | stable `data-vn-*` selectors, base theme, scene-image fit attribute, CSS isolation, network-fetch stripping |
| `src/backend/runtime/controller.ts` | host event handling, submission reconciliation, active-turn ownership, visual-state and portrait load/save |
| `src/backend/runtime/message-text.ts` | macro resolution and cleanup of assistant message text before planning |
| `src/backend/runtime/planner.ts` | sidecar request, JSON repair and fallback plan, scene/cue/choice construction, active-character and attire attribution, per-paragraph speaker nameplates, deterministic expression assignment |
| `src/backend/runtime/images.ts` | deterministic prompt compilation through the vendored Inlay compiler, image generation, per-provider asset scheduling, reference-portrait anchoring, asset updates |
| `src/backend/runtime/storage.ts` | versioned per-user config, per-chat state, protagonist visual state, per-chat portrait store, durable character-appearance memory, serialized writes |
| `src/backend/runtime/audio-catalog.ts` | audio library scan, BGM/SFX categorization, tag matching |
| `src/backend/inlay-prompt` | vendored Inlay prompt compiler: tag assembly, visibility filtering, environment sections, ComfyUI/NovelAI syntax rendering |
| `src/backend/core/visual-state.ts` | frozen protagonist tag block, schema-v1 profile migration |
| `src/backend/core` | host-neutral queues, contracts, continuity reduction, boundary decisions, stale guards |
| `src/shared/character.ts` | closed pose/expression catalogue (92 entries), pure expression selection, single-character identity types |
| `src/shared/contracts.ts` | strict Zod trust-boundary schemas |
| `src/protocol.ts` | narrow frontend/backend message protocol |

## Message intake and macro resolution

RisuAI cards ported through LumiRealm keep every alternative first scene inside one greeting message, wrapped in CBS `{{#when}}` blocks, plus a `$messageSelector` placeholder line that a display regex script turns into a scene picker. The raw stored text never changes; picking a scene only sets a chat variable.

Cue therefore never plans raw message text. Every path that feeds a message into planning (GENERATION_ENDED, swipe/edit reconcile, chat switch, `vn_get_state` bootstrap, retry, `vn_refresh`) goes through `resolveMessageIntake` in `src/backend/runtime/message-text.ts`:

1. Fast path: a message without macro syntax and without a placeholder-only line is returned byte-identical; the host is not called.
2. Bounded pure-subtree masking: a token is masked only when it is a safe, output-position pure transform subtree (containing only pure volatile macros `random`/`pick`/`roll`/`time`/`date`/`weekday`/`isotime`/`isodate`/`datetimeformat` and pure string transforms `upper`/`lower`/`capitalize`/`trim`/`reverse`, with no control structures, variable reads/writes, or external context). If a message contains any stateful/mutating macro (`counter`, `toggle`, `rcounter`, `setvar`, `addvar`, `let`, etc.) or volatile macros in control positions, masking is bypassed entirely: the host executes a single full-raw dry resolve, preserving exact whole-message evaluation order and shared environment state.
3. For maskable messages, `spindle.macros.resolve(maskedText, { chatId, commit: false })` runs the host macro engine once; this single call chooses the branch. Each masked token that survived in the chosen branch is then evaluated on its own (`commit: false`) to fill the planning text in. A resolve that throws, a missing API, or `{{#...}}` blocks surviving the resolve (no interceptor loaded) mark the intake `resolved: false`.
4. `cleanResolvedText` removes what is left: unresolved `{{#...}}...{{/...}}` blocks together with their content (an unselected scene must not leak), unresolved inline `{{...}}` tokens, and placeholder-only `$identifier` lines. `<pimg>`/`<img>` tags, `{{img::...}}` references, and `{{char}}`/`{{user}}` display macros pass through; the planner substitutes the display macros with real names.

The intake yields two strings. `text` is the exact narrative the planner, the stored turn (`resolvedSourceText`, also handed to the image/reference pipeline) and the frontend use. `selectionText` retains the indexed placeholders (`\u2062N\u2062`), so distinct tokens in alternative branches produce distinct fingerprints. The record stores `source: { version: 2, rawFingerprint }`. `relateRecord()` tests `rawFingerprint` first: an edited message (even changing only volatile token arguments) is immediately classified as a plain edit (`changed`), replanning with continuity intact and no identity reset. When raw text is identical and selection text matches, the record is `current`. When raw text is identical and selection text differs: if the message had only stable pure masking (`stableSelection: true`), it is classified as `reselected` (replan + single-turn greeting identity reset); if the message had unmaskable volatile macros in conditions or stateful blocks (`stableSelection: false`), it is classified as `changed` (replanned to show the newly rolled branch, but identity state is never reset). Limitation: unmaskable volatile macros in conditions/headers re-roll on reopen, incurring replan cost as on older builds; identity reset is prevented, avoiding data loss.

History messages read for planner context resolve the same way, bounded to messages that contain macro syntax and cached per message id + swipe within one planning run. `vn_get_state` re-resolves the latest assistant message when it contains macro syntax, so a selection change is picked up when the view opens; `vn_refresh` does the same on demand. When the host cannot resolve right now (`resolved: false`) and a turn is already stored for that message, the stored turn stays authoritative: it is sent as is, never replaced by a waiting state or a replan built from the stripped raw text.

Unselected-greeting rule (evidence-based, no bare length heuristic): a message enters the waiting state only when the raw text contained macro blocks or a placeholder line AND either the cleaned text is empty, or there is explicit default-branch evidence — the raw greeting carries a placeholder-only line, has at least two selectable blocks, and the cleaned text equals the first block's single-line body (the branch such cards fall into while the selection variable is unset). Then nothing is planned and no images are generated; the backend sends `vn_waiting` and the reading view shows a card asking the reader to pick a starting scene in the chat. A legitimate short opening scene without that evidence is planned normally. Detection is structural; the placeholder wording is never matched.

When a greeting is re-resolved to a different scene (same stored text, different selection) while it is the chat's only assistant turn, the chat's durable identity state (frozen protagonist visual state, character registry, per-chat appearance roster, scene lineage and scene-image cache scope) is reset to the empty pre-greeting baseline before replanning, so the discarded scene's cast cannot leak into the newly selected one. Mid-chat turns and plain edits of the greeting are never reset. Retry (`vn_retry_turn`) re-resolves macro-bearing messages first and replans when the resolution changed. A cancel, deletion, edit event, or new generation that arrives while a macro resolve is in flight supersedes that intake (per-chat intake epoch), so a stale resolve can never enqueue a plan afterwards.

## Turn flow

### View gating

The backend only does automatic work for chats whose Cue view is open. The
frontend announces visibility with `vn_view { chatId, open }` and repeats the
flag as `viewOpen` on every `vn_get_state` (activation, chat switch), so a
restarted backend relearns the state from the next request. The boot request
after a page load says nothing about the view (`vn_get_state` without
`viewOpen`, no `vn_view`): the backend may still hold the view open from before
the reload, and its image batch must survive until the config says whether
`autoEnter` reopens the view. The first `vn_state` reply then either activates
(which announces `open:true`) or sends an explicit `vn_view open:false`. The
backend keeps one open chat per user in `runtime/view-registry.ts`.

Gated triggers: `GENERATION_ENDED`, the `MESSAGE_SWIPED` / `SWIPE_EDITED` /
`MESSAGE_EDITED` reconciles, and the `vn_get_state` bootstrap. A gated trigger
is skipped entirely (no planner call, no image jobs) when `config.enabled` is
false or the chat's view is closed; with `debugLogging` on, each skip is traced
(`skipped ...: view closed`). Only an explicit announcement opens the view:
`vn_view` with `open:true`, `vn_get_state` with `viewOpen:true`, or an explicit
user action from the open view (`vn_submit`, `vn_retry_turn`, `vn_refresh`;
choices submit through `vn_submit`). A `vn_get_state` without `viewOpen` changes nothing, so a
background request never opens a closed view.

Closing the view (`vn_view` with `open:false`, or `vn_get_state` with
`viewOpen:false`) aborts the chat's in-flight work the same way
`GENERATION_STARTED` does — turn ownership dropped, asset batch aborted,
queued planning cancelled, scene-cache admission released — and persists
`cancelled` over queued/generating jobs of the aborted turn so nothing looks
stuck. That stamp is keyed to the aborted turn and skipped only when a new
owner (retry, reply, reuse) has claimed the chat's turn meanwhile; a view that
merely reopened still gets the stamp, plus `vn_asset` updates, because the
aborted batch never resumes on its own. Leaving a chat
sends `vn_view open:false` for the previous chat, and the home screen (empty
chat id) closes whichever view is open, so a chat nobody is watching never
keeps planning or generating. Opening one chat displaces the user's previously
open chat, whose work is aborted. Late results are rejected by the ownership
guards. Reopening goes through `vn_get_state`: a missing or stale stored
record (a newer reply arrived while closed, detected by message id/fingerprint)
plans the latest reply once, and `vn_state` then carries `turn: null` so the
stale turn is never rendered first. Cancelled jobs stay cancelled; they are never
resumed on their own — the reader reruns them by hand with `vn_retry_turn`.

```text
GENERATION_STARTED
  -> stage enters waiting state

GENERATION_ENDED
  -> locate saved assistant message
  -> key by chat + message + swipe + content fingerprint + revision
  -> enqueue one planning job for that chat
  -> gather recent and enabled structured context
  -> ask sidecar for scenes, cues, speakers, and optional choices
  -> resolve the protagonist identity (seed once from planner/card/appearance memory, then freeze) and map each cue's expression onto the closed catalogue
  -> attribute each paragraph to a literal speaker nameplate and each cue to its active character, attire, and optional BGM/SFX
  -> validate claimed scene changes against objective evidence
  -> persist TurnPlan, the visual state, and any new appearance memory
  -> show paragraph 0 immediately
  -> enqueue all cue images by visible/next/background priority

image result
  -> verify active turn and scene revision
  -> persist generated asset
  -> send URL to browser
  -> preload and decode without clearing previous image
  -> swap only after decode succeeds
  -> acknowledge browser readiness to backend
```

Entering VN mode in an existing chat with no stored projection first sends the current config, then plans the latest non-empty assistant message through this same queue. The stage shows `Planning scene` while that bootstrap is running. Simultaneous state requests deduplicate to one plan.

If an image is late, the previous decoded image remains visible. If an obsolete image finishes after an edit, swipe, deletion, or newer turn, the ownership guards reject it.

## Reveal and input state

The current paragraph is readable before any response control appears. Each click or supported key advances one paragraph. The final paragraph needs its own acknowledgement. Only then does the stage reveal either generated/authored choices or the typed composer.

Choices are presentation metadata. Selecting one submits its configured value as a normal user message. Standard input does the same with typed text. Both routes attach a unique request ID. If Lumiverse writes the user message but fails while starting generation, the backend checks chat history before reporting the failure and never retries the same request blindly.

## Scene image fit

The rendered scene image obeys a user-selectable fit preset persisted in `config.json`. `VnStage` writes a `data-vn-scene-image-fit` attribute on the scene image element whenever the saved config changes, and the base theme maps each value to the matching CSS `object-fit` (cover, contain, fill, none, scale-down). Cover stays the backward-compatible default, `object-position` remains centered, and the fit is driven from config rather than user-supplied custom CSS, so it applies inside the shadow DOM regardless of theme.

## Theme presets

The stage ships exactly five built-in visual presets: **Lumiverse**, **Golden hour**, **Boxed console**, **Paper novel**, and **Midnight noir**. Their ids are the single canonical, host-neutral `THEME_PRESET_IDS` tuple and the derived `VisualNovelThemePreset` type, both declared in `src/config.ts` so the shared config module never imports frontend or browser code. `src/frontend/theme/presets.ts` only supplies the CSS payload (`THEME_PRESET_CSS`) keyed by those ids; the settings selector (`THEME_PRESET_OPTIONS`) and the config normalizer consume the same tuple, so the three can never drift. The removed experimental `retro-crt` id is gone everywhere.

Inside the nested theme shadow root the style layers are appended in a fixed cascade order, made explicit by `THEME_STYLE_LAYER_ORDER` (`src/frontend/theme/style-layers.ts`):

1. `base` — the platform `VN_BASE_CSS` (stable `data-vn-*` selectors, the scene-image fit mapping, and default `--vn-*` values).
2. `preset` — exactly one scoped block from `THEME_PRESET_CSS`, selected by `data-vn-preset="<id>"`. The preset *style element* lives in `themeRoot`, after base and before user CSS, and never in the outer safety root.
3. `user` — the sanitized custom-CSS layer. It is always last, so a user rule of equal specificity wins over a preset rule.

The `Lumiverse` preset maps the VN custom properties onto the real host tokens the Lumiverse page exposes — `--lumiverse-text`, `--lumiverse-text-muted`, `--lumiverse-primary` (accent), `--lumiverse-card-bg` (card), `--lumiverse-border`, `--lumiverse-font-family`, and `--lumiverse-*` key-control tokens (`primary-contrast`, `fill-medium`, `bg-elevated`) — each with a hard fallback so a host that does not export a token still renders the original default look.

The frontend controller applies the whole presentation config (theme preset, scene-image fit, custom CSS) on every save and on every `vn_state` / `vn_config` response through a single `applyVisualConfigToStage` helper, always from the *merged* config, so a saved patch never leaves the stage on stale values. The Exit control stays in the outer safety root, unreachable by any preset or user CSS.

## Scene model

A scene owns:

- structured location, time, weather, lighting, and persistent environment elements
- a `cast` list, an optional active `character` and `attire`, and the identity/tag block (`identityPrompt`)
- a reusable base prompt and a declared composition lock (planner metadata; the current compiler does not forward `basePrompt`, the composition lock, or a 16:9 requirement to the provider)
- prior-scene lineage
- a scene-scoped `ambient` effect id (weather/mood layer; persists until the next scene boundary)
- zero or more paragraph cues, each carrying an expression id, an optional one-shot `effect` id, and optional per-cue character, attire, BGM, and SFX

The planner may propose a boundary, but the deterministic boundary reducer accepts a new scene only for an initial scene, location change, major time jump, environment replacement, or explicit force. Emotion, pose, ordinary action, punctuation, and camera wording do not create a new scene. Where a boundary is not justified, the previous scene (and its camera, composition, and base prompt) is reused so the frame stays stable.

Compiled camera framing is fixed to `upper body, eye level, straight-on` for every cue. The scene still carries a declared composition lock and the dialogue-safe-region intent as planner metadata, but the compiler does not currently emit them; framing stability comes from the fixed camera tags plus scene reuse. Keeping faces clear of the dialogue UI therefore depends on the provider/workflow dimensions rather than an enforced prompt constraint.

### Environment continuity, structured changes, and stale contradiction prevention

Continuing scenes (`!decision.startsNewScene`) preserve established descriptive environment fields by default. This prevents LLM paraphrases, generic atmospheric rewrites, or omitted fields from eroding established visual details (such as specific props, architectural elements, or lighting setups).

1. **Planner-only input vs stored contracts**:
   - `PlannerSceneSchema.environment` accepts `PlannerEnvironmentInputSchema` (`persistentElements`, `removedElements`, `changes`), and `PlannerSceneSchema.environmentChanges` accepts `PlannerEnvironmentChangesSchema`.
   - These patch operations are planner-only: during normalization, all operations are evaluated deterministically and the resulting scene is strictly parsed into `SceneEnvironmentSchema` (`location`, `timeOfDay`, `weather`, `lighting`, `description`, `persistentElements`).
   - Stored `SceneEnvironment` and `SceneState` never carry transient operation keys.

2. **Structured `environmentChanges` schema & semantics**:
   - `description?: string`: Explicitly replaces the textual scene description. There is intentionally no `clearDescription`; description can only be replaced, or is re-synthesized automatically when lighting/time/weather changes without a replacement description.
   - `lighting?: string | null`: Explicitly replaces or clears lighting state (null explicitly clears lighting).
   - `timeOfDay?: string | null`: Explicitly updates or clears time of day (null explicitly clears timeOfDay).
   - `weather?: string | null`: Explicitly updates or clears weather (null or 'none' explicitly clears weather).
   - Location changes belong only in `boundary` and `environment.location` — there is no location field in `environmentChanges`.
   - `addElements: string[]`: Appends new persistent elements without duplicating existing elements.
   - `removeElements: string[]`: Targeted removal of specific persistent elements by exact name or core noun.
   - `replaceElements: Array<{ from: string, to: string }>`: Replaces matching element `from` with `to` (e.g. `{ from: "closed window", to: "shattered window" }`).
   - `clearElements: boolean`: Clears all persistent elements to an empty array.
   - `clearLighting: boolean`: Clears lighting to `null`.
   - `clearWeather: boolean`: Clears weather to `null`.

3. **Preservation vs legacy progression**:
   - Absent an explicit `environmentChanges` patch, continuing scenes preserve `base.description`, `base.lighting`, and `base.persistentElements`.
   - Generic rewrites (even longer descriptive fluff) cannot erase established physical props (e.g. `television glowing`).
   - Legitimate backward-compatible progression in standard fields (`location`, `timeOfDay`, `weather`, and distinct lighting transitions like `lights off` or `dark`) is accepted.
   - Generic mentions (e.g. `bookshelf`) never downgrade or replace detailed established items (`tall mahogany bookshelf packed with grimoires`).
   - Compound props on furniture (e.g. `laptop on desk`) are added as new objects without replacing the furniture (`wooden desk`).

4. **Clear behavior and sentinel preservation**:
   - `normalizeWeather` preserves the `'none'` sentinel through normalization so `mergeEnvironment` can distinguish an explicit clear from an omitted/unchanged field (`null`).
   - `mergeEnvironment` clears `weather` to `null` upon receiving `'none'` or `clearWeather: true`.
   - Final scene construction sanitizes any residual `'none'` to `null`, ensuring literal `'none'` never leaks into compiled prompts.
   - `lighting: "none"` or `clearLighting: true` clears lighting to `null`. Valid ambient descriptors (`normal`, `ambient`, `clear`) are preserved as legitimate lighting conditions and are not converted to `null`.

5. **Stale contradiction neutralization**:
   - When lighting, timeOfDay, or weather changes and the planner does not provide a new description, stale contradictory prose (e.g. `at dusk`, `warm lamps lit`) is dropped and re-synthesized from the new location and lighting.
   - When lighting explicitly becomes dark or off (`lights off`, `dark`, `blackout`, `unlit`), light-state modifiers on persistent elements (`lit`, `brightly glowing`, `burning`) are neutralized (`lit lamp` -> `lamp`), preserving the object without contradictory glowing text in the compiled prompt.

## Stage effects

Stage visuals are layered procedurally (CSS + inline SVG only; no network assets):

- **Ambient effects** (scene scope): `rain`, `heavy_rain`, `snow`, `sakura`, `fog`, `fireflies`, `embers`, `vignette_dark`, `sepia_flashback`, `desaturate`, `dream_haze`, `danger_pulse`. The planner proposes one per scene (`PlannerSceneSchema.ambient`); omitted ambient values inherit on a continuing scene while its weather stays unchanged; changed weather derives a new weather overlay (or clears it). Explicit `null` always clears. Accepted scene boundaries resolve their own ambient, independently of image cues. Weather effects animate real particle fields (multi-depth falling rain/snow, fluttering petals, drifting firefly glow, rising embers); `heavy_rain` adds a wet-lens pass (subtle blur plus refraction droplets). Grade effects (vignette, sepia, desaturate, haze, danger pulse) apply only to the scene image or a dedicated overlay — never to the dialogue box.
- **One-shot cue effects** (paragraph scope): `shake`, `shake_hard`, `rumble`, `zoom_in`, `zoom_out`, `zoom_punch`, `tilt`, `heartbeat`, `flash_white`, `flash_red`, `lightning`, `speed_lines`, `blur_pulse`, `fade_to_black`, `fade_from_black`, `fade_to_white`, `sparkle_burst`, `hearts_burst`, `confetti`. The planner selects sparse, story-grounded beats, not a per-turn quota. `TurnPlan.effectCues` stores paragraph effects independently of the image limit; legacy records fall back to `visualCues[].effect`. An explicit empty `effectCues` list suppresses that fallback. Each effect runs once and self-clears.

Pipeline: the planner normalizer accepts supported aliases and wrapped values, then validates against the closed catalogues in `src/shared/contracts.ts`; unknown values never reach the renderer. Ambient resolution distinguishes omission from explicit `null`, derives weather overlays from rain/storm/snow/fog when applicable, and preserves a continuing scene's ambient when omitted. Misplaced cue ambients route to their containing proposed scene; scene effects route to its start paragraph. Cross-catalogue values route by their supported id, with the named catalogue taking priority for ambiguous aliases. Unsupported values emit debug-gated planner trace lines. `TurnView` exposes per-paragraph `effects` and `ambients` arrays. Effects are omitted when empty; ambient arrays are omitted only when every scene lacks ambient data, so an all-null array explicitly clears a prior overlay. Scene `startParagraph` boundaries drive ambient changes even when no image cue exists. The host preserves ambient when data is absent and forwards explicit clears. The `[VN] planned` trace lists paragraph effects and scene-start ambient values (including `null` and omitted values). The stage renders ambient markup inside the scene container (`data-vn-scene-ambient` drives scene-image filters; a sibling overlay carries particle/grade layers) and one-shot bursts in a dedicated fx overlay above the scene but below the dialogue box. Particle placement uses a seeded avalanche hash with stratified horizontal slots so fields cover the full width deterministically; all animation respects `prefers-reduced-motion` (static grade only, no motion).

## Asset state machine

```text
queued -> generating -> generated -> browser_ready
                    \-> failed
queued/generating   \-> cancelled
```

`generated` means the provider returned a persisted image ID and URL. `browser_ready` means the active browser decoded that image successfully. Those are intentionally separate states.

Jobs carry their owning turn key, scene ID, scene revision, paragraph index, prompt fingerprint, provider key, and priority. The prompt fingerprint includes the byte-identical identity/scene/camera block and the resolved pose id and suffix, so distinct poses and distinct identity states produce distinct jobs. Concurrency is enforced per provider. This lets another provider progress without violating a slow provider's limit.

## Canonical data and storage

Lumiverse chat messages are canonical. Paragraphs, choices, scenes, cues, and asset state are extension-owned projections. Stored records are versioned and scoped by user:

```text
config.json
character-appearance.json
character-appearance-migrated.json
chats/<chat-id>/state.json
chats/<chat-id>/visual-state.json
chats/<chat-id>/characters.json
chats/<chat-id>/character-registry.json
chats/<chat-id>/portraits.json
turns/<chat-id>/<assistant-message-id>/<swipe-id>.json
```

Per-path writes are serialized because `userStorage` has no transaction or compare-and-swap operation. Chat state points to the active turn and carries the latest accepted scene and terminal continuity. The visual-state record holds the frozen protagonist identity and the latest environment descriptor; the portrait record holds each character's canonical reference image ID and data; the appearance map remembers canonical tags per character name across chats.

## Image pipeline

The extension has one native image pipeline. `planner.ts` asks a sidecar model — or a deterministic fallback planner when the sidecar is unavailable — for scene boundaries, environments, paragraph cues (expression plus optional character, attire, BGM, SFX), per-paragraph speakers, and optional choices, then normalizes, repairs, and validates that JSON into a strict `TurnPlan` (one repair retry, then fallback). `images.ts` compiles each cue into a prompt with `compileImagePrompt` and turns the plan into asset jobs with `createAssetJobs`, which a per-provider scheduler (`AssetScheduler`) generates with bounded concurrency and delivers progressively to the browser.

Compilation goes through the vendored Inlay compiler (`src/backend/inlay-prompt`) configured for Anima-style tag assembly, ComfyUI section formatting, and `maxCharacters: 1`. The assembled prompt is, in order:

```text
prefix + subject (1girl|1boy|1other, solo) + identity tags (with any attire override applied)
       + expression/pose suffix + environment (location, time+weather, lighting, background elements)
       + camera (upper body, eye level, straight-on) + suffix
```

`PROMPT_PREFIX`, `PROMPT_SUFFIX`, and the negative prompt come from `config.ts`; the subject line comes from the cue's persisted `subjectCategory`, falling back to explicit gender words in the identity text; the pose suffix comes from the closed catalogue. The scene's `basePrompt`, composition lock, environment `description`, and declared aspect ratio are planner metadata that the current compiler does not forward to the provider.

Where the extension owns the data, behavior is deterministic: a byte-identical content fingerprint keyed by message, swipe, and content makes generation events, reconnects, and edits idempotent; stale asset results are rejected by ownership guards; previous visual state is reused so the scene prompt stays stable until a justified boundary; and the pipeline never mutates the canonical chat message.

### One subject per frame, switchable active character

The planner instruction requires exactly one visible character per frame and forbids a second character, a crowd, or a bystander; the compiler forces `solo` into every prompt. Unlike earlier builds, the visible character is not always the same protagonist: the planner's `characters` output may describe several cast members, a scene can name an active `character`, and an individual cue can switch it again (`cue.character`, with persona names filtered out). Attire can likewise be overridden per scene or per cue. The frame is single-subject, but the subject can change between cues.

### Identity, appearance memory, and tag block

Character and clothing events resolve in paragraph order before the image-count limit is applied. Each new cue stores its own `resolvedIdentity` and nullable `resolvedAttire`; scenes retain their opening state. `terminalVisualState` and indexed continuity deltas carry the final subject and wardrobe into the next turn, including changes after the last generated image. Opening-cue repair never copies a future character or outfit backward. Planner entries also accept open-ended `species` and `anatomy` fields, preserving unfamiliar visual traits without expanding a species whitelist.

The protagonist's physical identity is a frozen per-chat tag block stored at `chats/<chat-id>/visual-state.json` as:

```text
{ schemaVersion: 2, protagonist: { name, tags }, environment, updatedAt }
```

The active character's baseline is seeded from a matching planner entry, the card, or chat-scoped appearance memory. A later description of the same character cannot overwrite a usable baseline, but changing the active character selects that character's own baseline. Missing appearances never inherit another character's body. The image job reports an unresolved appearance and Retry reruns planning. Document-shaped noise is rejected rather than stored as canonical memory.

A roster at `chats/<chat-id>/characters.json` remembers appearances within each chat. Runtime planning and image generation do not import the old user-wide `character-appearance.json` by name, since unrelated chats can reuse names such as Guard. Existing per-chat visual state seeds the scoped roster on the next planned turn; other cast members are relearned from matching planner entries. Legacy storage APIs remain available for compatibility.

### Stable character ids, explicit aliases, and subject category

`chats/<chat-id>/character-registry.json` stores one entry per character: a stable `id` derived from the canonical name, the canonical `name`, explicit `aliases`, the appearance `tags`, and a durable `subjectCategory` (`female | male | nonbinary | nonhuman | unknown`). Chats created before the registry existed are promoted on load from `characters.json` and `visual-state.json`; nothing is written until a planned turn merges new declarations.

Aliases are explicit only. The planner sees each known character's id in `KNOWN CHARACTERS` and may set `characterId` on a scene, cue, or character entry, or list `aliases` on an entry, when the story makes clear that a label such as "Fox girl" names Kitsune. Every reference is resolved to its canonical name before the cue timeline runs, so the relabelled character keeps its body, wardrobe, portrait key, and continuity. Resolution is conservative: a label that already names a known entity is never re-pointed by an id (the id is ignored and the conflict is reported in `rejectedAliases`); only a `characters[]` entry persists an alias; a scene or cue id link canonicalizes the current turn only and never creates an entry; an id that does not exist is ignored. A label with no explicit link stays a new subject even when it shares a species or appearance; two kitsune characters are two entries. A usable baseline is never overwritten by an incomplete later description.

`subjectCategory` is persisted separately from anatomy. Species and animal-ear tags never set it; only explicit gender or presentation statements do (planner `subjectCategory`/`gender`, or explicit words in the tags such as "fox girl", "woman", "male", "nonbinary"). It is filled once and then durable: a later per-turn value that disagrees is reported in `rejectedSubjects` and ignored, so one planner slip cannot flip a character to `1other`. The only correction path is a request that matches explicit gender words in the entity's own frozen baseline tags (for example a `fox girl` baseline wrongly stored as `nonhuman` can become `female`); the per-turn description is never consulted, so partial descriptions cannot drift the category. A baseline without gender words has no in-app correction path and requires editing `character-registry.json`. Scenes, cues, and `terminalVisualState` carry `characterId` and `subjectCategory`; the prompt compiler uses the persisted category first and only falls back to text classification when it is `unknown`, after stripping species+anatomy compounds so "cat ears" can no longer produce `1other`.

### Reference portrait anchoring

When reference anchoring is enabled, supported providers capture a character portrait in `chats/<chat-id>/portraits.json`. Reuse requires a matching fingerprint of the normalized name, identity tags, provider, connection/workflow and configured model. Unversioned or incompatible portraits are not sent to the provider; a successful fresh capture replaces them. Concurrent cues for the same character wait for capture, while unrelated characters can render concurrently. NovelAI receives director reference images; ComfyUI and SwarmUI receive source images. Unsupported providers do not anchor.

### Closed pose/expression catalogue

Pose and expression belong to a closed, bounded catalogue (`POSE_EXPRESSION_CATALOGUE`, currently 92 entries, hard cap 128, unique ids, non-empty suffixes). The planner proposes a cue `expression`; a valid catalogue id is used directly, otherwise selection falls back to a pure function of `(paragraphIndex, paragraphText)` — keyword match first, then a stable paragraph-index wrap-around. Each cue stores only a `poseExpressionId`; `compileImagePrompt` resolves it back to its exact suffix. Unknown or absent ids fall back to the first catalogue entry, so old stored cues and corrupt ids still resolve deterministically. The same paragraph and plan always produce the same compiled prompt.

### Bounded action and prop support

Props and physical interactions are modeled via a bounded, validated schema (`src/shared/action-prop.ts`) rather than arbitrary prompt deltas. This permits narrative gestures (such as "holding brass key in right raised hand") while preventing arbitrary prose or prompt injection from leaking into generated images.

- **Action catalogue & relationships**: Allowed actions belong to a closed set of physical verbs (`holding`, `carrying`, `wielding`, `raising`, `pointing`, `touching`, `showing`, `presenting`, `inspecting`, `resting_hand_on`, etc.). Allowed placements belong to a closed enum of 20 canonical relationships (`in right raised hand`, `in left raised hand`, `in right hand`, `in left hand`, `with both hands`, `held aloft`, `at side`, etc.). Conflicting hand and relationship values are rejected by schema validation.
- **Visible object noun grammar**: Props must adhere to a deterministic noun grammar consisting of a closed base noun catalogue (70 entries including `key`, `lantern`, `book`, `staff`, `dagger`, `sword`, `coin`, `pocket watch`, `compass`) with up to two allowed modifiers (materials like `brass`, `wooden`, `silver`; colors like `crimson`, `blue`; styles like `ornate`, `ancient`). Camera directives (`body`, `shot`, `framing`, `close-up`), anatomy directives (`hair`, `eyes`, `skin`), subject directives (`another girl`, `1boy`), clothing directives (`dress`, `boots`, `jacket`), and prompt-injection keywords are strictly forbidden and rejected.
- **Legacy backward compatibility**: `ActionPropFieldSchema` uses a preprocessor that normalizes legacy invalid strings (e.g. `"Mira turns"`, `"speaks"`, `"raises a lantern"`) to `null` while normalizing valid action strings or structured objects to a canonical `ActionProp`. Omitted or null actions default safely to `null`.
- **Compiler deterministic assembly**: `compileActionProp` renders action tags with underscores replaced by natural spaces (e.g. `resting_hand_on` becomes `"resting hand on"`). The compiled action tag is appended immediately after the catalogue pose suffix in both fixed and dynamic perspective modes. The pose catalogue expression is never overridden or replaced.
- **Isolation from identity and memory**: Props live strictly on cue scope (`cue.action`). They are never appended to `resolvedIdentity`, `visibleTags`, scene `identityPrompt`, `terminalVisualState`, or `characterAppearance` memory, ensuring props never freeze into the character's durable appearance. `portraitIdentityFingerprint` does not include props (preventing spurious reference portrait recaptures), while `promptFingerprint` does include them so prop changes trigger fresh asset generation. Cues attributed to the user/persona discard companion actions.

### Temporary scene-image cache (reuse-first, generate-on-miss)

`core/scene-image-cache.ts` owns an in-memory, bounded (256 entries / 1 MiB
metadata, LRU) cache of generated scene images, keyed by a structured
exact-compatibility identity built in `runtime/images.ts`
(`sceneImageIdentityFor`): durable character id + subject class, resolved
appearance + wardrobe, effective environment, pose, bounded action, framing,
and the exact provider request. Scene ids, turn keys and `promptDelta` are
excluded; `cameraLock`/`compositionLock` are never rendered and excluded.
Entries carry a physical scene episode (`priorSceneId ?? "initial:<scope
generation>"`): speaker switches keep the episode, real boundaries retire
earlier entries. Admission epochs (bumped by the controller on
`GENERATION_STARTED`, message reconcile events, cancel, retry and every batch
start) reject late results; `MESSAGE_DELETED` and chat switches release the
whole scope; `IMAGE_DELETED` and failed `images.get` checks drop pointers.
Eviction never deletes images or rewrites turn records.

`generateAssets` looks the cache up inside the scheduled executor (scheduler
priorities, concurrency and job states unchanged), claims single ownership of
an in-flight key (waiters share the result; an aborted owner releases them,
a failed owner fails them once), and stores successful normal generations.
Cues beyond `maxImagesPerTurn` are kept by the planner as `TurnPlan.cacheCues`
(max 16): `resolveCacheCues` turns a candidate into a terminal `generated` job
with provider `cache` only on a verified hit (at planning time before the first
`vn_turn`, on retry, and after every budgeted store in the same batch); a miss
creates nothing. `AssetView.source: "cache"` lets the frontend exclude those
swaps from progress and retry wording. See README "Scene image reuse".

## Audio pipeline

Audio is optional and user-supplied. The settings panel imports audio files from any folder in chunks that fit the host's 4 MB message limit; `audio-catalog.ts` scans the stored library, classifies each file as BGM or SFX from its path keywords, and derives searchable tags from file and folder names. When a library is present, the planner is asked for optional per-cue `bgm` and `sfx` names, which the frontend `audio-engine` resolves against the catalog and plays with looped music and one-shot effects. Chats without an audio library skip all of this: the planner is not asked for audio cues.

## Staging host contract

The audited remote staging commit is `33dfa9ee62999fa3e2567066ed5cdadf61635323`.

- Full-screen content mounts with `ctx.ui.mountApp({ position: "app-overlay" })`.
- Runtime overrides register through `ctx.ui.registerComponentOverride({ host, mode, priority, component })`.
- Lower numeric override priority wins. This preview uses priority `10`.
- Destroying a handle restores the underlying host component.
- Permission revocation clears host overrides. The frontend also exits VN mode when it receives an `app_manipulation` revocation event.
- Published types `0.6.23` do not accurately declare the staging override call, so one narrow local adapter contains the mismatch.

Primary references:

- [Lumiverse app mounts](https://docs.lumiverse.chat/frontend-api/ui-placement/#app-mounts-requires-app_manipulation)
- [Lumiverse frontend/backend communication](https://docs.lumiverse.chat/frontend-api/backend-communication/)
- [Lumiverse chat mutation](https://docs.lumiverse.chat/backend-api/chat-mutation/#append-and-generate)
- [Lumiverse backend events](https://docs.lumiverse.chat/backend-api/events/)
- [Lumiverse image generation](https://docs.lumiverse.chat/backend-api/image-generation/)
- [staging override registry](https://github.com/prolix-oc/Lumiverse/blob/33dfa9ee62999fa3e2567066ed5cdadf61635323/frontend/src/lib/spindle/component-override-registry.tsx)

## Preview boundaries

The preview proves the overlay and interaction architecture. It does not promise image-provider speed or visual identity quality, which depend on the selected model, provider, and prompt settings. Regular non-stream image generation cannot be cancelled upstream, so cancellation is cooperative and stale completions are discarded.

The overlay contract should be treated as staging-only until the same API lands in Lumiverse main. A future type-package release should replace the local staging adapter once it exposes the real override signature.
