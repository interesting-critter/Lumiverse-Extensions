## 1.0.5 — Semantic Boost

- Renamed Boost’s app-wide **Smart Invert** mode to **Invert brightness** so the control describes what users actually see. **Shuffle colors** is now palette-only: it changes Primary/Secondary while preserving the active Recolor/Invert brightness mode, Brightness, Contrast, Original saturation, Text treatment, and Typography instead of silently snapping an inverted/light theme back to Recolor or retuning the sliders.
- Rebuilt Full App Boost around a material graph instead of the old binary name heuristic. **Primary accent** stays faithful as the direct dominant accent and leads canvas/ordinary surfaces; **Secondary accent** stays faithful as the supporting accent and owns raised/card/hover/border material families. Lumiverse semantic surface aliases and Chat Shell `--lcs-*` glass tokens are consumed directly, while resting controls/glass remain deliberately calmer and neutral overlays keep their structural character. Achromatic Primary/Secondary picks now borrow the other chromatic accent's hue when a material needs tint instead of amplifying the meaningless OKLCH hue of gray/white/black.
- Brightness/Contrast now transform material lightness in bounded log-odds space, preserving the native deep → canvas → raised → hover ladder even under the +100 Brightness / -100 Contrast smoke test instead of collapsing the app into one pastel slab. Accent-family variants preserve native relative depth without replacing the chosen accent lightness with the source theme lightness.
- Added **Text treatment** with Auto contrast and Custom foreground modes. Auto uses proper sRGB relative luminance/contrast math against transformed surfaces; Custom recolors the native text/icon hierarchy without flattening its alpha structure.
- Kept **Protect controls** surgical: it repairs paired interactive foregrounds rather than rewriting shared prose variables. Direct accent foregrounds (`primary-contrast` / deep contrast and compatible host aliases) now stay out of accent paint entirely until that repair pass, while resting/hover glass and subtle/interactive borders have separate material roles. Transform diagnostics expose the expanded routing counts and role-tagged samples.
- Retired the unused legacy palette bootstrap path now that current Lumiverse exposes the native theme contract directly. Boost continues to transform only the canonical worker-generated variable map so live output cannot feed back into its own source.
- Hardened Palette's browser test harness for Bun 1.4 canary + Happy DOM realm gaps (`SyntaxError`, `CSS.escape`, storage/computed-style shims), refreshed stale current-source expectations, and tightened semantic picker ownership so mounted `data-component` / Spindle surfaces cannot be renamed by noisy shared CSS-module guesses.
- Imported `.lumitheme` projects now preserve Lumiverse's imported Global CSS and native component CSS as a first-class **Theme Source** layer instead of flattening everything into Custom CSS. Theme Source is protected by default, can be explicitly **Forked & edited** inside the local Palette project, and round-trips component enabled state/metadata through native export.
- Theme Source import now inspects `theme.json` directly from the `.lumitheme` ZIP before the native bridge canonicalizes it. Authored Global CSS, component CSS/TSX, enabled state, and the archive asset manifest are preserved as the protected source snapshot (including rules such as `@import`); Lumiverse still owns archive validation, asset extraction, and native installation. Existing v45 imports migrate as an explicitly labeled host-draft fallback.
- New picker-authored **native overrides now default to Strong authority**, so an explicit Palette edit wins against stubborn native/theme rules without requiring a second trip into Target details. Normal remains available as an opt-down, and existing saved Normal/Strong choices are preserved.
- Polished the **Code** workspace around imported-theme remixing: Theme Source now leads with the theme identity and one-line archive metadata, empty Global CSS/TSX collapse into compact disclosures, a lone native component opens by default, Palette-generated CSS is folded away as compiler output, empty Custom CSS stays collapsed, and diagnostic CSS overlay controls moved under **Advanced**. The source/apply architecture and schema remain unchanged.
- Fixed imported-theme authoring accidentally double-stacking source CSS over the currently active native theme. Import is now genuinely inert; **Overlay preview** is explicit and clearly described as a composite preview rather than a clean native apply. Palette also surfaces when the native import bridge has visibly canonicalized source rules such as stripped `@import`.
- Docked Palette now cancels Lumiverse's native full-UI zoom for its own tooling chrome while continuing to honor the independent `--lumiverse-font-scale`. The isolation no longer shrinks Palette's own width/height footprint by the host scale, fixing the narrow "diet Palette" column seen at 0.8× UI scale.
- Fixed the mini widget and detached floating editor inheriting Lumiverse CSS zoom through their actual portal ancestry, which could leave the visible controls several pixels away from their pointer hitboxes. Portalled Palette chrome now counter-zooms only when its real parent chain is scaled, instead of blindly using the app-wide UI-scale token.
- Follow-up: the mini widget no longer counter-zooms or implements its own fixed-position drag geometry at all. It now mounts inside Spindle's native `createFloatWidget()` surface (the same host primitive used by SpotifyControls), delegates movement/position persistence to `moveTo()` / `getPosition()` / `onDragEnd()`, and only asks the host to resize around Palette content. This keeps the rendered widget, browser hit target, and Lumiverse UI-scale coordinate system under one owner.
- Follow-up: the full floating editor now clamps and drags in rendered viewport geometry under Lumiverse UI scale. Its desktop drag path uses `getBoundingClientRect()` dimensions instead of zoom-distorted `offsetWidth` / `offsetHeight`, converts viewport pointer deltas back through the actual ancestor zoom before writing `left` / `top`, and re-clamps after host-scale changes so the editor can reach the real right and bottom viewport edges without escaping them.
- Follow-up: fixed a remaining floating-editor **ratchet drag** under nested Lumiverse CSS zoom. The editor no longer assumes the ancestor UI-scale multiplier is also its fixed-position `left` / `top` scale; each desktop drag now synchronously probes the browser's actual X/Y positional response and uses that measured conversion for clamping, so one gesture reaches any viewport edge instead of requiring repeated release/re-grab passes.
- Follow-up: reverted the mini widget itself from Spindle `createFloatWidget()` after the native float surface proved unable to satisfy Palette's mobile overlay contract across route/tab stacking contexts. The mini widget is again a persistent `document.body` portal above mobile chrome, but now reuses the detached editor's measured zoom-aware viewport drag/clamp math, preserving the hitbox and anti-ratchet fixes without surrendering its overlay layer.
- Follow-up: Lumiverse staging confirmed the main viewport drawer is an ordinary fixed `z-index: 9992` surface, while Lumiverse also scales every direct `body` child through `body > *`. The mini widget now opts its direct body root out of that host scale and promotes the same fixed surface into the browser top layer with a non-modal manual popover when supported. This makes drawer/tab stacking contexts irrelevant without reintroducing Spindle-owned drag geometry; older WebViews retain the high-z body-portal fallback.
- Hardened native Style Library dock teardown: once Palette identifies the host dock shell it observes that shell's direct parent and consumes the actual `removedNodes` record instead of inferring closure from the transient connection state of the dock content. The browser lifecycle regression now drains Happy DOM's own async task queue and switches to Code before asserting Theme Source controls, avoiding Bun/Happy DOM scheduling and hidden-workspace false negatives without weakening the browser contract.
- Added repository `.gitignore`. Persisted project schema is now **v46** and minimum Lumiverse remains **1.1.6**.

## 1.0.4 — SVG / Icon Primitive

- Added capability-driven SVG discovery in Design: if the resolved element is or contains inline `<svg>` nodes, **SVG / Icon** is highlighted and can target the SVG itself, one nested SVG, or all nested SVGs without a component-specific selector recipe.
- SVG/Icon **Replace icon** keeps the real Lumiverse/React-owned control and native SVG layout box intact while replacing only the rendered glyph through compiler-owned CSS. Tintable mode can inherit `currentColor`; Preserve colors uses the SVG snapshot as an image. Optional fixed size, fit, opacity, and rotation are persisted.
- The existing SVG packet remains available as **Paint surface**, preserving prior pseudo-surface/ornament behavior. Legacy v42 SVG Asset packets migrate to this surface mode.
- Added a first-class **Palette library** sourced from the extension's existing built-in ornament SVG corpus, alongside the existing saved SVG wardrobe and one-off/custom SVG input. Built-ins and saved assets are sanitized snapshots; clearing a replacement restores the native SVG.
- Multi-SVG targets get an explicit target picker and stable structural child paths instead of generated CSS-module hashes.
- Persisted project/state schema is now **v43**. Minimum Lumiverse remains **1.1.6**.

## 1.0.3 — Community UX Pass

- 1.0.3 QA follow-up V: docked **Apply and edit** now collapses the native Style Library instead of destroying it; reopening Browse expands the same dock with its browse/pack/filter state intact. Fullscreen handoffs still close the fullscreen library normally.

- 1.0.3 QA follow-up IV: **Apply and edit** and other Style Library → Design handoffs now release the native dock before hiding the library, preventing a blank dock from continuing to reserve half the app after the library UI exits.

- 1.0.3 QA follow-up III: native Style Library dock collapse/expand now preserves the live library root instead of treating a temporary host detach as panel destruction; collapsed docks explicitly keep their Palette title visible.

- 1.0.3 QA follow-up II: fixed mobile browse grid overlap, compacted docked pack Recipe set defaults, and made open pack control sections full-width on narrow layouts.

- 1.0.3 QA follow-up: Quick Looks gain Apply/Revert toggling, pack-card navigation is whole-card robust, mobile library tabs match the dock strip with collapsible browse controls, and Current Target recommendations resolve the selected semantic role instead of only its component family.

- Desktop Style Library can switch between its full-screen workspace and Lumiverse's native **left dock**. The dock reuses the same Palette state, stays resizable, and keeps the themed chat visible while browsing.
- Inline **Quick Looks** can browse **Current target** or any individual style pack without opening the full library. Results move in horizontal snap pages of two cards with compact pagination.
- Mobile Style Library now respects host/safe-area top chrome, keeps the Close control inside a real touch target, and stops Browse/Tune Previews from competing for one tiny row.
- Mobile pack browsing uses a roomier single-pack rhythm and vertically composed cards on narrow phones.
- The mini-widget launch button inherits its shell radius, fixing the clipped square corner.
- Added the `ui_panels` permission required by the public native dock API.
- No theme recipe, selector, renderer-style, or persisted project/schema changes. Schema remains **v42** and minimum Lumiverse remains **1.1.6**.
- Includes the preceding **1.0.2 Message Scope Correctness** fix; 1.0.3 itself is intentionally UX-only.

## 1.0.2 — Resolver & Primitive Taxonomy Hotfix

- Fixed BubbleMessage user-side part selection incorrectly rewriting an outer CSS-module local such as `_content_1hvlc_*` to a nested `_contentUser_16x4f_*` just because the two locals shared the word `content`. Assistant/User variant pairing now requires mounted evidence that the locals belong to the **same CSS-module hash family**.
- Added the full mounted message-side grammar `foo / fooChar / fooUser`. Shared parts such as Name now resolve Assistant → `nameChar`, User → `nameUser`, and Both → exactly those two branches instead of carrying the assistant leaf onto the user side. The paired `*Char` / `*User` leaves are folded back into the semantic base part instead of appearing as duplicate anatomy entries.
- Disambiguated repeated normalized locals inside one component by preserving a nearby same-module structural anchor when needed. BubbleMessage's outer content wrapper now resolves through the Bubble → Content path instead of a broad `[class*="_content_"]` selector that can also catch nested MessageContent.
- Regressed **Assistant / User / Both** together for outer Content, nested MessageContent, shared+Char+User Name families, and picks originating from either speaker.
- Fixed a downstream Both-scope editor leak where Target Details showed the correct combined selector but the Style Stack could borrow an Assistant-only packet via the originally clicked representative DOM node. Both now treats authored matches as common only when they cover every mounted branch; matched message edits localize to the active facet rather than mutating a one-sided source override.
- Both selection guides and forced state previews now cover every mounted speaker branch instead of only the node that happened to be picked first.
- Split the overloaded **Image** packet into two first-class jobs. Image now always exposes source quality, tone/filter, crop/fit, focal position, and frame controls; **Mask** owns Native / None / Fade / Custom masking independently of whether the selected semantic target is an actual `<img>` leaf or a wrapper.
- Schema **v42** automatically separates legacy combined Image+mask packets into sibling Image and Mask packets on the same target/state. Legacy mask-only wrapper packets promote directly to Mask instead of leaving a meaningless default Image card behind. Sparse edited-field ownership and recipe-slot provenance are split with the packet so preset reset/edit behavior survives migration. Built-in packs were normalized to the same vocabulary.
- **Read Style** now rehydrates authored media treatment and authored CSS masks as separate Image and Mask packets. Image can still inspect semantic wrappers containing media for filter/source-quality intent, while fit/crop remains a media-leaf concern.
- Renamed the user-facing **Text Style** primitive to **Ink** and moved it into Paint beside SVG Asset. The persisted packet type remains `text`, so existing projects keep their color/gradient/stroke/glow data unchanged; Typography remains responsible for font structure.
- Persisted project/state schema is now **v42** because Mask is a new first-class packet type. Public extension version remains **1.0.2** and minimum Lumiverse remains **1.1.6**.

## 1.0.1 — HTTP Browser UUID Hotfix

- Fixed Palette failing during frontend setup on plain HTTP browser sessions where `crypto.randomUUID()` is unavailable, including LAN/Tailscale-hosted Lumiverse instances that work normally when installed as a secure-context PWA.
- Added a portable UUID helper: native `randomUUID()` when available, `crypto.getRandomValues()` UUID-v4 generation otherwise, plus a last-ditch compatibility fallback for runtimes without Web Crypto. Project IDs, migration-generated IDs, persistence request IDs, and theme-runtime request IDs now all use the same helper.
- Added regression coverage that boots Palette state with a `crypto` object that deliberately has no `randomUUID`, matching the release-day mobile failure reported through Eruda.
- No theme, renderer, editor-layout, or persisted schema changes. Schema remains **v41** and minimum Lumiverse remains **1.1.6**.

## 1.0.0 — Public Release

- First public release of **Palette**, the visual, component-aware theme editor for Lumiverse.
- Ships the stabilized semantic picker/scope model, visual style packets, Base + Mobile authoring, state styling, Smart Guides, Layout Groups, reusable My Styles, theme projects, native theme handoff, and the mobile floating workbench.
- Includes the fully mounted-QA pass for the built-in **Manga, Editorial, Visual Novel, and Journal** directions across BubbleMessage and MinimalMessage, including responsive renderer-specific anatomy, composer treatments, reasoning, actions, greetings, and swipe controls.
- Mobile Palette includes safe scroll gutters, compact inspector controls, separated workspace/window actions, explicit Minimize behavior, and the contained **100% → 80% → 60% → 100%** inspector-density cycle.
- Persisted Palette project/state schema is **v41**. Minimum supported Lumiverse version remains **1.1.6**.

## v28.53 — Mobile Density Containment

- Fixed the new Palette mobile density cycle after mounted QA showed the 80%/60% inspector expanding its layout width before zoom and getting clipped by the scroll viewport. Density no longer uses inverse 125%/166.67% widths.
- The inspector now stays at a hard 100%/max-100% layout width and only applies the requested CSS zoom. The scroll body centers the scaled inspector, so 80% and 60% remain fully inside the safe-gutter viewport instead of sacrificing the right-side value controls.
- Persistent header/workbar controls, the 100→80→60 cycle, scroll-safe gutters, theme isolation, Journal layout, persisted vocabulary, and schema remain unchanged at **v41**.

## v28.52 — Mobile Workbench

- Finished the Journal Minimal phone composition by centering both scrapbook portraits in normal flow and centering the shared mobile identity header. Desktop keeps the left/right collage asymmetry; mobile becomes an intentional stacked journal entry instead of zig-zagging across a narrow viewport.
- Rebuilt the floating Palette header for phones as two visibly separate control families: Design/Code/Themes live in a segmented workspace island, while edge toss, density, Minimize, and Close live in a window-action group. The old curved Return/Undo-looking glyph is replaced by an explicit minimize control.
- Added a persisted **100% → 80% → 60% → 100%** inspector-density cycle. Only the scrolling inspector body scales; the workbar and floating header remain full-size touch targets.
- Added permanent mobile scroll-safe side gutters so a thumb can always pan vertically without landing on a range input, plus tighter mobile field rhythm, bounded select typography, and a compact Smart control so themed fonts/scales cannot turn Palette itself into buttonmageddon.
- No persisted theme vocabulary changed; project/state schema remains **v41**.

## v28.51 — Journal Mobile Flow Portrait

- Replaced the Journal Minimal user portrait’s mobile `sticky` workaround with a true responsive **Flow reset**. The portrait now remains in normal document flow, occupies its own block row, and still Quick Aligns to the right via logical auto margins—without following the viewport during scroll.
- This leans on the existing mobile `display: block` message frame: assistant portrait stays naturally left in flow, user portrait gets the same row ownership with right-side placement. No desktop Journal geometry changes.
- Bubble Journal, VN, Editorial, Manga, persisted vocabulary, and schema remain unchanged at **v41**.

## v28.50 — Journal Bubble Mounted Parity

- Folded the final mounted BubbleMessage Journal corrections back into the pack instead of leaving them as one-off authored CSS: assistant and user action pills now keep separate desktop offsets and separate mobile layouts/positioning.
- Tightened Journal Bubble mobile identity geometry with renderer-side header rules: assistant header/meta/header-left use explicit container-query boundaries, while user header/header-left get their own responsive sizing instead of inheriting the assistant arrangement.
- Finished assistant Bubble swipe behavior on mobile: the pager stays a single-row end-aligned control inside a full-width responsive lane, with the existing sage stationery ink/chrome retained.
- Minimal Journal remains untouched in this pass. Persisted vocabulary is unchanged; schema remains **v41**.

## v28.49 — Journal Mounted Parity

- Folded the final mounted Journal Minimal desktop/mobile edits back into the pack instead of approximating them from screenshots. Both Minimal cards explicitly collapse to block flow on phones, and the assistant frame clears inherited translation before the mobile stack is laid out.
- Finished the side-aware scrapbook identity: the user polaroid now uses the approved 85×106 sage frame/shadow on desktop, becomes a compact right-aligned sticky portrait on mobile, and the assistant/user washi pieces keep separate desktop offsets plus the tested `top:-70px` phone offsets beside their respective portraits.
- Reworked Minimal paper actions to match the mounted DOM: the outer action docks own the one-column paper stack, the inner native action rows use implicit grid rows on desktop, and mobile switches to centered horizontal controls while the docks settle to the bottom edge. User buttons use the stronger sage outline/shadow from the mounted edit; both sides shrink to 20px controls on phones.
- Added the softened sage Minimal Greetings count treatment from mounted QA. Bubble Journal geometry, Visual Novel, Editorial, persisted vocabulary, and schema remain unchanged at **v41**.

## v28.48 — Journal Action Wrapper Fix

- Corrected the Journal Minimal action layout after mounted QA showed the one-column grid was authored on the outer dock while Lumiverse's inner `_actions_` row kept the buttons horizontal. The dock now owns positioning only; the actual assistant/user action rows own the one-column grid, 8px spacing, and start alignment.
- Kept the existing phone fallback as a wrapped row so the compact mobile card does not grow a seven-button tower.
- Added the mounted Journal Minimal thinking correction as semantic Quick Align: the reasoning container now centers itself with fit-content/auto-margin behavior instead of relying only on flex-item alignment.
- No Bubble Journal geometry, swipe colors, composer inset, or persisted vocabulary changed. Schema remains **v41**.

## v28.47 — Journal Margin Polish

- Finished the Journal Minimal action furniture: desktop assistant/user docks now use the same single-column paper-button grid language as Bubble actions, while mobile keeps a compact wrapped row so the column does not eat the phone.
- Darkened Journal swipe arrows and the page counter with the pack's recurring sage ink (`#46504e`) at the actual button/counter subroles, so native child colors can no longer wash the pager text out.
- Added a real writing inset to the Journal composer textarea and slightly increased Manga's existing inset. This fixes the corner-hugging placeholder while keeping typed text on the same baseline instead of relying on unsupported `::placeholder` padding.
- Journal card geometry, Visual Novel, Editorial, persisted vocabulary, and schema remain unchanged at **v41**.

## v28.46 — Journal Side-Aware Stationery

- Split the Journal Minimal header tape into **assistant** and **user** pseudo-element roles so the washi follows the portrait side instead of using one `::after` position for both mirrored layouts. Assistant tape stays left; user tape mirrors to the right, with matching mobile positions.
- Finished Journal Minimal action chrome by giving both assistant and user action buttons the same mint-paper button treatment as BubbleMessage actions while preserving their existing side-specific docks.
- Re-inked the Journal swipe pager with the same mint stationery family as the action buttons, adding a soft paper border, rounded pager shell, and readable dark ink.
- Journal message geometry, Bubble Journal, VN, Editorial, persisted vocabulary, and schema remain unchanged at **v41**.

## v28.45 — Visual Novel Mobile Center Lock

- Applied the final mounted Minimal VN phone correction: the **Inner Voice** reasoning container now uses Palette Quick Align center on mobile (`fit-content` + logical auto margins) instead of carrying the previous percentage-width override. This matches the successful mounted CSS exactly while leaving desktop reasoning geometry untouched.
- No other Visual Novel, Editorial, Bubble, project-schema, or persisted-vocabulary behavior changed. Schema remains **v41**.

## v28.44 — Visual Novel Mobile Lane

- Finished the last mobile pass for **Visual Novel Minimal** on both assistant and player stages. The portrait now hard-anchors into the upper-left on phones, while the transparent header becomes a real two-line identity lane to its right instead of keeping the desktop centered banner geometry.
- Released the compact mobile metadata pill from its frame-level absolute anchor. On mobile it now sits in normal header flow directly beneath the speaker name, fixing the ‘meta in the middle of the response’ drift without changing the desktop placement.
- Centered the remaining mobile system furniture so the **Inner Voice** plate stays the focal bridge between header and dialogue, while the route pager and action buttons read as deliberate centered footer controls instead of side-clinging leftovers.
- Desktop Minimal VN geometry, Bubble VN, Editorial, persisted vocabulary, and schema all remain unchanged at **v41**.

## v28.43 — Visual Novel Mirrored Stage

- Folded the mounted assistant fixes back into the canonical Minimal VN stage: the transparent 76% header is explicitly centered, desktop metadata stays in normal header flow instead of anchoring into long dialogue, while the compact mobile meta keeps its deliberate portrait-adjacent anchor.
- Released the route pager from absolute positioning and moved it into normal-flow **bottom-left** VN furniture. The custom SVG arrows remain intact; the flower ornament stays attached to the pager itself.
- Transposed the approved assistant dialogue-stage geometry onto the player side instead of maintaining a second, shrunken card grammar. Player Minimal now uses the same 92% / 1040px stage, 255×180 centered landscape portrait, centered transparent header, flowing name/meta lane, fading banner, and 86% / 880px double-line dialogue frame, recolored into the rose route family.
- Transposed the desktop text-command stack onto user actions as well; mobile still releases both action docks back into compact wrapped flow. Bubble VN and Editorial remain untouched. Persisted project vocabulary is unchanged; schema remains **v41**.

## v28.42 — Visual Novel Stage Tooling

- Fixed **Use generic** for dynamic `aria-label` / `title` targets. The suggested reusable scope now switches directly to the exact candidate instead of re-running message-family preservation and snapping straight back to the label-specific button. This matters especially for action rows such as Copy/Edit where the local selector is intentionally dynamic.
- Made Style Library recipe cards identify their real semantic footprint even when a thumbnail is present: cards now show **Component** and **Affects** summaries, so reset/edit work no longer requires reverse-engineering a vague preview.
- Made recoloring controls independent of Recents. Every packet color field now exposes an explicit native **Pick** swatch beside the text value (including gradient stops, stencil/tint/outline/shadow/glass controls), and Quick Recipe / Boost anchors use the same treatment.
- Rebuilt independent **Corners** controls as a spatial 2×2 map (`top-left / top-right / bottom-left / bottom-right`) with a center preview instead of listing bottom-right in the visual bottom-left slot. No packet/schema change was required.
- Hardened SwipeControls anatomy against current Lumiverse markup: pager button roles now target direct `SwipeControls > button` children instead of requiring the old `_btn_` CSS-module fragment. Previous/next roles use the direct first/last buttons, restoring custom VN arrow masks when the native class name disappears.
- Folded the safe parts of the mounted Visual Novel Minimal authoring experiment back into the canonical assistant stage: scanline frame texture, 255×180 landscape portrait, Trebuchet gradient name plate with asymmetric corners, stronger meta plate, double-line dialogue frame, stacked VN text-action plates, and a long narrow Greetings status bar. One-message coordinate hacks were deliberately not copied. Player-side work remains conservative because the mounted user experiment was unfinished.
- Editorial and Bubble VN remain untouched. Persisted project vocabulary is unchanged; schema remains **v41**.

## v28.41 — Visual Novel Dialogue Stage

- Re-authored **Visual Novel Minimal** around a classic VN dialogue-screen composition instead of continuing the backlog/editorial column language. Bubble VN remains the active cinematic scene; Minimal now uses a centered character portrait, fading scene banner, floating name plate, translucent patterned dialogue frame, and transparent system furniture.
- Turned the Minimal header into transparent staging space: speaker names are VN plates, metadata floats near the portrait, Greetings floats beside the character art, and subtle `::after` embellishments provide scene decoration without adding another card. The player side keeps the approved narrower/right-weighted asymmetry while adopting the same VN grammar.
- Rebuilt Minimal reasoning as an **Inner Voice plate** that sits beneath the name and overlaps the dialogue frame. The body continues inside the same translucent patterned surface rather than opening a generic reasoning lane.
- Converted Minimal message actions to generated **text controls** (`EDIT`, `COPY`, `HIDE`, `ANCHOR`, `FORK`, `PROMPT`, `DELETE`) while preserving the real native buttons as hit targets. `Copy` now has its own static Minimal action role and is protected from the omitted-action fallback.
- Reworked Minimal swipe furniture into a compact ornamental pager at the bottom-right of the dialogue stage. Previous/next controls replace native chevrons with built-in arrow SVG masks, the counter uses VN UI typography, and a small flower ornament marks the route control. Minimal long-message continuation also receives renderer-scoped VN chrome instead of inheriting the shared serif button.
- Removed `minimal-native-strip-off` from Visual Novel Apply All because Minimal VN now intentionally authors the message `::before` surface as its fading character banner. All new roles are static DOM anatomy; persisted project vocabulary is unchanged and schema remains **v41**.

## v28.40 — Visual Novel Backlog Signal

- Kept the v28.39 Minimal ADV backlog geometry and asymmetry intact, but removed the accidental **purple Editorial** styling language. Minimal VN now has its own backlog signal system: cool cyan route ink for assistant/history, dusty rose response ink for the player, neutral charcoal surfaces, and flatter square-ish portrait framing.
- Added a Minimal-only **ADV backlog type** recipe so shared Bubble VN can keep its cinematic Georgia hierarchy while MinimalMessage overrides paragraphs, headings, emphasis, blockquotes, and code with compact game-history sans/mono typography. This is renderer-scoped anatomy, not a cross-renderer prose rewrite.
- Re-inked Minimal speaker names, metadata, route rules, inner voice, greetings, swipes, and utility controls away from lavender-on-black into the cyan/rose route palette. Assistant dialogue gets only a faint cyan history wash; player entries keep the right-weighted response shape with a restrained rose signal.
- Added static `minimal.prose.*` semantic roles for scoped backlog typography and registered `visual-novel-minimal-prose` as Minimal-only. No persisted project vocabulary changed; schema remains **v41**.

## v28.39 — Visual Novel ADV Backlog

- Re-authored **Visual Novel Minimal** around its actual renderer thesis instead of shrinking Bubble VN into another purple card stack. Bubble remains the active cinematic scene; Minimal now reads as an **ADV backlog / route transcript**.
- Assistant Minimal messages are now mostly open canvas: compact 66×86 portrait rail, flat side-scoped byline, unboxed metadata, normalized content mount, a single slim route rule, and a centered 92% / 980px transcript measure. Mobile releases the sticky portrait and full-width content chain cleanly.
- Player Minimal messages become compact right-weighted backlog entries rather than a second choice window: 78% / 820px faint response wash, plain byline, 44×52 portrait, normalized content lane, and a quiet rose route rule.
- Rebuilt Minimal inner voice as inline narration instead of a nested reasoning card, and flattened greetings, swipe buttons/counter, and action controls into low-contrast backlog furniture. Actions stay pinned to the message footer rather than competing with the byline.
- Added static Minimal semantic anatomy for side-specific content rules plus swipe buttons/counter. No persisted authoring vocabulary changed, so schema remains **v41**.

## v28.38 — Editorial Marginalia

- Rebuilt Editorial reasoning in both **BubbleMessage and MinimalMessage** after mounted QA exposed the native outlined `Thought for …` lane as the last piece of generic system chrome inside the publication language.
- The reasoning shell is now transparent and borderless, constrained to a reading-column width on desktop, and released to full width on phones. A cool-slate gradient edge creates a quiet marginal rule without adding another card or pill.
- The live `Thought for …` toggle now uses compact Georgia italic typography and the existing Editorial thought mark; the open reasoning body continues the same faint rule/wash with restrained grain instead of becoming a second boxed panel.
- Kept the toggle subrole ink-only because current Lumiverse mounts the toggle button as the header itself; repainting chrome from both roles would make the later selector erase the marginalia gradient. No new persisted vocabulary was introduced, so schema remains **v41**.

## v28.37 — Editorial Mobile Folio

- Mounted phone QA exposed the remaining assistant footer conflict: `minimal-actions-overlay` was applied after Editorial's mobile release packet, so the generic absolute action dock won the cascade and climbed back into the byline. Editorial Apply All now applies the generic overlay first and the Author Rail second, making the mobile `position: static` reset the final pack-owned word while desktop keeps the same top-right rail geometry.
- Finished **Compact swipe pager** instead of styling only its outer shell. Swipe buttons and counter now have semantic subroles; native boxed controls are flattened into bare cool-slate chevrons plus a tiny monospaced folio count, with a tighter phone footprint.
- Tightened the assistant mobile proof row so swipes and actions read as one quiet footer sequence rather than two unrelated control islands. User correspondence and desktop chat remain untouched.
- Added only static DOM anatomy (`message.swipes.buttons` / `message.swipes.counter`); schema remains **v41**.

## v28.36 — Editorial Mobile Byline

- Mounted mobile QA was already almost clean, so this pass is deliberately mobile-only: desktop Editorial chat stays frozen.
- Reader Correspondence now anchors the 44px contributor portrait at the top-left of the mobile card and reserves that exact space inside the 44px byline row, keeping **name + metadata beside the portrait** instead of stacking beneath it.
- Editorial assistant Minimal now collapses the native mobile row to block flow, keeps its 48×66 portrait pinned beside the byline, normalizes the assistant bubble to the full content width, and releases the assistant actions into a normal bottom proof row like the user controls.
- No new persisted authoring vocabulary was introduced; schema remains **v41**.

## v28.35 — Editorial Masthead Kicker

- Left chat/message styling untouched after the desktop Editorial chat pass was approved.
- Fixed the Contributors masthead `::before` kicker by explicitly releasing native pseudo positioning back into flex flow, zeroing inherited pseudo padding, centering it as a fixed 82×18 rail item, and preventing it from shrinking into the first contributor portrait.
- Kept the roster label hidden on mobile pending the dedicated mobile audit. Schema remains v41.

## v28.34 — Editorial Flex Exorcism

- Mounted DevTools QA identified the final Reader Correspondence dead lane: the purple hatched region was Chromium visualizing **native MinimalMessage flex free-space**, not mysterious padding. Editorial had already anchored the user avatar/actions out of normal flow, but the host flex composition was still distributing a vacant rail.
- Reader Correspondence now explicitly collapses its user frame to ordinary **block flow** and normalizes the user bubble to `width/max-width: 100%` with `min-width: 0`. The custom 106px proof rail remains real frame padding; the bubble now receives the entire remaining content box instead of inheriting a ghost native lane.
- This is pack geometry only. No new persisted authoring vocabulary; schema remains **v41**.

## v28.33 — Editorial Left Rail Ink

- Corrected the Reader Correspondence portrait after mounted QA proved native Minimal keeps the user avatar in the **right** rail. Editorial now deliberately reserves a compact left proof rail on the message frame, anchors the contributor portrait into it, and centers the proofmark dock beneath the same axis.
- Kept the user header itself at zero padding / 40px flex-row geometry. The rail reservation now belongs to the frame instead of being faked with giant header padding, so identity/content alignment remains readable and editable.
- Deepened the quill/send ink from pale slate to a darker fountain blue (`#355d69`) and nudged the proofmark family toward the same adjacent blue-gray so the postage silhouette stays soft while the writing mark is actually legible.
- Mobile still releases the desktop rail back into flow. No new persisted authoring vocabulary was needed, so schema remains **v41**.

## v28.32 — Editorial Rail & Quill

- Added **Advanced padding** and **Advanced margin** beneath Spacing's normal linked sliders. Top/right/bottom/left can now be edited independently without making basic box-model work noisy; touching the main slider links all four sides again.
- Fixed Spacing compilation so intentionally negative margins survive authoring instead of being silently clamped back to zero. Existing BoxSpacing already stored independent sides, so schema remains **v41**.
- Rebuilt Editorial user identity around the mounted Minimal rail: the user header drops its giant asymmetric padding, becomes a 40px flex row, and lets the native left rail do the visual separation instead of faking it with empty header acreage.
- Re-centered the user proofmark dock beneath the contributor portrait and moved its ink from oxblood to the pack's muted blue-slate family.
- Replaced Editorial Send's red native arrow with a built-in **quill mask** while keeping the scalloped postage shell and native click target intact. The quill and inner stamp rule now use cool fountain-ink slate instead of the accent red that was fighting the paper palette.
- Updated Editorial previews and stale recipe tests to match the shipped v28.31/v28.32 geometry.

## v28.31 — Editorial Proof Desk

- Flattened **Greetings** in both renderers into quiet publication annotations: transparent chrome, tiny cool-slate iconography, uppercase index text, and an unboxed native count instead of the stray cream pill.
- Rebuilt **Reader correspondence** around the tighter mounted experiment: 74% / 780px finished-paper width, reduced header/content dead space, a 70px contributor portrait, and a narrower literary body measure.
- Moved Minimal user actions into the correspondence margin as small oxblood **proof marks**. Desktop uses a compact two-row editorial mark lane under the portrait; mobile releases the dock back into normal flow.
- Removed the composer’s inner **isolation cell**. The writing field is now a barely-tinted grain plane with no control border while Text Entry continues to own the real textarea inset and autosize-mirror metrics.
- Rebuilt Send as an actual **postage silhouette** using a built-in scalloped `postage-edge` mask. The shell owns the perforated paper edge, the real button becomes a printed inner frame, and the send glyph is larger without changing the native hit target.
- Flattened the group member UI into a **Contributors masthead**: unboxed author entries, smaller publication portraits, a generated `CONTRIBUTORS` rail label, an inset active underline, and a dashed add-member action left as the one obvious control.
- Added semantic subroles for greetings icon/count and the roster publication label. These are static anatomy additions; schema remains **v41**.

## v28.30 — Editorial Writing Desk

- Added **Text Entry**, a persisted normal-user primitive for composer text inset, typography metrics, and placeholder appearance. Text inset moves placeholder and entered text together; metric declarations are mirrored onto Lumiverse's hidden textarea autosize mirror so the field can still measure itself honestly. Project/state schema advances to **v41**.
- Added authoritative InputArea semantic roles for the real `textarea[name="chat-message"]`, its hidden textarea mirror, send shell, exact send button, and send icon. The send selector no longer confuses `_sendBtnShell_` with `_sendBtn_` when their CSS-module prefixes overlap.
- Re-authored **Editorial → Writing desk** as warm paper instead of a white slab: restrained grain, a proper writing field, Georgia Text Entry metrics, and a square dotted oxblood postage-stamp send treatment that preserves the native button hit target.
- Tightened **Reader correspondence** to a centered 82% / 820px paper card, cooled and darkened the stock, enlarged the contributor portrait to 82px, and enlarged the assistant author portrait to 94×118 so both identities stop reading like micro-icons beside long-form prose.
- Finished neglected Editorial furniture: Bubble/Minimal greetings now read as printed index tabs, group-member UI becomes a publication contributors strip, and the native scroll-to-bottom control becomes a restrained page marker. The furniture recipes are renderer-independent and ship through fresh-project Apply All.
- Updated the Composer workshop so the send shell is a first-class physical surface and Text Entry/placeholder styling previews the same ownership model Palette compiles.

## v28.29 — Editorial Minimal correspondence desk

- Re-authored **Editorial → Reader correspondence** so Minimal user messages finally answer the pack’s bright writing-desk composer instead of reading like a dimmed assistant variant.
- User correspondence now uses warm cream paper, subtle grain and lift, a circular contributor portrait with a paper ring, dark Georgia byline, muted publication metadata, and dark authored ink across prose descendants.
- Added the renderer-local `minimal.content.user.ink` semantic role so a light user correspondence surface can safely override Editorial’s shared dark-surface prose colors without contaminating assistant messages or BubbleMessage.
- The user header now reserves a real contributor identity lane beside the circular portrait; actions stay quiet/dark on the light card, and mobile keeps the same correspondence thesis with a compact flow portrait.
- Apply All now materializes the user correspondence recipe after shared Editorial prose so renderer-specific correspondence ink wins deterministically on a fresh project.
- Added a dedicated light correspondence preview in the Style Library. Schema remains **v40**.

## v28.28 — Manga Minimal mobile portrait lock

- Kept the newly-stable mobile reading lanes and bottom action strip intact; this pass only fixes portrait/identity geometry.
- Mobile assistant/user portraits now anchor at `top: 8px` instead of `38px`, aligning the compact 48×58 token with the name/meta identity row instead of dropping into the thinking/content lane.
- The Manga Temper mark follows the portrait upward (`top: 12px`) so its corner overlap remains intentional on both mirrored sides.
- Desktop 168×390 sticky portrait rails, Quick Align, content-lane resets, and action-row geometry are unchanged. Schema remains **v40**.

## v28.27 — Manga Minimal native content-lane amnesty

- Mounted mobile QA identified the real remaining collapse owner: Lumiverse wraps MinimalMessage prose in an intermediate native `_content_` / `_contentChar_`-style lane above `MessageContent`; that wrapper was still 14px wide, so the now-full-width frame simply inherited a 183,286px-tall child.
- Added side-aware `minimal.content.mount.assistant` / `minimal.content.mount.user` roles targeting the renderer-local content wrapper structurally via `:has([data-component="MessageContent"])`, avoiding reliance on one hashed CSS-module suffix.
- Manga Minimal mobile now resets that intermediate lane to block flow, 100% parent width, `min-width: 0`, `max-width: 100%`, and fit-content height before styling `MessageContent` itself.
- Desktop sticky rails and the actions → name → meta choreography remain untouched. Schema remains **v40**.

## v28.26 — Manga Minimal mobile grid exorcism

- Fixed the actual cause of the 14px-wide / six-mile-tall Manga Minimal mobile prose column: the MinimalMessage root was still participating in its native grid after the portrait had been pulled out into an anchored mobile token.
- Assistant and user MinimalMessage frames now explicitly reset to block flow on ≤720px, making any native grid-column placement on the inner bubble inert instead of collapsing the reading lane.
- Mobile frame, bubble, and MessageContent width contracts now also own `min-width: 0` alongside full-parent width/max-width so intrinsic/min-content sizing cannot resurrect the postage-stamp column.
- The intended phone choreography stays **actions → name → meta → content**; desktop sticky Manga rails are untouched. Schema remains **v40**.

## v28.25 — Manga Minimal mobile amnesty

- Re-authored Manga Minimal mobile as a compact reader instead of letting desktop portrait-rail geometry collapse into a narrow newspaper column.
- Minimal assistant/user bubbles explicitly switch to full-width block flow on ≤720px; MessageContent and the Manga thinking panel also claim the full available width.
- Mobile action chrome is now the first deliberate lane: full-width, single-row, nowrap, `space-between`, with a 4px minimum gap instead of wrapping into `PROMPTDELETE` / orphaned Delete rows.
- Speaker identity follows beneath the actions as a column — **actions → name → meta** — with full-width headers and compact 48×58 portrait tokens anchored beside the identity rather than consuming the prose lane.
- Desktop sticky rails remain untouched; mobile portraits are explicitly non-sticky and use anchored compact tokens.
- The Minimal Temper mark shrinks to a 20px portrait-corner accent on mobile so it no longer fights the action strip. Schema remains **v40**.

## v28.24 — Quick Align primitive

- Added **Quick Align**, a first-class placement packet for normal humans: Left / Center / Right / Fill and Top / Center / Bottom / Fill express visual intent without requiring users to know whether the current parent is block, Flex, or Grid.
- Horizontal Quick Align compiles through logical auto margins + `fit-content` with a harmless Grid alignment hint, so ordinary block-flow elements can finally move without inert `align-self` roulette. Explicit Size packets still win when the user intentionally owns width.
- Vertical Quick Align combines logical block margins with `align-self`; Palette warns when the mounted parent is ordinary block flow and therefore may not provide free vertical space to distribute.
- Layout Item remains the advanced Flex/Grid child-control packet and now points ordinary left/center/right users toward Quick Align instead of merely scolding them.
- Manga Minimal now dogfoods the primitive: assistant/user identity headers use semantic Quick Align start/end placement instead of inert Layout Item alignment against a block parent; Mobile returns both to start/full-flow behavior.
- This adds persisted placement intent, so project/state schema advances to **v40**. Existing v39 projects normalize forward without changing their authored CSS.

## v28.23 — Manga identity lock + prompt action + Browse Inside labels

- Manga Minimal now keeps its identity card physically adjacent to the portrait rail: assistant header hugs the left rail, user header hugs the mirrored right rail, while name/meta remain left-aligned inside that compact header. Mobile returns the header to full-width flow.
- Minimal Manga restores the assistant Prompt Breakdown control to the text action strip as `PROMPT`; the omission whitelist no longer hides prompt/breakdown actions.
- Browse Inside once again prioritizes `aria-label` / `title` / `name` on interactive controls before generic CSS-module class names, so action buttons show useful labels instead of seven rows of `Btn`.
- Schema remains v39.

## v28.22 — Manga Minimal header lanes

- Manga Minimal now gives Assistant and User their own header roles instead of relying on one generic Minimal header rule.
- Both headers reserve a dedicated top action rail, then left-align name + metadata beneath it; the user side mirrors the Temper-safe inset from the left.
- Header width is explicitly claimed at 100% so native compact-user sizing cannot re-center the identity lane.
- Action docks keep the v28.21 outer-dock / inner-row split; this pass only fixes their relationship to identity chrome. Schema remains v39.

## v28.21 — Manga action-row geometry

- Manga Minimal now separates the outer actions dock from Lumiverse's inner `_actions_` row: the dock owns placement/width while the real inner row owns `space-between` distribution.
- Removed the unstable left+right+translate positioning combination from Manga Minimal actions. Assistant and user docks now mirror from the Temper side with a bounded 40% / 420px lane.
- Added side-aware `minimal.actions.assistant.row` and `minimal.actions.user.row` KnownPart roles so future packs can style the real control row without pixel-hunting nested wrappers.
- Mobile resets both the dock width and inner-row layout back to normal flow/wrap. Schema remains v39.

## v28.20 — Minimal side-local edits + Manga rail lock

- Side-specific MinimalMessage picks no longer mutate a broader matching pack override when edited in Design; Palette materializes only the changed fields onto the selected Assistant/User selector.
- Minimal speaker-name registry roles are now explicitly Assistant/User rooted, with side-aware Minimal meta-pill roles added for pack authoring.
- Manga Minimal now authors assistant/user metadata separately and adopts the mounted action-strip coordinates from the latest QA pass.
- Manga action docks keep their mobile flow reset and mirrored desktop geometry. Schema remains v39.

# Changelog

## Semantic Boost follow-up

- Restored Preview compilation's default Boost inclusion while keeping Design scrubs explicitly Boost-free.
- Prefer stable ComposerActionBarLive semantic wrappers over localized title/ARIA selector fallbacks.
- Hardened Bun-canary Happy DOM compatibility tests and aligned Manga mobile portrait-rail expectations with the authored compact layout.

Public release history for Palette. The long internal v27/v28 development log is preserved in [`docs/development/INTERNAL_CHANGELOG-v27-v28.md`](docs/development/INTERNAL_CHANGELOG-v27-v28.md).

## Unreleased — release-candidate hardening

- Shortened Manga Minimal action chrome to the intentional five-control strip — **EDIT / HIDE / ANCHOR / FORK / DELETE** — with all other native actions omitted from this skin, space-between distribution, and Temper-red Delete ink.
- Generated Content authored on a normal element now compiles to that element's `::after` skin automatically; explicit pseudo-surface targets remain unchanged.
- Left-aligned Manga Minimal speaker identity/meta treatment so the mirrored user header reads like the assistant side instead of native compact-user chrome.
- Art-directed Manga Minimal as a true mirrored two-sided reader: user portrait now matches the assistant sticky rail on desktop, collapses back to normal flow on mobile, and uses matching speaker-name hierarchy.
- Mirrored the Minimal Manga Temper ornament by message side and gave the text action rail wider real-button spacing with white ink plus a crisp black outside outline.
- Removed the transitional ‘first real adapters’ helper and corrected Minimal Quick Look recommendations so they no longer surface Bubble-only Manga recipes.
- Enforced renderer-specific pack families: Bubble choreography stays Bubble-only, Minimal choreography stays Minimal-only, while genuinely renderer-independent prose/media/control targets can remain shared.
- Added side-aware Manga Minimal action-button/icon/label targets and a text-only printed action strip that skins the real native buttons rather than trying to group pseudo-elements.
- Added **Text Style → Outline → Edge / Outside**. Outside outline generates a crisp multi-shadow ring so thick outlines do not consume the glyph fill; ordinary text shadow remains independently editable.
- Generated Content can now mirror `title` or `aria-label` with CSS `attr(...)`, in addition to literal labels.
- Removed the stray zero-byte `src/ui/styles.ts.tmp` development file.

## 1.0.0 — Initial public release

Palette 1.0 packages the internal v28.16 / schema v39 build line as the first public release.

### Authoring

- Persistent picker with semantic owner/context, target ladder, **Edit Part**, **Browse Inside**, target surfaces, and message-side facets.
- Semantic Base/Mobile and state-aware styling packets for paint, typography, shape, layout, imagery, media, positioning, transforms, visibility, pseudo surfaces, generated content, and reusable SVG assets.
- Smart geometry Guides for spacing, dimensions, layout, positioning, images, and Layout Groups.
- Layout Groups arrange real sibling DOM through their shared parent without React reparenting or synthetic wrappers.

### Inspection and reuse

- **Read Style** and **Read Page** inspect mounted presentation lazily; observed values do not become authored CSS until edited.
- **Quick Looks**, pack recipes, and reversible recipe provenance compile through the same ordinary Palette styling model.
- **My Styles** saves reusable Targets, Components, and multi-component Bundles across theme projects.
- Theme Stash provides persistent multi-project management with swatch cards, duplicate, rename, delete, and active-state handling.

### App-wide theming

- Full App **Boost** transforms Lumiverse's canonical native theme variable map with continuity across commits.
- Backdrop treatment lives inside Boost; global **Typography** remains independently toggleable.
- Native Theme Assets, font registration, `.lumitheme` handoff, generated CSS inspection, and separate Custom CSS are available from the normal workflows.

### Packs and media

- Curated Manga, Editorial, Journal, and Visual Novel composition systems.
- **Media Flow** supports both raw Markdown/XML prose images and Lumiverse's native inline-attachment anatomy.
- Image source policy supports Native / Auto / Full quality behavior for enlarged authored imagery.

### Product polish

- Palette product identity and Brush drawer icon.
- Float/Dock/mobile-safe editor workflows and hideable mini widget.
- Readable Read Page, compact Quick Look actions, and structural Group navigator for wrapper-heavy Lumiverse DOM.
- Rebuilt one-page Guide with Palette workflows first and a practical CSS field guide second.

### Compatibility

- Project persistence schema: **v40**.
- Minimum Lumiverse version: **1.1.6**.
- Internal extension identifier remains `theme_studio` to preserve existing storage/runtime compatibility.
