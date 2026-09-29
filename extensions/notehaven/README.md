<!-- Mirror of upstream documentation. Everything below the metadata table is the original file, unmodified. -->

| Field | Value |
| --- | --- |
| Extension | Notehaven |
| Source repository | https://github.com/hans123m12m1/lumiverse-notehaven |
| Original link (from Lumiverse-Extensions README) | https://github.com/hans123m12m1/lumiverse-notehaven |
| Upstream path | `README.md` |
| Retrieved | 2026-09-29 @ `main` (`ff38688`) |

---

# Notehaven 🌙

A cozy **Lumiverse** (by [Prolix](https://github.com/prolix-oc/Lumiverse)) extension built on the **Spindle** framework:

- 📓 **Notes** — an Obsidian-inspired (but homier) notebook right in the sidebar drawer, with `[[wiki links]]`, `#tags`, pins, colors, live markdown preview, checklists, and **embedded media**: images, animated GIFs, **videos (mp4/webm/mov/mkv/avi/3gp…) and audio (mp3/wav/ogg/m4a/flac/opus…)** up to ~16 MB each. Uploads are type-checked against a media allow-list, so only real images/video/audio get stored.
- ✨ **The Halo** — your own logo floating over the Lumiverse chat. **Drag it anywhere, resize it (24–256 px), snap it to screen edges, or hide it entirely.** Double-click it to jump into your notes.

No build step required — `dist/` is hand-written, dependency-free ES2022 and installs as-is.

- 🗂 **File Tree Navigator (2.0)** — a second drawer tab with an Obsidian-style tree: nested subfolders, 📌 Shortcuts + 🕘 Recent + 🏷 Tags sections (collapsible & reorderable), 6 sort modes, smart search (`words -exclude #tag folder:name OR…`), drag & drop, and word/token badges that roll up through subfolders.
- 🗄 **Tabs & split panes (2.0)** — browser-style tabs (drag to reorder, right-click to pin/split), **Split right / Split down** with draggable dividers, up to 5+ nested panes. Unfocused panes keep a live reading view of their note.
- 🪟 **A real workspace window (2.0)** — drag it anywhere (position is remembered), minimize-to-header, fullscreen, resize from both corners and both edges, back/forward history, Find & Replace (Ctrl+F), and three view modes: ✍ Live · 📖 Reading · ⌨ Source.
- ⚙️ **Deep settings (2.1)** — the full File-Tree-spec preference sheet: editor behaviors (new-tab-vs-replace, default tab view, strict line breaks, fold headings/lists, line numbers, indent guides, RTL, line height, spellcheck + languages, auto-pair, smart lists, tab/space indents) and navigator behaviors (expand-on-select, one-branch, collapse scope, keep-selected, spring-loaded folders, icon overrides, color-to-icons-only, 7 counter toggles, shortcuts/recent/folder/tag visibility, tags-as-folder, “Untagged”, recent count) plus confirm-before-delete and a 🗑 Trash folder mode. All in Settings, applying live.
- 📌 **Real pinning (2.0)** — pinned notes own the *top* of the list in their own section; folders pin too. Notes & folders take icons from an svg library, emojis, or uploaded pictures (tap a folder icon for a preview popover — works on PC & mobile). Duplication names sequentially: `name.txt` → `name (1).txt`.


---

## What's new in 2.5.5 — clear backdrop 👁

"Moving it works, but I want to put it next to the chat without the
background obscuring." Exactly — new **Clear backdrop** toggle:

- 👁 Kills the dim + blur wall behind the notes window, applies live.
- 🖱 The chat behind stays **fully clickable/scrollable** — touches fall
  through everywhere except the notes window itself. Type in chat with your
  notes parked beside it (pairs beautifully with the phone float window or
  a dragged-aside desktop window).
- One tap in the **⋯ menu**, plus a switch in **Settings → Appearance**.
  Saved per user; flip back for focus mode anytime.

## What's new in 2.5.4 — halo transparency & ghost mode 👻

Requested: make the Halo translucent. Delivered two dials in
**Settings → Halo → Comfort controls**:

- 🫥 **Transparency slider** — 15% faint ghost to 100% solid, live preview
  as you slide, saved per user.
- ⭕ **Backdrop ring toggle** — turn off the tinted body/ring behind the
  logo for a clean transparent orb (great look for custom icons).
- 🔒 **Your custom icon is never touched** — these change only how the orb
  is painted, never which image it shows (broken images still self-heal to
  the built-in orb, as before).

## What's new in 2.5.3 — float is now the phone DEFAULT 📱🎈

"At 100% zoom it really does eat the whole screen." Fixed properly: **phones
now open notes in the centered, draggable float window by default** — no
full-screen sheet unless you ask for it.

- 📱 Existing phone users are **migrated** too (the old `phoneFloat` key is
  swept at boot) — nobody needs to find a hidden toggle.
- 🎈 One-time intro toast explains the new window and where the dock-back
  lives: ⋯ menu → **Dock to bottom**, or Settings → Editor →
  **"Dock to bottom — classic sheet"**.
- Everything from 2.5.0 still applies: drag by the top pill, double-tap to
  re-center, position persists, keyboard-safe clamps.

## What's new in 2.5.2 — the whole header went vector ✨

Emoji → SVG, everywhere in the chrome (requested with a very reasonable
annotated screenshot): history ◀▶, minimize/fullscreen (including their
toggled states), ⋯ and ⋮ menus, ✕ closes (header, find, settings), the ✍
Live / 📖 Read / ⌨ Source mode pills — now pen, open-book and code-bracket
vectors — plus the drawer brandmark moon. All inherit the accent color and
stay pixel-crisp on any screen.

## What's new in 2.5.1 — real sun & moon icons ☀️🌙➜✨

Friend request: the theme toggle's ☀️/🌙 emoji are now **real vector SVG
icons** (feather-style sun + crescent moon). They stay razor-sharp at any
size, inherit the accent color like every other Notehaven icon, and the
button keeps its tooltips. Icon buttons also got a sizing rule so any future
SVG drops in perfectly.

## What's new in 2.5.0 — phone float window 🎈

For Android (and any phone): the notes UI no longer HAS to be a full-screen
bottom sheet. Flip **Float window** and it becomes a real little window in
the middle of your screen that you drag around.

- 🎈 **Float toggle** — topbar ⋯ menu → *"Float window — drag the UI around"*
  (also in Settings → Editor). One tap, applies live, remembers your choice.
- 🖐 **Drag the whole UI anywhere** — the grab pill on top becomes a move
  handle (touch pointer events, Android + iOS). Position persists.
- 🎯 **Sits in the middle** — first float opens centered at ~78% height, the
  exact "the sheet covers most of their screen" complaint, solved.
- 👆 **Double-tap the pill** → jumps back to the middle.
- Keyboard opening or rotating the phone re-clamps the window so its header
  is never lost off-screen; nurse watchdog leaves float mode intact.

## What's new in 2.4.2 — the Halo you'll actually recognize 🌙📖

Plot twist from a user screenshot: the Halo was alive all along — the old
default logo (a plain lavender sparkle) looked EXACTLY like the host's own
floating buttons, so nobody could pick it out of the crowd. And if its image
ever failed to decode, the bubble had no body of its own = invisible glass
that every health check called "fine". Now:

- **New default glyph** — a crescent moon over an open book on the lavender
  orb. It reads as *notes*, not as yet another AI-sparkle button.
- **A real body** — the orb now has an accent-tinted backdrop + ring, so it
  is visible even while an image is loading or if one fails.
- **Dead-logo self-heal** — a broken custom logo swaps back to the built-in
  orb automatically, with a one-time toast saying it happened.
- **'Here I am' pulse** — after install/update it pulses twice and the hello
  toast literally describes what to look for. Plus a hover tooltip.

## What's new in 2.4.1 — Halo rescue kit 🛟

Still the stable 2.2.4 code — plus four small, purely-additive safeties that
can only make the Halo APPEAR, never hide it:

- **The logo-block merge** — the smoking gun. Boot merged every settings
  section except `logo`, so an old/corrupt save without it (or with a
  nonsense size) made `createHalo()` throw and the Halo silently never
  existed. Old saves now get healed defaults (`64px · visible · snap`).
- **Boot rescue timer** — 2s after start, if the bubble truly isn't on the
  page, it's force-rebuilt with sane defaults and a "Halo rescued ✨" toast.
- **Honest hidden state** — if the Halo is simply *hidden* by the 🙈 option,
  Notehaven says so and points at the way back (Extras → `toggle-halo`, or
  Settings → Logo → Show logo); "Reset spot" now un-hides it too, and the
  watchdog stops false-warning "browser blocked the logo" on purpose-hides.

## What's new in 2.4.0 — rollback to the stable 2.2.4 build 🔙

You asked, you got it: this release is **exactly the 2.2.4 code** — the
last build where the Halo and the whole workspace behaved on your install.
The 2.3.0 "visual-viewport seat" and 2.3.1 "unphaseable halo" experiments
are **fully removed**, not layered over. (It's *numbered* 2.4.0 only so the
Extensions panel accepts it as an update over the broken 2.3.1 — inside,
it's your good old 2.2.4.)

- ✅ Everything 2.2.4 had: boot retry + degrade, folder-icon healing,
  workspace mending, note imports, safe-area sheet, sideways-scrolling
  topbar, top-scroll pill, right-click menu fix.
- 🗑 Everything 2.3.x removed: overlay viewport pinning, anti-zoom font
  overrides, halo viewport-box clamping.

## What's new in 2.2.4

- **No more `failed to start: request timed out (get_settings)`.** Right
  after an update the backend worker can still be spinning up — Notehaven now
  retries the handshake (3×, with progress toasts and a 45s window), and if
  the backend truly won't answer, the panel **opens anyway with defaults**
  instead of dying. Your notes stay safe on disk; closing and reopening a few
  seconds later re-syncs everything.

## What's new in 2.2.3

- Even on webviews that ignore **both** `dvh` and CSS variables, the portrait
  sheet now snaps into place: when the seat watchdog detects the topbar off-
  screen it forces **raw measured pixel geometry** (visualViewport) directly
  onto the panel — nothing left for the browser to misinterpret. The pill's
  manual sizing releases that override automatically.

## What's new in 2.2.2 — every-screen portrait fix 📐

- Root cause of the clipped portrait topbar: mobile browsers **lie about the
  viewport height** (`100vh`/`100dvh`/`innerHeight` all include space hidden
  under URL bars or notches — some webviews don't even support `dvh`).
- The sheet is now sized by **`visualViewport.height`** measured in JS —
  the actual visible area on ANY device — and re-measured automatically when
  the URL bar shows/hides, the keyboard opens, or you pinch-zoom.
- The seat watchdog (2.2.1) now also catches a header clipped above the
  screen using the same measured truth.

## What's new in 2.2.1

- **Portrait topbar can never get lost again.** Notch handling moved out of
  fragile negative-margin math into the grab handle itself, and a seat
  watchdog checks the header is on-screen every time the panel opens, rotates
  or resizes — if a weird phone/webview ever pushes it away, the sheet is
  reseated automatically. Landscape/desktop layout is untouched.

## What's new in 2.2.0 — "mend my mess" release 🧹

- **Split-graveyard fix:** panes that multiplied into a field of
  “empty pane” placeholders get cleaned automatically at boot (when every pane
  is empty, the layout folds back to one), and you can reset anytime yourself:
  editor **⋮ → 🧹 Reset panes to one**, or **Settings → ⌨ Editor → 🧹 Reset to
  one pane**.
- **Import is now unmissable:** an **⬆ button** lives in the note rail header
  (next to the counter). It accepts .md / .json / .txt / code files, same flow
  as ⋯ → Import notes.
- **Broken folder pictures heal themselves:** if a folder's uploaded icon was
  deleted, the chip falls back to the default 📁 instead of a broken-image box.

## What's new in 2.1.9

- **Phone scroll controls:** scroll regions inside the panel now show a slim,
  visible, draggable scrollbar, and a floating **⤒ pill** appears once you've
  scrolled — one tap hops straight back to the top (toolbar / title). Handy
  on real phones and on emulator mice alike.
- Small flex fix so a long note can never push the header rows off-screen.

## What's new in 2.1.8

- **Right-click on the Halo opens the menu — not the notepad.** The tap
  detector now only counts the primary button, so a right-click release stops
  reading as a "clean tap". No more notes window opening behind the menu,
  and no more click-and-hold needed on PC (that's the phone gesture).

## What's new in 2.1.7

- **The chat stays scrollable behind the notes panel on phones.** Drag the
  sheet down with the grab pill, then keep reading/scrolling your chat above
  it while you jot notes — the dimmed backdrop no longer eats your touches.
  (Inside the panel everything scrolls with native momentum as always.)

## What's new in 2.1.6

- **Topbar swipes sideways on narrow screens** — the whole button row
  (history, list, minimize, fullscreen, theme, settings, New note, ⋯) is one
  clean horizontally-scrolling line. The close ✕ is always a swipe away.

## What's new in 2.1.5 — proper phone layout

- **Notch/Dynamic-Island fix:** the phone sheet now respects
  `env(safe-area-inset-top)`, so the header is no longer swallowed on notched
  iPhones (and the home-indicator area is padded at the bottom).
- **Adjustable sheet:** drag the grab pill at the top of the panel to size it
  between 55% and 100% of the screen — your height is remembered. Double-tap
  the pill to snap back to full screen. Same code covers Android.
- The sheet is anchored to the **bottom** and gets rounded corners when
  windowed, so the chat above stays peeking through.
- Toolbar rows pack tighter on small screens.

## What's new in 2.1.4

- **Fixed: tapping the floating logo on phones now actually opens notes.**
  The tap no longer relies on the synthetic `click` event (mobile browsers
  often swallow it); a clean lift-without-move is the tap, and the pointer
  capture waits until you really start dragging. The test suite simulates a
  phone tap to prove the modal opens.

## What's new in 2.1.3 — the "it actually boots now" hotfix

- **CRITICAL FIX:** v2.1.0–2.1.2 shipped a startup race — the Settings cards
  rendered before the new editor defaults were backfilled, so `lineHeight` was
  `undefined` and the whole frontend crashed on load (`settings.editor`…
  TypeError, caught only by the boot toast). Result: **no modal, no drawer
  tabs, no Halo, anywhere.** This was the real reason nobody saw the widget.
  Defaults are now a single shared table applied at BOTH setup and boot.
- **Fixed:** a leftover code fragment from the 2.1.2 Halo surgery that could
  swallow part of the Halo wiring.
- **New permanent regression net:** the test suite now executes the real
  `setup()` end-to-end in a stub host (boot, drawer tabs, input-bar actions,
  Halo mount, teardown) — broken-boot bugs cannot ship again.
- Everything from 2.1.2 stays: the Halo renders itself (no permissions, works
  on phones), drag/snap/tap/long-press, self-heal watchdog, zero-permission
  install, `{{notehaven::Title}}` macro + `{{nhnote::…}}` alias.

## What's new in 2.1.2

- **The Halo is finally, reliably VISIBLE — on desktop AND phones.** The host
  floating-widget API is gone: Notehaven now renders the logo itself, straight
  onto the page. No permissions to grant, nothing for the host to silently hide.
  Drag it (mouse or touch), snap-to-edge, tap to open, long-press/right-click for
  sizes, Ctrl+scroll to resize on PC — everything persisted.
- A self-healing **watchdog** rebuilds the logo if your browser ever eats it.
- Zero-permission install: the extension no longer asks for `ui_panels`.
- If you hid the logo earlier and forgot: input-bar **Extras → “Show Chat Logo”**.

## What's new in 2.1.1

- **Fixed:** `Cannot override built-in macro: note` — Lumiverse owns a built-in
  `{{note}}` macro, so Notehaven now registers **`{{notehaven::Title}}`** plus the
  short alias **`{{nhnote::Title}}`** instead. Update any prompts using the old name.
- **Fixed:** the Halo could be completely invisible (reported on both mobile and
  desktop) when the host refused the floating widget (`ui_panels` not granted or
  widgets unsupported). Notehaven now falls back to its **own self-rendered bubble** —
  same logo, drag it around, snap-to-edge, position persists, taps/long-press/ctrl-scroll
  all work. Grant `ui_panels` whenever you want the native widget back.
- **Reminder:** on any device you can always open your notes from the chat input bar
  **Extras → “Open Notes”**.

## Installation

1. Push this folder to a GitHub repository (replace the `github` / `homepage` fields and `author` in `spindle.json` with yours).
2. Open **Lumiverse → Extensions** panel.
3. Click **Install Extension** and paste your repo URL
   (or `POST /api/v1/spindle/install`).
4. Grant the one requested permission: **`ui_panels`** (needed for the floating logo; notes work entirely on free-tier APIs).
5. Look for the **Notes** and **Halo** tabs in the sidebar drawer — or press `Ctrl/Cmd+K` and type "notes".

The extension auto-seeds a *Welcome* note on first run so you always land somewhere friendly.

## The Notes modal

Notes open in a big, comfy **centered window** (the sidebar **Notes** tab is a launcher with recent notes — or double-click the Halo logo):

- **Left rail** — all your notes: search, click to open, right-click to manage
- **Right side** — the editor: title, formatting toolbar, Edit/Split/Read views, status bar

| Feature | How it works |
| --- | --- |
| Create / open / delete | `＋ New note` button, list click, trash (two-click confirm) |
| **Rename** | Edit the title at the top, or right-click a note → ✏️ Rename |
| **Pin / recolor / duplicate** | Right-click any note in the list |
| **Images** | 🖼 toolbar button → pick a file → it's downscaled (1600 px), stored per-user, embedded as `![name](nh-img://id)`. Click an image in the preview for **fullscreen zoom** or **remove**. |
| `[[Wiki links]]` | Type `[[Note Title]]` — click to open; **pink links create the note on click** (Obsidian-style) |
| `#tags` | Typed anywhere in a note; shown as chips and matched by the search bar |
| Checklists | `- [ ]` / `- [x]` — **toggleable right in the text** |
| ✍ Write / 📖 Read | One **live-preview surface**: images, wiki-links, tasks and formatting render inline while you type. Read mode locks the surface and hides the toolbar |
| Search | Filters by title, content snippet and tags |
| Autosave | Debounced ~1 s, with a live "Saving…/Saved ✓" indicator + word & character count |
| Close | ✕ button, `Esc` key, or click the dimmed backdrop |

Everything is stored **per user** via `spindle.userStorage`, so multi-user installs keep everyone's notes private.

## The Halo (chat logo)

All Halo controls live **inside Notehaven → ⚙ Settings** now — no separate drawer tab eating space:

- **Upload logo** — any PNG/JPEG/WebP/SVG, or an **animated GIF** (kept fully animated!), persists across sessions. Big files upload in chunks — no more timeouts.
- **Resize** — a comfy slider (24–256 px) in the tab, or right-click the logo for Small/Medium/Large/Giant presets.
- **Move** — just drag it; **snap-to-edge** docks it like a magnet when you let go. It parks itself near the bottom-right corner by default; reset its spot anytime.
- **Show / hide** — the tab switch, the right-click menu, *or* the **"Toggle Chat Logo"** action in the chat input bar's Extras popover (there's also an **"Open Notes"** action there).
- **Tap / click** the logo to pop your notes open instantly (drag-tolerant — it won't misfire while you move it, and it listens on several event paths so host drag handlers can't swallow your click). On phones, **long-press** it for the menu.
- **Resize on the spot (PC)** — **Ctrl+scroll** right on the logo; the Settings slider and right-click size presets work too.

## Folders, organization & bulk actions

- **Folders / categories** — no side bars, no tabs: folders live right inside the **All** list as section headers with their notes grouped neatly beneath. Tap a folder row (chevron included) to **collapse it like a dropdown** ▾ — your collapsed choices are remembered across sessions. Create folders with **＋ New folder** at the bottom, rename inline, and right-click (long-press) a folder for more: emoji or **custom image icons**, delete (keep notes → Inbox, or delete everything)
- **Dark & light themes** — the ☀️/🌙 button in the notes header flips instantly; Settings → **Theme** offers Auto (follow Lumiverse) / Dark / Light
- **One-click pins** — 📌 the pin button toggles instantly (no menus): pinned notes sort to the top of the list, and errors surface as toasts instead of failing silently
- **Custom background** — set any picture behind your notes from Settings → Appearance, with a "veil" slider that keeps text readable in dark *and* light mode 🖼
- **Move notes** — drag them onto a folder's section header (PC), right-click → *📁 Move to…*, or create notes directly inside a folder; new notes land in the folder you're browsing
- **Multi-select** — tap ☑ in the list header: check individual notes or **All**, then **Move** or **Delete** in bulk
- **Resizable UI** — drag the rail's right edge on PC (slider in Settings too), drag the window's bottom-right corner to resize the whole sheet (phones stay comfy full-screen)

## Import & Export

Open the **⋯ menu** in the notes modal header:

| Option | What you get |
| --- | --- |
| **Export this note (.md)** | Plain Markdown with the title as an `# H1` — Obsidian-friendly. Embedded images stay as `nh-img://` references. |
| **Export this note (.json)** | The note *plus* its embedded images, re-importable anywhere. |
| **Export ALL notes (backup .json)** | Full backup of every note + every image. |
| **Import (.md / .json)** | `.md`/`.txt` files become new notes (a leading `# Heading` becomes the title). Notehaven `.json` files restore notes *with* images; id collisions are safely regenerated. |

Right-clicking a note in the list also has **Export (.md)**.

**Import understands more than Markdown:**

| File | What happens |
| --- | --- |
| `.md` / `.markdown` / `.txt` | Becomes a note (first `# H1` → title) |
| `.json` | Notehaven backup restored *with images*; any other JSON becomes a fenced code note |
| `.docx` | **Real Word text extraction** (dependency-free OOXML zip parsing) |
| `.toml`, `.yaml`, `.csv`, `.ini`, `.log`, `.py`, `.js`, … | Imported as a fenced code-block note (` ```toml ` etc.) |

## Settings (⚙ next to the list toggle)

- **Accent color** (or follow the app), **window opacity**, **backdrop dim & blur**, **editor font size**, **corner roundness**
- **Custom CSS** — free-form styling, applied live
- **Export / Import theme** — bundle it all as `notehaven-theme.json` and share it

Everything persists per user at `{DATA_DIR}/users/{userId}/extensions/notehaven/`.

## Phones & tablets

- The notes modal goes **full-screen** on small screens; the note rail slides over the editor and tucks itself away when you pick a note.
- **Split mode stacks vertically** (editor on top, preview below) on narrow screens.
- Touch-sized buttons, tap-to-open Halo, long-press context menus everywhere.

## Prompt macro

Insert any note into a prompt or preset block:

```
{{notehaven::Story Ideas}}   # or the short alias: {{nhnote::Story Ideas}}
```

Matches titles case-insensitively (exact, then prefix), strips embedded images into `[Image: …]` placeholders, and is marked `volatile` so it always reads fresh content.

## Repository layout

```
notehaven/
├── spindle.json        # manifest (identifier: notehaven)
├── dist/
│   ├── backend.js      # userStorage persistence, RPC handlers, {{notehaven}} macro
│   └── frontend.js     # drawer tabs, editor, preview renderer, Halo float widget
└── README.md
```

### How it talks

The frontend and backend communicate over Spindle's JSON messaging with a typed
request/response envelope (`{ type, requestId }` ⇢ `{ type: 'nh:result', requestId, ok, data|error }`).
All writes are text/JSON — images are stored as base64 data URLs, which keeps the
extension on free-tier APIs and away from binary scanner heuristics.

### Permissions requested

| Permission | Why |
| --- | --- |
| `ui_panels` | Floating Halo logo widget (`ctx.ui.createFloatWidget`) — gracefully degrades if not granted |

Everything else (drawer tabs, input-bar action, context menus, storage, macros, toasts, file picker) is **free-tier**.

## License

[Lumiverse Community License v2.0](LICENSE.md) — the same license as the Lumiverse platform itself. Personal & non-profit use, view it, fork it internally; contribute improvements back; no public redistribution or commercial hosting without permission.
