## v28.16 · Guidebook

- Replaces the built-in release-note wall with a real one-page Palette manual: quick start, target/scope model, style packets, groups, lazy Read Style/Page, Quick Looks/My Styles/packs, Boost/Typography, widget/native handoff, and a practical CSS field guide.
- Adds tables, selector examples, responsive/image/pseudo/layout recipes, and a debugging checklist instead of forcing users to reverse-engineer the changelog to learn the extension.
- Guide jump links now use Palette-owned click navigation that finds the rendered heading and scrolls it into view. This avoids relying on Lumiverse's current Markdown heading-id behavior, which does not reliably move the Guide modal.
- No persistence/schema changes; schema remains **v39**.

## v28.15 · Cockpit Fit + Group Navigator

- Quick Look card actions stay on one row; Reset no longer drops into a lonely second row, and unapplied compact looks do not reserve dead reset space.
- Themes folds Backdrop into the Boost card while keeping Typography as a genuinely standalone app-wide layer.
- Read Page gets a full readability pass: larger modal, search, target rows, source metadata, section headers, and mobile spacing instead of text-for-ants density.
- New Layout Group gains structural target navigation. Each picked member can retarget to nearby descendants/ancestors, temporarily tolerate depth mismatch while you line members up, and add eligible siblings directly from the established shared parent without pixel-hunting nested Lumi wrappers.
- No persistence/schema changes; schema remains v39.

## v28.14 · Palette brush identity

- Replaces the interim crossed-brush drawer glyph with the supplied Alum Design paintbrush artwork.
- Crops the icon to the actual brush/sparkle geometry, removes the embedded Noun Project attribution text from the rendered glyph, and keeps attribution in the Guide as required.
- No persistence/schema changes; schema remains v39.

## v28.13 — Palette: Drawer Identity + Readability

- Spindle drawer roots such as `data-spindle-drawer-tab="personas"` now act as stable semantic surface boundaries. Picker ownership and My Styles grouping prefer the drawer name (for example **Personas**) over unrelated CSS-module guesses such as `QwenCustomVoiceManager`; legacy saved targets are also relabeled from their live mounted drawer when that relationship is unambiguous.
- Save Style's dropdown is anchored to its trigger inside Float instead of centered around it, fixing the half-clipped menu at narrow editor widths.
- My Styles, Choose Parts, Style Library navigation, and the densest Design helper copy get a small readability bump after the UI-maxxing pass drifted into 6–8px ant text.
- Palette keeps the crossed-brush drawer identity. Guide attribution records **paintbrush by Alum Design (CC BY 3.0)**.
- Schema remains **v39**.

## v28.12 — Palette: My Styles Bundles + Fit Pass

- Schema **v39** expands global **My Styles** from Target/Component captures to hand-picked **Bundles**. Design → Save style → Choose parts opens an authored-target checklist grouped by native component, so one reusable style can deliberately span Persona Browser, Persona Editor, footer controls, or any other persistent Palette targets without copying generated CSS.
- Saved Bundle cards show an Apply preview with mounted/not-mounted targets plus merge/new-target counts. Not-mounted stable selectors remain legal authored state; applying still merges semantic packets and preserves unrelated destination packet types.
- Save style tools now fit the floating editor: the tool row wraps and the popover centers/clamps to the viewport instead of escaping off the side.
- Code editors soft-wrap long generated/custom CSS lines and hide horizontal overflow by default.
- Palette's drawer icon switches from the Lumiverse-theme-adjacent palette silhouette to crossed brushes.
- Full App Boost drops the stacked `N active` badge; the header now quietly names the enabled layers (`Colors + Typography`, etc.).

## v28.11 — Palette: My Styles + Cockpit Pass

- Visible product identity graduates from **Theme Studio** to **Palette** while the stable extension identifier, persistence keys, storage paths, and Spindle contracts remain `theme_studio` for compatibility. Generated CSS and native `.lumitheme` handoff metadata use the Palette name too.
- Schema **v38** adds global **My Styles** bundles. From Design, **Save style** can capture the current target (including its generated front/back planes) or the selected native component's authored section. Saved styles keep semantic Base/Mobile + state packets, live outside any one theme project, and can be applied into another project as ordinary editable Palette overrides.
- Style Library gains a dedicated **My Styles** shelf with search, Apply, Rename, and Delete controls. Applying a saved style merges only its packet types into matching selectors, preserving unrelated authored styling in the destination theme.
- Theme Stash becomes a compact responsive card grid with palette swatches, target/group/Boost counts, active state, and inline Rename / Duplicate / Delete actions.
- The completed Native Bridge capability-status panel leaves the normal Themes workspace. Native handoff remains where it is useful in Code, whose actions now wrap cleanly on narrow layouts.
- Recipe Palette becomes a compact disclosure, disabled Boost cards collapse to switch-sized rows, and routine Design scope/state chrome is flattened so packet cards remain the visual hierarchy instead of every control becoming another bordered box.

## v28.10 — Packpocalypse XCIV: VN Greetings Corner Lock

- Visual Novel Route greetings now anchors the alternate-greetings launcher to the Bubble's top-right corner instead of inheriting its unstable natural position.
- Desktop uses 12px top/right; Mobile uses 8px top/right, both on a raised z-layer.
- The launcher itself now keeps intentional button padding (6px × 10px desktop, 4px × 8px mobile) so the top-corner chip remains readable instead of collapsing around its indicator. Internal content padding is zeroed to avoid double-padding.
- Schema remains v37.

> **v28.09 VN mobile static-position lock:** the cinematic Visual Novel corner caps keep their desktop edge anchors, but Mobile now follows the mounted fix that finally behaved: absolute pseudo positioning with all inset edges left automatic, then explicit translate offsets (`-13px/-11px` for the upper cap and `-13px/-46px` for the lower cap). The assistant values come directly from the mounted inspector; Second Speaker mirrors them as the best matching `contentUser` estimate. This keeps the caps tied to the `_content_` / `_contentUser_` static positions instead of fighting percentage/container geometry. While type-checking the pass, a latent VN CG-insert bug was also caught: three native-attachment recipe steps referenced an undeclared Media Flow local, so those steps now instantiate their own full/unclipped flow packet instead of throwing when applied. Greetings and all v28.08 content-frame targeting remain intact. Schema stays **v37**.

> **v28.08 VN content-frame + greetings:** the cinematic Visual Novel corner caps return to the outer `_content_` body wrapper (`assistant.content.mount` / `user.content.mount`) after mounted DOM inspection confirmed that this wrapper is the painted dialogue card while `MessageContent` is the prose node inside it. The responsive pseudo-position inheritance and explicit Fill-width geometry remain intact. Visual Novel also gains a dedicated Route greetings recipe so the alternate-greetings launcher participates in the pack chrome and Apply All. Selected Choice keeps its existing mount-level label/frame contract. Schema stays **v37**.

> **v28.07 VN inner-content frame retarget:** the cinematic Visual Novel corner caps stop living on the outer `_content_` mounting board and move onto the actual `MessageContent` text box (`assistant.content` / `user.content`). This makes the ornament geometry follow the padded dialogue body the user actually sees instead of the broader wrapper around it. Selected Choice intentionally keeps its existing mount-level pseudo label/frame contract. v28.04 responsive pseudo-position inheritance and v28.06 Fill-width cap geometry remain unchanged. Schema stays **v37**.

> **v28.06 VN cap resurrection:** v28.05 proved that auto/native width plus dual-edge pinning can collapse the real VN pseudo plane in mounted Lumiverse. The cinematic assistant and Second-speaker dialogue caps therefore return to the last empirically visible geometry: explicit Fill/100% width from the left edge, with v28.04 responsive top/bottom anchor inheritance preserved. This is an intentional rollback of only the failed centering experiment; exact visual centering stays a mounted-geometry polish problem rather than another speculative pseudo sizing rewrite. Schema stays **v37**.

> **v28.05 VN dialogue-cap centering:** the VN upper/lower corner pseudo-planes stop using `width: 100%` from the absolute left edge. Assistant and Second-speaker caps now pin both horizontal edges (`left: 0; right: 0`) and leave width native/auto, so the ornament stretches symmetrically with the actual dialogue mount on desktop and mobile. Responsive Base-position inheritance from v28.04 remains intact. Schema stays **v37**.

> **v28.04 VN responsive pseudo inheritance:** Mobile pseudo-surface scaffolding now inherits Base anchored Position semantics when the Mobile scope only changes paint/size. This fixes the VN dialogue frame regression where the lower rotated corner cap was silently reset to `inset: 0` on phones and collapsed onto the top cap, making the four-corner frame look like a decorative top strip. The fix is compiler-level so other responsive pseudo ornaments keep their Base anchor without recipes duplicating Position packets. Schema stays **v37**.

> **v28.03 VN dialogue-frame lock:** the VN corner plane is back on the actual dialogue mount, not the outer BubbleMessage. Assistant and Second speaker stages now use the content-hugging `MessageContent` frame for both top and bottom corner art, while the accidental outer-stage corner planes are removed so the top is neither doubled nor lost behind the hero area. The touch-safe avatar backdrop, right-docked pager, prose/media fixes, and schema **v37** remain unchanged.

> **v28.02 VN touch-backdrop exorcism:** Lumiverse hides `_avatarBg_` on coarse/touch pointers with a native `display:none`; Theme Studio backdrop recipes now explicitly restore the backdrop frame as a block when the visual intent is to use it. Visual Novel keeps the wide corner art on the full message stage only (no duplicate dialogue-box corner plane), and Route HUD SwipeControls shrink back to content width and dock at the far right while the separate HUD rail remains the wide background. Ghost backdrop and Scene backdrop Basics receive the same touch-safe display ownership. Schema stays **v37**.

> **v28.01 VN regression sweep:** Visual Novel restores its scene/backdrop contract after the recent Manga passes: dedicated backdrop images fill their own frame without redundant responsive Size boundaries, Second speaker clears Lumiverse's native avatar-backdrop mask, assistant name Mobile inherits the proven Nudge instead of re-entering anchor wars, Route HUD swipes align to the far edge, and the VN corner language now also frames the full message stage. VN gains an H1–H4/body/code prose hierarchy, and Editorial / Visual Novel / Journal all adopt Media Flow across raw Markdown/XML images plus native Lumi attachments. Schema stays **v37**.

> **v28.00 Widget visibility toggle:** the collapsed mini widget now opens a tiny context menu on desktop right-click or touch long-press instead of disappearing immediately; the menu owns the explicit Hide action. The Design workbar widget button is now a true on/off visibility toggle, so the same control that restores a hidden widget can hide it again. Visibility remains locally persisted and schema v37 is unchanged.

> **v27.99 Manga speaker-lane parity:** mobile user BubbleMessage now expands the actual Header → HeaderLeft → Avatar chain instead of only the outer frame/bubble, so the portrait stage can use the full message width. User speaker names are explicitly anchored to the user Header with the validated -2px / -36px nudge, keeping them inside the image stage instead of hanging below it. Schema v37 is unchanged.

> **v27.98 Manga mobile parity:** phone user BubbleMessage/frame geometry now explicitly claims the full available width instead of inheriting Lumiverse's compact user cap, and Manga user speaker names use the exact same 25px Base / 19px Mobile heavy italic display treatment as assistant names. Desktop geometry, pager spacing, and schema v37 are unchanged.

> **v27.97 Manga pager breathing room:** phone assistant BubbleActions reserve a slightly wider 140px lower-right lane whenever SwipeControls are mounted, keeping the action strip visually separate from the pager without moving the pager itself. Desktop/user controls and schema v37 are unchanged.

> **v27.96 Manga pager truce:** on phones, assistant BubbleActions only shift left when the same message actually mounts SwipeControls, reserving the lower-right pager lane without penalizing messages that have no swipes. Desktop/user controls stay unchanged; schema remains **v37**.

# v27.95 — Packpocalypse LXXX: Manga Mobile Trim

- **Manga user-side names get the full display identity treatment.** The robust user-name role is no longer a tiny 11px utility label: Bubble user names now use a 20px italic uppercase print treatment on Base and 17px on Mobile, with a restrained dark stroke so Narrator/user identity reads as part of the same panel language as assistant names.
- **Mobile BubbleActions stop renting a second apartment.** Manga now re-anchors the action pill to the actual Header on phones, pins it to the lower-right of the image stage, trims the pill to a 20px utility strip, and tightens padding/gap. This replaces the old top-right Message-frame placement and avoids large translate-based viewport tetris while preserving the desktop corner controls.
- Schema stays **v37**. Existing Manga projects should **Reset Manga → Apply All** once to refresh the user-name and Mobile action layers.

# v27.94 — Packpocalypse LXXIX: Manga Cast Pass

- **Manga user identity now uses the robust user-side name role.** The Bubble recipe targets the current user speaker selector family (`_nameUser_` or shared `_name_` under the user marker) instead of the older `_nameUser_`-only shortcut, so user-side names receive the same compact uppercase/italic print language.
- Added Manga **Cast strip** for the mounted `MessageList` group-character roster. The real wrapper/bar/member/active/avatar/name/add-member anatomy gets square monochrome cards, grayscale portraits, hard rules, subtle diagonal print texture, and a matching active-member treatment. It is included in Manga Apply All and appears under Scene furniture / Global UI.
- **Use generic selector now prefers persistence over brevity.** When a title/ARIA instance selector offers both `[class*="_member_"]` and `._member_<build hash>`, the stable CSS-module local selector wins instead of the shorter exact generated class. Exact hashes remain available as advanced selector candidates but are no longer recommended as the reusable escape hatch.
- Schema stays **v37**. Existing Manga projects should **Reset Manga → Apply All** once to add Cast strip and refresh the user-name target.

# v27.93 — Packpocalypse LXXVIII: Manga Panel Pass

- **Manga stops assuming every portrait wants to live in the upper quarter.** The main Bubble panel now starts at a saner `52% 46%` focal position on desktop and `50% 46%` on mobile; foreground rail/sticker portrait variants reset to neutral vertical focus instead of inheriting the old upward bias. Atmospheric backdrop art keeps its intentional crop.
- **The Bubble header is recomposed as one graphic panel instead of box-after-box UI.** A masked diagonal hatch accent lives on the header back plane, metadata pins to the upper-left utility lane, and the italic uppercase character name overlays the lower-left edge of the artwork. These are ordinary Theme Studio packets, so the mounted result remains editable in Design.
- **Temper mark now uses a real SVG Asset on the actual Bubble/Minimal card front plane.** The cleaned rage glyph is rendered as a vivid red stencil with fixed pixel geometry and safe top-right placement, replacing the older pseudo/background-mask route that could disappear behind the card/actions composition.
- Added Manga furniture basic **Panel down cue** for `[aria-label="Scroll to bottom"]`: square monochrome shell, hard border/shadow, and a matching printed navigation treatment. Manga Apply All includes it.
- **Manga composer field padding now owns the visual inset for both typed text and `::placeholder`.** Placeholder remains a conditional pseudo target, but the host textarea supplies the actual breathing room so copy no longer hugs the upper-left corner.
- Schema stays **v37**. Existing Manga projects should **Reset Manga → Apply All** once to replace the old focal/Temper/composer layers and add the new scroll cue.

# v27.92 — Packpocalypse LXXVII: Ink Truce

- **Text Style now separates default ink from forced glyph paint (schema v37).** Solid text exposes **Descendant colors → Respect / Override**. Respect emits ordinary `color` only, so authored child colors such as Lumiverse `proseDialogue`, legacy `<font color>`, and custom HTML can win naturally. Override additionally owns `-webkit-text-fill-color` for hostile controls such as the current InputArea textarea. Gradient text keeps its required transparent WebKit-fill machinery.
- Fresh Text Style packets default to **Respect**. Pre-v37 authored packets normalize to **Override** to preserve existing user intent rather than silently changing hand-built themes. Read Style marks a solid text observation as Override only when the inspected target itself actually authors `-webkit-text-fill-color`.
- **All pack/basic recipe text now passes through one semantic ink audit.** `input.*` composer roles force ink because current Lumiverse controls are the proven WebKit-fill conflict zone; every other solid Text packet uses cascade-safe ink. This covers Manga, Editorial, Journal, Visual Novel, headings, prose, thinking, metadata, user/assistant dialogue bodies, names, actions, and standalone Basics without hand-maintained per-recipe exceptions.
- In particular, Journal can still establish dark ink on its light paper and Manga/Editorial/VN can establish their own default body colors **without bleaching descendant dialogue spans or font-tag colors**. Existing applied pack layers should be Reset + reapplied once to receive the audited v37 recipe packets.

# v27.91 — Packpocalypse LXXVI: Media Amnesty

- **Message media becomes first-class semantic anatomy instead of an attachment-only special case.** Theme Studio now recognizes prose/Markdown image paragraphs, linked image wrappers, raw MessageContent images, and Lumiverse's native attachment → inline button → inline frame → image lane as separate reusable targets.
- Added the persisted **Media Flow** packet (schema **v36**): Native leaves layout alone, Natural returns media to normal block flow with intrinsic aspect ratio, and Full width additionally claims the reading measure. **Unclipped** clears thumbnail-style overflow constraints. The packet compiles the defensive display/width/height/max-height/aspect/float/clear/overflow reset that previously required hand-authored CSS.
- Added standalone **Full-width media** and **Inset media card** Basics so raw Markdown/XML images and native Lumi attachments can share one coherent treatment without adopting a full pack.
- Manga gains **Ink media panel** and Apply All now includes it. Raw prose images and native attachments use the same full-width monochrome panel language, Auto source quality, natural height, contained image fit, hard edges, and unclipped wrappers instead of leaving attachment cats as unrelated thumbnail widgets.
- Manga **Temper mark** now uses the cleaned imported anime-rage SVG as its built-in ornament instead of relying on font/ASCII punctuation, is explicitly vivid red, and moves inward from the top-right action lane on Bubble and Minimal layouts so it can actually remain visible.

# v27.90 — Packpocalypse LXXV: Pre-Pack Polish

- **Composer Composer preserves both anatomy-rail and sidecar scroll while switching parts.** Clicking a lower Frame/Controls/State entry no longer rerenders the one-column anatomy navigator back to its top; swapping targets inside an already-open sidecar also keeps the editor's current scroll depth instead of snapping upward.
- **`display: none` gets a visible red hidden marker.** Target-ladder rungs, the active Selected header, and Composer Composer anatomy rows now show a small red × when their exact element target owns a Gone Visibility packet. Generated Front/Back layers do not falsely mark their host as hidden.
- **Composer Placeholder becomes a real conditional semantic target.** Theme Studio now recognizes the mounted textarea behind `::placeholder`; the Placeholder row is enabled only when that textarea actually has a `placeholder` attribute. The mock uses the live placeholder text and lets clicking that text open the placeholder's normal Design stack.
- Added standalone Composer basic **Jewel send**, promoting Composer Composer's compact circular send treatment into a reusable two-part recipe for the real send button + glyph. Schema stays v35.

# v27.89 — Packpocalypse LXXIV: SVG Customs

- **Saved SVG import now accepts ordinary standalone-file preambles.** UTF-8 BOMs, XML declarations, leading comments, and legacy `<!DOCTYPE …>` wrappers from vector exporters are stripped before validation/storage, then the existing executable/remote-content sanitizer runs on the real `<svg>…</svg>` root. The wardrobe stores only the embeddable SVG body; PUBLIC/SYSTEM DTD references never survive into project state or CSS data URIs.
- Old-school exported artwork such as traced/vectorized glyphs with `pt` dimensions, nested `<g>` transforms, and path data can therefore be reused as Composer icons or generic SVG Assets without manually deleting XML/DOCTYPE boilerplate. Schema remains v35.

# v27.88 — Packpocalypse LXXIII: Composer Tailor

- **Composer Composer now targets native actions one-by-one.** Clicking a mock toolbar glyph resolves the exact stable `[data-composer-action="…"][data-toolbar-action="…"]` wrapper and opens that action as the sidecar target; choosing the broader **Action buttons / Native toolbar** anatomy still edits the whole action family deliberately. Composer Icons compilation recognizes exact action scopes and emits only that action instead of nesting/duplicating the toolbar selector or recoloring every glyph.
- **The workshop header is compressed into a real navigator strip.** Anatomy becomes one narrow, scrollable column; the live-ish mock composer sits beside it at a compact height; the real Design packet editor owns the full row beneath. Mobile keeps the same anatomy → preview → editor hierarchy without hiding the navigator.
- **The mock composer is now a preview as well as a selector.** Authored shell/actionbar/field/control treatment is reflected approximately in the schematic, and Composer Icon families/custom SVG overrides use their real mask glyphs there. It remains a teaching preview rather than a promise of pixel-identical mounted layout.
- **Saved SVGs graduate from Composer into a project-wide asset wardrobe (schema v35).** Existing v34 `composerSvgs` migrate losslessly to `svgAssets`; Composer Composer continues using the same library, while Design gains a reusable **SVG Asset** packet that can render a stored SVG as a tinted stencil or ordinary background image. Applying an asset snapshots sanitized SVG source into the packet so later wardrobe cleanup cannot break authored output.
- SVG Asset packets work on ordinary targets and generated Front/Back pseudo-surfaces, making the wardrobe useful for `::before` / `::after` ornaments, badges, dividers, stamps, and future accessory systems instead of being a composer-only island.

# v27.87 — Packpocalypse LXXII: Composer Wardrobe

- **Composer Composer is rebalanced around the actual work.** The compact mock composer stays at the top as a visual navigator; selecting a mounted part keeps the anatomy index visible and opens the existing Design packet stack below the preview instead of dedicating most of the canvas to empty preview acreage. The mock itself is clickable: toolbar slots, attachment, writing field, send button, and send icon jump straight to their semantic editor.
- Composer Icons becomes a visible accessory workflow rather than pack-only plumbing. The sidecar offers an explicit **＋ Icon wardrobe** action when the native toolbar has no Composer Icons packet yet, then the normal Design packet exposes Native / Manga / Editorial / Journal / Visual Novel families plus per-action overrides.
- Added a project-owned **Saved SVG wardrobe** (schema **v34**). Users can choose a stable native composer action, apply any stored SVG to that one slot, return it to the family default, import a local `.svg`, paste SVG source, name it, reuse it, or remove it from the wardrobe. Applied packets embed a sanitized SVG snapshot so deleting the library entry does not break an already-authored icon.
- Stored SVGs are sanitized before persistence/use: scripts, styles, foreignObject/image embeds, remote/event hrefs, inline event handlers, and style attributes are stripped; unsafe URL/javascript payloads fail closed. The compiler still replaces only glyphs inside stable `data-composer-action` + `data-toolbar-action` wrappers, leaving button behavior, badges, order, and extension-only actions untouched.
- Sparse ownership remains honest: changing only Composer Icons glyph size does not materialize an icon family, while an explicit family or custom per-action SVG does. Recipe-slot migration also recognizes Composer Icons ownership so pack provenance remains reversible.

# v27.86 — Packpocalypse LXXI: Composer Atelier

- **Composer Composer becomes an in-Library sidecar instead of a teleport.** Selecting Shell / toolbar / field / controls / state keeps Style Library mounted and opens the existing Design packet stack beside the anatomy browser. A Back control returns to anatomy; **Open full Design ↗** remains the explicit escalation path for picker/selector surgery. Sidecar scroll survives live packet commits.
- The Composer Composer typography/layout is enlarged out of ant territory. Anatomy rows are now real navigation, the schematic has room to teach the surface, and narrow screens use a one-level-at-a-time flow rather than squeezing anatomy + Design together.
- Added persisted **Composer Icons** semantic packets (schema **v33**) with Native / Manga / Editorial / Journal / Visual Novel families plus glyph size. The compiler replaces only SVG glyphs inside stable native `data-composer-action` + `data-toolbar-action` wrappers; button chrome, order, badges, click behavior, and extension-only actions remain untouched.
- Manga, Editorial, Journal, and Visual Novel composer recipes now ship their own first icon family, making the pack composer language recognizable at the toolbar level instead of sharing the same native glyph set. This is the pack-first seed for future standalone Basic icon wardrobes / text-only / icon+label presentation.
- No new parallel composer styling engine was introduced. Sidecar edits still author ordinary Design packets and pack-applied Composer Icons remain editable/resettable through normal recipe provenance.

# v27.85 — Packpocalypse LXX: Composer Composer Seed

- Composer contrast gets a second, source-correct hotfix. Solid **Text Style** now emits both `color` and `-webkit-text-fill-color`, so authored ink can beat Lumiverse surfaces that explicitly own WebKit text fill instead of appearing pale/transparent despite a winning `color` declaration.
- Manga and Editorial composer textarea/placeholder/send ink are marked palette-stable, preventing pack palette refinement from recoloring deliberately dark/high-contrast control text back into unreadable pale pack text.
- Added semantic composer roles for the writing-field wrapper, the standalone attachment control, and the nested send icon/control. Manga, Editorial, and Journal now explicitly own attachment/send-icon contrast instead of hoping broader toolbar or button styling catches them.
- The Style Library **Composer** surface now seeds **Composer Composer**: a mounted-anatomy workshop with direct shortcuts into Design for shell, toolbars, writing field, action buttons, attachment, text, placeholder, send button/icon, badges, selected state, popover, rows, and labels. This is the landing area for future Native / SVG set / Text-only action presentation; that replacement system is intentionally not part of this release.
- Existing popover styling and badge suppression remain intact. No schema migration; project schema stays **v32**.

# v27.84 — Packpocalypse LXIX: Composer Surface Cohesion

- Composer pack recipes now treat persona/guided-generation badges as optional auxiliary chrome. Manga, Editorial, Journal, and Visual Novel suppress those badge spans through the stable `data-composer-action` + `data-toolbar-action` wrappers instead of generic badge selectors.
- Persistent persona/guides selection chrome is styled separately from badge visibility, so hiding the tiny badge does not leave an accidental native selected-state treatment. Manga flattens it into the command strip, Editorial keeps a quiet ink/paper selection, Journal uses a soft stationery marker, and Visual Novel keeps a restrained purple armed state.
- Composer-triggered InputArea popovers are now pack-owned surfaces: shell, menu rows, hover treatment, secondary copy, dividers, and persona search inherit each pack's visual language instead of dropping back to unrelated native chrome.
- Editorial Composer gets a legibility hotfix: native action bars remain fully opaque, icon ink is substantially darker, and placeholder contrast is raised while preserving the light writing-desk thesis.
- The future Native / SVG set / Text-only composer presentation system is intentionally deferred; this pass stabilizes the current native-action surface family first. Schema remains **v32**.

# v27.83 — Packpocalypse LXVIII: Semantic Separation

- Journal folds the washi ornament into Polaroid Note and Pinned Journal Card; the old standalone Washi Photo recipe is hidden from the library but retained internally so older provenance can still reset cleanly.
- Manga Sticker Portrait is now a genuinely pasted monochrome comic cutout with rotation, white paper border, and hard offset print shadow.
- Editorial Author Portrait is now a muted circular publication medallion with fine rule and soft depth, visually distinct from Manga.
- Manga Composer becomes a high-contrast black/white command strip with boxed native action controls, paper input field, and inverted send control.
- Editorial Composer becomes a light writing-desk surface with dark serif ink, subdued toolbars, and a restrained circular send control.
- Recipe previews were updated so Manga and Editorial remain distinguishable even with labels hidden.

## v27.82 — Packpocalypse LXVII · Mask Lab

- **Image → Mask** becomes a first-class semantic control with four explicit modes: **Native** preserves the app/theme mask, **None** actively suppresses inherited masks, **Fade** keeps the friendly one-edge controls, and **Custom** exposes multi-edge mask layers without requiring users to author CSS.
- Custom masks can independently enable a side, top, and bottom fade, choose left/right for the side layer, tune **Solid until** and **Transparent by** with sliders, and combine layers with Intersect / Add / Subtract / Exclude. Theme Studio emits both standard `mask-*` declarations and the corresponding WebKit compositing operations.
- Read Style now recognizes supported native one- and multi-edge masks as observed Image/Mask state. Inspecting does not claim them; changing a mask control materializes only the semantic mask operation. Unsupported mask grammar remains Native rather than being falsely round-tripped.
- **Scene backdrop** now explicitly uses Mask → None on the real backdrop frame, neutralizing Lumiverse's native `_avatarBg_` mask stack so full-strength scene art is actually full-strength. Ghost backdrop intentionally keeps its atmospheric/native fade behavior.
- Existing Image packets remain backward-compatible: old directional fades still compile as Fade when no explicit mask mode exists. Schema remains **v32**.

## v27.81 — Packpocalypse LXVI · Scene Backdrop

- Added **Scene backdrop**, a louder sibling to Ghost backdrop that uses the same repaired frame/image/scrim anatomy but lets the backdrop portrait read as actual header art instead of atmosphere.
- Scene backdrop keeps neutral 50/50 framing and Auto source quality, then uses full image opacity, stronger contrast/saturation, and a lighter lower scrim so the artwork stays visibly present. Ghost backdrop remains intentionally dim and unchanged.
- Both backdrop basics now share safe BubbleMessage-relative geometry and matched Base/Mobile stage heights. No schema/compiler/pack changes; schema remains **v32**.

## v27.80 — Packpocalypse LXV · Ghost Backdrop Rehab

- **Ghost backdrop** is rebuilt against the real three-layer Bubble avatar-backdrop anatomy instead of trying to turn `_avatarBgImg_` into a full header by itself. The recipe now owns the backdrop frame, image, and scrim as separate semantic targets.
- The backdrop frame is pinned to the real `BubbleMessage` frame at the top of the message, given the same explicit height as the header, and the image fills that frame with a neutral 50/50 focal point. No `535%` width escape hatch or message-id-specific responsive boundary is needed.
- The native backdrop scrim becomes a controlled lower fade while the foreground avatar/name/meta stay native. Mobile gets a shorter matched header/backdrop height rather than inheriting desktop acreage.
- Added generic `avatar.backdrop.frame`, `avatar.backdrop.image`, and `avatar.backdrop.scrim` roles so non-pack basics can use the same proven backdrop anatomy as Packpocalypse without borrowing assistant/user pack recipes. Schema remains **v32**.

## v27.79 — Packpocalypse LXIV · Basics Shelf

- The Style Library gains a visible **Show pack-owned** switch. Pack recipe pieces remain discoverable by default, but can now be hidden from the individual-style shelves so the smaller independent basics are easier to browse. Pack collection cards remain visible either way.
- Pack-owned visibility is provenance-based: a style counts as pack-owned when its preset ID is actually referenced by a registered pack, rather than by family-name guessing.
- With four current packs, the pack shelf is now an intentional **2 × 2 desktop grid** instead of leaving Journal orphaned on a second row beneath three cards. Responsive layouts keep two columns on tablets and one on narrow phones.
- No recipe/compiler/schema behavior changes; schema remains **v32**.

## v27.78 — Packpocalypse LXIII · Surface Rail

- Style Library **Surface** moves out of the filter dialog and becomes a persistent pill rail beside the current browse heading: All, Messages, Prose, Avatars, Composer, and Global UI now switch immediately without opening another layer.
- The Filters button is renamed **More filters** and now contains only Message layout + Style family. Its badge counts only those hidden constraints, so choosing a visible surface no longer pretends there is a mystery filter active.
- Clearing More filters preserves the visible surface selection; the **All** surface pill is now the explicit way back to the whole library.
- The surface rail scrolls horizontally on narrow layouts instead of collapsing back into a modal. No recipe/compiler/schema behavior changes; schema remains **v32**.

## v27.77 — Packpocalypse LXII · Library Grown-Up Mode

- The full-screen Style Library gets a dedicated legibility pass: larger navigation, result chrome, search labels, section headings, recipe titles, badges, and workbench controls replace the inherited 6–9px microtype that made a giant workspace read like text for ants.
- Recipe and pack cards stop reserving the deleted description row. Library cards now size to preview + real metadata + actions instead of keeping an empty `minmax(0,1fr)` bedroom for copy that no longer exists.
- Pack workbenches now use the pack preview as a real full-width banner, with compact status overlaid on the art instead of wasting half the hero on a blank summary field.
- Pack sections become a real sticky navigation strip with section counts and scroll-aware active state. Clicking a section scrolls the pack canvas itself rather than relying on generic document `scrollIntoView()`.
- Visual Novel finally receives its own banner/thumbnail art direction: cinematic scene pane, framed dialogue window, VN jewel/heart language, and the pack palette instead of the generic default preview.
- The right-side workbench rail keeps the same functionality but grows readable type and touch targets. No recipe/compiler/schema behavior changes; schema remains **v32**.

## v27.76 — Packpocalypse LXI · VN Neutral Focal

- Visual Novel's assistant cinematic image no longer ships with an implicit upward focal bias. `assistant.avatar.image` now starts at **50% / 50%** on both Base and Mobile instead of 50/28 desktop and 50/25 mobile.
- Cover, Auto source quality, fill-frame behavior, and the existing tone treatment stay unchanged; the pack simply stops moving portrait art unless the user deliberately moves it with **Move inside frame**.
- A regression assertion now locks the assistant scene's Base and Mobile focal Y to neutral center.
- Schema remains **v32**. Reset/reapply **Cinematic dialogue stage** (or Reset Visual Novel → Apply All) to replace an already-mounted recipe layer.

## v27.75 — Packpocalypse LX · VN Backdrop Cleanup

- Visual Novel retires the assistant-side native **background-avatar sprite** entirely. The main mounted avatar/portrait already owns the cinematic hero art; the extra `avatarBg` frame/image/scrim are now explicitly hidden on desktop and mobile so the old secondary cutout can no longer photobomb the scene/dialogue seam.
- The optional Visual Novel **scene-art asset slot** now targets the real assistant portrait stage (`assistant.avatar`) instead of the retired background-avatar frame, so alternate scene art still has a live anchor.
- **Second speaker stage** gets a calmer user scrim: it now spans the full scene width but only the lower 62% (68% mobile), fading vertically from transparent at the top into a dark readability bed near the dialogue seam. This preserves the top of the user artwork instead of cutting across it with the old right-side opaque wedge.
- Schema remains **v32**. Reset Visual Novel → Apply All to replace the old assistant backdrop packets; reset Selected choice before mounting Second speaker stage as usual.

## v27.74 — Packpocalypse LIX · Second Speaker Content Hug

- Visual Novel's optional **Second speaker stage** finishes its dialogue-window geometry: the user dialogue mount now uses `fit-content` width/height with a bounded `48%–88%` desktop width, so short replies stop stretching across the cinematic stage while long replies still wrap safely. Mobile keeps a predictable `94%` width but remains content-height.
- The alternate's old centered `vn-corners` ornament is replaced by the same **wide top/bottom corner-cap system** used by the assistant dialogue frame. The top cap uses `vn-wide-corners`; the lower cap mirrors it with a 180° transform. Mobile swaps both to `vn-mobile-corners`, preventing aspect-ratio squash on narrow screens.
- The mount's real border remains the scalable enclosure, while the decorative caps now hug the actual dialogue box rather than rendering as a small bracket glyph in the middle of a much wider panel. The old heart-jewel pseudo is intentionally retired for this alternate so the two pseudo planes can own the full top/bottom frame pair.
- Schema remains **v32**. This release changes only **Second speaker stage**; Assistant and Selected choice remain untouched. Reset Selected choice before mounting the alternate.

## v27.73 — Packpocalypse LVIII · Second Speaker Stage Reblock

- Visual Novel's optional **Second speaker stage** gets a full composition rewrite after the forgotten alternate proved its old right-side identity geometry had drifted far behind Selected Choice and the assistant stage. The user message frame is now a centered `92%` cinematic panel with an explicit content-height Bubble and its own reserved lower HUD lane.
- The alternate's **Header is now ordinary flex layout** (`218px` desktop / `170px` mobile). HeaderLeft no longer anchors anywhere: it stays in flow as a `row-reverse` speaker cluster at the lower-right, so the portrait sits on the outside edge while name/identity text remains grouped immediately to its left. This intentionally avoids the containing-block/anchor saga that consumed the assistant speaker name.
- User metadata leaves the scene entirely and becomes a compact **bottom-left HUD capsule** owned by the user Bubble. BubbleActions get a matching **bottom-right micro-HUD**, including the same faint heart-underlay language as the rest of Visual Novel.
- The user dialogue plane is widened to `94%`, nudged upward only `14px` into the scene seam, and keeps its own VN corner/jewel treatment. Mobile switches to `96%` with smaller scene/identity/HUD geometry rather than scaling the desktop offsets.
- Schema remains **v32**. Second speaker stage is still an alternate recipe: reset **Selected choice** before mounting it. Assistant, Selected Choice, CG inserts, Thinking, and the shared screen furniture are unchanged by this release.

## v27.72 — Packpocalypse LVII · VN Speaker Translate Treaty

- Visual Novel's desktop **assistant speaker name** stops participating in the anchor/containing-block saga. Its Base Position packet now stays in natural layout and applies the Workbench-proven `translate: 0 -5rem`, with the existing custom layer retained. This matches the live edit that remains stable across normal long/short replies without making Header/HeaderLeft a desktop containing block.
- Mobile assistant name handling remains intentionally different: the existing **Header Left · Ancestor** anchor at `top: 130px` stays in place because that packet is already stable on narrow phones.
- **Second speaker stage** now explicitly owns visibility, Size, and a neutral Nudge Position packet for `user.name`, plus its mobile typography. It keeps its independent right-weighted HeaderLeft composition instead of borrowing Assistant speaker geometry, so the alternate character-vs-character user mode remains a first-class maintained path.
- Schema remains **v32**. For the assistant change, use **Reset Visual Novel → Apply All** so the old desktop anchored Position packet is replaced by the translate-based packet. Second speaker stage remains an optional alternate and should still be mounted separately after resetting Selected choice.

## v27.71 — Packpocalypse LVI · Exact Header Left Speaker Anchor

- Visual Novel now copies the Workbench-proven speaker-name anchor literally: the assistant name is anchored to the mounted `Header Left · Ancestor` selector (`[data-component="BubbleMessage"]:not([class*="_user_"]) [class*="_headerLeft_"]`) at `top: 180px` desktop / `130px` mobile.
- Route HUD metadata remains independently Bubble-owned; this pass intentionally changes no HUD, attachment, CG-lane, or dialogue geometry.
- Regression coverage now asserts the exact mounted Header Left ancestor selector rather than accepting a semantically similar pack role.

## v27.70 — Packpocalypse LV · VN Containing-Block Truce

- Visual Novel assistant speaker name no longer anchors to a positioned Header. The now-real Header remains a fixed scene-flow box, while the name returns to the full assistant Bubble as its absolute containing block at the same proven `top: 180px` desktop / `130px` mobile scene coordinates.
- This resolves a CSS containing-block collision introduced in v27.69: positioning Header for the name made the nested metadata pill resolve its own Bubble-relative `bottom` offsets against Header instead, causing metadata to jump into the scene.
- Header keeps its real `238px` / `176px` scene height, CG ordering and attachment ownership remain unchanged, and Route HUD metadata again shares the assistant Bubble containing block with the speaker name.
- Schema remains **v32**. Reset Visual Novel → Apply All to replace the v27.69 name Position packet and its Header anchor helper.

## v27.69 — Packpocalypse LIV · Header Is Real Again

- Visual Novel fixes the assistant speaker anchor at the actual layout-contract level: the assistant Header is no longer `display: contents`. It is a real full-width scene box matching the hero portrait height (`238px` desktop / `176px` mobile), so Header-relative absolute positioning finally has a containing block.
- HeaderLeft and MetaWrap remain dissolved, preserving the flexible VN identity/HUD composition without deleting the Header box the speaker name needs.
- The existing speaker coordinates (`top: 180px` desktop / `130px` mobile) are unchanged; this release makes those coordinates resolve against the intended Header instead of falling through to Bubble/CG/dialogue ancestors.
- CG attachment ordering/sizing from v27.67 remains unchanged. Schema remains **v32**. Reset Visual Novel and Apply All to replace the old Header Layout packet.

## v27.68 — Packpocalypse LIII · Exact Header Ancestor Speaker Lock

- Corrected the Visual Novel speaker-name anchor to the exact **Header · Ancestor** selector proven by mounted Workbench editing: `[data-component="BubbleMessage"]:not([class*="_user_"]) [class*="_header_"]`. The previous recipe used Theme Studio’s narrower `assistant.header` role selector, which looked equivalent in source but did not reproduce the stable live-editor positioning once attachments entered the stack.
- Speaker geometry itself is unchanged: `top: 180px` desktop and `130px` mobile, full-width centered Georgia name, custom layer `10`. The only change is ownership of the positioning context.
- Attachment/CG ownership from v27.67 is intentionally untouched — Cat is finally behaving.
- Schema remains **v32**. Reset Visual Novel → Apply All to replace the previous speaker Position packet.

## v27.67 — Packpocalypse LII · Header-Locked Speaker + Attachment Ownership

- Visual Novel assistant speaker names now anchor to the real **Header / speaker lane** rather than the broader Bubble. Mounted testing proved the Header-relative anchor stays glued to the hero scene even when attachment/reasoning content expands underneath it.
- Inline attachment sizing ownership moves to **Inline Image Btn**, the outer attachment control that actually participates in Lumiverse layout. It now uses Auto width + Fit height with **Attachments** as its responsive boundary; the inner frame/image remain presentation-only media surfaces.
- Assistant CG lane bottom clearance now explicitly absorbs the dialogue frame's authored negative Nudge (`-42px` desktop / `-32px` mobile). This preserves the intended dialogue overlap when no attachment exists without letting MessageContent bury the bottom of a CG insert.
- Schema remains **v32**. Reset Visual Novel → Apply All to replace the previous speaker anchor and CG sizing/clearance packets.

## v27.66 — Packpocalypse LI · VN CG Ordering + Inner Monologue

- Fixed the remaining Visual Novel attachment lane bug at the actual Bubble flex level. Assistant image attachments are mounted by Lumiverse in their own sibling content wrapper between Reasoning and MessageContent; Theme Studio now exposes that wrapper as **Assistant CG lane**, assigns it explicit flex order `3`, and moves the dialogue frame to order `4`. The stable stack is now scene → inner voice → CG insert → dialogue, so attachment media can no longer jump ahead of Thinking or steal the speaker-name band.
- Tightened **Assistant inner voice body** to the real ReasoningBlock DOM chain (`bodyWrapper → bodyInner → body`) instead of a broad descendant body-class fallback. This prevents unrelated BubbleMessage body elements from falsely inheriting the Inner voice recipe/provenance in Pick.
- Restyled the expanded VN reasoning body as a real **inner-monologue panel**: three-stop violet-black glass, faint grain, Georgia reading typography, stronger spacing, and an `INNER MONOLOGUE` generated caption. Mobile gets a slightly tighter reading scale without reverting to terminal-style prose.
- Schema remains **v32**. Reset Visual Novel → Apply All to replace the previous CG ordering and reasoning packets.

## v27.65 — Packpocalypse L · VN CG lane + speaker lock

- Visual Novel assistant speaker names are now anchored to the assistant Bubble at stable scene coordinates instead of living as translated flex items. Attachments can no longer push the speaker label down over an inline image.
- The Inner voice ribbon compensates for the name leaving normal flow, preserving the established scene → thought → dialogue spacing on desktop and mobile.
- Visual Novel **CG insert** now uses an explicit media canvas: 330×230px desktop and 236×176px mobile. Inline images fill that canvas with `object-fit: contain`, so arbitrary landscape/portrait attachments letterbox when needed instead of being cropped.
- The attachment lane gains deliberate top/bottom spacing so media reads as a separate VN insert rather than another speaker-layer element.
- Schema remains **v32**. Reset Visual Novel → Apply All to replace the old name-flow and attachment-size packets.

## v27.64 — Packpocalypse XLIX · VN Mobile Frame + CG Inserts

- Visual Novel fixes the compressed phone dialogue frame at the source: desktop keeps the wide `1000×64` corner-cap ornament, while Mobile uses a dedicated `400×64` VN cap. `background-size: contain` can now preserve real vertical corner geometry on narrow widths instead of mathematically collapsing the decoration to ~20px tall. Both top and bottom mobile caps own a 64px plane.
- Mounted inline media is now first-class Theme Studio anatomy: **Inline image button**, **Inline image frame**, and **Inline image** recognize the current `_inlineImageBtn_` / `_inlineImageWrap_` / `_inlineImage_` attachment family while retaining the broader attachment fallbacks. The public host still exposes no stable attachment `data-component`, so these remain medium-confidence DOM-scoped receipts rather than invented native IDs.
- Visual Novel gains **CG insert** and includes it in Apply All. Attachment containers are restored to normal centered flow, inline-image controls lose native layout surprises, media receives a violet glass frame, and image dimensions are capped independently for desktop and Mobile (`344×280` desktop; `270×230` mobile maximums) without forcing a crop.
- Schema remains **v32**. Reset/reapply Visual Novel to pick up the mobile frame and new attachment recipe.

## v27.63 — Packpocalypse XLVIII · Narrow-Phone VN Lock

- Visual Novel mobile assistant actions now use the live-editor values proven on narrow phones: `right:12px`, `bottom:11px`, Nudge `+2 / -47`. Desktop HUD packets are unchanged.
- Visual Novel **Continue route** now gets a mobile-only `-19px` Nudge so the long-message reveal tucks back toward the dialogue plane instead of opening a dead lane above the HUD.
- User/mobile RPG window geometry is unchanged from v27.62. Schema remains **v32**.

## v27.62 — Packpocalypse XLVII · Mobile Player Header + Packet-Locked Controls

- Visual Novel **Selected choice** keeps its now-stable desktop geometry and gets a phone-specific identity layout: the 50px portrait remains top-left while the user name becomes a longer 120px plaque beside it instead of stacking beneath the portrait and truncating names.
- The user BubbleActions Mobile Position packet now bakes the live-editor values that proved stable on phone: `right 22`, `bottom 13`, `nudgeX -3`, `nudgeY -12`. This deliberately uses the actual persisted Position packet contract rather than another viewport-relative approximation.
- Schema remains v32. Reset/reapply Visual Novel → Selected choice to pick up the Mobile delta; desktop stays unchanged.

## v27.61 — Packpocalypse XLVI · Absolute Player Identity + Ephemeral Disclosure + Continue Route

- Visual Novel Selected Choice corrects the player identity math: the live editor offsets from v27.59 were deltas layered over the existing anchored recipe, not standalone Nudge coordinates. The baked desktop positions now preserve the original choice-window anchor and add those deltas (`portrait 54/33`, `name 54/101`).
- Ephemeral Pick inspection now discloses the matching Theme Studio source immediately: the Style Stack banner lists the actual matching packet count, recipe/manual owners, and packet types, while the packet cards below remain bound to the real owning override.
- Visual Novel gains **Continue route**, a three-stop violet long-message reveal capsule for the native Read more control. It is included in Visual Novel Apply All.
- Schema remains v32.

## v27.60 — Packpocalypse XLV · Ephemeral Style Stack + Player Position Bake-In

- Design's Style Stack now follows actual CSS matching for mounted picks instead of requiring the picker's reusable selector string to be byte-for-byte identical to an authored Theme Studio override selector. If the selected DOM node matches additional Theme Studio selectors, their real packets are surfaced in the stack with their actual override/packet IDs and Pack/manual provenance.
- Matching authored packets remain fully editable: scrub previews and committed edits resolve the override that actually owns the packet rather than materializing a duplicate rule under the picker's alternate selector. A compact **Applied Theme Studio styles** banner calls out when extra authored selectors matched the current mounted node.
- This closes the gap where Generated CSS visibly contained pack rules for a picked `MessageContent`, avatar, or name while Design incorrectly reported **No normal-specific changes** simply because the resolver and recipe used different reusable selector forms for the same element.
- Visual Novel **Selected choice** bakes in the live-editor player identity placement from the successful manual mockup: user portrait Position becomes Nudge `+32 / +15`, and the name plaque becomes Nudge `+32 / +23` on desktop. Mobile retains its contained anchored layout.
- Schema remains **v32**. Reset/reapply Visual Novel → Selected choice for the baked identity coordinates; the Style Stack reader fix applies immediately.

## v27.59 — Packpocalypse XLIV · Player Window Polish + Pack Packet Provenance

- Visual Novel **Selected choice** gets its last containment polish: the portrait/name cassette moves a few pixels inward, the outer rim and `CHOICE SELECTED` tab now anchor to the user content mount instead of spanning the broader Bubble scaffold, and the user micro-action tray moves higher/inward into MessageContent's reserved control lane.
- Pack/recipe-owned packets now surface provenance directly in Design. Every active recipe-owned packet gets the same compact ownership badge family as sparse manual edits (for example `7 edited · Pack`), with the recipe name in the tooltip.
- Recipe provenance is presentation-only. Theme Studio deliberately does **not** populate `editedFields` on full recipe packets, because `editedFields` is the sparse-compiler contract and recipes sometimes intentionally author values equal to packet defaults in order to override native CSS.
- Schema remains **v32**. Reset/reapply Visual Novel → Selected choice for the user-window coordinate/rim changes; packet provenance appears immediately for existing persisted recipe layers.

## v27.58 — Packpocalypse XLIII · Player Window Content Ownership

- Visual Novel **Selected choice** promotes native `MessageContent` to the actual RPG window surface. The outer Bubble/content wrappers are now transparent layout scaffolding instead of competing nested boxes.
- User Header / HeaderLeft / MetaWrap dissolve with Layout → Contents. Portrait and name are independently anchored to the real user Bubble containing block, preventing native header geometry from dragging the identity pieces away from their cassette.
- The portrait and name plaque remain inside the left side of the player window, while MessageContent reserves their gutter and now also reserves a lower control lane. Short and long choices therefore share one stable silhouette.
- User BubbleActions move **inside** the choice window: they anchor to the Bubble bottom-right with positive inset instead of hanging below the content mount. Mobile uses the same containment model.
- `CHOICE SELECTED` and the outer rim now anchor to the user Bubble as the stable ancestor; the rim follows the whole window instead of depending on percentage pseudo height.
- Schema remains **v32**. Reset Visual Novel / Selected choice before applying so the old Header cassette and below-frame action coordinates are cleared.

## v27.57 — Packpocalypse XLII · Visual Novel Player Identity Contained

- Visual Novel **Selected choice** keeps the resurrected user portrait + name, but the identity module no longer hangs outside the left edge where native clipping can reduce it to a portrait sliver.
- The user Header now renders as an 84px framed identity cassette **inside** the choice window: compact portrait, attached name plaque, violet-black window chrome, and a restrained lift shadow.
- User choice prose now reserves a 118px desktop left gutter for that cassette. Mobile switches to a top-left identity cassette with vertical text clearance instead.
- The choice frame gains a minimum height so very short user replies cannot collapse underneath the portrait/name module. Long replies still grow normally with content.
- User metadata remains intentionally hidden in the Selected choice variant; this is still a player command window rather than a miniature assistant stage.
- Schema remains **v32**. Reset Visual Novel / Selected choice before applying so the old outside-left Header position is fully cleared.

## v27.56 — Packpocalypse XLI · Player Choice Window

- **Visual Novel → Selected choice** is rebuilt as a compact RPG-style player response window instead of a mostly anonymous framed bubble. The user avatar and speaker name are intentionally resurrected as a small portrait token + attached name plaque that hang from the left edge of the choice card.
- User metadata stays suppressed in this variant: the revived header exists only to carry identity, preserving the deliberate Assistant = cinematic stage / User = command-window distinction.
- The user choice body gets a stronger three-stop violet glass surface and a real double-frame treatment: the native card border owns the inner enclosure while the `::before` plane becomes a separate outer rim that scales with arbitrary message height. `CHOICE SELECTED` remains the state tab on the upper-right edge.
- BubbleActions now dock to the lower-right edge of the choice frame as a dedicated micro-control tray. User action buttons get the same faint heart-underlay vocabulary as the assistant HUD through a new **User system heart** pseudo role, but stay smaller and pink-weighted so the player side remains its own UI species.
- Desktop Selected Choice widens slightly and reduces its historical rightward shove so the protruding portrait has real composition space. Mobile moves the portrait into the card's upper-left and increases top text inset instead of allowing the identity token to collide with prose.
- Schema remains **v32**. Use **Reset Visual Novel → Apply All** (or reset/reapply Selected choice) because the user Header/Avatar/Name surfaces are newly owned by the default user recipe.

## v27.55 — Packpocalypse XL · Compact Cast Rail

- **Visual Novel → Route roster** now owns explicit compact geometry instead of inheriting the host member card/image dimensions. Member tiles are 92×76px on desktop (78×68px mobile), with 46px/40px portrait tokens and a matching compact add-member tile.
- The roster rail/wrapper explicitly collapse to content height and keep a horizontal flex layout, preventing character images from expanding the member strip into full-size portrait panels.
- The active-speaker glow and VN glass treatment remain intact; this pass only constrains the roster silhouette.
- Schema remains **v32**. Use **Reset Visual Novel → Apply All** (or reset/reapply Route roster) to replace the previous unconstrained member/avatar packets.

## v27.54 — Packpocalypse XXXIX · Visual Novel Screen Furniture

- Visual Novel finally expands beyond message cards. **Dialogue composer** themes current staging's split InputArea contract: native `ComposerActionBarLive`, the separate `chat_toolbar` extension rail, their actual controls, the textarea/placeholder, send control, and the scroll-to-bottom button now share the pack's violet-black VN system language.
- Added first-class **GroupChatMemberBar** recipe roles for the mounted character roster: wrapper, rail, member card, active member, member avatar/name, and add-member control. Selectors are scoped under `MessageList` and follow current staging's `barWrapper / bar / member / memberActive / avatar / name / addMemberBtn` CSS-module anatomy so generic class names do not leak across the app.
- The Visual Novel pack now advertises Composer + Global furniture areas and Apply All includes **Route roster** + **Dialogue composer** after the assistant/user message core. One-to-one chats simply no-op the roster recipe when GroupChatMemberBar is not mounted.
- Schema remains **v32**. No project migration. Existing Visual Novel mounts should Reset/Apply All if you want the newly owned screen furniture.

## v27.53 — Packpocalypse XXXVIII · ComposerActionBarLive Compatibility + VN HUD Lock

- InputArea anatomy now follows current staging's split composer contract. The native action bar stays on the real `_actionBar_` box, while reorderable native actions gain semantic roles through stable `data-composer-action` + `data-toolbar-action` wrappers. Those wrappers are treated as identity scopes only because upstream renders them with `display: contents`; visual control recipes target their descendant buttons/icons.
- Added separate **Extension toolbar** and **Extension toolbar controls** roles for the dedicated `chat_toolbar` Spindle mount, with `_extensionToolbar_` as the visual-root selector and `[data-spindle-mount="chat_toolbar"]` as the semantic mount fallback. Native and extension toolbars are no longer assumed to share one DOM box.
- Existing `input.actionbar.controls` remains a compatibility role, but now prefers the stable composer-action wrappers before falling back to the historical `_actionBar_` descendant selector. Manga, Editorial, and Journal composer recipes also style the extension toolbar alongside the native bar; Journal mirrors its control ink into extension controls.
- Picker/target resolution recognizes composer-action wrappers as high-stability semantic boundaries. Picking a native toolbar button can therefore resolve through e.g. `[data-component="InputArea"] [data-composer-action="home"][data-toolbar-action="home"] button` instead of depending on translated `title`/`aria-label` copy. Browse Inside labels these wrappers as `Composer action · …` and identifies the `chat_toolbar` extension mount explicitly.
- Visual Novel BubbleActions lock to the live-editor alignment that actually centered the HUD: **Nudge X -9px / Y -13px** at the existing right/bottom anchor. This replaces the incremental pixel guesses from v27.51–v27.52.
- Schema remains **v32**. No persisted-project migration. Composer recipes should be Reset/Apply All if an older pack layer was authored against the pre-split InputArea.

## v27.52 — Packpocalypse XXXVII · VN HUD Pixel Lock II + Inspector Anatomy Split

- Visual Novel BubbleActions move the final **2px upward** on desktop (1px on Mobile) while metadata and the route pager remain untouched. This is the last HUD baseline correction over v27.51.
- **Edit Part** and **Browse Inside** are deliberately distinct again. Edit Part follows the native component family owned by the currently edited target/ladder rung and can expose sibling/mounted CSS-module parts across that component. Browse Inside follows only the exact mounted descendants of the current target.
- Native selections now retain Edit Part inventories for **all discovered native component contexts** in the target chain, not only the nearest component. Moving between component-bearing ladder rungs can therefore swap the part catalog instead of leaving the original list frozen.
- Browse Inside now roots itself at the **current edited target**, including sibling parts reached through Edit Part, instead of remaining pinned to the originally picked element.
- Schema remains **v32**. No persisted-project migration.

## v27.51 — Packpocalypse XXXVI · VN HUD Pixel Lock

- Visual Novel's route pager moves **3px lower** and gets a slightly brighter read so the arrows/counter visually meet the metadata baseline instead of floating a hair above it.
- BubbleActions moves **3px higher** on desktop (2px on mobile), converging on the same HUD centerline without changing the rail, metadata, or button styling.
- No anatomy, engine, schema, or pack architecture changes. Schema remains v32. This is deliberately the tiny final HUD alignment pass over v27.50.

## v27.50 — Packpocalypse XXXV · VN HUD Marriage + Wider Frame

- Visual Novel's lower HUD gets the final alignment pass: the shared rail is more visible, meta/actions sit on its true centerline, and the native route pager receives a small visual correction so its actual arrows/counter land on the same baseline instead of hovering one control-height above the others.
- The `vn-wide-corners` ornament itself is widened rather than inflating its host box. Both upper and lower corner arms now reach substantially farther into the dialogue plane while remaining fixed-height and safe for arbitrarily long replies.
- The assistant bubble reserves only the HUD space it actually needs, trimming a little of the black no-man's-land between the dialogue frame and the system rail.
- Schema remains v32. Recipe/ornament tuning only. Use **Reset Visual Novel → Apply All** when testing over v27.49.

## v27.49 — Packpocalypse XXXIV · Visual Novel HUD Rail + Wide Frame

- Visual Novel's three lower controls now share one authored **HUD rail** instead of rendering as three unrelated floating capsules. The Bubble owns a single dark-violet rounded rail; metadata sits left, SwipeControls spans the rail and centers its native pager, and BubbleActions sits right. Individual meta/swipe/action shells are transparent so the system reads as one component.
- Added a semantic **Assistant HUD rail** pseudo target on the assistant Bubble (`::after`) so future packs can reuse the same composition without inventing a DOM wrapper.
- The assistant dialogue plane grows from **90% to 96%** width with a larger max-width, and both VN corner caps use a taller mask plane so the wide corner artwork can occupy the full frame instead of shrinking inward to preserve its SVG aspect ratio.
- Schema remains **v32**. Reset Visual Novel → Apply All is recommended because Route HUD and dialogue-frame Size packets changed.

## v27.48 — Packpocalypse XXXIII · Studio UX Debt + VN System Pass

- Visual Novel's lower **Route HUD** now reads as one control family instead of three unrelated widgets: metadata, route pager, and BubbleActions share the same 30px dark-violet capsule, border strength, lift, and baseline. Action buttons flatten inside that parent capsule while their heart motifs remain local to the controls.
- VN dialogue ornamentation is split into a true **upper corner cap + lower corner cap**. The lower cap reuses the same wide-corner stencil rotated 180° rather than stretching one full-height frame through long prose. The decorative heart moves off the clipped dialogue pseudo plane and becomes a separate speaker-name jewel.
- The secondary/background portrait gets a quieter authored seam position and stronger fade so it behaves like a supporting VN sprite rather than an accidental card peeking from the left edge.
- Theme Studio gains first-class **attachment discovery** for mounted BubbleMessage attachment groups, items, and images. Lumiverse does not currently expose a guaranteed public attachment component id, so these targets are marked as mounted semantic fallbacks rather than pretending the selector is stronger than it is. Pick/Browse Inside now calls them `Attachment` / `Attachment image` instead of generic div/image furniture.
- Message-side editing now survives target-controller jumps such as **User → Edit avatar**. Theme Studio preserves the current Assistant/User facet when the destination scope has a matching side variant instead of silently falling back to the general/Both avatar rule.
- Background and text gradients can now author **2–6 stops** directly in Design. Existing pack-authored third stops are finally editable: each stop can be removed while at least two remain, and **Add stop** inserts a midpoint into the widest interval.
- **Layout Item** now identifies its real parent, current display mode, and parent selector. The non-flex/grid warning includes an **Edit parent** action that resolves the mounted parent into the ordinary Theme Studio editor rather than merely telling you the layout is unsupported.
- The compact floating widget can be completely hidden with a **650ms long-press** on its collapsed launcher. The hidden state persists locally; opening the widget from the Theme Studio sidebar explicitly re-enables it.
- Instance-specific selectors that rely on dynamic `title=` or `aria-label=` values now surface a **Use generic** suggestion when Theme Studio already knows a reusable persistent selector for the same mounted element. This makes dynamic member/button groups editable without forcing a detour through Edit Part.
- Schema remains **v32**. These are editor UX, semantic discovery, and Visual Novel recipe changes; no persisted-project migration is required. For the VN visual changes, use **Reset Visual Novel → Apply All** so the old HUD/corner/sprite packets do not survive provenance.

## v27.47 — Packpocalypse XXXII · VN HUD Dock + Portrait Blocking + Corner Cap

- Visual Novel's assistant dialogue ornament stops pretending to be a miniature full frame. The **VN wide corner cap** now contains only the upper left/right bracket language, uses a shorter fixed-height pseudo plane, and stays decorative even when the response is thousands of tokens long.
- The native background-avatar layer is now an authored **secondary portrait sprite** instead of an undecided floating card. It docks at the scene/dialogue seam on desktop, grows into a taller cutout, fades toward the dialogue plane, and uses its native scrim as a lower dissolve. Mobile still hides the secondary sprite to protect the reading width.
- **Anchored Position now resets unauthored edges to `auto`**. A bottom/right anchor therefore clears stale native top/left coordinates instead of over-constraining the element. This fixes Visual Novel BubbleActions refusing to leave their native top-right scene position and makes Anchored semantics safer for any component that already ships positional offsets.
- The Route HUD keeps metadata left, swipes centered, and BubbleActions right in the reserved lower system lane; the action recipe itself remains content-sized and heart-decorated.
- Schema remains **v32**. Test with **Reset Visual Novel → Apply All** because the system-controls Position packet and assistant secondary-portrait/frame planes changed.

## v27.46 — Packpocalypse XXXI · Visual Novel Mockup Distilled + Layout Contents

- Schema v32 adds **Layout → Contents / dissolve wrapper** (`display: contents`). It removes only the wrapper's layout box; its real children stay in the DOM and participate in the parent layout. Visual Novel uses this to dissolve BubbleMessage's structural Header / HeaderLeft / MetaWrap wrappers without reparenting React-owned nodes.
- Visual Novel's assistant stage now follows the successful off-camera mockup without its one-message-only hacks: the regular avatar becomes a wide fixed-height scene image, the native avatar-background layer becomes a secondary faded portrait cutout, the speaker name overlays the scene via Flex order + Nudge, and Thinking docks at the scene/dialogue seam.
- The assistant dialogue frame is content-height safe for long replies. The old full-height `VN corner brackets` surface is replaced by a new **VN wide corner cap** so decorative geometry stays compact instead of stretching down thousands of pixels with long prose.
- The lower Route HUD now reserves its own Bubble padding and pins metadata / swipes / heart-action controls against the real Bubble bottom, so short and long replies share one bottom-system grammar instead of relying on viewport heights or `translate: 25rem` mockup offsets.
- The secondary portrait is hidden on Mobile; the primary scene, centered name, thought seam, dialogue, and HUD remain responsive.
- Because schema and structural layout ownership changed, test with **Reset Visual Novel → Apply All**.

## v27.45 — Packpocalypse XXX · Anchor Containment + VN Stage Blocking

- **Anchor containment no longer overrides an anchor's own authored Position mode.** Generated anchor helpers now use a deliberately low-specificity `:where(...) { position: relative; }` fallback instead of an authoritative rule. Nested anchors can therefore use an already-absolute/sticky/fixed parent without silently turning it back into a flow-relative box.
- Visual Novel's assistant scene stops nesting anchored Header/Scrim children through positioned parents. The 190px cinematic backdrop remains genuinely absolute, while the Header intentionally owns a 174px normal-flow scene lane and bottom-aligns the portrait/name identity row inside it.
- The Inner voice ribbon no longer reserves a 154px spacer. It follows the scene lane naturally and uses an 8px visual Nudge upward into the scene/dialogue seam; mobile uses the same choreography with a 6px nudge.
- This is the first Visual Novel pass where the vertical composition is intentionally simple: **scene lane → thought ribbon → dialogue plane**. Existing HUD/meta/swipe/action positioning remains unchanged.
- Schema remains **v31**. Because the compiler behavior for nested Anchors changed, use **Reset Visual Novel → Apply All** when validating this build.

## v27.44 — Packpocalypse XXIX · Visual Novel Composition Pass

- Visual Novel moves past selector/geometry triage into an actual composition pass. The assistant scene banner is shortened, the header becomes a true overlay instead of reserving another full-height block, and the identity lane is pulled into the lower-left scene zone.
- The assistant speaker name now explicitly owns visibility, inline layout, content sizing, and stronger VN typography so native name presentation cannot silently collapse it.
- The inner-voice block stops relying on a large negative Nudge. It now reserves deliberate flow space at the scene/dialogue seam, making the thought ribbon part of the cinematic stack rather than a displaced native control.
- The dialogue frame keeps a small visual overlap with the scene via Nudge, but otherwise grows with real response content. Scene backdrop height is reduced to 190px desktop / 150px mobile, with matching scrim proportions and tighter frame padding.
- Schema remains v31. This is pack recipe composition tuning only; test with Reset Visual Novel → Apply All so the earlier geometry packets do not survive provenance.

## v27.43 — Packpocalypse XXVIII · Visual Novel Exact Anatomy

- Visual Novel stops discovering BubbleMessage anatomy by broad CSS-module substring search and now follows the host's real direct DOM tree. Assistant/user frames use Lumiverse's native `data-part` facet; main bubble, header, identity lane, dialogue wrapper, MessageContent, reasoning, and SwipeControls are scoped through explicit parent → child paths.
- The VN dialogue frame no longer risks confusing BubbleMessage's outer `_content_…` wrapper with MessageContent's independent `_content_…` class family. Only the direct BubbleMessage content wrapper that directly owns `data-component="MessageContent"` receives dialogue-frame geometry.
- Assistant stage explicitly owns the message card as Block layout and the real main bubble as a single Flex Column. This removes the remaining native row/flex ambiguity that could squeeze the dialogue wrapper into a microscopic column and produce enormous wrapped height.
- Inner Voice now targets the exact direct ReasoningBlock mounted beneath the main BubbleMessage bubble (`container + bubble → reasoning toggle → body`) instead of matching arbitrary `_container_` / `_bubble_` descendants.
- User Selected Choice keeps its existing visual treatment but inherits the same exact `data-part="user"` and direct-content targeting. The optional Second Speaker recipe therefore shares the corrected anatomy without copying assistant geometry.
- Schema remains **v31**. This is selector/anatomy hardening plus layout containment; no migration. Test with **Reset Visual Novel → Apply All** so XXVII's fuzzy structural targets do not remain in provenance.

## v27.42 — Packpocalypse XXVII · Visual Novel Geometry Containment

- Visual Novel’s first live mount exposed native BubbleMessage wrappers that retain percentage-height/stretch contracts. The assistant card, MessageContent, and BubbleActions could therefore participate in a circular `height: 100%` layout and expand to ~175,000px. VN structural roles now explicitly own `fit-content` or fixed heights instead of assuming native auto-height behavior.
- Assistant backdrop/header geometry is deterministic: the scene is 232px on desktop / 176px mobile, the native scrim is forced into a rectangular lower-left plane, and the identity lane is placed against that known scene geometry.
- Dialogue overlap now uses **Nudge** (`translate`) rather than negative margin. Theme Studio spacing intentionally clamps negative margin values, so the previous recipe never actually authored its requested overlap.
- Dialogue frame, MessageContent, reasoning shell/toggle/body, route metadata, swipes, and BubbleActions now explicitly collapse to content height where appropriate, preventing native stretch rules from turning HUD furniture into page-height columns.
- Selected Choice receives the same containment pass so the default user-side role cannot inherit accidental full-height geometry.
- Schema remains **v31**; this is recipe-level geometry correction only. Reset Visual Novel and Apply All before testing so the original VN height packets do not remain in provenance.

## v27.41 — Packpocalypse XXVI · Visual Novel First Stage

- Adds the first **Visual Novel** pack family for BubbleMessage. Its default composition deliberately gives Assistant and User different jobs instead of mirroring one generic card.
- **Cinematic dialogue stage** turns the assistant avatar backdrop into a tall full-width scene plane, keeps a small foreground portrait token, uses the native backdrop scrim as a heavy lower-left identity plate, and overlaps a dark glass dialogue frame onto the scene.
- Dialogue framing gets dedicated Visual Novel ornament vocabulary: **VN corner brackets** and a small **heart jewel** ride generated pseudo planes instead of touching MessageContent descendants.
- **Inner voice ribbon** pulls native Thinking/Reasoning into the seam beneath the speaker identity and above dialogue. The live toggle remains native; expanded reasoning becomes a quieter inset panel.
- **Route HUD** relocates the real metadata pill to the lower system lane, turns native meta-dot separators into heart stencils, brings SwipeControls into the same lower HUD, and styles BubbleActions as compact system buttons with heart ornaments.
- User messages ship with two mutually exclusive recipes: **Selected choice** (the default compact player-choice card) and **Second speaker stage** (an alternate mirrored cinematic character presentation). Apply All chooses Selected choice; reset it before applying Second speaker if switching an already-mounted project.
- Adds side-specific Bubble semantic targets for assistant/user scene art, scrim, identity, dialogue, metadata, actions, and reasoning so the pack does not rely on accidental shared selectors.
- Schema remains **v31**. This is a first visible Visual Novel composition pass; MinimalMessage is intentionally not claimed yet.

## v27.40 — Packpocalypse XXV · Reasoning Container Scope + Private Notes Tab Rescue

- Journal now absorbs the useful part of the manual reasoning-container tweak **semantically**: the Private note uses a responsive `60%` desktop width capped at `590px`, a tiny 2px shell inset, and centered layout-item alignment instead of a broad `[class*="_container_"]` override.
- The reasoning memo gets a clearly intentional **42px desktop bottom gap** before the scrapbook mounting board, so it reads as its own note rather than hovering one accidental pixel above the main spread. Mobile uses a smaller 24px gap and full-width flow.
- **PRIVATE NOTES** moves from the native inner toggle pseudo to the outer reasoning shell pseudo. This keeps the generated tab independent from native toggle geometry, preventing the little label from being clipped/swallowed when the reasoning container is resized or padded.
- The tab itself gets slightly more padding, stronger edge/shadow, and a higher local layer so it reads like a physical label tucked over the note edge.
- Remove old custom rules targeting generic BubbleMessage `[class*="_container_"]` descendants before testing; Custom CSS loads after generated pack CSS and will intentionally override these recipe values.
- Schema remains **v31**. No migration; this is semantic-target and recipe cleanup.

## v27.39 — Packpocalypse XXIV · Thought Docking + Shadow Triage

- Journal's **Private note** now stops hovering in awkward limbo between the header collage and the notebook spread. The reasoning memo gets an explicit top margin and a slight negative bottom margin so it reads as intentionally docked into the main page instead of almost-touching it by accident. Mobile still resets to a normal non-overlapping flow.
- The brown **content mounting board** keeps its dotted faux-table treatment, but its shadow is trimmed down so the board still reads lifted without creating a muddy halo around the composition.
- **MessageContent** loses the giant ambiguous dark bloom and now uses a smaller, clearer paper shadow. The notebook page still has depth, but the source of the shadow is readable again instead of feeling like a black cloud under the spread.
- Schema remains **v31**. This is recipe tuning only: no new roles, no new migration. Because both the notebook page and private-note packets changed, test with **Reset Journal → Apply All** if older overrides were already baked into the project.

## v27.36 — Packpocalypse XX · Journal Filing Desk Cleanup

- Journal's **Filing ledger** no longer relocates the native metadata pill to the BubbleMessage frame or reserves artificial space beneath MessageContent. The real meta pill stays inside the existing identity stationery card and becomes a compact three-column `ENTRY / FILED / LENGTH` strip around the live native values.
- The identity card widens and the Bubble header sheds excess bottom padding, keeping the polaroid + name + filing strip as one readable collage instead of letting metadata overwhelm or clip the speaker identity.
- Ledger cells lose their individual specimen-box borders in favor of a shared outer filing sheet with faint per-cell tint, stronger live-value ink, and tighter generated labels.
- **Private note** now explicitly styles the real reasoning toggle's ink/opacity, gives the folded memo a deliberate fixed desktop width, stronger paper edge/shadow, and a faint ruled open body. The generated `PRIVATE NOTE` caption remains static while the thought duration stays native.
- Journal BubbleActions remain native-position-owned Nudge, but their Base offset is tightened from `-5 / +161` to `-12 / +138` so the utility strip sits closer to the identity collage instead of floating in the header acreage. Mobile still explicitly resets to `0 / 0`.
- Schema remains **v31**. This is mounted composition cleanup using existing semantic targets, Generated Content, Flow Centering, Nudge, Pattern, Transform, and provenance. Because the Filing ledger recipe dropped old owned Position/MessageContent-spacing packets, test with **Reset Journal → Apply All**.

## v27.35 — Packpocalypse XIX · Generated Content + Filing Ledger + Private Note

- Schema v31 adds **Generated Content**, a literal pseudo-surface label primitive under Typography. It compiles safe escaped `content: "…"` strings for `::before` / `::after`; it does not invent runtime values or introduce a token language.
- BubbleMessage and MinimalMessage metadata anatomy now exposes the three live `_metaSegment_` siblings as semantic **Message number / Timestamp / Token count** targets, plus their dot separators and generated-label pseudo surfaces. The values remain native Lumiverse text.
- Journal adds a Bubble-only **Filing ledger** that editorializes the live message metadata into three labeled cells: **ENTRY / FILED / LENGTH**. Generated captions are static art direction while `#94`, the timestamp, and token count remain real DOM values.
- Journal Thinking becomes **Private note**: a compact crooked memo with a generated `PRIVATE NOTE` caption, the native thought duration retained as live metadata, and the open reasoning body styled as the unfolded paper.
- Journal Apply All includes the filing ledger and Private note recipes. This establishes reusable metadata/content vocabulary for later Manuscript, Archive/HUD, Gothic, Mode, and other composition families.

## v27.34 — Packpocalypse XVIII · Flow Centering + Ink Shadows

- Schema v30 adds **Position → Flow placement → Center block**. It compiles `margin-inline: auto` for Flow/Nudge without assuming a flex/grid parent, so capped-width Journal sheets can center in ordinary document flow.
- Journal Bubble notebook sheets now use Flow centering on Base; Mobile keeps the centered baseline while naturally filling the narrower parent.
- Journal paper, portrait, identity-note, and pasted-ephemera shadows move from barely-visible green haze to darker neutral ink-gray depth with a restrained teal undertone.
- Pasted ephemera receives stronger host/backing shadows while retaining isolated negative-layer backing and positive-z tape.

## v27.33 — Packpocalypse XVII · Centered Sheets + Physical Ephemera

- Journal Bubble now treats the notebook body as the sole reading sheet when no sidecar occupies its row: MessageContent requests a parent-width/max-760 sheet and centers itself with Layout Item alignment on desktop, then returns to stretch behavior on Mobile.
- The notebook sheet shadow was softened into a broader lifted-paper shadow so the centered sheet reads as a physical page rather than a hard offset card.
- **Pasted ephemera** gained a second physical depth pass: the island host now has its own ambient lift, the dark backing has a stronger short paper shadow and slightly richer teal gradient/grain, and Mobile reduces the lift while straightening the clipping.
- Schema remains **v29**. This pass uses existing Size, Layout Item, Shadow, Transform, and pseudo-surface semantics; no new persistence primitive was needed.

## v27.32 — Packpocalypse XVI · Pasted Ephemera + Safe Negative Layers

- Journal now treats regex/custom HTML islands as **Pasted ephemera**: arbitrary island markup keeps its own internal colors while a dark scrapbook backing sheet lives on `::before` and crooked washi rides `::after`.
- Added semantic HTML-island Back/Front ornament roles. Journal Apply All includes Pasted ephemera, with Base/Mobile spacing and rotation so foreign cards read like clippings instead of unreadable white-on-cream embeds.
- Negative pseudo planes are now compiler-isolated automatically: when a generated `::before`/`::after` surface owns a custom negative z-index, Theme Studio adds `isolation: isolate` to the real host. `z-index: -1` therefore means “behind this target's contents,” not “escape behind some ancestor and pray.”
- Journal's notebook content recipes explicitly own `opacity: 1`; softness stays in individual background/text alpha so embedded HTML can remain truly opaque.
- Schema remains **v29**. Isolation is a compiler safety invariant for negative pseudo surfaces, not a new persisted style primitive.

## v27.31 — Packpocalypse XV · Nudge Means Nudge

- Fixed Position → Nudge so it preserves the target's native positioning mode and emits only `translate` (plus requested z-index).
- Fixes Journal BubbleActions turning into a giant stretched paper slab when Nudge overrode Lumiverse's native absolute `.actionsPill` positioning with `position: relative`.
- Zero-valued responsive Nudge still emits `translate: 0 0`, so Mobile can reset Base movement without taking ownership of positioning.
- Added compiler regression coverage for native-position-safe Nudge.
- Schema remains v29.

# v27.30 — Packpocalypse XIV: False Anchor Exorcism

- Reverted Journal BubbleActions from the v27.29 header anchor after mounted testing proved BubbleActions is visually over the header but not safely owned by that header's containing block. Anchoring changed layout semantics and could stretch the control into a full-height side slab.
- Journal **Paper actions** now stays in BubbleActions' native layout context and uses the proven responsive Nudge path (`-5px / +161px` on Base, explicit `0 / 0` on Mobile) to clear the portrait and corner ornament without reparenting or inventing ancestry.
- Position/Nudge now always emits its `translate` baseline, including `0 0`, so a Mobile recipe can explicitly neutralize a Base nudge instead of inheriting it. Anchored positioning keeps its own zero-translate protection as before.
- Schema remains **v29**. This is a recipe/position-semantics correction over existing packets and provenance.

# v27.29 — Packpocalypse XIII: Journal Utility Shelf

- Promoted ChatView's stable `button[aria-label="Scroll to bottom"]` into a semantic **Scroll to bottom** target and a Journal **Page-down stamp** recipe.
- Replaced Journal Bubble's manual giant-Y translate workaround for BubbleActions with a real anchored home: the header now reserves a utility shelf and the action pill anchors to its bottom-right corner on Base/Mobile. Anchored Position now emits an explicit zero translation when unnudged so stale/manual Nudge cannot keep offsetting the newly anchored element.
- Folded the successful mounted Journal tweaks back into the curated recipe: ruled notebook-paper stripes, harder lifted-sheet shadowing, a more physical polaroid shadow, and a stamped metadata shadow.
- Schema remains **v29**. No new persistence primitive was needed; this pass uses Position, Pattern, Shadow, Size, Transform, and existing semantic target plumbing.

# v27.28 — Packpocalypse XII: The Journal Demands Rotation

- Added schema/state **v29** with a first-class **Transform** semantic packet. Transform owns friendly Rotate, linked/unlinked Scale, and Skew X/Y intent instead of forcing users into raw CSS.
- Rotate and Scale compile through modern individual CSS properties (`rotate`, `scale`) so they compose cleanly with Position & Layer's existing `translate`. Skew uses `transform: skew(...)` and the UI warns that it owns the legacy transform property.
- Added Transform to Design's Shape styles with quick ±5/±2/0 rotation presets, exact slider/number controls, linked scale, advanced skew controls, and one-click reset. Base/Mobile authoring and recipe provenance work through the normal packet stack.
- Journal immediately uses the new primitive: Bubble polaroids tilt/scale slightly, the stamped identity note sits subtly off-axis, Minimal pins lean the opposite direction, and Washi tape is visibly crooked. Mobile recipes calm those rotations back toward zero.
- Transform is available for shared Group Members as well, so repeated sibling cards can receive a coordinated visual tilt without custom CSS or DOM wrappers.

# v27.27 — Packpocalypse XI: Scrapbook Contrast + Real Temper

- Journal gets a visibly stronger composition pass instead of another low-contrast material tweak. Bubble now has a mint-washed header sheet, a stamped identity note, a heavier polaroid lift, an inset notebook reading sheet with a left margin rule, and a star-divider ornament. Minimal gets the same vocabulary in a compact pinned-card form with its own header note, inset reading sheet, margin rule, and divider.
- Added unclipped **Header ornament** / **Minimal header ornament** pseudo roles. Journal's Washi photo recipe now staples tape to those header planes instead of the avatar wrapper, so tape can visibly overlap the portrait even when native avatar overflow clips its own pseudo-element.
- Journal's default corner decoration is now bolder stationery: a postage-star mark on Bubble and a small flower on Minimal. The page keeps user-swappable ornament slots, but Apply All now has enough visual punctuation to read as scrapbook at a glance.
- Stationery Composer contrast is pushed harder: stronger mint/cream gradient and grain, an explicit toolbar paper chip, full-opacity dark toolbar controls, a more opaque writing field, and a darker forced placeholder with opacity 1.
- Replaced Manga's hand-drawn Temper approximation with the actual U+1F4A2 anger-symbol silhouette converted to a monochrome stencil path. It now reads as the familiar four-lobed anime anger vein instead of a sparkle/crosshair.
- Persistence remains **schema v28**. New ornament surfaces are semantic role vocabulary only; all composition still compiles through existing Background/Stencil/Position/Size packets and recipe provenance.

# v27.26 — Packpocalypse X: Paper Depth + Backdrop Exorcism

- Journal now suppresses the complete Bubble avatar backdrop stack proven by mounted receipts: `_avatarBg_`, its nested image, and `_avatarBgImg_` are targeted together with semantic Visibility Gone. The cream page no longer leaves a native dark scrim/vignette behind.
- Journal's Bubble and Minimal leads now use authored warm paper gradients, low-alpha grain, deeper card/photo shadows, and stronger washi presence so the family reads as layered stationery rather than a flat cream slab.
- Stationery Composer now owns readable toolbar controls and the textarea placeholder in addition to shell/text/send. The shell gets a paper gradient + grain, the field gets darker ink and a clearer inner surface, and the send control gets stronger stationery contrast.
- Added narrow semantic roles for **Composer toolbar controls** and **Composer placeholder**. The selector compiler now treats `::placeholder` like other pseudo-elements when applying Strong authority, keeping the pseudo-element after Theme Studio's specificity guards.
- Redrew Manga's built-in **Temper mark** into a proper four-lobed anime anger-vein stencil instead of the previous broken-spark silhouette.
- Persistence remains **schema v28**. This pass is selector/recipe/compiler polish over existing semantic packets and provenance.

# v27.25 — Packpocalypse IX: Cream Ink + Utility Strips

- Journal now treats its cream paper / dark ink relationship as authored composition intent. Recipe steps may opt out of pack-palette Text remapping with `tuneText: false`; Journal uses that on frame/content/prose/reasoning/chrome/composer ink so the cream family stays readable even when the workbench accent changes. Accent-driven borders and ornament Stencils remain refinable.
- Journal Bubble explicitly suppresses the native avatar backdrop image, removing the dark vignette/scrim that survived behind the paper composition. Minimal gets the same fixed-ink philosophy across its frame, identity, prose, and controls.
- Added narrow semantic **action-control** roles for BubbleActions and Minimal actions descendants. Journal's new **Paper actions** recipe now owns the cream action shell plus button/SVG foreground ink instead of relying on native white icons to inherit politely.
- Added **Journal · Page pager**, a dedicated fixed-ink swipe treatment used by Journal Apply All instead of the shared palette-tuned compact pager.
- Manga's Bubble lead now reserves a shallow **utility strip** above the portrait. Actions are anchored into that band with room left at the far corner for Temper, preventing the action row from overlapping either the hero image or the ornament. Mobile gets its own smaller reserve/offset.
- Persistence remains **schema v28**. The only engine-level addition is recipe-local Text palette tuning control; all visual fixes remain ordinary semantic packets, selectors, responsive recipe layers, and existing provenance.

# v27.24 — Packpocalypse VIII: Journal + Ornament Rack

- Added the third full pack family: **Journal**, an assemblage-first Bubble/Minimal composition built from ordinary Theme Studio primitives. Bubble gets a paper **Polaroid note** masthead; Minimal gets a separate **Pinned journal card** lead; both share notebook prose, clipped quotes, a margin-thought note, ticket Greetings, notebook-tab Read More, and a stationery composer. Mobile collapses the pinned-photo choreography back into safe document flow instead of preserving desktop overlap.
- Added Theme Studio's first bundled **ornament rack**: Temper mark, Impact burst, Four-point sparkle, Sweat drop, Washi tape, Paper clip, Postage star, Tiny flower, Scribble heart, and Star divider. Built-ins are safe percent-encoded SVG data URLs and remain ordinary editable Background/Stencil intent rather than a bespoke decoration engine.
- Pack asset slots can now advertise curated built-in ornament choices. Added **plane-stencil** placement so a wrapper `::before` / `::after` surface can use asset alpha as a palette-tintable mask without hiding the wrapper's real contents. Journal exposes Photo pin, Page sticker, Divider ornament, and Thinking mark slots; external project/current-theme assets remain valid alternatives.
- Manga gained **Temper mark**, a tiny responsive anime anger ornament stapled to the message corner through the same pseudo-surface Stencil system. Its workbench slot can swap the default mark for burst/spark/sweat variants or a user asset.
- Fixed pseudo-surface geometry proven by corner ornaments: generated pseudo layers with an authored Anchored Position packet now use `inset: auto`, so top/right/bottom/left offsets actually control the ornament. Frame/rule pseudo surfaces without authored positioning still use `inset: 0` and retain their full-wrapper behavior.
- Editorial Apply All now includes **Reader correspondence** for Minimal user messages, aligning the smaller user portrait with the prose start instead of leaving it parked in the assistant author-rail acreage.
- Extended palette refinement so every Background Image Stencil, including bundled ornaments, inherits the pack accent/tint just like the reasoning icon replacement.
- Persistence remains **schema v28**. Journal, ornaments, plane-stencils, and the Editorial alignment pass are composition/library vocabulary over existing semantic packets and recipe provenance.

# v27.23 — Packpocalypse VII: Long Message Chrome

- Promoted Lumiverse's newly added long-message reveal control into Theme Studio's semantic anatomy. **Long message toggle** targets `MessageContent [class*="_longMessageTogglePill_"]`; **Long message toggle label** owns its inner span. MessageContent is the native semantic owner, so the same recipes work in Bubble and Minimal without cross-layout context echoes.
- Added **Manga · Read more tab**: square black/white shell, inset label padding, heavy uppercase print type, and a tiny offset ink-shadow. It is part of Manga Apply All for both message layouts.
- Added **Editorial · Continue reading**: restrained ink/steel shell with compact Georgia label treatment. It is part of Editorial Apply All for both message layouts.
- Extended the Bubble/Minimal anatomy audit with the mounted `_longMessageTogglePill_` receipt and regression coverage so future message-layout changes do not quietly strand the new control in native chrome.
- Persistence remains **schema v28**; this release adds recipe/selector vocabulary only.

# v27.22 — Packpocalypse VI: Stencils + Native Icon Skinning

- Added schema/state **v28** for a proven missing visual intent in Background Image: **Image / Stencil** rendering. Stencil uses an asset's alpha as a tintable CSS mask while preserving the target element's native layout box; optional **Replace native contents** hides only direct SVG/icon children. Existing v27 Background images migrate to ordinary Image mode.
- Replaced the reasoning-brain pseudo-surface experiment with direct **native icon-box skinning**. Thinking now prefers the mounted `container+bubble > toggle > brain` anatomy demonstrated by Lumiverse, and Manga/Editorial apply their built-in burst/diamond directly to the `_brain_` box as Stencil packets. No extra mark is positioned beside the toggle.
- Manga/Editorial **Thought mark** asset slots now use `placement: stencil`: a chosen PNG/SVG replaces the native brain in-place, inherits the pack accent as tint, exposes only the useful size control, and remains a normal editable Background packet in Design.
- Tightened reasoning semantic receipts around the actual combined `_container_` + `_bubble_` root, direct `_toggle_` wrapper, and `_body_` content while retaining stable `data-reasoning-toggle` and historical fallbacks.
- Added a small active-theme asset-source guard so creating/using a Theme Studio project bundle during the session does not immediately forget the previously observed native theme bundle as a borrowable source. Canonical adoption/export behavior remains unchanged.
- Added persistence/compiler/library regression coverage for v27 → v28 Background migration, Stencil mask compilation, direct-child suppression, native brain-box recipes, and stencil pack slots.

# v27.21 — Packpocalypse V: Minimal Spectacle + Borrowed Assets

- Rebuilt the **Manga MinimalMessage** lead as a real **Portrait rail** composition instead of a reduced Bubble preset: tall anchored monochrome portrait on desktop, printed identity, hard reading rule, right-corner actions, generous content plane, and a compact in-flow mobile collapse. **Editorial MinimalMessage** now has its own quieter **Author rail** with slim portrait, serif identity, cool publication rule, and literary measure.
- Manga **Apply All** now includes **Remove native strip**, so MinimalMessage's native root `::before` decoration is deliberately suppressed before the pack adds its own composition surfaces.
- Added explicit Bubble/Minimal **user-name** semantic roles and pack treatments. User messages no longer keep an unrelated native name style while assistant identity participates in Manga/Editorial.
- Finished the Reasoning chrome receipt by widening the semantic `_toggle_` role to wrappers/descendants proven by the stable `data-reasoning-toggle="true"` button. Manga/Editorial now hide the native reasoning SVG and add a pack-owned vector mark on the toggle's generated `::after` surface. The built-in burst/diamond use safe percent-encoded SVG data URLs and can be overridden from a new **Thought mark** asset slot.
- Fixed the remaining assetless workflow. Theme Studio now reads the active native theme's public asset bundle through `ctx.theme.assets.getActiveBundleId()`, includes those images as read-only **borrowable** sources, and resolves both `assets/…` and `./assets/…` spellings to `contentUrl` in live preview.
- Borrowed active-theme assets are copied into the Theme Studio project's own native bundle on first use from either a pack slot or ordinary Background-image editing. Generated/exported CSS therefore keeps canonical project-owned `./assets/…` paths instead of creating preview-only dependencies on another theme. Resources now shows project-owned assets and current-theme borrowable assets separately.
- Pack asset slots gained curated initial corner/offset/size metadata, letting tiny SVG marks enter at useful dimensions while preserving the existing Back/Front pseudo-plane editor.
- No schema bump. All new composition, icon replacement, and borrowed-asset behavior remains ordinary semantic packets/pseudo surfaces plus the existing public native asset bridge.

# v27.20 — Packpocalypse IV: Asset Planes + Live Path Firewall

- Fixed native asset paths in Theme Studio's live preview. Project state and generated/exported CSS still own canonical `./assets/…` references, while extension-owned preview stylesheets now resolve only known project asset paths to their session `contentUrl`. Relative pack paths no longer resolve against the app document and disappear.
- Asset preview resolution also applies to transient Design scrubs and Custom CSS preview without mutating either canonical project state or native export output. Switching projects clears the previous project's live asset map before refreshing the new bundle, preventing cross-project preview leakage.
- Expanded pack asset slots into explicit decorative **Back (`::before`) / Front (`::after`) planes** on semantic wrappers. Slots can choose from a broader set of message/header/meta/thinking/actions/composer anchors, then use corner, 2D offset, and size controls while compiling through ordinary Background / Position / Size packets.
- Manga **Corner art** now defaults to the Message frame Back plane; Editorial **Masthead mark** defaults to the Header Front plane. These are safe defaults only—bound assets remain freely movable in the workbench.
- Finished the latest mounted chrome receipts: Manga/Editorial Greetings put padding on the real launcher button instead of an internal span, and Thinking gains a dedicated `_toggle_` chrome role so the remaining native toggle background/border/shadow can be explicitly neutralized.
- No schema bump. Canonical asset identity, recipe provenance, and pseudo-surface compilation already had the necessary ownership model.

# v27.19 — Packpocalypse III: Live Chrome Receipts

- Rebased **Greetings** on the mounted interactive owner: `button[title="Browse alternate greetings"]`. The old `_indicator_` selector remains only as fallback evidence. Packs now style the actual launcher instead of decorating an internal class that may not own the button chrome.
- Added a separate **Greetings content** semantic role for the inner indicator/span. Manga and Editorial can therefore zero the button shell and own the internal padding/gap without double-bordering descendants.
- Finished the **Thinking toggle** reset by explicitly owning border, corners, and shadow on the live reasoning toggle. Native pill treatment can no longer peek through the Manga/Editorial thought panel.
- Promoted mounted **BubbleActions** (`data-component="BubbleActions"` + pill class) ahead of the historical `_actionsPill_` fallback. Editorial now ships a **Margin actions rail**: vertical on desktop, compact horizontal on Mobile.
- Turned the user-proven Editorial reading-column experiment into ordinary semantic packets: Bubble MessageContent gets a generated `::before` **reading rule** through a known pseudo-surface role. No custom-CSS pack escape hatch was added.
- No schema bump. This pass is selector/anatomy correction plus stronger use of existing pseudo surfaces, layout, positioning, and responsive recipe provenance.

# v27.18 — Packpocalypse II: Thought Panels + Split Masthead

- Repaired the **Thinking / Reasoning** semantic target for current Lumiverse. The mounted Reasoning block is now located from its stable `data-reasoning-toggle="true"` button inside BubbleMessage/MinimalMessage, with scoped `:has()` ancestry for the container and body. The old ThinkingBlock/ReasoningBlock data-component selectors remain only as compatibility fallbacks.
- Manga **Thought panel** now actually lands: hard monochrome frame, halftone surface, printed toggle strip, serif internal-narration body, and a lighter mobile edge. It remains an ordinary pack recipe with Base/Mobile provenance.
- Manga **Panel portrait** now deliberately styles BubbleMessage's native avatar backdrop as a high-contrast monochrome ghost layer using Auto source quality instead of leaving the native background image visually unrelated.
- Rebuilt Editorial's Bubble lead as a **Split masthead** rather than a recolored Manga hero. Desktop arranges the portrait plate and byline/meta column side by side; Mobile collapses them into a vertical masthead. The reading plane gets a narrower offset column and the avatar backdrop becomes a subdued photographic bleed.
- Editorial's curated palette moved from warm rose/flesh into **cool steel + warm paper** (`#92a6b3` / `#ece8df`), and Editorial preview surfaces now use a cooler ink base. Refine Palette is still fully editable.
- No schema bump: v27 recipe provenance already expresses every new treatment. No raw pack CSS path, DOM wrapper, or selector-system refactor was added.
- Added regression coverage for the live reasoning selectors, Manga backdrop/Thinking anatomy, Editorial split-masthead Base/Mobile direction, and the new Editorial default palette.

# v27.17 — Packpocalypse I: Editorial + Manga Spectacle

- Added schema/state **v27** so Quick Style / pack recipe provenance can own **Base and Mobile target packets independently**. Recipe steps may now declare a phone-only semantic delta, old v26 recipe slots normalize safely to Base, and manual packet deletion only detaches provenance for the responsive scope the user actually removed.
- Pack workbenches now enter with a **pack-curated default palette** instead of inheriting the global purple Quick Style accent. Manga opens monochrome; Editorial opens with its warm rose/ink palette. Refine Palette remains fully editable after entry.
- Reworked **Manga** toward an actual black-and-white print composition: harder panel frames, substantially more monochrome portrait treatment, tighter phone choreography, coordinated body-copy emphasis, square composer behavior, and new Greetings + Swipe Pager recipes. Apply All now carries those message-chrome pieces as part of the curated composition.
- Reworked **Editorial** with subtle grain, stronger feature/column rhythm, deliberate phone-scale portrait/header changes, coordinated paragraph/bold/italic prose, and a tighter mobile writing desk. Apply All now includes the Editorial Greetings treatment and compact pager so the native message chrome participates in the publication language.
- Large authored portraits remain **Auto** source quality rather than silently requesting originals. All new responsive behavior still compiles through ordinary Theme Studio packets and the existing Base/Mobile engine; no pack-only CSS path was introduced.
- Added persistence/library regression coverage for v26 → v27 recipe-slot migration, Base/Mobile provenance separation, pack palettes, new Manga core recipes, and responsive spectacle recipes.

# v27.16 — Pack Foundation

- Promoted Quick Style recipe ownership from an in-memory session map into project persistence. Recipe layer slots now survive reloads, so **Applied**, per-recipe Reset, **Reset selection**, and **Reset pack** keep their provenance after restarting Lumiverse instead of forgetting what a pack owns. Manual/non-recipe packet baselines remain underneath the ledger.
- Added schema/state **v26** for the persistent recipe ledger. Project duplication remaps active packet IDs while preserving lower recipe/base snapshots, so duplicated themes retain reversible pack history without sharing packet identities with the source project.
- Added declarative **recipe-owned Layout Groups** to the Style Library recipe schema. A recipe can now declare a real shared parent, member roles, Base/Mobile composition, and optional Members / Contents / Frame styling. Applying the recipe creates or refreshes that semantic group; resetting the recipe removes only groups carrying that recipe provenance.
- Known-role summaries/search now include group parent/member roles, and any recipe that declares a group is classified as a layout-scale recipe automatically. This lets pack-pocalypse recipes describe composition with the same semantic vocabulary as the manual Group editor instead of hardcoding custom CSS.
- Kept the important boundary: recipe groups still use the CSS-only Layout Group engine. Pack application never reparents React-owned DOM or invents wrappers.
- Manual **Restore target** and packet deletion now detach matching recipe-ledger ownership too, so explicit cleanup cannot leave a ghost **Applied** state or let a later pack Reset resurrect styling the user deliberately removed.

# v27.15 — Group Styling

- Expanded Layout Groups beyond composition with dedicated **Layout / Members / Contents / Frame** tabs. Pick and Group remain separate target modes; Guides/Smart remain a separate bordered tool cluster.
- **Members** now applies shared semantic packet stacks across all selected member roots without creating a wrapper. The first safe set covers Background, Pattern, Text Style, Typography, Border, Corners, Spacing, Shadow, Glass, Opacity, and Visibility. Individual members can still be edited separately through Pick.
- **Contents** can style repeated **Icons / Text / Buttons / Images** inside every group member. Each content target exposes only packet categories that make sense for that descendant family, allowing operations such as recoloring every icon in a control cluster from one group editor.
- **Frame** now creates one generated `::before` grid item on the real shared parent and places it behind the rows occupied by the selected members. Background, Pattern, Border, Corners, Shadow, Glass, and Opacity can visually bind a virtual group without DOM insertion or React reparenting. Frame activation gives group members explicit grid cells so the generated layer can overlap the same tracks deterministically.
- Added Base/Mobile group-style persistence as schema/state **v25**. Mobile group styling is an override layer over Base, while responsive layout remains independent. Project duplication refreshes group-style packet IDs.
- Added compiler/migration coverage for shared member paint, repeated icon targeting, and parent-backed group frames.

# v27.14 — Layout Groups

- Added a first-class **Pick | Group** target-mode split. Pick remains the single-element styling workflow; Group opens a separate composition UI for multi-selection. **Pick / Group** and **Guides / Smart** are now separate bordered clusters in the sticky workbar.
- Added persistent CSS-only **Layout Groups** for two or more reusable sibling selectors under one shared direct parent. Group selection is persistent, numbered on-page, click-again-to-remove, and rejects different-parent or unsafe/temporary targets. No React-owned DOM is reparented.
- Added a dedicated Group editor with **Row / Column / Grid**, Grid columns, Gap, Across/Down placement, **Other siblings: Full width / Join layout**, Base/Mobile states, naming, saved-group selection, and Ungroup. Paint/effect packets are intentionally absent because a semantic group has no synthetic visual wrapper.
- Added canonical group compilation. The shared parent becomes a grid; unselected direct children span the full width by default and selected members opt back into tracks. Mobile group state compiles through the existing 720px responsive boundary. Group CSS is emitted after ordinary element overrides so composition remains authoritative.
- Added group persistence as schema/state **v24**, duplication-safe group/member IDs, migration validation, compiler tests, and multi-element geometry highlighting.

## V27.13 — Native Owner Confidence

- Fixed native-surface naming/scoping heuristics being able to overrule stronger semantic ownership. A real non-generic `data-component` or unambiguous public catalog owner now outranks CSS-module overlap, so generic class families such as `manager`, `row`, `actions`, and `folderRow` cannot rename unrelated Persona/editor UI to `QwenCustomVoiceManager`.
- CSS-module component inference no longer rewards longer component labels. Shared local names are weighted as weak evidence, rarer locals carry more weight, and a module guess still needs a meaningful lead over the runner-up.
- The native bridge now records runtime-only root module-family fingerprints for picker identity. If a normalized public selector currently matches several unrelated CSS-module families (for example multiple different `_manager_` roots), that selector remains usable for component browsing/Edit Part but is barred from naming a picked surface by module overlap alone.
- Once a trustworthy non-generic semantic/registry owner is present, conflicting module-only guesses inside that owner are removed from the Scope list. Generic App remains available as the intentional broader scope, and Similar remains the explicit cross-panel escape hatch.
- Existing authored selectors are not migrated or renamed automatically. Repicking/next editing pass refreshes target metadata when the selector is unchanged; ambiguous old rules are never silently reassigned. No schema or native Lumiverse change is required.

## V27.12 — Native Surface Scope

- Native UI scoping now evaluates every matching public catalog/module context around a pick instead of accepting the first CSS-module family it encounters. A specific mounted surface such as **CharacterBrowser** or **PersonaBrowser** can therefore outrank generic **App** when the host catalog provides stronger evidence.
- The selected-target ladder and the compiler now agree about surface ownership. Picking an image inside CharacterBrowser defaults to a component-local scope such as **Image in CharacterBrowser** instead of silently collapsing to `App > Avatar > img`; PersonaBrowser receives its own independent context.
- Broader authoring remains explicit and useful. **Image across App panels** keeps the generic App scope, while **Similar images in all panels** remains the intentional broad sibling scope. Theme Studio does not remove those escape hatches just because a more specific default exists.
- Ambiguous public catalog selectors are not guessed. If multiple native components expose the exact same mounted selector, registry evidence is ignored until `data-component` or CSS-module-family evidence can actually distinguish them; module inference also requires a meaningful score margin over the runner-up.
- Existing App-wide overrides are deliberately not migrated into CharacterBrowser/PersonaBrowser. Their original intended panel cannot be reconstructed safely, so repicking creates the new specific target while the old broad scope remains visible/restorable as broad state.
- No persistence/schema or native bridge change is required; this is selector-resolution policy on top of the existing `theme-catalog-v1` metadata.

## V27.11 — Cleanup Crew + Auto Image Source

- Added **Image → Source quality → Native / Auto / Full**. Auto starts from Lumiverse's native thumbnail, watches the mounted image size, and promotes supported sources `sm → lg → original` only when the loaded intrinsic pixels are no longer enough. Its density target is capped, it waits for each promoted candidate to load before considering the next tier, and it does not repeatedly downshift while a surface is being resized. Full remains the explicit original-file mode and now carries a short bandwidth warning.
- Large built-in hero/lead portrait recipes now request Auto rather than Full, keeping pack-pocalypse from silently turning every enlarged portrait into an original-file download.
- Typography-only Full App Boost no longer requests or transforms the native palette baseline. With Colors and Backdrop off it applies only the requested font-family/font-scale root variables; project switches, preview, startup stabilization, and canonical rebasing skip the color-source pipeline for that case.
- Anchored Position now gets the same 2D movement pad as Nudge. Anchor edge offsets remain intact while the pad compiles a fine `translate` adjustment, so an absolutely positioned element can be pinned to an ancestor and then moved visually without sign gymnastics.
- Trimmed the tutorial-y Easy Effect chrome: Text keeps Outline/Glow, Shadow keeps Lift/Press + Strength, Glass keeps Frost, and image edge fades stay direct controls. Advanced recipes remain available without the extra “CSS trick / just slide it / depth trick” framing.
- Container gets the explanation instead: it now states that it arranges direct children and briefly explains Direction, Distribute, Align, and Grid columns in layout terms.
- Style Library pack/recipe cards are preview-first. Long preset descriptions remain searchable metadata but are no longer printed under every card or pack header; the workbench hero/action copy was shortened too.
- The Guide remains one Spindle Markdown page, but now starts with standard in-page heading links so the long page can be jumped by section where the native Markdown renderer supports heading anchors.
- Schema v23 persists Auto source quality. Guide/README and embedded Guide are synchronized.

## V27.10 — Context Echo Repair

- Fixes the compiler guard that mistook every `nativeContextSelector` for a message context; non-message targets such as App/Badge/ChatView no longer get their native root prepended a second time.
- Canonicalizes saved native context/local selector metadata at compile time as well as migration time, repairing already-persisted `App > App > Avatar > img` without flattening legitimate recursive selectors.
- Schema v22 repairs already-saved duplicated image selectors on load.
- Full Source now benefits from the same canonical selector path as generated CSS, so supported `/api/v1/images/:id?size=sm|lg` targets are reachable after selector repair instead of silently matching zero images.

# v27.9 · Avatar family scoping + full-source image bridge

- Fixed the other half of the avatar collision. A message-owned Avatar was already prevented from leaking outward, but a broad native `App > Avatar` target could still leak inward because Lumiverse reuses the `_avatar_` CSS-module local across message, character/persona, profile, and other UI families. New picks under broad CSS-module roots retain the nearest useful module-family ancestor when it actually narrows the mounted match set (for example `App > CharacterCard > Avatar`) instead of collapsing to `App > Avatar`.
- Kept real `data-component` owners intentionally simple. BubbleMessage/MinimalMessage and other semantic component roots do not gain incidental wrapper segments just because a nearby Header/row also has a CSS-module class; their existing component-local scoping remains the stable boundary.
- Historical exact `App > Avatar` native-aware rules are now quarantined at compile time rather than guessed. Theme Studio cannot safely infer whether an old ambiguous rule meant a Character avatar, Persona avatar, profile avatar, or message avatar. Explicit **Similar … everywhere** remains the opt-in global escape hatch, and repicking the intended surface produces the new family-anchored selector.
- Added experimental **Image → Source quality → Full source**. This is a constrained extension-side DOM presentation bridge for native image props that CSS cannot improve: supported Lumiverse avatar/image thumbnail URLs are swapped to their full resolver URL, thumbnail `srcset` is temporarily removed, React-authored source changes are observed/reapplied after rerenders, and disabling/removing the packet restores the latest native source.
- Full-source mode works whether the Image packet targets the actual `<img>` or a wrapper containing it. Unknown/data/blob URLs are deliberately left untouched; this is not a generic TSX/component monkey-patcher.
- Schema v21 persists Image source quality and adds regression coverage for App-avatar quarantine/family anchoring plus full-source URL derivation.

# v27.8 · Selector paths + mounted structure browser

- Fixed contextual selector composition being able to prepend the same native/local context twice. Already-rooted local selectors and `:scope` locals are now composed idempotently instead of producing shapes such as `[class*="_row_"] [class*="_row_"] [class*="_desc_"]`.
- Added a conservative live-DOM simplifier for adjacent repeated CSS-module selector segments. Theme Studio collapses a repeated segment only when the original and shorter selectors resolve to the exact same mounted elements; recursive structures where both segments are genuinely meaningful stay untouched.
- Schema v20 repairs saved overrides whose own `nativeContextSelector` + `localSelector` metadata proves they were persisted with a duplicated context prefix, including `::before` / `::after` variants. This is intentionally narrower than blindly deduplicating every repeated class token.
- The selected-target ladder now wraps inside the inspector instead of disappearing into a hidden horizontal strip on narrow/docked/mobile editors. Crumbs can shrink and wrap while the current target remains reachable.
- Added **Browse inside**, a mounted-DOM structure browser below the target controls. It walks interesting descendants of the picked subtree (including plain wrappers, controls, headings, images, semantic/ARIA nodes, and CSS-module parts), displays them as a vertically scrolling nested tree, and routes a chosen node back through the normal selector resolver. Deep children no longer depend on being discoverable through the curated native part list.
- Expanded native **Edit part** discovery from 24 to 64 mounted CSS-module parts. Edit Part stays the quick curated part picker; Browse inside is the deeper structural escape hatch.
- Added selector/migration regressions for idempotent context composition, safe duplicate collapse, recursive duplicate preservation, and schema-v20 saved-context repair.

# v27.7 · Boost source firewall + first-load stabilization

- Removed the last live Boost feedback paths instead of adding another startup special case. Full App Boost now accepts **only** the worker-side `spindle.theme.generateVariables()` map as transform input; the frontend variable catalog and mutable `<html>` declarations remain reference/presentation surfaces and can never become Boost's source.
- Root-style MutationObserver recovery no longer copies unexpected inline declarations into the cached transform baseline. Native rewrites are still remembered underneath for Reset, but Theme Studio reasserts live Boost and then asks the worker for a fresh canonical baseline. This prevents a prior/current Boost payload from being promoted into its own next input.
- Startup canonicalization now runs even when the initially selected Theme Studio project has Boost disabled. This closes the hot-reload sequence where stale root authority could survive an unboosted boot, lose its marker, and poison the first Boost project opened afterward.
- Fully disabling Boost invalidates the cached baseline. The next activation therefore starts from a fresh worker-generated map instead of assuming the native theme stayed unchanged while Boost was off.
- Project activation is now a source boundary: selecting an enabled Boost project fetches a fresh canonical map before atomically replacing the previous project's live transform, including switches between projects with identical Boost settings.
- Added a short post-ready startup source watch. If Lumiverse finishes installing/changing the selected native theme after Theme Studio setup has already begun, Theme Studio notices the changed `generateVariables()` fingerprint and rebases the still-visible Boost automatically—the manual **Refresh source** timing fix is now built into first load.
- Added source-firewall regressions proving the catalog is never read for live Boost, an unboosted startup cannot poison the first later activation, and a late canonical native-theme change rebases cleanly.

# v27.6 · Read Style border activation

- Fixed a misleading "dead selector" case in lazy **Read Style**. A source element with no visible border can report an observed border color with alpha `0`; editing only Width + Color previously compiled the whole observed shorthand, producing a perfectly scoped but fully transparent border.
- Sparse Border edits now compile as independent longhands (`border-width`, `border-style`, `border-color`) instead of claiming the untouched observed shorthand. This keeps Read Style's "only what you touched" contract intact.
- Choosing a Border color while its observed alpha is fully transparent now activates that paint at full opacity. New edits promote the alpha immediately, and schema v19 repairs v27.5 packets that already saved Color + Width with an unedited zero alpha.
- Added targeted compiler/migration regressions for the Persona-row failure shape (`20px`, cyan, observed alpha `0`).

# v27.5 · Scope guard narrowed + v27.4 decontamination

- Fixed the v27.4 scope quarantine overreaching onto unrelated native CSS-module targets. Native-aware parts with real non-message ownership (for example ChatView's Bar Wrapper / Chat Toolbar) compile normally again instead of being silently suppressed.
- Kept the strict message invariant where it belongs: BubbleMessage/MinimalMessage local parts are still re-rooted under their message component, while only truly metadata-dead native-aware CSS-module orphans fail closed.
- Fixed the v27.4 project-level orphan recovery being too eager. A project containing one message family no longer causes unrelated native targets to be adopted by that family.
- Added schema v18 repair for records v27.4 already contaminated. When a saved selector was injected under MinimalMessage/BubbleMessage but its surviving native component metadata points somewhere else, Theme Studio restores the original local/native scope instead of preserving the false message root.
- Added regressions for non-message native styling, metadata-dead quarantine, message-local repair, and v27.4 false-rehome recovery.

# v27.4 · Scope quarantine + clean Boost boot

- Fixed another historical message-scope failure mode where an older save had already discarded `nativeComponentId` / `nativeContextSelector` before the v27.2/v27.3 migration could see them. If the project still has one unambiguous message family, contextual orphan targets such as `Avatar · Ancestor` are re-homed under that BubbleMessage/MinimalMessage root. Projects that intentionally style both message layouts are not guessed.
- Persistence now preserves an explicitly saved `source: native-aware` even when old component metadata is incomplete instead of silently demoting it to DOM-scoped and destroying the last remaining ownership clue.
- Added a compiler fail-closed invariant: a non-global native-aware CSS-module target can never compile as a naked app-wide selector. If context survives, the compiler restores it defensively; if context is truly gone, the orphan stays in project state but emits no global CSS until it can be retargeted. Explicit **Similar … everywhere** targets remain legal.
- Fixed the remaining startup Boost-on-Boost drift. A forced detox/startup refresh now trusts the canonical worker `spindle.theme.generateVariables()` payload that performed the detox instead of immediately replacing it with a potentially stale frontend catalog snapshot. This is why manual **Refresh source** could fix startup before: by then the catalog had settled. Startup and manual refresh now use the same clean canonical source deterministically.
- Schema v17 adds regressions for metadata-lost message orphans, ambiguous dual-message projects, compiler scope quarantine, and a stale startup catalog that disagrees with the canonical worker baseline.

# v27.3 · Alek emancipation act

- Fixed the v27.2 scope-leak repair being incorrectly gated on the saved project schema. A project that had already been normalized to schema v15 could keep a stranded bare message selector forever, so `[class*="_avatar_"]` could continue hitting Character-page avatars.
- Schema v16 treats message scoping as an invariant instead of a one-shot migration: any non-global BubbleMessage/MinimalMessage CSS-module target with message ownership metadata is re-anchored under its message root regardless of the incoming version stamp.
- Mixed selector lists are repaired branch-by-branch, and message ownership can also be recovered from the saved target label when older metadata is incomplete. Explicit **Similar … everywhere** / global scopes remain untouched.
- Added persistence coverage for schema-v15 stranded selectors, current-version imported leaks, and mixed rooted/bare selector lists.

# v27.2 · Native scope leak guard

- Fixed legacy native-aware CSS-module overrides such as bare `[class*="_avatar_"]` rules leaking out of BubbleMessage/MinimalMessage and styling unrelated app surfaces (for example character-list avatars).
- Schema v15 repairs only old message-local ladder/ancestor targets that carried message component metadata but persisted without their component context. A MinimalMessage avatar now migrates to `[data-component="MinimalMessage"] [class*="_avatar_"]`; explicitly broad scopes labelled Similar/everywhere/global remain broad.
- Added resolver coverage asserting message ancestor scopes remain component-contextual and compiler coverage asserting Strong authority preserves the full ancestor selector rather than collapsing to the local class.
- This repair is deliberately migration-scoped: new v27.1+ Assistant/User selectors already carry their message root, while genuinely global CSS-module scopes are left untouched.

# v27.1 · Message-side authoring

- Added a first-class **Assistant / User / Both** facet for BubbleMessage and MinimalMessage targets. Picking an assistant or user message starts on that side automatically; Both is an explicit opt-in.
- The resolver now understands current Lumiverse's root `_user_` message modifier and pairs mounted `*User` CSS-module parts (for example `name` / `nameUser`) into one civilian-facing Edit Part entry.
- Shared message parts that do not have a `*User` counterpart are isolated through the message root: Assistant uses `:not([class*="_user_"])`, User uses the `_user_` root, and Both compiles the two safe branches instead of broadening blindly.
- Edit Part preserves the current message side while moving between component anatomy, so browsing from Name to Content no longer drops back to an unrelated role.
- Selector-list compilation is now safe across Strong authority, hover/focus/disabled state selectors, previews, and generated `::before` / `::after` surfaces. This lets Both remain one semantic target even when the concrete assistant/user selectors differ.
- Message-side discovery remains DOM-derived and works without the native component catalog; no new Lumiverse bridge capability or persistence schema was required.

## v26.2 — Edit Part browser restoration

- Restored **Edit part** as a component-internal class browser rather than a same-node selector chooser.
- Part discovery now scans CSS-module local classes under the nearest mounted `data-component`, so off-ladder parts such as BubbleMessage avatar backdrop/scrim/image layers remain targetable even when the native component catalog returns no entries.
- Native catalog metadata still enriches component parts when available, but is no longer required for DOM part browsing.
- Removed the redundant Selector dropdown added in v26.1; Scope remains about reuse breadth, the target ladder remains ancestor navigation, and Edit part owns sibling/descendant component anatomy.

## v26.1 — selector chooser restoration

- Restored the literal selector chooser beside Surface/Scope so a picked DOM node can target any selector identity discovered on that same node (semantic attributes, normalized CSS-module locals, exact classes, or the temporary mounted path).
- Selector choice is separate from Scope: Scope controls reach; Selector controls which class/data-component identity names the current node.
- Same-node selector candidates are retained by the resolver even when the native component catalog is unavailable, so CSS-module parts such as avatar/ghost backdrop classes remain directly targetable.

# v26 · Responsive Workbench + canonical style slots

- Added first-class **Base / Mobile** authoring to the semantic project model (schema v14). On a phone-width viewport Theme Studio starts in Mobile scope automatically; choosing Base or Mobile manually pins that scope while you work. Mobile packets are sparse deltas and compile under `@media (max-width: 720px)` instead of rewriting the all-sizes theme.
- Read Style stays lazy inside responsive scopes. Editing an observed value on Mobile materializes only that field into the Mobile stack; resetting/removing the field lets the browser fall through to Base or the native responsive rule underneath.
- Reworked generated Design CSS into **canonical style slots**. Each target × state × responsive scope produces one deterministic slot per semantic category, with stale duplicate categories collapsed last-wins. Internal Quick Style history stays reversible, but generated CSS no longer grows append-only strata every time the same category changes.
- Style-pack detail is now a true **fullscreen workbench**. The recipe canvas owns the viewport, the workbench rail stays fixed-width and independently useful, and recipe grids scale from four/three columns down to two/one on smaller screens.
- Promoted common message chrome into reusable roles: **Minimal actions dock**, **Minimal native `::before` strip**, **Greetings**, and **Swipe pager**. Added Overlay actions dock, Remove native strip, Quiet greetings, and Compact swipe pager recipes. Editorial Minimal defaults now reclaim the dormant actions rail and remove the native strip without disabling the hover actions themselves.
- Hardened the v25.4 Boost fix on startup: if a saved project has Boost enabled, Theme Studio performs the guarded native-baseline refresh before applying the saved transform. This makes the working **Refresh source** detox path the normal reopen path rather than relying on stale-marker inference.

# v25.4 · Boost source-sampling guard

- Fixed the remaining **saved lavender / rendered swamp-green** failure mode: the public variable catalog can report Theme Studio's own current root authority if Boost is still mounted when the source is sampled. Source acquisition now happens inside an explicit guarded sampling window instead of reading the effective catalog blindly.
- Added a single `sampleNativeBaseline()` path. Theme Studio disconnects/guards its root observer, temporarily releases any root declarations owned by the current runtime, samples the native/theme source, then reasserts the existing live/preview Boost layer without letting its own writes masquerade as a native theme change.
- Hot-reload recovery now detects stale `data-theme-studio-boost-*` ownership left by an older runtime whose in-memory underlying snapshot is gone. It asks the clean worker-side base-theme generator for a detox seed, removes the stale `important` authority, then samples the public catalog from that clean root before applying the saved Boost controls.
- **Refresh source** uses the same guarded path with forced detox, giving users one deterministic escape hatch from any stale pre-v25.4 root state instead of transforming whatever happens to be winning at the moment.
- Added regressions for stale-root recovery and for Refresh source sampling only the released/native palette while the root observer is suppressed.

# v25.3 · Boost baseline-stability hotfix

- Fixed **Boost-on-Boost palette drift** after saving/reloading, refreshing source, or changing the native theme while Full App Boost was active. V25.2 still mirrored the transformed variable map through `spindle.theme`; because that API participates in Lumiverse's resolved theme map, the frontend catalog could later report Boost's own output as the next source palette. Re-transforming that output is what turned the saved lavender look into an unrelated teal/green palette.
- Full App Boost now has one live authority only: Theme Studio's reversible `important` inline declarations on `<html>`. The worker theme override is no longer applied during normal Boost updates, so the transform source cannot become its own output.
- On startup Theme Studio clears any stale worker-side Boost override left by v25.2 or an interrupted hot reload before capturing the native variable baseline.
- While Boost is active, genuine Lumiverse root-theme reapplications update both the reversible underlying declaration snapshot and Theme Studio's cached transform baseline before the live Boost layer is reasserted. The next slider change therefore transforms the newest native theme rather than stale source colors.
- `spindle.theme.clear()` remains as backward-compatibility cleanup on startup/reset/unload; persistent theme authoring continues to use `ctx.theme`.

# v25.2 · Boost root-authority hotfix

- Fixed Full App Boost still doing nothing on current Lumiverse even with an otherwise blank Theme Studio project. Boost now takes its baseline from the public frontend variable catalog first, writes the transformed map directly onto Lumiverse's own inline `<html>` declarations with `important` priority, and only then mirrors the same payload through the worker `spindle.theme` API. A muted/failed worker mirror can no longer turn the explicit local Boost action into a no-op.
- Added a root-style observer while Boost is active. If Lumiverse reapplies its native theme variables, Theme Studio remembers the newest underlying inline values and immediately reasserts only the active Boost variables before the next paint. Reset/unload restores the latest underlying declarations and then clears the scoped worker override.
- Boost preview/committed authority now share one root-inline ownership path; saved-theme `:root` rules and native inline theme applications can no longer silently outrank the live Boost.
- Canvas/wallpaper helper styles are now created through Spindle's tracked DOM factory rather than raw unowned style nodes.
- Full App Boost is no longer emitted into ordinary Generated CSS/Code output. It remains world state driven by the runtime bridge; native handoff and `.lumitheme` export continue to contain only actual project theme CSS.

# v25.1 · Boost cascade hotfix

- Fixed Full App Boost losing the cascade after native/saved theme CSS loads or redeclares the same `:root` variables. While Boost is enabled, Theme Studio now maintains an authoritative frontend `html:root` variable layer with `!important`, while still mirroring the same payload through the existing scoped `spindle.theme` lifecycle.
- Boost scrub previews use a higher-authority temporary layer so live slider feedback can override the committed Boost layer without touching project CSS.
- Native `.lumitheme` / Send to Lumiverse drafts no longer bake Full App Boost into persistent Global CSS. Boost remains extension world-state, preventing an installed Theme Studio theme from shadowing its own later Boost edits.
- Clearing, disabling, or unloading Boost removes the authoritative layer and returns control to the underlying native/theme cascade.

# v25 · Native authoring bridge adoption

- Replaced Theme Studio's private/read-only asset fetch and build-time native catalog snapshot with Lumiverse's public frontend `ctx.theme` authoring facade.
- Runtime capability detection now follows `theme-assets-v1`, `theme-packs-v1`, `theme-catalog-v1`, and `theme-editor-navigation-v1`; unsupported hosts simply disable those leaves instead of falling back to private APIs.
- Native component and CSS-variable references now come from `ctx.theme.catalog`. Mounted public component roots are locally enriched with visible CSS-module parts for picker ergonomics without importing native source paths.
- Added project-owned native asset bundles. Resources can create/list/upload/delete assets, optimize images through native WebP support, preview via `contentUrl`, and persist canonical `./assets/<slug>` CSS paths. Background Image can choose those native image paths directly, and native font assets remain registerable as `@font-face`.
- Added the Code workspace **Native handoff**: Send to Lumiverse, canonical `.lumitheme` export, inert `.lumitheme` import, and native editor navigation. Theme Studio emits a CSS-only `SpindleThemePackDraft`; Lumiverse owns archive/install semantics and executable TSX never enters the extension draft.
- Split native compilation into recognized native component sections plus safe global CSS. Older/stale native component IDs degrade to global CSS rather than losing the user's rules.
- Full App Boost now derives from the runtime public variable catalog instead of a generated variable snapshot while retaining the worker-side reversible `spindle.theme` lifecycle.
- Removed `generated-native-data.ts`, `sync-native-catalog.ts`, and the related build scripts. Bumped project/state schema to v13 for the native bundle ID and preview URL metadata.
- Kept Manga, Editorial, the v23 pack workbench, v22 library browser, and v20 lazy Read Style behavior intact; pack-level asset-slot binding remains the next Theme Studio-side adapter rather than a native API gap.

# v24 · Editorial pack

- Added **Editorial**, the second full style pack and the first pack designed as Manga's structural opposite: literary spacing, serif display type, restrained rules, art-directed portrait treatments, and a writing-desk composer.
- Editorial ships eight reversible recipes: **Feature lead** (Bubble), **Column rule** (Minimal), **Byline identity**, **Publication headings**, **Pull quote**, **Reading column**, **Author portrait**, and **Editorial composer**.
- The curated Apply All core is anatomy-aware. Bubble gets Feature lead; Minimal gets Column rule; shared prose/composer pieces stay available in both. Byline identity and Author portrait are intentional alternates and are not forced into the default composition.
- Added a distinct Editorial pack/recipe preview grammar so pack thumbnails no longer inherit Manga's halftone/panel choreography. The existing Manga Bubble/Minimal hero morph is now explicitly scoped to Manga.
- Editorial declares only two optional asset slots—current character portrait and a future masthead mark—to prove that packs can stay mostly procedural instead of requiring an asset bundle.
- Locked the forward pack vocabulary to **Terminal → Visual Novel → Casefile** after Editorial, with **Casefile** reserved for the investigation/report system.

# v23 · Pack workbench

- Turned pack detail into a real **workbench** instead of a read-only collection shelf. Each pack can now stage a custom recipe subset before applying it.
- Added per-pack **Both / Bubble / Minimal** anatomy modes. Compatibility is explicit: switching to Minimal disables Bubble-only pieces and vice versa, while shared prose/avatar/composer recipes remain available.
- **Apply all** is now layout-aware and still means the pack's curated defaults; it does not suddenly opt into alternates.
- Added **Apply selection**, **Reset selection**, Defaults / All compatible / Clear selection helpers, section-level choose/clear controls, and a compact recipe manifest in the pack rail.
- Recipe cards show a dedicated **Choose / Chosen** workbench control and visibly dim pieces that belong to the other message anatomy without hiding them from the pack inventory.
- Added a sticky pack section jump bar so larger future packs can grow without becoming scroll archaeology.
- Manga's pack preview now reflects Bubble vs Minimal workbench mode with a panel vs margin-rail composition cue.
- Pack workbench selections are UI-session state only; applying still creates the same ordinary reversible recipe layers and does not change project schema.

# v22 · Library browser + nested filters

- Reworked the Style Library into a real browser workspace: persistent Browse / Recent / Applied / Favorites navigation lives in a slim rail while search and results own the roomy canvas.
- Removed the always-visible area/family/layout tag wall and moved filters into a nested **Filter styles** dialog. Active filters return as removable chips above results.
- Style families now classify/filter recipe cards only; pack cards remain first-class collection results instead of being accidentally swept into family filtering. Family badges on individual style cards are clickable shortcuts to browse that family directly.
- Search now treats space-separated terms as AND terms (for example `minimal portrait`) and updates visible style/pack counts live.
- Moved the shared preview palette into a collapsible **Tune previews** rail control, so browsing starts near the actual content instead of below a full Build-a-Bear form.
- Pack cards now use a future-proof 3/2/1-column album-cover grid and open pack detail rather than behaving like oversized recipe cards.
- Mobile turns the rail into a compact view shelf and the nested filter dialog into a bottom sheet.
- No project schema changes.

# v21 · Pack controls sidebar

- Reworked pack detail into a two-pane workspace: recipe preview/cards scroll in the main canvas while pack-level controls live in a persistent right sidebar.
- Moved **Apply all**, **Reset pack**, the shared Recipe Palette, and semantic asset slots into the pack rail so customization no longer pushes the actual styles below the fold.
- The pack hero is shorter and informational-only; individual style sections now begin immediately beneath it.
- Palette and asset groups are collapsible in the rail. On compact/mobile layouts they start collapsed and the rail becomes a slim control shelf above the independently scrolling recipe canvas.
- Pack main/sidebar scroll positions survive apply/reset renders, and palette swatches/intensity now update through the existing CSS variables without rebuilding the modal.
- No project schema changes.

# v20 · Lazy Read Style + Style Map

- Changed **Read Style** from eager capture to inspection-first observed state. Reverse-engineered controls can be explored without adding anything to generated CSS or project ownership.
- Added sparse packet ownership through `editedFields`. The first user edit materializes only the semantic field/bundle that changed; untouched observed values continue to come from the source cascade.
- Sparse Typography, Text, Background, Image, Spacing, Size, Layout, Position, Glass, and other packet output is filtered by semantic ownership in the compiler. Helper rules are emitted only when the edited semantic operation requires them.
- Sparse packet views can hydrate their unowned fields from a fresh Read Style session, so a later source-theme change does not force stale observed values into the editor.
- Added **Read page** / Style Map: a document-level, non-destructive scan of visible meaningful targets, grouped by component with compact computed summaries and matching authored source/selector provenance. Selecting a map row opens the real target in Design and starts lazy Read Style.
- Added authored-source reporting for inline styles and matching stylesheet rules, including `!important` properties and active nested conditions when readable.
- Bumped project/state schema to v12 so sparse ownership metadata persists safely.

## v17 — Strong means Strong

- Strong overrides no longer rely on `!important` alone. The compiler zeroes the original selector specificity with `:where(...)` and applies a fixed Theme Studio authority tier, so an older complex Quick Style selector cannot out-specificity a newer local Strong edit.
- Editing or promoting an existing target moves it to the end of the generated Design cascade. Equal-authority rules therefore resolve by the user’s most recent explicit edit instead of selector complexity.
- Transient slider previews use the same authority selector for transition suppression and packet preview, so a hot-pink Strong scrub cannot visibly lose to the lavender Strong source it is replacing.

# v16 · Boost continuity + two-edge mobile Float

- Added a frontend Boost continuity handoff: while Spindle replaces an extension Theme override with its clear → apply sequence, the already-active Boost variable map remains mounted locally, preventing base-theme flashes/disco-ball frames during Boost commits.
- Boost remains world state under local Design scrubs; the transient Design stylesheet still contains only the target being edited.
- Mobile Float can now attach to either the bottom or top edge with one next-action arrow in the title bar. The chosen edge is remembered locally.
- Added a dedicated mobile sheet resize handle. When the sheet is bottom-attached the handle sits on top; when top-attached it moves to the bottom while the rest of the editor keeps the same layout/order.
- Resize math now follows the anchored edge, preserving the current sheet height on first touch and snapping to the existing Peek / Work / Full heights on release.
- Corrected the compact mobile Return selector so adding the new edge button does not steal the Return icon treatment.
- No project schema changes.

# v15 · Mobile QA + widget flow

- Compact-widget **Pick is one-shot**; the main editor keeps persistent DevTools-style picking. Picking no longer auto-expands a collapsed widget.
- Zap stays open after each zap and across one-shot target changes, so repeated control cleanup becomes Pick → Zap → Pick → Zap without reopening the tray.
- Guide geometry now explicitly invalidates and re-measures across target changes, resize/scroll/orientation, VisualViewport movement, ResizeObserver activity, pageshow, and app foregrounding.
- Fixed the mobile Float handle’s first-touch jump by seeding the drag height from the sheet’s current rendered geometry before drag mode activates.
- Polished compact-widget tool spacing with real SVG controls and added a direct Shuffle SVG beside the Boost strip; the main Boost Shuffle action now carries the same icon.

# Theme Studio v14 — Read Style authority fix

- Fixed Read Style packets appearing editable in the cockpit while losing every visual change to the CSS they were reverse-engineered from.
- Read Style now promotes the captured local target to **Strong** by default, so Quick Style/native `!important` declarations do not keep winning after capture.
- Subsequent sliders, font changes, colors, spacing, and other packet edits use that same Strong target in drawer, Float, and mobile sheet because all three shells share the same semantic source.
- Users can still switch the target back to Normal under Target details when they deliberately want ordinary cascade behavior.
- No project schema changes.

# Theme Studio v13 — mobile cockpit diet

- Floating Theme Studio no longer repeats the project selector/title row. The full drawer remains the primary project-management surface.
- Design, Code, and Themes are compact SVG workspace controls in the floating title bar; Pick/Guides remain their own core tool row directly beneath.
- Mobile Float is substantially slimmer at the top, including an icon-only Return control while preserving the draggable sheet handle.
- Moved **Read style** from Selected into the Style stack header, where the reverse-engineering action semantically belongs.
- Moved Shuffle colors, Refresh source, and Reset Boost to the top of Full App Boost so they stay reachable when Backdrop/Typography are expanded.
- No project schema changes.

# Theme Studio v12 — Read Style + inspector contrast

- Added **Read style** to the Selected target header. It reverse-engineers the winning visible presentation into ordinary editable semantic packets on the current scope/surface.
- Read Style uses matched authored declarations as a filter when available, so it can capture native CSS, Quick Styles applied through broader role selectors, and custom/theme CSS without blindly freezing browser-computed defaults such as `width: auto` into fixed pixels.
- Captures supported Background/gradient, Text Style, Typography, Border, Corners, Spacing, Shadow, Glass, Opacity, Visibility, Image treatment, Position, Flex/Grid layout, Layout Item, and explicitly authored Size properties.
- Read Style works on Element, Back layer, and Front layer surfaces; pseudo-element captures read the corresponding computed pseudo style.
- Existing packets of the same semantic type are updated while unrelated packets remain intact.
- Theme Studio primary buttons no longer trust Boosted Lumiverse contrast tokens. Primary actions now use the extension's soft-accent surface plus normal inspector text, keeping Apply/Shuffle/Done readable across aggressive palettes.
- Kept the v11 Backdrop treatment, coordinated H1–H4 systems, reversible Quick Style layers, and persistent Boost/Design separation intact.

## V18 — Style Library shell + MinimalMessage map

- Moved the full Quick Style catalog out of the Themes sidebar into a document-level Style Library modal/sheet.
- Themes now keeps a compact Quick Styles front porch: shared recipe palette, context-aware starter cards, and **Browse styles**.
- Added searchable recipe metadata for every existing Quick Style: area, look family, scale, layout compatibility, and keywords.
- Added Style Library filters for area, look family, and Bubble/Minimal message layout.
- Kept Apply / pencil / Reset semantics identical inside the library; Apply keeps browsing, pencil opens Design.
- Added an explicit MinimalMessage anatomy audit from the shipped native catalog. Existing BubbleMessage compositions are intentionally not mislabeled as Minimal-compatible.
- Renamed the old internal testing names to the plainer **Glow hierarchy** / **Glow set**.
- No persistence schema bump; this is catalog/UI infrastructure.

# v19 · Pack browser + Manga

- Added Style Library views for **Browse**, **Recent**, **Applied**, and persistent **Favorites**.
- Added pack metadata and an Obsidian-style pack detail view. Packs can expose a large preview, shared palette, coverage, asset slots, **Apply all**, pack reset, and the same per-style Apply / pencil / Reset controls as ordinary recipes.
- Added a first asset-slot contract (`image` / `procedural`, optional/required, semantic purpose) without hard-wiring file paths into recipe definitions.
- Added the first full pack: **Manga**. Its default Apply All coordinates BubbleMessage, MinimalMessage, shared MessageContent prose, and InputArea without forcing the two message layouts through the same choreography.
- Manga includes **Panel portrait** (Bubble), **Margin speaker** (Minimal), **Ink frame** (both), **Chapter headings** (H1–H4), **Caption box**, **Sticker portrait** (both), **Manga composer**, and **Printed code**.
- Apply All deliberately excludes the alternate **Ink frame** and **Sticker portrait** recipes so the default composition does not override its own hero/avatar treatment. Applying an alternate later layers cleanly; Reset reveals the pack style underneath.
- Added explicit MinimalMessage roles to the recipe registry plus layout-neutral MessageContent heading/blockquote/code roles and InputArea toolbar/textarea/send-button roles.
- No project schema bump; library history/favorites are UI-local, while applied recipes still compile to ordinary semantic targets.