<!-- Mirror of upstream documentation. Everything below the metadata table is the original file, unmodified. -->

| Field | Value |
| --- | --- |
| Extension | Vishrun |
| Source repository | https://github.com/Vishousdev/vishrun |
| Original link (from Lumiverse-Extensions README) | https://github.com/Vishousdev/vishrun |
| Upstream path | `README.md` |
| Retrieved | 2026-09-29 @ `main` (`514dba5`) |

---

# Vishrun

Vishrun is a Lumiverse Spindle extension that renders embedded HTML/CSS/JS widgets from SillyTavern character cards (`regex_scripts`).

## What it does

SillyTavern character cards can carry embedded UI widgets in a `regex_scripts` field — small HTML/CSS/JS payloads that render inside chat messages when the AI emits a trigger such as `【FOO】` or `<TAG>...data...</TAG>`. Lumiverse imports these cards but does not interpret `regex_scripts`, so the widgets remain inert. Vishrun fills that gap.

When the active character has `regex_scripts` entries, Vishrun watches messages as they render. On a placeholder trigger, it replaces the trigger inline with the widget — a sandboxed iframe if the payload contains `<script>`, otherwise a styled `<div>`. On a paired-tag trigger, it strips the tag from view and renders the widget with the captured data substituted via `$1..$N` backreferences. Rendering is idempotent: reloads, swipes, and message edits all re-render cleanly without modifying the persisted chat history.

The engine is card-agnostic. No card-specific code lives in the extension; any card whose `regex_scripts` use the supported feature subset (placeholders or paired tags, raw regex source, `$0`/`$N` backreferences, self-contained HTML/CSS/JS replaceString) works without modification.

## External assets

Widgets often reference external image URLs (e.g. character chibis hosted on catbox.moe). Vishrun supports them transparently:

- **`<img src="https://...">`** in widget HTML — both static in the replaceString and runtime-inserted via `innerHTML` / `outerHTML` / `insertAdjacentHTML` / `setAttribute` / `img.src = ...`.
- **CSS `background-image: url(https://...)`** in `<style>` blocks and inline `style="..."` attributes within the replaceString.

External URLs are fetched server-side through Lumiverse's CORS proxy and served back to the widget as `blob:` URLs, which the sandbox iframe's CSP allows.

**Requires the `cors_proxy` permission to be granted.** This permission is privileged: declaring it in `spindle.json` is not enough — an owner/admin must grant it manually from the Extensions panel after install. Without the grant, widgets render but external images stay broken.

## Status

Verified end-to-end on Lumiverse staging (`d157784` and forward) with cards using placeholder triggers (including embedded JS), paired tags with zero, single, and multiple capture groups, and CSS `background-image` external URLs in `<style>` blocks and inline `style` attributes.

## Not yet supported

- **External audio (`<audio src="https://...">`).** The sandbox iframe CSP currently blocks `media-src` for external URLs. Cards with BGM render visually but stay silent.
- **Unpaired HTML-like tags (`<Opening>`, `<Selection>` without a closing tag).** Lumiverse's tag interceptor only matches paired `<TAG>...</TAG>` shapes, and DOMPurify strips unknown HTML tags before Vishrun sees them. On hold awaiting upstream support for `matchOpenOnly` semantics.
- **MVU (`MagVarUpdate`).** Cards that depend on `<UpdateVariable>` parsing, `_.set()` execution, or persistent `stat_data` will not have reactive widget updates. Planned as the next major feature (backend interceptor).
- **Runtime CSS manipulation** (`el.style.foo = 'url(...)'`, `setProperty`, `cssText`, `insertRule`, adopted stylesheets). External URLs assigned via these APIs at runtime are not proxied. Static CSS in the replaceString is fully supported.
- **JS-Slash-Runner globals beyond `setChatMessages`.** Other globals (`getChatMessages`, `createChatMessages`, `triggerSlash`, etc.) are not shimmed. Add per-card if a real card surfaces a need.
- **Prompt-side substitution (`placement: 1`).** Vishrun operates on the render side only and does not modify the prompt sent to the LLM.

## Install

- Open Lumiverse, go to Extensions, click Add extensions, choose Install from source, paste this repo's GitHub URL, and click Install from source. Once installed, enable the extension, grant the `cors_proxy` permission from the Extensions panel, and refresh the browser tab.
- For local development, clone the repo to `Lumiverse/data/extensions/vishrun/repo` and follow the standard Spindle build flow (`bun build` produces both `dist/frontend.js` and `dist/backend.js`).
