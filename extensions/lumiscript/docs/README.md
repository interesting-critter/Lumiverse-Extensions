# LumiScript Documentation

User-facing reference material for LumiScript — the Lumiverse scripting extension.

## Contents

- [**Documentation index**](./index.md) — start here: the full concept / guide / cookbook tree, plus how scripts fire and what `api` is.
- [**What's new in v2.0**](./whats-new-v2.md) — the headline QuickJS isolate engine, the new `api.*` surface, and the one upgrade heads-up (private-network egress).
- [**Execution engine & isolation**](./concepts/engine.md) — the v2.0 engine choice (AsyncFunction vs the QuickJS isolate) and per-script isolation, with the related [worker pool](./concepts/workers.md), [network egress](./guides/network.md), and [diagnostics](./guides/diagnostics.md) settings.
- [**CLI tools**](./cli-tools.md) — `pack2js` and `js2pack`, for converting between LumiScript pack archives and directories of individual `.js` files. Use these when you want to edit scripts in your own editor, keep them under version control, or author packs by hand.

More docs will land here as the feature surface grows.

## Other documentation surfaces

- **In-app reference** — LumiScript's Script Manager has a **Reference** tab that documents the runtime surface (events, permissions, macros, key types, `api.*` functions, built-in libraries, script-pack import/export). The tab's toolbar has an `Export Markdown` button that produces an exportable copy.
- **Source comments** — the `src/` tree is JSDoc-rich; header comment blocks on engine modules, API builders, and shared utilities describe intent and edge cases that aren't repeated elsewhere.
