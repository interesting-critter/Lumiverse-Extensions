# Cue speech (TTS) — scope, contracts, and truthful limitations

Cue can read the current assistant paragraph aloud using the user's **saved
Lumiverse TTS connection profiles**. The feature is **off by default** and lives
entirely in the frontend overlay.

## How it works

- Settings live in the "Speech (read paragraphs aloud)" card under the Visual
  novel settings tab (`src/frontend/speech/settings-ui.ts`), persisted in the
  extension config as `config.speech` (`src/speech-config.ts`).
- Voices are `{ connectionId, voice }` references: an opaque Lumiverse TTS
  profile id plus an optional provider voice id (empty = the profile's own
  default voice). Cue never stores provider URLs, keys, or secrets; the server
  resolves credentials.
- Resolution mirrors the stage nameplate semantics exactly
  (`src/frontend/speech/voice-resolution.ts`): paragraph speaker `""` is the
  narrator, a name is that character, `null`/absent falls back to the turn
  speaker. Chains: narrator → character default; character override → character
  default → narrator. Per-character overrides are keyed
  `chat::<chatId>::<lowercased name>` so equal names in different chats never
  collide. When nothing applicable is configured, Cue reports an honest
  "unconfigured" status and stays silent — it never falls back to an arbitrary
  first profile.
- Playback: a manual Play/Pause/Stop dock (`src/frontend/speech/ui.ts`)
  for the current paragraph, on a dedicated speech player (never the BGM/SFX
  channels). The dock is a top-center pill, clear of the bottom dialogue box
  and the reading controls, offset by `env(safe-area-inset-top)`. Pausing
  during startup takes effect; a play that settles after Stop or teardown
  stays stopped. Optional auto-play is a double opt-in: the setting must be
  enabled AND one successful gesture Play must have happened in the session.
- Synthesis uses the host's buffered `POST /api/v1/tts/synthesize` endpoint via
  a same-origin authenticated fetch (`src/frontend/speech/transport.ts`), only
  when the user plays speech. There is no backend-triggered synthesis, no
  prefetch, and no automatic retry. Limits: one in-flight request, 2000
  characters per paragraph (longer paragraphs report "too long" instead of
  truncating), 8 MiB response cap enforced while streaming, 60 s timeout
  reported as a visible error (never a silent reset),
  audio-MIME validation, session-only LRU cache (16 items) whose object URLs
  are revoked on eviction, chat change, close, and disposal.
- **Profile revision refresh**: every fresh Play first re-reads the selected
  profile's metadata (`GET /api/v1/tts-connections/:id` — a host database read,
  never a provider call or synthesis). The cache key includes the profile's
  `updated_at`, model, effective voice (override or the profile's default), a
  sorted-key fingerprint of `default_parameters`, and the exact outbound text.
  A host-side profile edit bumps `updated_at`, so the next Play misses the
  cache and synthesizes anew — cached audio from an older revision is never
  silently reused. Resuming a *paused* player replays the already-fetched audio
  without a new metadata read; the revision check applies to fresh Play
  dispatches. If the profile was deleted or the metadata read fails, Cue
  refuses to synthesize and shows the error instead of guessing.
- Lifecycle: cursor change, backward/forward navigation, chat switch/fork,
  swipe/edit rebroadcast handling, user submission, overlay close, hidden page,
  permission revoke, settings change, and teardown all stop playback and abort
  the in-flight request; late responses are dropped, never played or cached.
  Same-turn rebroadcasts (e.g. image/asset updates) never replay speech.

## Delivery styles (Gemini audio tags)

`Delivery style → Gemini audio tags` prepends **one user-chosen inline tag**
(for example `[whispers]`) to the *spoken* text only; the visible prose never
changes and Cue makes **no extra LLM calls** to pick emotions. The tag is
reduced to one safe token (letters, digits, space, hyphen, underscore; 40
characters max) — bracket or markup content is stripped, never sent.

**Route gating**: before each Play, Cue checks the selected profile's actual
model id (from the same metadata read that keys the cache). The tag is injected
only when that model is Gemini-family (`/gemini/i`). Any other provider/model
receives the tag **only** through the clearly-labeled compatibility opt-in
("also send the tag to non-Gemini profiles", off by default), because non-Gemini
providers may read the bracket text aloud. Tagged and untagged renditions are
distinct cache entries. This follows the
official Gemini speech-generation guide (fetched 2025; page last updated
2026-09-02 UTC): tags are probabilistic performance guidance with **no
exhaustive supported list**, English tags are recommended, and there is no
closing-tag syntax. Other providers may read the bracket text aloud, which is
why the mode is opt-in per user. Cue does not claim `parameters.instructions`
support for Gemini via OpenRouter (the host only forwards `instructions` for
`gpt-4o-mini-tts` models) and does not manufacture Gemini model ids; the
profile's configured model is always used.

## Truthful limitations (v1)

- **API surface**: there is no versioned `spindle.tts` consumer API or TTS
  consumer permission in `lumiverse-spindle-types` 0.6.23. Cue calls the host's
  authenticated REST routes (`/api/v1/tts-connections`, `/api/v1/tts/synthesize`)
  from the overlay, which runs in the host page and shares the session cookie.
  These routes are host-internal and may change between Lumiverse versions.
- **API base**: only the same-origin relative base `/api/v1` is used. A
  deployment that serves the UI from a different origin than the API is
  unsupported for Cue speech and fails with a normal error (Cue never reads
  tokens or proxies credentials to work around it).
- **One voice per paragraph**: attribution metadata is per-paragraph name only,
  so a paragraph mixing narration and quoted dialogue is spoken by one voice.
- **Host TTS auto-play overlap**: Lumiverse's own message auto-play (if enabled)
  may speak the same message at the same time. Coordination between the two
  players is **not verified**; Cue warns in settings and never mutates the
  Lumiverse global setting.
- **Cancellation is best-effort upstream**: aborting a request may not stop
  provider-side billing; stale completions are still discarded locally.
- **Profile edits mid-session**: every fresh Play re-reads the profile revision
  and the cache key covers profile id, `updated_at`, model, effective voice,
  parameter fingerprint, delivery formatting, and exact outbound text, so a
  host-side edit misses the cache and synthesizes anew. Two deliberate limits
  remain: cached audio cannot replay while the host API is unreachable (the
  revision read comes first, by design), and an edit landing between the
  revision read and the synthesis request is resolved by the host at synthesis
  time (the route accepts no pinned revision).
- **No measured audio quality claims**: no live synthesis was performed while
  building or testing this feature; all tests are offline with mocked
  transport/fetch, and profile/voice listing in settings happens only on
  explicit button presses (metadata only, never synthesis or profile tests).
