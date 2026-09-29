<!-- Mirror of upstream documentation. Everything below the metadata table is the original file, unmodified. -->

| Field | Value |
| --- | --- |
| Extension | LumiWHYYY |
| Source repository | https://github.com/Archkr/Lumiverse-LumiWHYYY |
| Original link (from Lumiverse-Extensions README) | https://github.com/Archkr/Lumiverse-LumiWHYYY |
| Upstream path | `README.md` |
| Retrieved | 2026-09-29 @ `main` (`d95c92b`) |

---

<div align="center">

# LumiWHYYY

**Configurable scheduled fullscreen jumpscares for Lumiverse.**

[![Version](https://img.shields.io/badge/version-1.0.0-e75b3d)](./spindle.json)
[![Lumiverse](https://img.shields.io/badge/Lumiverse-%E2%89%A5%201.1.0-d4a35a)](https://github.com/prolix-oc/Lumiverse)
[![Permission](https://img.shields.io/badge/permission-ui__panels-6f9f78)](./spindle.json)

</div>

LumiWHYYY displays a fullscreen image and plays bundled audio after a configurable amount of visible Lumiverse time. A new countdown begins after each playback.

## Features

- Any positive finite interval, with no arbitrary maximum
- Seconds, minutes, and hours controls
- Fullscreen, cover-fitted Foxy image
- Bundled MP3 with volume and 0.25×–4× playback-speed controls
- Countdown that pauses while the Lumiverse tab is hidden
- Click or `Escape` to dismiss immediately
- A test button that does not arm recurring scares
- User-scoped settings that survive reloads
- Responsive, theme-aware drawer UI built with Lumiverse host components
- No network calls and no external media dependencies

## Installation

1. Open **Lumiverse → Extensions**.
2. Install `https://github.com/Archkr/Lumiverse-LumiWHYYY`.
3. Enable the extension and grant its `ui_panels` permission.
4. Open **LumiWHYYY** from the drawer or command palette.
5. Choose an interval, run a preview, and enable recurring playback.

Recurring playback is disabled on fresh installations. Enabling it or running a preview provides the user gesture browsers require before timed audio can play. After a reload, an enabled schedule waits for the first click or keypress in that session, then starts from the full configured interval.

## Behavior

- Only time spent with Lumiverse visible counts toward the next scare.
- A hidden tab preserves its remaining countdown instead of catching up on return.
- The next full interval starts after the audio ends or the overlay is dismissed.
- Revoking `ui_panels` removes the overlay and pauses the scheduler while preserving the enabled setting.
- Playback speed changes the current and future scare without altering the countdown interval.
- If audio playback fails, the fullscreen image remains visible for approximately the speed-adjusted MP3 duration and the drawer reports the problem.

## Permission

LumiWHYYY requests only `ui_panels`. Lumiverse gates fullscreen float widgets behind that permission. The JPG and MP3 are compiled directly into `dist/frontend.js`, so the extension does not need `media`, `images`, `cors_proxy`, or network access.

## Development

```bash
bun install
bun run typecheck
bun run test
bun run build
bun run check:dist
bun run contract
```

`dist/backend.js` and `dist/frontend.js` are committed release artifacts. The frontend build inlines `src/assets/jumpscare.jpg` and `src/assets/jumpscare.mp3` as data URLs because Lumiverse serves extension frontend code as a single JavaScript bundle.

## Project layout

```text
src/
  backend.ts          User-scoped settings and permission state
  frontend.ts         Spindle frontend entry
  runtime.ts          Browser lifecycle and host integration
  scheduler.ts        Foreground-only countdown state machine
  audio.ts            Reusable audio playback controller
  presenter.ts        Fullscreen overlay lifecycle
  ui/                 Preact drawer and themed styles
  assets/             Bundled Foxy JPG and MP3
tests/                Unit, UI, and source-contract coverage
scripts/              Deterministic production build checks
```

## License

LumiWHYYY uses the Lumiverse Community License 2.0. See [LICENSE.md](./LICENSE.md).
