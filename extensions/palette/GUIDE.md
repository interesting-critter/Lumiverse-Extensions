# Palette

> **Visual theme authoring for Lumiverse.** Pick the thing you mean, describe the visual intent, and Palette turns it into scoped, reusable CSS. Generated CSS is output; your semantic Palette project is the source of truth.

**Palette 1.0.5 · schema v46**  
Release history lives in the project changelog; this page is the actual manual.

**Jump to:** [Start here](#palette-guide-start) · [Pick & scope](#palette-guide-pick) · [Style packets](#palette-guide-packets) · [Groups](#palette-guide-groups) · [Read styles](#palette-guide-read) · [Reuse](#palette-guide-reuse) · [Boost](#palette-guide-boost) · [Widget & Code](#palette-guide-code) · [CSS field guide](#palette-guide-css) · [Debugging](#palette-guide-debug)

## Start here

Palette is easiest to understand as **visual DevTools with memory**. It knows enough about Lumiverse anatomy to avoid making you write selectors for normal work, but it never traps advanced users inside a black box.

| Workspace | What it is for |
| --- | --- |
| **Design** | Pick a real mounted element and author semantic style packets. |
| **Themes** | App-wide Boost, Typography, Quick Looks, pack workbenches, and theme projects. |
| **Style Library** | Browse reusable looks and your cross-project **My Styles**. |
| **Code** | Inspect compiler-owned CSS, add Custom CSS, and use native `.lumitheme` handoff. |

A normal workflow is:

1. Open **Design** and turn on **Pick**.
2. Click the thing you actually want to change.
3. Use the breadcrumb, **Edit Part**, or **Browse Inside** if Lumi wrapped your target in three divs and a trench coat.
4. Add packets such as Background, Typography, Size, Image, or Position.
5. Switch **Base / Mobile** or **Normal / Hover / Active / Focus / Disabled** when needed.
6. Save the result as a **My Style**, keep building the current theme, or inspect the generated CSS in **Code**.

> **Rule of thumb:** if you are reaching for Custom CSS during ordinary styling, first ask whether Palette already has a visual packet for that intent. The packet is safer, responsive, reusable, and pack-aware.

## Pick & scope

### Pick stays armed

The main Design picker stays active until **Done** or **Esc**. This makes it practical to inspect several nearby pieces without reopening the picker every time. The floating mini-widget uses a lighter one-shot picking flow so it does not become a tiny second DevTools.

Palette separates four things that browsers often blur together:

| Layer | Example | Why it matters |
| --- | --- | --- |
| **Clicked node** | the exact `<img>` under your cursor | What you physically picked. |
| **DOM ladder** | Image → Avatar → Header Left → Header → Bubble | How the mounted tree is built. |
| **Semantic owner** | BubbleMessage / Personas / InputArea | Which Lumiverse surface owns it. |
| **Persistent scope** | Avatar in BubbleMessage | What future rerenders should keep styling. |

### Edit Part vs Browse Inside

**Edit Part** is the curated semantic map of useful parts for the current component. It is where you go when you know you want *Avatar*, *Meta Wrap*, *Actions*, *Message Content*, and similar authored pieces.

**Browse Inside** follows the actual mounted subtree. Use it when the thing you need is deeper, conditional, or too weird to belong in the curated list.

They intentionally do not show the same list.

### Dynamic labels and Use generic

Interactive controls sometimes need their live `aria-label` or `title` to distinguish one mounted button from its siblings. Palette may therefore select something precise such as `button[aria-label="Copy"]`. When the target panel offers **Use generic**, that is an explicit escape hatch to the reusable sibling selector (for example the whole Minimal action-button group). The switch is exact and keeps the current message side; it should not bounce back to the label-specific target.

This is useful when one button taught you the styling recipe but the intent is really “all buttons in this row.” Keep the precise target when the buttons genuinely need different appearances.

### Stable selectors beat generated hashes

Palette prefers public semantic boundaries such as:

```css
[data-component="BubbleMessage"]
[data-component="InputArea"]
[data-spindle-drawer-tab="personas"]
[data-composer-action="send"]
```

Then it can narrow to a stable CSS-module family:

```css
[data-component="BubbleMessage"] [class*="_avatar_"]
```

instead of persisting an exact generated class such as:

```css
._avatar_1hvlc_181
```

Generated hashes can change. The family selector communicates the actual visual part.

### Assistant, User, and Both

Message targets can be side-aware. When Lumi shares internal classes between both sides, Palette keeps the side discriminator in the selector instead of pretending one `_avatar_` means every avatar in civilization.

Typical shapes:

```css
/* Assistant */
[data-component="BubbleMessage"]:not([data-part="user"]) [class*="_avatar_"]

/* User */
[data-component="BubbleMessage"][data-part="user"] [class*="_avatar_"]
```

Use **Both** only when you actually want the same treatment on both speakers. In Design, Both is a real combined scope rather than shorthand for the speaker you clicked first: its editable stack represents styling common to the mounted Assistant/User branches, and new edits persist against the combined selector. One-sided authored packets remain available from their Assistant or User facet instead of masquerading as Both.

### Surface means the real thing, not just its box

Many Lumi parts have multiple useful surfaces:

- Element
- `::before` / Back layer
- `::after` / Front layer
- `::placeholder` when the mounted control actually has placeholder text

Generated surfaces are useful for labels, ornaments, stickers, frames, SVG stencils, and other decorative planes without inserting DOM.

## Style packets

Palette packets describe intent. You should not need to remember the CSS property names for ordinary work.

| Packet | Think of it as | Common CSS underneath |
| --- | --- | --- |
| **Background** | paint the surface | `background`, gradients, images |
| **Ink** | visual ink for text, glyphs, and current-color graphics | `color`, gradient fill, stroke, shadow/glow |
| **Typography** | type structure | family, size, weight, line-height, spacing |
| **Text Entry** | where typing starts + how it measures | textarea inset/metrics + synchronized autosize mirror + placeholder appearance |
| **Border / Corners** | edge treatment | border longhands, radius |
| **Spacing** | breathing room | padding + margin |
| **Shadow / Glass** | depth/material | shadow, blur, backdrop-filter |
| **Opacity** | fade the entire target | `opacity` |
| **Container Layout** | arrange direct children | flex/grid, gap, distribute, align |
| **Quick Align** | put this element where you mean | logical auto margins + safe self-alignment |
| **Layout Item** | advanced behavior inside Flex/Grid | grow/shrink, order, self-alignment |
| **Size** | fit, fill, or fix the box | width/height/min/max |
| **Image** | crop and treat media pixels | source quality, object-fit, object-position, filter |
| **Mask** | fade or clip a visual surface | mask-image, multi-edge masks, compositing |
| **Media Flow** | make prose/native media behave | natural height, unclipping, full-width flow |
| **Position & Layer** | move/place/stack | translate, anchored position, sticky/fixed, z-index |
| **Transform** | pose it | rotate, scale, skew |
| **Background Image** | decorative image layer | background-image/position/size |
| **Visibility** | visible / hidden / gone | visibility/display |
| **Generated Content** | label a generated surface | CSS `content` on `::after` / explicit pseudo surfaces |
| **SVG Asset** | reusable icon/ornament | sanitized project SVG stencil |

### Image, Mask, and Ink are separate jobs

**Image** is for media treatment: source quality, brightness/saturation/contrast and other tone controls, crop/fit, and focal position. It stays an Image packet even when the semantic Palette target is a wrapper; use **Browse Inside** when object-fit/object-position must land on the actual media leaf.

**Mask** is surface geometry, not an image-only effect. Use it to preserve the native mask, clear masking, add a directional/radial fade, or build multi-edge custom masks. Because Mask is independent, it can style ordinary wrappers, generated surfaces, media, and other CSS-mask-capable targets without pretending they are images.

**Ink** is the renamed visual-paint packet formerly shown as **Text Style**. The persisted packet type remains compatible with older projects, but the UI name reflects what it really does: solid/gradient color, stroke/outline, and glow for text, icon glyphs, SVG/currentColor surfaces, and similar painted marks. Typography remains responsible for type structure such as family, size, weight, line-height, spacing, and case.

Schema v42 migrates pre-v42 combined Image packets automatically. A legacy Image that owned both tone/crop settings and a mask is normalized into sibling **Image + Mask** packets on the same target/state, including recipe provenance, so old themes keep their rendered intent while new edits use the clearer taxonomy.

Color-bearing packets always expose a real **Pick** swatch beside the editable color string. **Recents** are convenience history, not the only way to open a picker; pack-authored colors therefore remain editable even when they were never picked manually in the current project. Independent Corners are arranged spatially as top-left / top-right over bottom-left / bottom-right, matching the box you are actually shaping.

### Text outlines: Edge vs Outside

**Edge** uses the browser glyph stroke and is fast/clean for thin lettering. **Outside** is the "do not eat my fill" option: Palette manufactures a crisp ring of zero-blur text shadows behind the glyph and exposes it through the same Thickness / Color / Opacity controls. Your ordinary directional text shadow can still coexist with that outline.

### Generated Content can mirror native labels

Generated Content can use a literal string, or mirror the owner's existing `title` / `aria-label` through CSS `attr(...)`. If you add it to a normal element, Palette automatically emits the content on that element's `::after` skin because Chromium does not reliably render `content` on ordinary elements. If you explicitly selected Back/Front, Palette keeps that pseudo-surface. This is useful for skins that replace a native icon with text while keeping the actual button as the semantic/layout target.

Pseudo-elements are generated surfaces, not DOM siblings. If several labeled buttons need even distribution, group/style the **real buttons** and use their `::before`/`::after` only as visual skins.

### Quick Align: say where, not how

Use **Quick Align** when your intent is simply “put this on the left / center / right” (or the vertical equivalent). Palette resolves that intent through layout-safe CSS instead of making you memorize when `align-self`, `justify-self`, or auto margins happen to work.

For ordinary horizontal placement, Palette uses logical auto margins and `fit-content`, so it works in normal block flow as well as common Flex/Grid contexts:

```css
/* Right */
width: fit-content;
margin-inline-start: auto;
margin-inline-end: 0;
```

An explicit **Size** packet still wins if you deliberately set Width, so Quick Align does not silently undo authored sizing. Vertical placement can use Flex/Grid alignment when that layout exists; Palette warns when a plain block parent has no free vertical space to distribute. Use **Position & Layer** when you need a pinned top/bottom relationship instead.

**Quick Align** is the friendly placement primitive. **Layout Item** remains the advanced control for grow/shrink/order and raw Flex/Grid self-alignment.

### Spacing: simple first, per-side when you need it

Spacing keeps **Padding** and **Margin** as fast linked sliders for normal work. Open **Advanced padding** or **Advanced margin** directly underneath when one side needs a different value; Palette exposes Top / Right / Bottom / Left there and leaves the other sides alone. Moving the main slider afterward deliberately links all four sides again. Negative values are allowed for margin, not padding.

### Text Entry: move the typing origin honestly

Use **Text Entry** on the mounted composer textarea when the intent is simply “typing should begin here.” Horizontal/vertical inset moves entered text and the placeholder together, while font metrics stay synchronized with Lumiverse's hidden textarea mirror so auto-height measurement does not drift.

Palette deliberately keeps two responsibilities separate:

- **Text inset + metrics** belong to the textarea and its hidden autosize mirror.
- **Placeholder appearance** (ink, opacity, italic/weight) belongs only to `textarea::placeholder`.

That means Palette does **not** fake placeholder placement with transforms or pseudo positioning. If the placeholder is annoyingly glued to the upper-left corner, change Text Entry inset; the real typed text will start in the same honest place.

### Size: Fit, Fill, Fixed

Use **Fit** when the box should hug its content. Use **Fill** when it should claim the available space. Use **Fixed** when you really want a number.

If Fill appears not to fill anything, inspect the parent. A flex/grid child can only fill the space its parent actually gives it.

### Position: Nudge vs Anchored

**Nudge** is visual movement that keeps the native layout contract intact. It compiles as translation and is excellent for “this is correct, just 12px too high.”

**Anchored** means Palette owns the positioning relationship: top/right/bottom/left relative to a chosen containing block.

Use Nudge when Lumi already knows where an element belongs. Use Anchored when you are intentionally building a new composition.

### Image: wrapper vs image

This distinction saves hours.

```css
/* Avatar frame / wrapper */
[data-component="BubbleMessage"] [class*="_avatar_"]

/* Pixels inside that frame */
[data-component="BubbleMessage"] [class*="_avatar_"] img

/* Large native ghost/backdrop image — different target */
[data-component="BubbleMessage"] [class*="_avatarBgImg_"]
```

Resize the wrapper when you need a larger stage. Use **Image → Move inside frame** when the frame is correct and only the crop/focal point is wrong.

### Media Flow

Lumiverse prose images and native attachments are not the same DOM species. Palette treats both lanes deliberately:

**Markdown/XML lane**  
image paragraph → linked/span wrapper → image

**Native attachment lane**  
attachments → inline image button → inline image frame → inline image

Use **Full width** or **Unclipped** when native thumbnail chrome is fighting an authored reading layout. Do not solve every media problem with a global `img { width:100% }` missile.

### Guides

**Guides → Smart** follows the active packet:

- Spacing → box model
- Size → dimensions + containing block
- Layout → flex/grid geometry
- Quick Align / Layout Item → target + layout parent
- Position → placement/anchor relationship
- Image → crop/focal frame
- Background/Text/Shadow → lightweight outline

If a tiny text node has no useful box of its own, inspect its meaningful wrapper or parent boundary instead of assuming the guide is broken.

## Groups

**Group** is for several real siblings that should behave as one authored composition. Palette does **not** reparent React DOM and does not invent wrappers.

Group mode now has two ways to select members:

- **Visual:** click siblings on the page.
- **Structural:** retarget a picked member through nearby DOM levels, then use **Add sibling** once Palette knows the shared parent.

This is specifically useful for Lumi's favorite anatomy:

```text
row
└─ wrapper
   └─ inner wrapper
      └─ actual control
```

A valid saved group requires one real shared direct parent. Palette can temporarily let draft members sit at different depths while you line them up, but **Create group** stays disabled until the structural relationship is safe.

A Layout Group has four jobs:

| Tab | Purpose |
| --- | --- |
| **Layout** | Row / Column / Grid, columns, gap, responsive behavior. |
| **Members** | Shared packets on the member roots. |
| **Contents** | Shared descendant treatment for icons, text, buttons, or images. |
| **Frame** | One generated visual plane behind the group using the real parent. |

Unsafe cases reject instead of generating nth-child spaghetti: different parents, ancestor/descendant members, or selectors that cannot be distinguished safely.

## Read styles

### Read Style

**Read Style** reverse-engineers the selected target into familiar Palette controls.

The important contract is **Read is inspection; editing is capture**.

Observed values can appear in the controls without becoming project state. Palette only materializes the semantic operation you actually change. If you inspect a heading and only change its font size, Palette should not suddenly claim its native border, margin, color, shadow, and seventeen inherited properties.

### Read Page

**Read Page** is the larger map. It inventories mounted, visibly styled targets by semantic surface and lets you inspect where a result came from before deciding to edit it.

Use the search box for things like:

```text
heading
button
InputArea
avatar
::before
```

Source information is diagnostic context; the semantic target is the star. Click a target to inspect/capture it rather than translating an entire foreign stylesheet into semantic oatmeal at once.

## Reuse

Palette has three different reuse layers because they solve different jobs.

### Quick Looks

Quick Looks are curated starter recipes. **Apply** merges the packet types the recipe owns. The pencil applies it and opens the result in Design. Reset only peels that recipe layer back off.

The compact Quick Looks browser can switch between **Current target** and any Style Library pack without opening the full warehouse. Looks are paged two at a time in a horizontal snap strip, with tiny page markers below it. Pack browsing only changes the preview source/palette; nothing is applied until you press **Apply**.

### Fullscreen or native dock

On desktop, the Style Library opens fullscreen and exposes **Dock left** in its header. That moves the same live library state into Lumiverse's native resizable left dock so the themed conversation remains visible while you browse. The native dock can be collapsed to its labeled edge tab and expanded again without discarding Palette's live library state. When **Apply and edit** hands you into Design from a docked library, Palette tucks that dock away instead of destroying it; **Browse styles** expands the same dock again with your browse/pack/filter context intact. **Fullscreen** moves it back without reopening or cloning the library. Phones keep the full-screen presentation, with safe-area-aware geometry and a reachable Close control.

### My Styles

My Styles are your reusable semantic styling, stored above individual theme projects.

| Save mode | Captures |
| --- | --- |
| **This target** | current target + decorative surfaces |
| **This component** | authored styling under the current semantic component |
| **Choose parts…** | any checked multi-component combination as a Bundle |

Applying a My Style merges packet types. If the destination already has Border + Shadow and your saved style contains Background + Typography, the unrelated Border + Shadow survive.

Stable targets do not need to be mounted at apply time to remain reusable.

### Library cards show what they touch

Preview art is only a visual hint. Reusable recipe cards also show their **Component** and **Affects** summaries so you can tell “action row” from “content frame” before Apply, Edit, or Reset. Do not rely on preview silhouettes as semantic names.

### Packs

Packs are curated compositions built from the same semantic engine as Design: targets, packets, Base/Mobile, groups, media policy, asset slots, and provenance.

**Apply pack** should never mean “paste a giant CSS blob.” After applying a pack, every owned piece remains editable through ordinary Design controls.

**Reset pack** removes the pack's layers and reveals whatever was underneath. Manual styling and other packs remain unless that reset actually owns them.

#### Bubble and Minimal are recipe families, not one selector missile

A pack can support **BubbleMessage**, **MinimalMessage**, or both, but renderer-specific choreography stays renderer-specific. A Minimal portrait/actions/thinking recipe targets Minimal only; its Bubble counterpart targets Bubble only. Choosing/applying **Both** installs both families so switching Lumiverse's message renderer reveals the already-authored matching composition.

Only genuinely renderer-independent targets such as `MessageContent` prose, exact semantic controls, or other shared surfaces should be authored once for both layouts. "Same aesthetic" does not require "same recipe."

## Boost

App-wide controls are split by what they actually depend on.

### Boost

Boost transforms Lumiverse's native variable map. **Colors** is the main recolor layer; **Backdrop** lives inside Boost because wallpaper/canvas treatment participates in that same transformed world.

The source flow is intentionally one-way:

```text
Lumiverse canonical generated variables
        ↓
Palette transform
        ↓
Presented app variables
```

Palette should never recursively recolor its already transformed root output.

The mode control describes what happens to the native light/dark structure:

- **Recolor** keeps the source theme’s light/dark balance while applying the chosen palette/material transform.
- **Invert brightness** reverses that balance while preserving the chosen palette and material relationships. It is a persistent mode, not a one-shot shuffle effect.
- **Shuffle colors** changes only Primary and Secondary. It deliberately preserves Recolor/Invert brightness, Brightness, Contrast, Original saturation, Text treatment, and Typography so a light/inverted theme cannot silently fall back to a dark recolor just because the palette was shuffled.

Color routing is semantic rather than a two-bucket hue swap:

- **Primary accent** stays faithful to the picked color for direct primary-family tokens and leads canvas, ordinary surface, control, and glass tinting. Native primary variants keep their relative depth rather than borrowing the old theme's absolute lightness.
- **Secondary accent** stays faithful to the picked color for direct secondary-family tokens and owns the supporting **Raised / Card / Hover / Border** material families. Opposing accents are not averaged into mud: supporting materials choose a chromatic owner, while a genuinely neutral Secondary can still soften the result without injecting a fake hue.
- Lumiverse's semantic material aliases (`surface`, `surface-raised`, `surface-hover`, `surface-muted`, `input-bg`, `border-subtle`) and Chat Shell `--lcs-glass-*` colors are first-class Boost inputs. Resting glass/subtle borders stay calmer than their hover counterparts, and older `bg-*`, `card-*`, and border primitives remain mapped as compatibility fallbacks.
- **Brightness / Contrast** use a bounded lightness transform that preserves the native material ladder. The deliberate smoke test is `Brightness +100 / Contrast -100`: it should become washed-out, but deep/canvas/glass/control/raised/hover must remain visibly distinct. Neutral overlays and shadows keep their structural black/white character in Recolor mode instead of turning into pastel paint.
- **Text treatment → Auto contrast** keeps the native foreground character but repairs the main text anchor against the transformed surface using real sRGB contrast math. **Custom** lets you supply a foreground anchor while preserving the native text/muted hierarchy.
- **Protect controls** is a final local safety pass for paired control foregrounds (`primary-contrast`, deep contrast, and compatible host aliases); those foreground tokens are never recolored as accents, and the repair does not own ordinary prose.

Transform diagnostics include per-role counts for Primary, Secondary, Canvas, Surface, Raised, Hover, Card, Glass, Control, Text, Muted, Border, Neutral, Semantic, and Pass-through. If a material suddenly lands in the wrong family, the routing itself is telling you where to look.

### Typography

Typography is standalone. You can change global font family/scale without enabling palette recoloring.

This is a valid configuration:

```text
Boost colors: OFF
Backdrop:     OFF
Typography:   ON
```

### Acceptance-test nonsense that is actually useful

If you are developing Palette itself:

- **Blood Mode** — absurd red anchors; catches untransformed surfaces.
- **Pinkpocalypse** — absurd pink; catches old complex gradient values.
- Typography-only — catches accidental palette coupling.
- Wallpaper test — catches backdrop layers that remain hidden under a wash.

## Widget & Code

### Floating mini-widget

The widget is a utility cockpit, not a second full editor:

- Pick
- Guides
- Zap
- Code/selector peek
- Float/dock

Desktop: right-click the collapsed widget for its context menu.  
Touch: long-press it.  
**Hide mini widget** is reversible from Palette's sidebar/workbar control.

On phones the floating editor can attach to the top or bottom edge; the resize handle follows the anchored edge. Mobile Float also has a **density** control that cycles **100% → 80% → 60% → 100%**. Density applies only to the scrolling inspector body, so the workspace tabs, Pick/Guides workbar, edge control, Minimize, and Close remain full-size touch targets.

The mobile inspector keeps narrow empty gutters on both sides of its scroll body. Those gutters are intentional touch-safe vertical pan lanes: if a packet is mostly sliders, drag the gutter instead of negotiating with a range thumb. Palette also pins its own mobile control typography/height so the active Lumiverse theme cannot make editor dropdown labels oversized or clipped.

### Theme Source vs Generated CSS vs Custom CSS

Imported `.lumitheme` projects keep the archive's authored `theme.json` source as **Theme Source**. Palette reads Global CSS, native component CSS/TSX, component enabled state, and the asset manifest directly from the downloaded archive before asking Lumiverse to validate/import it. That keeps authored rules such as `@import` intact even when Lumiverse canonicalizes its own runtime draft. **Fork & edit** explicitly unlocks the local source snapshot; **Protect source** can lock it again without discarding edits. Older v45 projects that only contain the native bridge's canonicalized draft remain supported and are labeled as a host-draft fallback instead of pretending to have archive fidelity.

Import is genuinely inert. **Overlay preview** is an explicit opt-in because it layers Theme Source over whatever native theme is currently active in Lumiverse; that composite is useful for editing but is not equivalent to a clean native theme application. **Send to Lumiverse** is the clean apply path.

**Generated CSS** is compiler-owned and read-only. When source overlay is enabled it loads after Theme Source, so visual Design edits remain explicit overrides instead of rewriting downloaded CSS.

**Custom CSS** is your deliberate escape hatch. It stays separate from semantic project state and loads last, so it can deliberately override both Theme Source and generated Design output.

Do not treat generated CSS as persistence input. Palette should regenerate it deterministically from semantic state. Imported Theme Source is persistence input because it is source material received from the native import bridge, not compiler output.

### Native handoff and assets

The Code workspace can send/export the current work through Lumiverse's native theme bridge and import compatible `.lumitheme` data. Import is inert: it creates a new Palette project, preserves the archive's authored Global/component source as protected Theme Source, lets Lumiverse own validation and asset extraction, keeps source overlay preview off, and does not apply the theme. Export keeps native component source sections structured instead of flattening them into Global CSS.

Native Theme Assets are project-owned. Palette stores canonical `./assets/...` references so an exported native theme does not depend on a temporary browser URL. Images can be optimized, and font assets can be registered into the Typography browser.

SVG imports are sanitized before entering the project wardrobe. XML declarations and harmless old exporter wrappers can be cleaned, but scripts/events/remote-content are not treated as decoration.

## CSS field guide

Palette is designed so you do not *need* CSS, but knowing a few patterns makes debugging and Custom CSS dramatically easier.

### 1. Scope before specificity

Prefer a meaningful owner plus a stable part:

```css
[data-component="BubbleMessage"] [class*="_nameChar_"] {
  color: white;
}
```

over a naked reusable family:

```css
[class*="_nameChar_"] {
  color: white;
}
```

The second rule may style another component that happens to reuse the same local class word.

### 2. `:where()` makes strong-looking selectors easier to override

Palette often wraps authored selectors in `:where(...)` so the structural scope itself contributes zero specificity while Palette controls authority deliberately:

```css
:where([data-component="BubbleMessage"] [class*="_avatar_"] img) {
  object-fit: cover;
}
```

This is different from solving everything by piling on more IDs/classes.

### 3. `!important` is not automatically evil

A visual theme editor sometimes has to beat native component CSS. The real sin is using `!important` without correct scope.

Good:

```css
[data-component="InputArea"] [data-composer-action="send"] button {
  color: white !important;
}
```

Bad:

```css
button {
  color: white !important;
}
```

### 4. Flex and Grid: style the parent to arrange children

If you want siblings to become columns, the important rule usually belongs to their parent:

```css
.parent {
  display: grid;
  grid-template-columns: repeat(2, minmax(0, 1fr));
  gap: 8px;
}
```

A child cannot make its siblings become a grid by itself. Layout Item therefore warns when the actual parent is not a layout container. If your actual intent is only left / center / right placement, use Quick Align instead; it is designed to work in ordinary block flow too.

### 5. `width:100%` only means 100% of the available containing width

If a child still looks tiny after `width:100%`, inspect the parent chain. A 100%-wide child inside a 112px wrapper is still 112px.

Useful debugging sequence:

```text
Target width
→ parent width
→ parent display/layout
→ min/max constraints
→ overflow/clipping
```

### 6. Move the image *inside* the frame with `object-position`

For cropped images:

```css
.avatar img {
  width: 100%;
  height: 100%;
  object-fit: cover;
  object-position: 50% 28%;
}
```

Changing `object-position` moves the focal point inside the existing frame. Translating `.avatar` moves the whole frame instead.

### 7. Fades are masks, not opacity gradients

A common portrait fade:

```css
.avatar img {
  -webkit-mask-image: linear-gradient(
    to bottom,
    #000 0%,
    #000 68%,
    transparent 100%
  );
  mask-image: linear-gradient(
    to bottom,
    #000 0%,
    #000 68%,
    transparent 100%
  );
}
```

If native Lumi already applies a mask and you want the full image, explicitly clear it:

```css
mask-image: none;
-webkit-mask-image: none;
```

### 8. Pseudo-elements need `content`

A `::before` or `::after` does not exist visually until content is generated:

```css
.target::before {
  content: "";
  position: absolute;
  inset: 0;
  pointer-events: none;
}
```

For a host-local decorative back plane, the host usually needs a positioning context:

```css
.target {
  position: relative;
  isolation: isolate;
}
```

Palette adds isolation automatically when one of its generated pseudo surfaces uses a custom negative z-index.

### 9. Nudge is safer than re-owning native positioning

If Lumi already has a correct absolute/sticky/flow relationship and you only need a visual adjustment:

```css
.target {
  translate: 0 -12px;
}
```

is often safer than changing `position`, `top`, and `left` just to move it twelve pixels.

### 10. Mobile deltas should override only what changes

```css
.target {
  font-size: 30px;
  translate: 0 -40px;
}

@media (max-width: 720px) {
  .target {
    font-size: 22px;
  }
}
```

If the mobile composition should keep the Base Nudge, do not unnecessarily re-author positioning. If you need to cancel the Base movement, explicitly set the mobile translation back to zero.

### 11. Composer actions have stable identities

Prefer the stable wrappers on current Lumiverse:

```css
[data-component="InputArea"]
[data-composer-action="persona"]
[data-toolbar-action="persona"] {
  /* persona action */
}
```

The Spindle `chat_toolbar` mount is a separate extension toolbar. Do not confuse it with native composer actions just because they appear beside each other.

The current native typing lane is also intentionally split into stable semantic parts:

```css
[data-component="InputArea"] textarea[name="chat-message"]
[data-component="InputArea"] [class*="_textareaMirror_"]
[data-component="InputArea"] [class*="_sendBtnShell_"]
```

The textarea mirror is measurement infrastructure, not a second visible text box. Let **Text Entry** synchronize its metrics instead of styling the mirror independently.

### 12. CSS variables are the app-wide language

Lumiverse theme variables look like:

```css
:root {
  --lumiverse-bg: #111;
  --lumiverse-text: #f7f2fa;
  --lumiverse-primary: #b78cff;
  --lumiverse-card-bg: linear-gradient(...);
}
```

Boost transforms the canonical variable map rather than hue-rotating the rendered app. That is why images/media can remain intact while the UI changes palette.

### 13. A few useful selector recipes

```css
/* Assistant message only */
[data-component="BubbleMessage"]:not([data-part="user"]) { }

/* User message only */
[data-component="BubbleMessage"][data-part="user"] { }

/* Message prose */
[data-component="BubbleMessage"] [data-component="MessageContent"] { }

/* H2 inside message prose */
[data-component="MessageContent"] h2 { }

/* Persona drawer surface */
[data-spindle-drawer-tab="personas"] { }

/* Native send action */
[data-component="InputArea"] [data-composer-action="send"] { }

/* Only paragraphs that contain an image */
[data-component="MessageContent"] p:has(img) { }
```

### 14. Full-width prose image without stretching its aspect ratio

```css
[data-component="MessageContent"] p:has(img) {
  width: 100%;
  max-width: 100%;
  margin: 14px 0 22px;
  text-align: center;
}

[data-component="MessageContent"] img {
  display: block;
  width: 100%;
  max-width: 100%;
  height: auto;
  margin-inline: auto;
  object-fit: contain;
}
```

For native Lumi attachments, use Palette's **Media Flow** rather than assuming this raw prose selector also owns the attachment button/frame wrappers.

## Debugging

When something “does nothing,” check these before inventing new infrastructure:

| Symptom | First thing to inspect |
| --- | --- |
| Border is mysteriously invisible | color alpha / opacity is not literally 0%. Yes, this happened. |
| `width:100%` is still tiny | containing parent width + max-width. |
| Element will not move as expected | native `position`/inset contract; try Nudge. |
| Image crop is wrong | wrapper size vs image `object-position`. |
| Mobile layer vanished | native `display:none` can defeat mere `visibility:visible`. |
| Style leaks to unrelated UI | selector lost its semantic owner/context. |
| Read Style created too much CSS | observed values should stay unowned until edited. |
| Pack reset resurrects something deleted | provenance/ownership detach is wrong. |
| `::after` decoration sits on top instead of bottom | mobile override accidentally reset Base positioning. |
| Composer selector broke after upstream change | use stable `data-composer-action` / `data-toolbar-action` identity. |

A good escalation order is:

1. Check the actual authored packet values.
2. Check the mounted target and parent geometry.
3. Inspect the generated selector in **Code**.
4. Use browser DevTools to inspect the winning declaration/cascade.
5. Only then decide whether Palette is missing a primitive or selector boundary.

> **Mounted visual truth wins.** A mathematically elegant selector or positioning scheme that does not match the real screen is still wrong.

## Credits

Palette drawer icon: **paintbrush by Alum Design (CC BY 3.0)**.  
Palette is built as a Lumiverse Spindle extension and intentionally keeps its internal `theme_studio` identity stable for persistence/runtime compatibility.


### Editorial left proof rail

Reader Correspondence intentionally replaces native Minimal user-side rail distribution with its own left editorial proof rail. The frame owns the real 106px rail reservation and switches to ordinary block flow so the host flex row cannot keep a ghost lane after the avatar/actions are anchored out of flow. The header stays padding-free, while portrait and proofmark actions share the same rail axis.

On mobile the desktop proof rail is released, but the portrait is still anchored against the card so it does not consume a separate row. The compact byline reserves only the portrait width, which keeps name + metadata beside the image while prose and proofmark actions continue in ordinary full-width flow below. Editorial assistant mobile uses the same principle: portrait + byline at the top, actions released to the bottom instead of occupying the native right-side action lane. The generic Minimal actions overlay must materialize before Editorial's Author Rail so the phone-only flow reset wins the pack cascade. SwipeControls use their own button/counter semantic subroles; flatten those native controls directly instead of compensating around the outer pager shell.

### Editorial Contributors kicker

The Contributors masthead uses `chat.roster.bar::before` as a generated publication kicker. The host roster can carry its own pseudo positioning, so Editorial explicitly resets that surface to normal flow, zeros inherited pseudo padding, gives it a compact fixed footprint, and centers it as a non-shrinking flex item before the contributor cards. Do not compensate for overlap by moving the member cards themselves.


### Editorial reasoning marginalia

Editorial reasoning deliberately avoids the native full-width outlined lane. Both BubbleMessage and MinimalMessage use the same publication language: a transparent shell, a narrow cool-slate marginal rule/wash, small Georgia italic live duration text, and a restrained serif body when expanded. Desktop reasoning is constrained to the reading column; mobile releases it to full width rather than preserving a percentage lane.

Current Lumiverse mounts `button[data-reasoning-toggle="true"]` as both the reasoning header and the toggle surface. Keep the header responsible for chrome and the toggle role responsible for ink only. If both roles author background/border, the later toggle packet can erase the header treatment even though Palette appears to have styled both correctly.


### Visual Novel Minimal: classic dialogue stage, not Editorial-with-neon

Visual Novel deliberately uses two renderer grammars. **BubbleMessage is the active cinematic scene**: large scene choreography, choice windows, route HUD, and more theatrical game chrome belong there. **MinimalMessage is the compact classic VN dialogue stage**: centered landscape portrait, fading character banner, a game-like gradient speaker plate, translucent/double-line patterned dialogue frame, and authored VN furniture. The assistant action group may use stacked text plates while mobile releases it back into compact flow; Greetings can become a narrow status bar below dialogue rather than a generic pill. Do not collapse Minimal back into an Editorial author column or a smaller Bubble card stack.

The Minimal root intentionally owns generated scene decoration. `minimal.decorative-rail.assistant` / `.user` target the message `::before` surface as a fading banner behind the centered portrait and name plate; the side-specific corner-ornament roles use `::after` for restrained VN embellishment. Because that `::before` is authored content, the Visual Novel pack must **not** apply `minimal-native-strip-off`.

Identity lives above the dialogue frame rather than inside an opaque header card. The header itself stays transparent and centered; the speaker name becomes a gradient plate while desktop metadata remains in ordinary header flow so long dialogue cannot drag the pill into the middle of the response. Mobile may deliberately anchor the compact metadata back near the portrait. Greetings is flattened into a narrow status bar below the dialogue frame instead of competing with the portrait.

After mounted QA, the player side now **transposes the same stage geometry** instead of shrinking into a separate card grammar: both sides share the 92% stage, centered landscape portrait, 76% transparent identity lane, and broad double-line dialogue window, with assistant blue/lilac and player rose route families carrying the side distinction. Desktop actions use the same stacked text-command grammar on both sides; mobile releases both into compact wrapped flow.

Reasoning is an **Inner Voice plate** beneath the name and slightly over the dialogue frame. Message actions are text-first VN controls generated over the real native buttons, including dedicated static roles for Edit, Copy, Hide, Anchor, Fork, Prompt, and Delete. Native SVGs are hidden without replacing the click targets. The omitted-action fallback must exclude every named action, including Copy.

Swipe navigation belongs at the **bottom-left** as compact route furniture in normal flow. `minimal.swipes.previous` and `.next` replace native chevrons with maskable built-in arrow SVGs; `minimal.swipes.counter` owns the route count; `minimal.swipes.ornament` supplies a small decorative flower. Keeping the pager in flow prevents long messages from separating the controls from the actual dialogue footer. The long-message toggle is also scoped under Minimal so it cannot inherit Bubble's serif/purple continuation chrome.

Typography remains renderer-specific. Bubble VN keeps its cinematic serif language. Minimal dialogue/body copy uses compact UI sans and mono metadata through `minimal.prose.*`, applied after shared `visual-novel-prose`, so fresh Apply All preserves the renderer split. Those VN roles themselves remain static DOM anatomy. Palette project/state schema is currently **v46**. Imported `.lumitheme` CSS persists as a protected, preview-off Theme Source layer, separate from Palette-generated Design CSS and Custom CSS; v43 remains the SVG / Icon replacement metadata boundary and v42 remains the Image/Mask split migration boundary.
