# Changelog

## 1.0.16

### Added

- Generation protocol settings now include an explicit Anima/Illustrious
  prompt-family switch with family-specific subject serialization and matching
  copyable examples.
- Tagged prompts can use an internal scene-plan shape that binds character and
  persona identity text to visually anchored subject sections.
- Multi-subject scene plans now compile to deterministic, overlapping native
  SwarmUI regions with subject-local conditioning and a background region.
  Illustrious uses full regional strength; Anima uses a conservative
  experimental strength. Single-subject and ordinary POV scenes remain on the
  standard serializer.
- Named Character Looks inherit the existing visual identity and add outfit,
  negative, checkpoint, LoRA, reference, thumbnail, trigger, and note layers.
  Inline requests can select a Look explicitly or infer it from saved aliases.
- A native Character editor Visuals tab integrates with Lumiverse's PR #268
  character surface; Visual Lore now shares the main Visuals page instead of
  claiming a second drawer tab.
- Activated lore entries can persist visual identities for characters,
  locations, objects, creatures, outfits, and styles and merge those layers
  into tagged generation.
- Output Library now opens on folder preview cards and exposes character canon
  through a focused Prompts & visuals modal inside each bound folder.
- Output Library landing cards and opened-folder contents now use isolated,
  mutually exclusive surfaces, with folder controls above the image grid and a
  responsive sibling modal for base prompts and named Looks.
- The Library visual modal can create and fully edit character Looks in place,
  and its folder-toolbar icons share a consistent horizontal baseline.
- Visuals now has Character, Persona, and Lore tabs. Lore loads lazily after the
  backend listener exists, renders loading/permission/error shells, and puts
  entries activated in the current chat first.
- Character Look and Visual Lore reference images now resolve into the same
  bounded reference-conditioning payload used by Studio img2img, with a 0.60
  creativity default and graceful text-to-image fallback when resolution fails.

### Changed

- Multi-character guidance now enforces exact visible counts, spatial subject
  anchors, one-fact/one-owner rules, and separate shared interaction, camera,
  and environment layers. A character card is treated as an identity source
  that can resolve to multiple visible NPC subjects rather than a fixed count
  of one. POV guidance now distinguishes an invisible persona/viewer from
  `character="none"` scenery semantics.
- Illustration fallback cards use a compact horizontal strip so Generate,
  Retry, and Attach actions remain visible in narrow chat placeholders.
- Inspector reuse and init-image actions resolve exact embedded generation
  metadata using the verified Swarm output path before applying parameters.

## 1.0.15

### Added

- The parser-model override now offers connection-aware model suggestions. Studio
  asks the selected Lumiverse text connection for its model list when that host
  capability is available, then falls back to the connection's active model and
  cached model metadata without inventing provider-specific entries. Free-form
  overrides remain supported.
- Extension backend failures now print a scoped message, details, and stack trace
  through Lumiverse's terminal logger, with `console.error` as a fallback. Browser
  errors continue to surface normally.
- Outputs can now be starred from the current-output stage and the full-size
  inspector. Starred generations appear in a permanent **Favorites** collection
  while retaining their ordinary or character-folder membership.
- Library selection mode now supports **Non-starred only**, so **Select page** can
  safely target disposable outputs without selecting favorites.
- Bulk-selected outputs can be favorited or unfavorited in one action. Moving
  outputs to **Favorites** also stars them automatically.

### Changed

- Favorites behaves as a protected label-like collection rather than an exclusive
  destination: starring never removes an output from its existing folder, and
  the Favorites collection itself cannot be deleted.

## 1.0.14

### Added

- Studio settings now have a dedicated **Generation** tab for inline-image
  behavior, composition, image-count requirements, and the editable protocol
  prompt.
- Image requests can use either the existing one-pass **Inline protocol** or a
  new two-stage **Parser model** flow. Parser mode lets the chat model place a
  lightweight visual request in its reply, then expands that request in place
  through a selected Lumiverse text connection before SwarmUI generation.
- Parser mode hydrates Lumiverse's saved text connections, follows its default
  connection when no explicit choice is stored, and accepts an optional
  per-parser model override without changing the model used by the active chat.
- Parser failures are reported to both the browser console and Studio's normal
  backend error channel instead of leaving a silent unfinished request.

### Changed

- The `{{swarm_image_protocol}}` macro and automatic prompt injection now switch
  between the complete inline protocol and the compact parser-placement
  protocol according to the selected Generation mode.
- Swarm Studio requests Lumiverse's **Generation** permission only for parser
  connection discovery, generation lifecycle handling, and quiet parser calls.
  Existing inline-protocol behavior remains the default.

## 1.0.13

### Added

- Finished in-chat illustrations now offer **Remove from chat (keeps Library
  copy)** in their image-actions menu. Removal updates the saved chat message,
  clears its inline attachment metadata and retired job state, and deliberately
  preserves the generated Lumiverse Library output.

### Fixed

- `persona="active"` no longer falls back to the active persona's full
  biographical description when no explicit Studio visual profile is bound.
  Only the positive identity prompt from an enabled, bound persona visual
  profile is submitted to SwarmUI; otherwise the persona identity layer stays
  empty.
- Live message reconciliation now accepts an empty message body, allowing an
  image-only assistant message to be cleanly removed without requiring a page
  refresh.

## 1.0.12

### Fixed

- Inline-image regeneration now tracks the stable tagged-image job ID separately
  from the fresh per-attempt generation ID used for every random-seed retry.
  Replacing an old attempt no longer leaves Studio or Quick Create waiting for a
  terminal event under the superseded ID.
- Regeneration with current Studio settings, original settings, and a prompt
  edited through Quick Create now share the same terminal ownership cleanup.
  When the final image is attached to chat, both Studio and the floating player
  retire their progress, preview, and Stop/Interrupt controls together.
- Generic generation-start events preserve an already identified tagged-image
  source instead of reclassifying the retry as a manual Studio generation.
- Concurrent inline illustrations still hand the shared preview surface to the
  next active attempt after one completes.

## 1.0.11

### Fixed

- Lumiverse's shared `IMAGE_GEN_COMPLETE` event is now an authoritative
  completion fallback for Studio and Quick Create. Because that event already
  contains the saved image ID and URL, both surfaces immediately retire their
  progress and interrupt controls and display the final output even when the
  extension-specific result message is delayed or never delivered.
- The fallback preserves normal result hydration when it arrives later and
  continues to hand off correctly between multiple queued message images.
- Settled job IDs are now remembered across Studio, Quick Create, tagged jobs,
  host progress, and interruption/error paths. Late `generation_started` or
  progress messages can no longer resurrect a completed generation's controls.

## 1.0.10

### Fixed

- Completed message illustrations now emit an explicit terminal `ready` event
  before their full result payload. Studio and Quick Create retire their progress
  UI from either terminal event, so a delayed result can no longer leave
  “Preparing generation” and “Interrupt generation” stuck over a finished image.
- Tagged generation state now tracks concurrent message illustrations separately
  from manual Studio generations. A finished job hands the preview to the next
  genuinely active tagged job, while late queued/progress messages for settled
  jobs are ignored instead of resurrecting the spinner.

## 1.0.9

### Added

- Replaced the crowded gear popover with a responsive Studio settings modal:
  desktop uses a left navigation rail, while mobile uses a horizontally scrolling
  top tab strip. General, Theme, and Metadata now have independent pages.
- The complete inline-image protocol is editable in General settings, with
  explicit save/reset controls and a collapsible macro reference. The
  `{{swarm_dynamic_guidance}}` insertion point keeps image-count, identity,
  composition, checkpoint, LoRA, and preset guidance live inside a custom
  protocol.
- Added default-off controls to strip a character-bound LoRA stack from
  persona-only requests and to automatically prepend the active character
  positive prompt. Leaving automatic character printing off lets the model
  select only the relevant concrete identity tags from multi-NPC visual blocks.
- Added centered inline-image scale controls for full, 75%, and 50% display.

### Fixed

- Chat-tagged generations now register as the active shared generation in both
  Studio and the floating Quick Create player. Swarm stream preview frames and
  step progress are therefore visible again without restoring the removed
  in-message loading strip.
- The settings layer is mounted at the Studio shell instead of inside the
  transformed header controls, preventing clipping and stacking-context bugs on
  desktop and mobile.

## 1.0.8

### Fixed

- Inline image attachment recovery now identifies a completed tag by its
  immutable tag fingerprint instead of comparing the raw scene text with the
  fully layered generation prompt.
- The backend now scans the persisted chat for the matching tag and remaps a
  transient streaming message ID to the final saved assistant message before
  mutation. Recovery no longer depends entirely on the browser observing a
  second, non-streaming render pass.
- Remapped jobs acquire their mutation lock using the final message ID, keeping
  multiple images completing in the same message serialized without one
  attachment overwriting another.
- This failure was unrelated to IPv4 or remote mobile access; the same race
  could occur locally whenever character/persona layers or preset directives
  changed the submitted prompt.

## 1.0.7

### Fixed

- Enabled Swarm prompt presets now use native `<preset:exact saved name>`
  directives inside Studio's complete composed positive prompt. This preserves
  the user's scene prompt, character/persona visual layers, inherited LoRA
  triggers, and selected preset behavior in the same Swarm request.
- Studio strips the conflicting raw `presets` override from manual generation,
  including stale overrides restored from older saved Studio state. This
  prevents Swarm from replacing the submitted prompt after Lumiverse builds the
  request.
- Existing native preset directives are detected case-insensitively and are not
  duplicated. The `{{swarm_preset}}` macro expands to the enabled directive
  list in manual generation just as it does for tagged chat illustrations.

## 1.0.6

### Fixed

- Saved Swarm presets now serialize `loras` and `loraweights` using SwarmUI's
  native comma-separated parameter format instead of JSON-array strings.
- Preset application continues to accept legacy Studio-created JSON-array
  values, allowing their LoRA contents to be recovered into the editable stack.

### Diagnostics

- Every backend `studio_error`, image-generation error event, Studio error
  status, Quick Create failure, and Chat Visuals failure now prints its complete
  message to the browser console.
- Backend request failures also print the original `Error` object and stack to
  the Lumiverse server console before the sanitized frontend error is emitted.
- Browser error logs retain the complete backend payload as an expandable
  object, avoiding truncation by compact status bars and mobile layouts.

## 1.0.5

- Restored Extension UI registration on plain-HTTP remote sessions.
- Made inline tag interception optional on older Lumiverse builds.
- Registered the core drawer before optional host integrations.
