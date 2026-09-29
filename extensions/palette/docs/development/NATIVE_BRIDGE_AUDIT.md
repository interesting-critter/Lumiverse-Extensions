# Palette native bridge audit · v27.53

> **v27.74 Second speaker content hug:** the optional character-vs-character user dialogue now sizes from its actual MessageContent (`fit-content` with bounded desktop width) and uses wide top/bottom VN corner caps that follow the real dialogue box. Mobile keeps a stable 94% content-height window and uses the dedicated mobile corner-cap asset.

> **v27.73 Second speaker stage:** the optional character-vs-character user mode now uses a real flex-positioned scene header instead of anchored HeaderLeft geometry. Portrait/name form one lower-right identity cluster in normal flow; user metadata and BubbleActions live in matching bottom HUD capsules owned by the user Bubble. Reset Selected choice before mounting the alternate.

## v27.53 InputArea / ComposerActionBarLive receipt

- Current staging renders native composer actions through `ComposerActionBarLive`, then exposes the extension-only `chat_toolbar` separately. Palette no longer models those as one toolbar surface.
- Native reorder units expose stable `data-composer-action` + `data-toolbar-action` attributes while their wrappers use `display: contents`. Palette uses those attributes for semantic identity and scopes visual styling to descendant controls rather than pretending the wrapper owns paintable geometry.
- The native `_actionBar_` box remains the native toolbar surface. The extension mount is modeled independently through `_extensionToolbar_` with `[data-spindle-mount="chat_toolbar"]` as a semantic fallback.
- Selector resolution now prefers action-attribute semantics over localized `title`/`aria-label` copy when a native composer control lives inside one of these wrappers. This is DOM presentation metadata only; no private React state or reorder implementation is touched.
- Schema remains v32.

## Previous native bridge audit history

## v27.23 long-message control receipt

- Mounted Lumiverse now exposes a long-message reveal pill inside `MessageContent` via the CSS-module family `_longMessageTogglePill_`. Palette treats **MessageContent** as the semantic owner rather than binding the control to MinimalMessage-only ancestry.
- The outer pill and its inner `span` are separate known parts so packs can own shell geometry and label typography independently.
- Bubble/Minimal message anatomy maps both record the new control as high-confidence mounted chrome; pack recipes remain renderer-compatible without selector context composition.
- No native bridge API or schema change was required. Persistence remains schema v28.


## v27.22 stencil icon skinning

- Reasoning icon replacement remains presentation-only but no longer creates a generated sibling mark. Palette targets the mounted `_brain_` icon box and compiles a normal Background Image packet in **Stencil** mode (`mask-image` / `-webkit-mask-image`), optionally hiding only that box's direct native SVG children.
- The semantic reasoning receipt now prefers the observed combined `_container_` + `_bubble_` root and direct `_toggle_` wrapper, while `button[data-reasoning-toggle="true"]` remains stable fallback evidence. No React/private component state is touched.
- The public asset bridge is unchanged. Stencil images use the same canonical project asset paths, live `contentUrl` resolution, and active-theme borrowing/adoption flow as ordinary Background images.

## v27.21 active-theme asset borrowing

- Palette now reads `ctx.theme.assets.getActiveBundleId()` as the public source for assets belonging to the currently applied native theme. Those assets are exposed read-only in Resources/pack pickers and use their public `contentUrl` for live preview.
- Selecting an active-theme asset from a pack slot or ordinary Background image field copies it through public `getBytes()` + `upload()` into the Palette project bundle before semantic state is committed. The authored packet therefore points at the copied asset's canonical `cssPath`; exported `.lumitheme` drafts do not depend on another theme's bundle.
- Live preview treats `assets/foo.png` and `./assets/foo.png` as aliases only while resolving known bundle assets to `contentUrl`. Stored/generated/native CSS remains canonical and is never rewritten to session URLs.
- SVG replacement for pack chrome remains CSS/presentation-side: the native reasoning icon is made transparent and a generated pseudo-surface carries a safe inline/vector or project asset Background. No React/TSX replacement API is introduced.

## v27.19 mounted chrome receipts

- Greetings launcher: prefer the stable `button[title="Browse alternate greetings"]`; `_indicator_` is internal/fallback anatomy, not the primary button owner.
- Bubble actions: prefer `[data-component="BubbleActions"][class*="_pill_"]`; historical `_actionsPill_` remains a compatibility fallback.
- Reasoning toggle: `button[data-reasoning-toggle="true"]` remains the stable interaction anchor; packs now explicitly neutralize its native border/corner/shadow treatment when composing thought panels.

Palette no longer reverse-engineers Lumiverse's private theme internals at runtime. The v26 bridge is intentionally split between two public surfaces with different lifetimes.

> v27.16 pack preparation remains extension-side. Persistent recipe provenance is project storage owned by Palette, and recipe-declared Layout Groups use the same Members/Contents/Frame compiler as hand-built groups. No new Lumiverse bridge capability or DOM reparenting is required.

### Message reasoning surface

Current Lumiverse `ReasoningBlock` does not expose a dedicated `data-component` root. Palette therefore treats Thinking as a DOM-scoped message surface anchored to `button[data-reasoning-toggle="true"]` inside a known `BubbleMessage` or `MinimalMessage` root. The containing box uses a scoped `:has(> button[data-reasoning-toggle="true"])`; the header targets the toggle directly; the content target resolves the reasoning `_body_` descendant. Historical `ThinkingBlock` / `ThinkingBox` / `ReasoningBlock` data-component selectors remain compatibility fallbacks only. This avoids inventing a native owner while keeping the selector message-local.

## 1. Frontend native authoring: `ctx.theme`

Palette feature-detects four versioned host capabilities and only enables the corresponding UI when the host advertises them.

| Capability | Palette use | No longer needed |
| --- | --- | --- |
| `theme-catalog-v1` | component picker catalog, live CSS-variable reference, typography/reference UI | generated component/variable snapshot, source-path parsing |
| `theme-assets-v1` | project bundle creation, list/upload/delete/optimize, font/image resources | `/api/v1/settings` and `/api/v1/theme-assets` fetch adapters |
| `theme-packs-v1` | canonical `.lumitheme` export/import and native install/save | extension-owned archive codec or settings mutation |
| `theme-editor-navigation-v1` | supported Global/Assets editor handoff | native modal DOM/store poking |

The public catalog contains presentation metadata only. Palette may inspect the currently mounted DOM under a public component selector to discover visible CSS-module local class names for picker ergonomics, but it never treats those hashes or private file paths as canonical native metadata.

## 2. Asset identity

Native asset DTOs deliberately expose two addresses and Palette preserves that distinction:

- `contentUrl` is the runtime/session address used by Palette preview surfaces. The live preview stylesheet resolves known canonical asset URLs to this address at injection time only.
- `cssPath` is the canonical pack-relative `./assets/<slug>` identity persisted in packets, fonts, projects, generated CSS, and exported CSS. It is never replaced in project state by a session URL.

A Palette project stores its own native **source bundle** ID. The first project upload or first borrowed active-theme asset creates that bundle through `ctx.theme.assets.createBundle()`. Assets from `ctx.theme.assets.getActiveBundleId()` are source candidates only; first use copies them into the project source bundle before authoring so export remains self-contained. Native installation deliberately creates/localizes a fresh installation bundle; Palette keeps the project source bundle unchanged so further editing and reinstalling never mutate the installed copy.

## 3. Theme pack boundary

Palette produces `SpindleThemePackDraft`, not a `.lumitheme` archive. The draft contains project metadata, compiled global CSS, recognized native component CSS sections, and an optional asset bundle ID. It has no TSX input surface.

Lumiverse owns archive encoding, archive parsing, asset localization, install semantics, theme-library persistence, and imported-TSX safety. Palette's import flow is inert: after native parsing it creates a new project and stores arbitrary imported global/component CSS in the Custom CSS layer because semantic packets cannot losslessly represent unknown CSS. Nothing is applied until the user explicitly sends/installs it.

## 4. Worker live presentation remains separate

Full App Boost now uses reversible frontend root-inline authority. Palette still clears the backend `spindle.theme` override on startup/reset/teardown for compatibility with older builds, but it does not feed transformed Boost values through that path because those values participate in Lumiverse’s resolved source map.

This gives Palette two explicit contracts:

```text
ctx.theme       persistent/native authoring, assets, catalog, pack handoff
root inline      reversible extension-owned live variable presentation
spindle.theme   canonical Boost source + legacy cleanup
```

## 5. Permission and capability behavior

Palette declares `app_manipulation` plus `ui_panels` for the optional native Style Library dock. Lumiverse remains responsible for enforcing authority at mutating native leaves. Palette still checks capability versions to control UX and degrades to visual editing/Custom CSS when a host does not provide a native authoring subtree. There are no private REST/store fallbacks.

## 6. Remaining deliberate gap

Palette still does not treat arbitrary TSX/React structure as an extension authoring surface. Pack asset slots bind canonical native assets through ordinary semantic packets, and pack recipes can now declare CSS-only Layout Groups, but groups never reparent host-owned nodes. If a future composition truly requires structural React changes, that should be a deliberate public host capability rather than a private DOM/React hack.

## Upstream landing note

The bridge now targets the landed Lumiverse implementation and published Spindle Types `0.6.19`. Lumiverse's follow-up canonical-path fix normalizes asset slugs through its own `toThemeAssetRelativePath()` helper, so Palette deliberately trusts `SpindleThemeAsset.cssPath` verbatim and never reconstructs `./assets/…` from `slug`. Project asset bundles remain source bundles: **Send to Lumiverse** installs a fresh native bundle without replacing the project-owned source bundle.


## Boost root authority

Current Lumiverse applies its resolved native/extension variable map directly to `document.documentElement.style`. Palette therefore keeps Full App Boost on that same root inline declaration block with `important` priority and does **not** mirror transformed values back into `spindle.theme`; doing both creates a source/output feedback loop. The transform baseline is firewall-strict and comes only from worker-side `spindle.theme.generateVariables()`. A MutationObserver watches the root `style` attribute while Boost is active, records newer native declarations only as the underlying state Reset should reveal, reasserts the explicit Boost values, and schedules a fresh worker-generated rebase when Lumiverse appears to have changed the native theme. Boost is excluded from project/native `.lumitheme` CSS.


## v26 boundary note

Responsive Base/Mobile state, canonical compiler slots, fullscreen pack choreography, and message-chrome recipes are entirely Palette-side semantics; they require no further native bridge expansion. Mobile deltas compile to ordinary CSS media queries, pseudo-elements remain ordinary selector surfaces, and Greetings/Swipe/action-dock recipes only restyle mounted native UI.

Frontend startup canonicalizes the worker-generated baseline even when the initially selected Palette project has Boost disabled, then applies saved Boost if needed. A short post-ready worker source watch handles Lumiverse themes that finish installing after extension setup; selecting another enabled Boost project also fetches a fresh canonical source. The frontend catalog is no longer part of live Boost source capture, so presentation output cannot become transform input. This does not add a new native contract.

## v27.1 message-side boundary note

Assistant/User/Both authoring does not require another native bridge surface. Current BubbleMessage and MinimalMessage expose their role distinction in mounted presentation anatomy through the user-side CSS-module modifier and, for some parts, `*User` variants. Palette derives the facet from the mounted DOM underneath the public component boundary and keeps it as selector/compiler metadata only. It does not add a private store/REST dependency, filesystem metadata, or a new persistence contract.

## v27.9 image-source presentation bridge

Palette still does not patch private React components, replace TSX modules, or depend on an internal store. Some native avatar surfaces intentionally render thumbnail-tier image URLs; CSS can enlarge those boxes but cannot recover pixels that were never requested. The optional Image **Full source** treatment therefore uses a narrowly scoped extension-owned DOM presentation bridge: for images selected by an authored Palette target, it recognizes supported Lumiverse avatar/image thumbnail URL shapes, swaps the mounted `img.src` to the corresponding full resolver URL, suppresses thumbnail `srcset` while active, observes normal React rerenders, and restores the latest native source when the treatment is removed or the extension unloads. Unknown URLs are left untouched.

This bridge is deliberately not a generic prop/TSX replacement API. If a future visual edit requires application behavior that cannot be expressed through CSS or a reversible presentation-level DOM attribute, it should use a public Spindle capability or a native Lumiverse change rather than monkey-patching React internals.

Selector scoping remains extension-side as well. Reused CSS-module locals under very broad native roots such as App retain a nearby module-family ancestor only when that ancestor demonstrably narrows the mounted match set. Real semantic `data-component` boundaries remain the preferred scope, and historical exact `App > Avatar` overrides are quarantined instead of guessed across Character/Persona/Profile/message families.


## V27.11 cleanup notes

- Typography-only Boost no longer samples the native theme variable map. With Colors and Backdrop disabled, the frontend owns only `--lumiverse-font-family` / `--lumiverse-font-scale`; canonical `generateVariables()` remains reserved for palette/canvas transforms and explicit Refresh source.
- Image source quality remains a DOM presentation bridge rather than a component replacement API. **Auto** promotes only recognized Lumiverse thumbnail routes and bases promotion on mounted rendered size plus actual loaded intrinsic dimensions; **Full** remains explicit original-file behavior. Neither mode rewrites arbitrary URLs or React internals.
- Style-pack hero recipes that enlarge native portraits use Auto by default, so pack application does not silently request original assets unless the large thumbnail is genuinely insufficient.


## V27.12 native-surface scope note

CharacterBrowser/PersonaBrowser scoping uses the existing public `theme-catalog-v1` component metadata plus mounted CSS-module evidence already permitted for picker ergonomics. Palette now evaluates all matching component contexts instead of taking the first CSS-module family on a root. Specific browser ownership can outrank generic App only when catalog/module evidence distinguishes it; identical ambiguous catalog selectors are ignored rather than guessed. App-wide and Similar scopes remain explicit Palette selector choices. No new Spindle capability, private store, REST route, or TSX hook is introduced.


## V27.13 native-owner confidence note

`theme-catalog-v1` remains the only native component metadata source. Palette now treats mounted CSS-module overlap as fallback evidence rather than authority: a non-generic `data-component` or unambiguous catalog selector outranks module inference. The frontend bridge records only runtime picker fingerprints (common generated CSS-module hashes observed on roots matched by a public selector); those hashes are never persisted or exported. When one normalized catalog selector spans several unrelated mounted module families, its class inventory may still enrich Edit Part, but it cannot name a picked native surface. This closes false labels/scopes such as Persona/editor rows being attributed to `QwenCustomVoiceManager` because both use generic `manager`/`row` vocabulary. No new Spindle capability or private source path is used.

## v27.14 layout-group note

Layout Groups remain extension-side and CSS-only. They persist semantic sibling membership plus a real shared-parent target, then compile row/column/grid composition through that parent. Palette does not reparent React-owned DOM nodes and this feature requires no additional native Spindle bridge.

## V27.25 cream/action boundary note

Journal's fixed ink and Manga's utility strip remain Palette-side recipe semantics. `tuneText: false` only prevents the pack workbench palette tuner from rewriting an authored Text packet before normal compilation; it does not add a native bridge or new persistence contract. The new Bubble action-control role is scoped under public `data-component="BubbleActions"` with the historical `_actionsPill_` family as fallback, while Minimal action controls remain inside `data-component="MinimalMessage"`. Journal's avatar-backdrop suppression targets the already-known Bubble `_avatarBgImg_` presentation layer. No broad app-button selector, private store access, or React mutation is introduced.

## V27.26 Journal backdrop/composer receipt

Mounted BubbleMessage receipts prove the visual avatar backdrop is a stack rather than a single image: `[class*="_avatarBg_"]`, its nested `img`, and `[class*="_avatarBgImg_"]`. Journal now owns a narrow aggregate semantic target for that stack and compiles Visibility Gone instead of relying on opacity on one descendant. InputArea also gains narrow descendant roles for action-bar controls and `textarea[name="chat-message"]::placeholder`; compiler authority recognizes `::placeholder` as a pseudo-element so selector guards remain valid.


## V27.27 decorative-plane receipt

Journal proved that avatar-owned pseudo surfaces can be clipped by native portrait overflow, even when the semantic ornament packet itself is valid. Palette therefore adds Bubble/Minimal header `::after` roles as alternate decorative planes and moves the built-in washi recipe there; no DOM wrapper or private React access is introduced. The actual avatar stack remains untouched.

The Manga Temper asset is still an ordinary bundled Stencil. Only its built-in SVG silhouette changed to the recognizable U+1F4A2 anger-symbol shape; user assets and pack provenance behavior are unchanged.


## v27.31 BubbleActions positioning receipt

Current Lumiverse mounts `BubbleActions` as one element carrying both its compact `pill` class and BubbleMessage's `actionsPill` class. The native `actionsPill` contract is absolute positioning (`top`/`right`, with user-side variants). Palette therefore must not force `position: relative` for Nudge; doing so removes the element from its native absolute contract and can make it participate/stretch in the BubbleMessage flow. Journal Paper Actions now relies on pure `translate` while preserving Lumiverse's native absolute positioning.


## v27.32 HTML-island layering receipt

Mounted regex/custom HTML is reachable as `[data-component="MessageContent"] [class*="_htmlIsland_"]`. Journal keeps descendant markup untouched and uses generated `::before` / `::after` surfaces for the clipping backing and tape. Negative pseudo z-index is contained with compiler-generated `isolation: isolate` on the island host; this is a stacking safety rule, not a DOM or native-bridge mutation.


## v27.33 Journal centered-sheet audit

Journal Bubble MessageContent remains a normal semantic MessageContent target. The pack uses existing Layout Item `align-self: center` plus parent width/max-width rather than inventing a wrapper or absolute anchor; Mobile returns to stretch. Pasted ephemera keeps the v27.32 isolated negative pseudo backing and adds ordinary host/backing Shadow packets only.

## v27.34 Flow-centering audit

Mounted Journal testing showed `Layout Item → align-self:center` could not center Bubble MessageContent because the native parent does not expose a reliable flex/grid item contract for that child. Schema v30 therefore adds a logical Flow placement field to Position. `Center block` emits `margin-inline:auto` only for Flow/Nudge and never changes positioning mode or parent display. Journal uses it on the capped 760px notebook sheet at Base and lets the same logical centering remain harmless on Mobile because the sheet fills its narrower parent.

Journal shadow tuning is deliberately neutral/ink-led rather than palette-led: physical depth must remain visible on cream paper even when the mint accent is pale.


## v27.35 Generated Content + metadata-segment audit

Mounted BubbleMessage metadata exposes the message number, timestamp, and token count as ordered sibling spans carrying the `_metaSegment_` class fragment inside `_metaPill_`; separator glyphs use `_metaDot_`. Palette promotes those mounted descendants into semantic message-number / timestamp / token-count roles using order-derived selectors and leaves their text untouched. No private store, REST route, React prop access, or duplicated runtime value is introduced. If Lumiverse changes the mounted segment order, the semantic adapter must be updated once rather than letting individual packs guess independently.

Schema v31 **Generated Content** is static pseudo-surface art direction only. It compiles an escaped literal `content` declaration for `::before` / `::after` and does not evaluate templates, macros, DOM text, or native state. Journal combines those two independent mechanisms in Filing ledger and Private note: generated labels provide `ENTRY / FILED / LENGTH / PRIVATE NOTE`, while live message metadata and reasoning duration remain native mounted content.

## v27.64 mounted inline-media receipt

Current mounted BubbleMessage inspection exposes attachment ancestry as `Content → Attachments → Inline Image Btn → Inline Image Wrap → Inline Image`. No stable public attachment `data-component` is present in that receipt. Palette therefore recognizes `_inlineImageBtn_`, `_inlineImageWrap_`, and `_inlineImage_` as **medium-confidence DOM-scoped** attachment targets while retaining generic `_attachments_` / `_attachment_` / `MessageAttachments` fallbacks. Packs may style these nodes, but they must not document the class family as a guaranteed Lumiverse API.

### v27.66 — Bubble attachment/reasoning order receipt

Current BubbleMessageDefault mounts assistant reasoning first, then assistant attachments inside a separate `styles.content` sibling, then the main `styles.content` sibling containing `MessageContent`. Visual Novel therefore styles the attachment wrapper as a distinct **Assistant CG lane** rather than assuming the inner `MessageAttachments` node can participate in Bubble-level flex ordering. Reasoning body targeting follows ReasoningBlock's `bodyWrapper → bodyInner → body` chain to avoid collisions with unrelated body-class families.


### v27.67 — attachment layout-owner + speaker-anchor receipt

Mounted Workbench inspection confirms the inline attachment ancestry `Attachments → Inline Image Btn → Inline Image Wrap → Inline Image` has **Inline Image Btn** as the effective layout-owning control. Visual Novel therefore authors Auto width / Fit height and the responsive **Attachments** boundary on that outer button, while the nested Wrap/Image retain frame and object-fit presentation. The assistant speaker name is anchored relative to the mounted Header/speaker lane rather than Bubble, which keeps the label tied to scene geometry when CG/reasoning siblings change height. Because Assistant dialogue intentionally nudges upward (`-42px` Base, `-32px` Mobile), the CG lane reserves matching bottom clearance plus a small visual gap so MessageContent never covers the attachment.


### Visual Novel Header containing block

VN keeps the native BubbleMessage Header as a real box because the assistant speaker name uses it as a positioning ancestor. `display: contents` on Header invalidates that containing-block contract even if generated CSS adds `position: relative`; only the inner HeaderLeft/MetaWrap structural wrappers are dissolved.
