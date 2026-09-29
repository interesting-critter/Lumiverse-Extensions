# Diagnostics

When something's off — a script won't run, QuickJS scripts seem to be silently falling back, memory feels high — the **Diagnostics** panel is the first place to look. It's also what you export when you ask for help.

## Where it is

Open the LumiScript panel → **Settings** → **Support** → **View Diagnostics**. The modal collects a live health snapshot across every subsystem. The **Download Report** button saves it as a timestamped Markdown file (`lumiscript-diagnostics-YYYY-MM-DD-HHMMSS.md`) — paste that into a bug report or Discord thread rather than screenshotting.

## What's in it

The report is a series of sections, each a list of checks that read *pass* / *warn* / *fail* / *info* (*fail* is the red, most-actionable one, and it drives the report's overall tone):

- **LumiScript** — extension-level state (enabled, version, key settings).
- **Script-runner subprocess** — the worker pool: how many workers are alive, per-worker memory, idle/eviction state. See [Worker pool](../concepts/workers.md).
- **Engine** — the active execution engine (below).
- **Active context** — the current chat / character the runner sees.
- **Registrations** — live handlers, tools, macros, subscriptions.
- **Storage** — per-script storage usage.
- **Assistant (Lisa)** — the in-app assistant's connection + config.
- **Editor / Monaco** — browser-side checks, collected in the app and appended after the backend report arrives: `document.fonts` state and readiness, blob-URL worker support, and Monaco CDN reachability (corporate-proxy / firewall detection — that probe can take up to ~5 s).

## The Engine section (v2.0)

The Engine section reflects whichever [engine](../concepts/engine.md) is active — it's titled **Engine (AsyncFunction)** or **Engine (QuickJS-WASM)** accordingly. On QuickJS it's the fastest way to answer *"is the isolate actually working on my machine, and did anything fall back?"* It reports:

- **WASM availability** — whether the isolate instantiated on your platform, and the cold-start time. Right after you switch to QuickJS, before any script has run under it, this reads *not probed*; it populates on the first QuickJS run. If it *failed*, every QuickJS run degrades to AsyncFunction.
- **Runs by engine** — the quickjs-vs-asyncfn run split.
- **Degraded runs** — QuickJS-requested runs that fell back to AsyncFunction (a platform / WASM-load signal). A non-zero count here explains "I switched to QuickJS but nothing changed."
- **Timeouts** — each one forces a whole-child respawn; the load-bearing stability signal.
- **In-VM out-of-memory** — a script hit its per-context memory ceiling (per-script isolation).
- **Streams** — `generateStream` opened vs cancelled-early.
- **Context pool** — under per-script isolation: live/cap contexts, how many are pinned, and the per-context memory limit. Reads *n/a* under shared isolation, where the pool is inactive.

## See also

- [Execution engine](../concepts/engine.md) — what the engine and isolation choices mean.
- [Worker pool](../concepts/workers.md) — the worker + memory state the Script-runner section reports.
