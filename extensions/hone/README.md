<!-- Mirror of upstream documentation. Everything below the metadata table is the original file, unmodified. -->

| Field | Value |
| --- | --- |
| Extension | Hone |
| Source repository | https://github.com/AMousePad/Hone |
| Original link (from Lumiverse-Extensions README) | https://github.com/AMousePad/Hone |
| Upstream path | `README.md` |
| Retrieved | 2026-09-29 @ `main` (`0538b85`) |

---

# Lumiverse Hone

LLM-powered message refinement for [Lumiverse](https://lumiverse.chat/). Refine AI responses, enhance your own drafts, and run multi-stage quality passes asynchronously.

> Full documentation lives in the **[Wiki](https://github.com/AMousePad/Hone/wiki)**. Start with [Installation](https://github.com/AMousePad/Hone/wiki/Installation) and [Quick Start](https://github.com/AMousePad/Hone/wiki/Quick-Start).

## Video Demo

TODO

## Why Hone?

LLMs that handle long creative context tend to drift. Anti-slop instructions, banlists, and "don't do X" prompts fight the model's own distribution. You're forcing it to suppress high-probability tokens *while also* writing creatively, and it gets harder to maintain quality writing while adhering to these rules the longer the cotext runs.

Hone lets your storywriting model generate creatively with full context, then edits the result in a second pass with a shorter context and targeted instructions. Whether you're polishing prose, translating a passage, enforcing lorebook consistency, or adding UI elements, only the message itself plus relevant context is needed. The main chat stays clean, and Hone quietly applies a revertable second pass whenever you want it.

More on the philosophy in [Core Concepts](https://github.com/AMousePad/Hone/wiki/Core-Concepts).

Full feature tour: [Interaction Surfaces](https://github.com/AMousePad/Hone/wiki/Interaction-Surfaces), [Pipeline Editor](https://github.com/AMousePad/Hone/wiki/Pipeline-Editor), [Strategies](https://github.com/AMousePad/Hone/wiki/Strategies), [Automation](https://github.com/AMousePad/Hone/wiki/Automation).

## Installation

Install via Lumiverse's Extensions panel using this repository's URL. See [Installation](https://github.com/AMousePad/Hone/wiki/Installation) for required permissions and first-run setup.

## Building from source

Requires [Bun](https://bun.sh/).

```bash
bun install
bun run build
```

Produces `dist/backend.js` and `dist/frontend.js`.

## Bugs & feedback

- [GitHub issues](https://github.com/AMousePad/Hone/issues): enable Debug Logging in Advanced settings, reproduce, click **Copy Debug Logs**, and paste into the issue
- Discord: `amousepad`

## License & credits

Hone has a split-license structure.

- **Source code, build scripts, tests, and documentation** outside `built-in-presets/` are under the **Hone Community License (Version 1.0)** (see [LICENSE](LICENSE) and [NOTICE](NOTICE)). Adapted by permission from the [Lumiverse Community License](https://github.com/Lumiverse-LLC/Lumiverse/blob/main/LICENSE.md) by Prolix OCs. Permits personal, academic, and non-profit use; requires improvements to be contributed back; forbids redistribution, public deployment, commercial use, government use, and AI/ML training without separate permission.
- **Bundled prompt presets in `built-in-presets/`** are under **CC BY-NC-SA 4.0** (see [built-in-presets/LICENSE](built-in-presets/LICENSE) and [built-in-presets/NOTICE.md](built-in-presets/NOTICE.md)). The prompt text is adapted from **[ReDraft](https://github.com/MeowCatboyMeow/ReDraft)** by MeowCatboyMeow (Discord: `catboytimemeow`), originally released under CC BY-NC-SA 4.0.

Derivative distributions may omit `built-in-presets/` to avoid the CC-BY-NC-SA obligations; the rest of the repository remains governed by the Hone Community License. See [License and Credits](https://github.com/AMousePad/Hone/wiki/License-and-Credits).
