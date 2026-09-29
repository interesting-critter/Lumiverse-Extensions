<!-- Mirror of upstream documentation. Everything below the metadata table is the original file, unmodified. -->

| Field | Value |
| --- | --- |
| Extension | Lummate |
| Source repository | https://github.com/WanderingMonk8/lummate |
| Original link (from Lumiverse-Extensions README) | https://github.com/WanderingMonk8/lummate.git |
| Upstream path | `README.md` |
| Retrieved | 2026-09-29 @ `main` (`d9772d1`) |

---

# Lummate

Lummate is a Lumiverse extension that turns erotic roleplay scenes into tactile playback for XToys.

Instead of firing blindly on every message, Lummate reads the scene, figures out who is involved, identifies the relevant contact, builds a beat-by-beat action plan, and sends the result to XToys in a form that can actually be used in real setups.

## Highlights

- **On-demand tactile playback**
  Add a `Play / Stop` control directly to Lumiverse messages and trigger tactile playback only when you want it.

- **Scene-aware parsing**
  Lummate parses erotic roleplay text into semantic actions like `thrust`, `ride`, `suction`, `stroke`, `finger`, `grind`, `lick`, and more.
  Hip-driven cues such as riding, rocking, bouncing, and twerking are folded into `ride` unless the scene is clearly describing circular grinding friction instead.
  The parser also hardens against common false positives by separating executed contact from quoted dialogue, promises, requests, and future-intent language.

- **Tracked-character targeting**
  Choose who the extension should follow in a scene instead of assuming the user is always the active participant.

- **Contact-zone filtering**
  Target the relevant zone for the tracked participant, including default and custom contact zones, so parsing stays grounded in the part of the scene you actually care about.

- **Group-scene support**
  Works across multi-character scenes, with participant-aware filtering and weighting instead of treating every scene like a one-on-one exchange.

- **Beat-by-beat action planning**
  Converts messages into ordered tactile beats with timing, continuity, and playback decisions rather than a single flat trigger.

- **Held-state and continuity handling**
  Preserves ongoing actions across messages so scenes can continue, transition, pause, resume, loop, or hold more naturally.

- **Per-action tactile calibration**
  Tune how each semantic action should feel on your own setup instead of relying on one universal interpretation.

- **XToys-ready webhook delivery**
  Sends XToys-compatible action triggers together with per-beat `intensity` and `seconds` values in the same webhook request.

- **Built-in safety controls**
  Apply a global intensity cap and a user-defined maximum ramp-time scale before values are sent to XToys.

- **Desktop and mobile controls**
  Supports desktop hover breakout controls and mobile long-press controls for a cleaner per-message UI.

## Current Status

This repository is now release-ready for the current v1 scope.

The Lumiverse-side extension has been verified in Lumiverse far enough to confirm:

- the extension installs successfully
- the frontend loads successfully
- a custom per-message action button can be injected into the native message action row
- frontend/backend messaging and runtime per-chat session plumbing are working
- per-message breakout controls work on desktop hover and mobile long-press
- the hidden parser-session arming/disarming lifecycle is implemented
- a Lummate settings drawer is available for parser connection selection, tracked-participant targeting, contact-zone targeting, debug toggling, XToys webhook configuration, and Lumiverse-side tactile calibration presets
- participant tactile profiles are derived, cached, regenerable, and reused
- group chats can cache more than one character profile when Lumiverse exposes the participant roster
- tracked participant selection is available per chat, with persisted contact-zone targeting
- Play now produces runtime message plans with ordered semantic/resolved beats
- current physical/emotional state parsing is implemented with baseline fallback
- the parser can use structured LLM output with deterministic zone-scoped fallback
- contact filtering now respects tracked participant ownership, tracked contact zones, and multi-participant scene context
- parser hardening now filters out many dialogue-only, promise-only, and prospective-intent misparses before they become tactile beats
- calibrated tactile resolution is now active, including preset fallback, XToys action-name mapping, participant contribution modulation, and safety-clamped resolved beats
- tactile calibration changes now affect resolved beat output
- runtime-only held-state continuity is now active, including entry-beat transition selection and end-of-playback resolution for replace/modulate/basic blend flows
- Lumiverse-side beat scheduling is now active, including beat-by-beat timer ownership, cancellation, loop handling, hold completion, and runtime scheduler-state verification
- XToys webhook delivery now follows the working private-webhook query-string path, sending one base action plus per-beat `intensity` and `seconds` values
- global XToys safety controls are now available for maximum intensity and maximum ramp-time scaling

The parser still mixes LLM output with deterministic fallback logic while the scene-matching rules are being tuned. Blend behavior is still a Lumiverse-side heuristic for now rather than a true XToys-side concurrent execution model. But the Lumiverse-side shell, state model, tracked-participant flow, profile cache, held-state controller, scheduler, and XToys compatibility delivery path are now active and usable inside Lumiverse.

## Version

Current release: `1.1.2`

The main working documents are:

- [lumiverse-xtoys-extension-design.md](./lumiverse-xtoys-extension-design.md) — end-to-end product and systems design
- [lumiverse-xtoys-implementation-plan.md](./lumiverse-xtoys-implementation-plan.md) — Lumiverse-side implementation breakdown

## Project Direction

The planned extension will:

1. attach playback controls to Lumiverse assistant messages
2. parse selected scenes on demand rather than auto-play on generation
3. resolve semantic scene actions into calibrated tactile plans
4. use Lumiverse as the sequencing controller
5. send XToys control events through a backend integration layer

## Near-Term Goals

- keep tightening tracked-participant and tracked-zone parsing across mixed multi-party scenes
- refine held-state transition heuristics, especially where concurrent actions may look like blend vs replace
- tighten the quantized XToys action contract and script examples
- reduce dependence on deterministic parser fallback as structured parser coverage improves
- deepen the calibrated contribution model and expose clearer resolution diagnostics where needed
- revisit deferred XToys-side live modulation and multi-toy routing after real-device feedback
