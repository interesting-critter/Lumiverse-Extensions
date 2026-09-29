<!-- Mirror of upstream documentation. Everything below the metadata table is the original file, unmodified. -->

| Field | Value |
| --- | --- |
| Extension | Prism |
| Source repository | https://github.com/kittyafterdark/Prism |
| Original link (from Lumiverse-Extensions README) | https://github.com/kittyafterdark/Prism |
| Upstream path | `README.md` |
| Retrieved | 2026-09-29 @ `main` (`a66c2cf`) |

---

# Prism

Prism gives Lumiverse scenes deterministic, reversible dialogue and thought colors without making roleplayers type formatting tags. Hybrid is the default for new installs: it emits portable `<font color>` identity tags, rehydrates the finished response, and asks only whether genuinely new tagged speakers should enter the confirmed registry.

## Master switch

Prism has a user-wide master switch at the top of the palette. Turning it off pauses prompt injection/macros, automatic persona coloring, DOM attribution/paint, Hybrid discovery, and post-generation hydration without deleting the saved registry, colors, or per-chat configuration. The disabled state persists across characters and chats until Prism is turned back on.

Existing persistent color tags already stored in message content are left untouched; the switch prevents Prism from adding or enhancing new runtime color behavior rather than destructively rewriting chat history.

## Engines

- **Local** — colors the rendered DOM only. Saved messages and model context are unchanged.
- **Hybrid** — asks the model for portable canonical font tags, enhances those tags locally, fills confident untagged gaps, and asks only whether genuinely new tagged speakers should be registered, kept temporary, or ignored.
- **LLM** — asks the model for canonical font tags and enhances those tags locally, without heuristic gap filling.

Existing engine choices survive migration. Hybrid is the default only for a fresh configuration.

## Prompt delivery

Prism can deliver its live registry automatically or expose it to a preset:

- **Automatic** appends the compiled Prism instruction through the generation interceptor.
- **Preset macro** disables that automatic append. Place `{{prismPrompt}}` wherever the full instruction belongs in the preset, or use `{{prismHexes}}` for only the current confirmed and provisional speaker-to-hex rows.

The Prompt settings include an autosaved custom instruction editor. Templates can use `{{prismRegistry}}`, `{{prismHexes}}`, `{{prismProvisional}}`, and `{{prismPalette}}`; Prism appends essential registry data when a custom template omits it.

## Everyday workflow

1. Open Prism from the chat toolbar.
2. Use **Set up scene** to import Cortex/transcript colors and generate collision-safe colors for anything missing.
3. Edit solid or two/three-stop gradient paint directly in the roster editor. The first dialogue stop is always the character’s canonical registry color.
4. In Local or Hybrid, right-click, long-press, or keyboard-open any detected segment to correct its speaker or content type. Enable **Bake manual corrections** under Cortex & attribution if those corrections should also be written into the stored message as portable font tags.
5. In Hybrid, click the yellow **Awaiting review** status only when Prism finds a clearly named, genuinely new tagged speaker. Register recurring characters, keep cameos temporary, or ignore them.

During generation, gradient text temporarily renders as its first canonical stop. Prism restores the full gradient only when the response completes or is cancelled, preventing streaming repaint flicker.

## Persona behavior

The Persona tab separates two jobs that used to overlap:

- **Color persona messages** automatically colors the active persona's user-role messages. Its enabled/disabled state is remembered as the default for new chats, and existing pre-1.0.2.7 chat state seeds that preference on migration.
- **Persona is part of the cast** is chat-local and intended for impersonation / Speak for User workflows. It lets assistant-side dialogue resolve to the active persona, includes that persona in the assistant speaker registry, and exposes the persona as a manual recoloring target.

These switches are independent. Cast mode can therefore identify and color an impersonated persona in assistant output even when automatic coloring of user-role messages is disabled.

## Existing formatting

Portable `<font color>`, escaped legacy tags, BBCode colors, and explicit inline colors are protected. Prism may use a uniquely matched tag as speaker evidence and may apply a reversible local solid/gradient overlay, but ordinary rendering does not destructively rewrite saved markup.

**Bake manual corrections** is the explicit exception: when enabled in Local or Hybrid, a manual speaker correction writes the canonical speaker color into the active stored swipe. Prism recolors an exact existing font tag when possible or wraps only the corrected segment when markup was missing. Each successful bake creates a transcript recovery backup first.

Lumi theme colors applied as presentation are not treated as legacy speaker evidence.

The rendering trust order is:

1. Manual correction.
2. Confirmed tag consistent with explicit prose.
3. Explicit speaker label or reporting clause.
4. Confirmed tag without contradictory prose.
5. Tentative observed tag.
6. Paragraph/action continuity.
7. Bubble ownership and carefully seeded alternation.
8. Unresolved, teachable text.

## Hybrid closed loop

Hybrid compiles one registry snapshot for prompt injection, persona tags, DOM matching, collision detection, and hydration. Snapshots are scoped by operator and chat, correlated to a generation, capped, and expired.

After generation, Prism inspects only the requested assistant message. Delayed storage is retried; Prism never substitutes the previous assistant message. Known characters and conflicts resolve silently. Only clearly named new speakers with unused tag colors enter review; temporary speakers preserve chat continuity without joining the permanent roster.

Provisional model echoes do not count as independent evidence. Weak discoveries require two independent sightings, and provisional hints expire after inactivity.

## Transcript tools

**Normalize existing font tags** performs a dry run, changes only matching legacy tags, skips ambiguous shared colors, and stores a recovery backup. It does not bake gradients or heuristic DOM paint and does not add persona formatting.

**Historical persona colors** is a separate opt-in operation for older user messages. Both operations roll back partial failures and expose **Restore last backup**.

## Registry and diagnostics

Settings can copy privacy-safe diagnostics, export the confirmed registry as JSON, import a prior Prism registry, retry the last hydration, rescan only the current message, or reset temporary evidence without touching confirmed colors.

Diagnostics omit message text and include the Prism/schema version, engine, registry revision, speaker/collision counts, unresolved count, Cortex health, hydration state, toolbar/DOM health, and backend roundtrip time.

## Layout and accessibility

Settings are grouped into **Look & feel**, **Prompt**, and **Cortex & attribution**. Look & feel controls the roster layout, compact/large presentation, interface scale, corner style, expanded mode, and the saved/sync indicator.

On phone-sized and iOS PWA viewports, Prism opens in its own safe-area-aware portal rather than inheriting Lumiverse's transformed modal tree. The Close control remains inside the visible viewport, Settings is available in every layout, and horizontal roster gestures are isolated from the vertically scrolling editor.

## Cortex

Prism uses supported Spindle surfaces:

- Reads `{{characterColors}}`.
- Reconciles character names and aliases through Cortex entities.
- Writes portable canonical font tags in Hybrid/LLM and configured persona output.

**Sync missing** fills genuinely absent links. **Repair links** can replace generated/library/transcript guesses while preserving manual pinned decisions as visible conflicts.

## Install

Install the repository or release archive through Lumiverse’s Spindle extension manager, or copy the unpacked folder into the local extension directory. Grant the requested chat, character, persona, memory, mutation, interceptor, and generation-event permissions.

The toolbar mount uses the stable `[class*="chatToolbar"]` fragment and also registers Prism in the input-bar Extras menu.

## Development

Requirements: Node.js 22 or newer.

```sh
npm ci
npm run build
npm test
npm run verify
```

`npm run build` compiles both Spindle entrypoints to ESM. The regression harness checks attribution/scene fixtures, canonical color migration, operator-scoped snapshots, exact-message hydration, state garbage collection, prompt budgets and sanitization, provisional anti-feedback behavior, transcript dry runs/rollback, streaming paint lifecycle, and source/dist consistency.

CI rebuilds the distribution and fails if `dist/` differs from source.

## Privacy and limits

- Local mode never injects instructions or changes assistant messages.
- Hybrid/LLM prompt data is sanitized, alias/name limited, capped at 48 confirmed speakers and 12 provisional hints, and hard-limited to 8,000 characters.
- Manual quote corrections stay chat-local and expire when their source content changes.
- Temporary scene-name colors remain chat-local unless confirmed by a stable card/Cortex identity or manually pinned.
- Thought detection is opt-in because roleplay italics are gloriously ambiguous.

See [CHANGELOG.md](CHANGELOG.md) for release history.


### High-scale editor cards

At narrow widths or elevated Lumi UI scale, Prism uses horizontally swipeable Paint and Identity cards instead of relying on vertical modal scrolling.

### Dense split layout on narrow screens

When Split roster is used below 980 CSS pixels, Auto size now keeps Prism in a shorter landscape/square card and applies a layout-aware desktop density. This mirrors the cleaner zoomed-out presentation without requiring users to change Lumi's global browser zoom. Explicit Large/Expanded sizing and the horizontal accessibility workspace are unaffected. In that narrow landscape card, the cast moves into a contained horizontal carousel with color-orb chips, edge fades, scroll snapping, visible navigation controls, and a slim themed scrollbar; wide Split mode keeps the traditional side roster.


### Responsive layouts

Auto uses the horizontal cast carousel. At 1.5× effective Lumi UI scale or higher, Prism switches to the accessibility Paint/Identity card workspace. The vertical sidebar is available as a manual preference.
