<!-- Mirror of upstream documentation. Everything below the metadata table is the original file, unmodified. -->

| Field | Value |
| --- | --- |
| Extension | Vellum II |
| Source repository | https://github.com/valdoix/vellum-engine |
| Original link (from Lumiverse-Extensions README) | https://github.com/valdoix/vellum-engine/tree/main |
| Upstream path | `README.md` |
| Retrieved | 2026-09-29 @ `main` (`69eca23`) |

---

# VELLUM II — *Palimpsest*

**A memory and continuity engine for [Lumiverse](https://lumiverse.chat) roleplay.**

VELLUM II watches your story as you play it and quietly keeps track of everything that matters — who's in the room, how people feel about each other, what each character secretly knows (and what they're wrong about), what's happening off-screen, and what happened fifty messages ago that the AI would otherwise forget. It then feeds the *relevant* slice of that history back into the AI on every turn, so your characters stay consistent and the world keeps its shape over a long story.

**VELLUM II is a matched set: preset + extension, designed to work together.** The preset tells the AI how to write and what to track. The extension reads that tracking data and builds a living chronicle of your story. You *can* use one without the other, but they're strongest as a pair — like a pen and paper, not two separate tools.

- **The preset** — the instructions sent to the AI. It shapes the *prose* (how it writes) and asks the AI to append a small, hidden status report at the end of each reply.
- **The extension** — a Lumiverse add-on that reads those hidden reports, builds a living "chronicle" of your story, and shows it to you in a beautiful panel with tabs for cast, relationships, a relationship graph, a timeline, and more.

> A *palimpsest* is a manuscript page that's been written, scraped clean, and written over again — with the older layers still faintly readable underneath. That's the whole idea: your story's present is the top layer, and its entire history stays legible beneath it.

---

## Creator's Note

This extension has been crafted with care but hasn't been tested on mobile devices yet. For the best experience on desktop, I recommend adjusting your Lumiverse panel width:

**Settings → Panel Width → Custom → 42vw** (or more, to taste)

I personally use 42vw — it gives the Chronicle tabs breathing room without crowding the chat. Adjust to your preference.

**On native thinking:** you don't need it. The preset's Reverie block already does the model's planning and reasoning on the page, so native/extended thinking is redundant and I don't recommend turning it on. That said, nothing breaks if you do — leave it on if you prefer it.

This project was built with the assistance of **Claude Opus 4.8** — engineered through conversation, iterated in real time, and tested against real long-form stories.

---

## What's New in This Version

> **2.1.0-beta.7** — more reliable state-block repair on reasoning models, and an opt-in "show the model its last block" format aid.

### Block repair now works on DeepSeek (and other reasoning models)

A few users reported that even the new **Repair state block** button showed *"Could not reconstruct the state block from the prose"* on DeepSeek. Two things were fighting the repair:

- DeepSeek (and other reasoners) leak their `<think>…</think>` reasoning into the reply even when thinking is asked off. The repair used to grab the first `{…}` it saw — which was often a stray brace *inside* the reasoning — and give up when that didn't validate. It now **strips reasoning tags and code fences first**, then scans **every** JSON object in the reply and keeps the first one that's actually a valid state block.
- Reasoning models often return an empty reply when thinking is forced off. The repair now **escalates automatically**: if the first (thinking-off) attempt comes back empty or unusable, it retries once with thinking on and a larger budget, so the block can land in the reasoning channel where it's still recoverable.

The repair prompt was also hardened against **Gemini's** habit of dressing structured output up as prose — it now insists on raw JSON only, no preamble, no code fence, no commentary.

### Show the model its own last block (opt-in format aid)

There's a new **⟦⟧ Block example** toggle in the Actions menu. When on, VELLUM injects your **previous turn's actual `<vellum>` block** as a worked example, placed right at the end of the injection — closest to where the model starts writing, where it's most likely to be imitated.

This is for models that keep forgetting the format or drifting from it: instead of an abstract spec, they see a concrete, story-specific example of exactly what they wrote last time. It's **off by default** because it costs roughly 400–700 extra tokens per turn; turn it on only for a model that needs the nudge.

---

### Previous version highlights (2.1.0-beta.6)

> A manual state-block repair button, and a fixed greeting that seeds the cast correctly.

### Repair a missing state block on demand

Auto-Repair (in the Actions menu) already rebuilds a dropped `<vellum>` block for you automatically — but only when it's toggled on, and only once per reply. If it's off, or its single automatic attempt didn't land, there was no way to try again short of regenerating the whole turn.

The floating window's title bar now has a **Repair state block** button (↻⌸). Click it and VELLUM reads the latest reply's prose, reconstructs the missing `<vellum>` block, and folds it into the chronicle — no toggle required, and you can retry as many times as you like. It works even on replies that never emitted a **reverie** planning block (some models, DeepSeek especially, routinely skip it), because the button only cares about the state block. If the latest turn already has a valid block, it tells you so instead of stacking a duplicate.

### The opening scene's character now shows up immediately

A first-message greeting almost never carries a `<vellum>` block, so the character the card is about used to stay missing from the cast until the model happened to re-introduce them a turn or two later. VELLUM now **seeds the cast from the character card** on the opening turn, so the tracker isn't blank when you first open it.

This also fixes a bug where an untitled chat could seed a cast member named after the chat's creation timestamp (e.g. *"Jul 19, 2026, 10:37:00 PM"*). Timestamp-like names are now rejected outright, so only a real character name is ever seeded.

---

### Previous version highlights (2.1.0-beta.5)

> A variables editor right inside the preset tab, a live sample-paragraph preview, and significantly smarter per-model errata.

### Configure your variables without leaving the preset editor

When you open **VELLUM II** in Lumiverse's preset editor, a **VELLUM tab** now appears alongside the usual tabs. Inside it you'll find a **grouped variables editor** — every prompt variable neatly laid out with labels, organized by category, using the same dropdown/toggle/slider controls you already know from the Prompt Variables menu.

There's no need to scroll through raw JSON or navigate away from the preset editor. Change a setting, hit **Save variables**, and you'll see a "Saved ✓" confirmation. Changes go straight to the preset in the database.

> **One thing to know:** Lumiverse's own "Configure prompt variables" modal (in the native preset editor) won't show your changes until you **reload Lumiverse**. This is a host limitation — the preset data is correct in the database, but the native view reads from its own in-memory copy and only refreshes on page load. The note in the tab says this plainly.

### Sample paragraph preview

Below the stat strip there's a new **Sample Paragraph** panel. Once you have a chat open:

1. Click **Generate sample ✦**
2. The extension assembles only the **prose-shaping blocks** (register, era, tonal cast, genre, fine-tuning dials, anti-slop, and more) with your current variable values — no character card, no story history, no scene context.
3. It runs a quiet background generation and displays the result as a short prose paragraph demonstrating what your variable combination actually sounds like.

After a result appears, a **↻ regenerate** button lets you get a fresh sample any time. Changing a variable automatically resets the panel to the "Generate sample" button so you always know whether the displayed sample reflects your current settings.

The preview uses the model you're connected to and counts against your token budget, which is why it's manual rather than automatic.

### Much smarter Model Errata

The **Model Errata** setting (in Planning & Engine → Model Errata) now contains significantly expanded, model-specific guidance. Each option is now split into two parts:

**Prose failures** — specific writing tics each model reaches for reflexively, named precisely so the model can avoid them. For example:
- **Claude**: the "and yet" upward pivot after a hard beat, post-gesture explanation, "something twisted in her chest" emotion-avoidance phrases, and the reflective coda that turns a live scene into past summary.
- **Gemini**: rhetorical questions used as fake interiority, em-dashes as dramatic pause, floating universal-truth sentences, and bare repetition with ellipsis.
- **DeepSeek**: adverb-modified verbs, "in that moment" significance-announce phrases, character voice collapsing into the narration register, passive voice under pressure.
- **Kimi**: beat compression (cramming approach + exchange + retreat into one sentence), dialogue with no physical grounding, comma splices under momentum.
- **GLM**: emotional flatness, generic blocking verbs ("nodded", "smiled"), flat SVO rhythm, character voice that sounds identical regardless of who's speaking.

**State block & reverie failures** — each model has a specific *false completion signal* that makes it drop the `<vellum>` block. These are now named directly:
- Claude feels the prose is "done" when the emotional beat lands — the errata now names this mechanism explicitly and explains why it's wrong.
- Gemini aestheticizes the reverie into literary paragraphs — the errata caps it at 80 words and specifies bullet-short note format.
- DeepSeek writes prose inside the `<vellum>` block instead of JSON — the errata shows the wrong format and the correct format side by side.
- Kimi skips the reverie entirely when momentum is high — the errata gives a strict three-step sequence.
- GLM embeds state information in the final prose sentence — the errata explicitly names this as "the block is still missing."
- Reasoning models put the JSON in their hidden thinking channel — the errata states "your thinking channel is not output" and tells them to copy it out.

If you've been frustrated with a particular model forgetting the state block, ignoring your prose settings, or inserting analysis before writing — update to the Model Errata option for that model.

---

### Previous version highlights (2.1.0-beta.4)

> A slimmer theme lineup, a configurable floating window, and simpler display cards.

### Choose which tabs show in the floating window

The compact floating window is no longer locked to a fixed set of tabs. A new **Tabs** button (☷) in the window's title bar opens a picker where you decide **which** tabs appear and **in what order** — and you can now pull in the "tools" tabs (Journal, Graph, Vault, Context) alongside the usual five (Now, Cast, Bonds, Chronicle, Director).

Check a tab to show it, uncheck to hide it, use the ↑↓ arrows to reorder, and hit **Reset to default** to go back to the original five. Your choice is remembered across reloads (and synced the same way your theme is). At least one tab always stays visible, so you can't accidentally empty the window.

### A tighter theme lineup

Three of the more niche art/craft themes — **Aquarelle** (watercolor), **Werkbund** (Bauhaus), and **Terrazzo** (Memphis) — have been retired to keep the theme picker focused on the looks people actually use. The lineup is now **eighteen chromes** instead of twenty-one.

If you were using one of the removed themes, nothing breaks: your saved theme is automatically migrated to the closest surviving look (Aquarelle and Terrazzo → Bloom, Werkbund → Modern) the next time the panel loads.

### Simpler state & reverie display cards

The preset used to ship a separate hand-styled `<vellum>` state card and Reverie card for *each* theme (Fantasy, Modern, Futuristic, Bloom, Ember, and more) — dozens of near-duplicate regex scripts you had to manually enable and disable to match your chrome.

Those per-theme variants are gone. There's now a **single unified State Block card and Reverie card** that automatically inherit your active theme's colors and fonts — the same way the visual cards already do. One card, every theme, no toggling. This makes the preset lighter and removes a whole category of "why are two cards showing" confusion.

> Upgrading note: if you had manually enabled one of the old per-theme cards, re-import `vellum-ii.json` so the unified defaults take over.

---

### Previous version highlights (2.1.0-beta.3)

The features below shipped in beta.3 and are still part of this release:

**The AI follows your prose settings more reliably**

This is the biggest invisible improvement in this release. Settings like **Pacing**, **Narrative Distance**, **Tense**, and **Genre** are now re-stated to the AI right before it writes — not just once at the top of the prompt where they get buried as the story grows. Think of it as a sticky note the AI re-reads every turn instead of a memo it filed away weeks ago.

In practice: if you set Pacing to *Lingering*, the AI will now actually slow down. If you picked *Present Tense*, it won't silently drift back to past. The instructions were always there — they just weren't in the right place.

The option descriptions themselves were also sharpened. "Propulsive pacing" used to be one word; it's now a full rule with a concrete test ("if a sentence can be removed without losing anything, cut it"). Same for every distance and tense option. Less room for the AI to fudge it.

### More genres to choose from

Eight new genres were added to the **Genre** setting in the Prompt Variables menu:

**Drama, Tragedy, Noir, Coming-of-Age, Political, Western, Wuxia, Xianxia**

You can now blend two genres (use the Secondary Genre slot) for combinations like Western + Horror or Political + Drama. Each genre shapes how the story is framed, not just what setting it's in.

### The AI bounces back from a missing state block automatically

The `<vellum>` data block at the end of each AI reply is how VELLUM tracks everything — who's in the scene, relationship changes, new facts. Sometimes a model forgets to write it, or cuts it off mid-reply.

There's now an **Auto-Repair** option (off by default, enable via the Actions menu). When enabled: if the AI's reply has no state block, VELLUM quietly reads the prose itself, figures out what changed, and writes the block in — all without interrupting you. You'll see a small toast notification when this happens. If the repair fails, it falls back to the existing prose-mining system it already had.

The repair also became smarter in general — it now recovers more kinds of state (clock/day tracking, NPC thoughts, character traits, and more) than it did before.

### Preset linking from the Actions menu (works on mobile too)

Previously you could only link or unlink a preset from the desktop **Preset Editor tab** — a tab that simply doesn't exist on mobile Lumiverse. Now the same controls live in the **Actions menu** (the ≡ toolbar button), so linking works everywhere.

A bug was also fixed where the Actions menu would show a preset as "Not linked" even after you'd just linked it from the desktop tab. Both views now always agree.

### New and redesigned visual cards (VTK)

If you use the **Visual Toolkit** (the feature that lets the AI render scenes as small illustrated cards — a letter, a codex entry, a portrait), this release adds six new card types and overhauls how all cards look:

**New tags the AI can use:**
- `[TITLE]` — a chapter or scene heading card
- `[MAP]` — a gazetteer-style location plate
- `[ITEM]` — an object description plate
- `[VERSE]` — a poem or song lyric, set in display type
- `[TEXT]` — a modern SMS-style message (for contemporary settings)
- `[HALO]` — a holographic display (for sci-fi / space-age settings)

All cards now inherit your active theme's colors and fonts instead of being hardcoded, so a Gatsby card looks gold-and-midnight and a Faewild card looks sage-and-lilac automatically.

The regex patterns that render these cards were also tightened to handle multi-line content and edge cases that previously caused broken display.

### Custom font URL in Customize → Type

The panel's **Customize → Type** tab now has a text box where you can paste any **Google Fonts URL** to use a font that isn't in the built-in picker. Get a URL from [fonts.google.com](https://fonts.google.com) — pick a family, click "Get font", copy the `<link>` URL — paste it in, and the panel restyles immediately and remembers it. Clear the box (or hit the ↺ reset) to go back to the picker.

Only `fonts.googleapis.com` links are accepted for safety; arbitrary external stylesheets are rejected.

### Graphite theme cards

Two new **display cards** for the Graphite chrome were added to the preset: a machined-steel state card and a matching reverie card, both using the Graphite palette (charcoal, steel-blue, Inter font). Enable them from the preset's regex scripts tab and disable the Default pair to match.

### Under-the-hood fixes

A handful of quieter bugs were squashed:

- Several preset conditions that compare dial settings were broken — the **Craft Anchor**, **Social Autonomy**, **Faction Politics**, and **Ambient Breath** blocks were always picking the same branch regardless of what you set. Fixed.
- The Preset Editor tab now uses the correct API for writing link metadata, ending a loop where the version stamp kept triggering redundant re-links.
- The preset panel now correctly identifies which preset is *actually in use* in the current chat (it was sometimes picking the wrong one, so the health check and link badge were wrong).
- Repair and recovery now show cleaner, more informative toast messages instead of the generic warning.

---

### Previous version highlights (2.1.0-beta.2)

The features below shipped in beta.2 and are still part of this release:

**Redesigned Chronicle views** — Memory, Knowledge, Secrets, Scars, and Codex were rebuilt with visual hierarchy, grouping, and color-coding to make long stories scannable at a glance.

**Time Sync tab** — the desync inspector (for catching plot threads that fell behind after a time-skip) now has its own dedicated tab instead of being buried at the bottom of World.

**Preset Editor tab** — when the VELLUM preset is open in Lumiverse's preset editor, a VELLUM tab appears beside it showing link status, health check, what the extension would inject this turn, and whether recent turns were read correctly.

### Lumiverse Themes (Bonus)

The `lumithemes/` folder includes **five Lumiverse host themes** inspired by VELLUM's visual language — illuminated-manuscript palettes, custom typography, and layout touches designed to complement the engine's aesthetic. Each theme comes in **two versions: static and animated**.

| Theme | Description |
|---|---|
| **Bloom** | blush & sage on plum dusk |
| **Ember** | lilac & mint over the indigo void |
| **Faewild** | sage & lilac among the fairy lights |
| **Fantasy** | gilt ink & rubric red on aged vellum |
| **Gatsby** | gilt gold on midnight & champagne |

**To install a theme:**

1. In Lumiverse, open **Settings → Appearance → Theme**
2. Go to the **Custom CSS Editor** tab
3. Click **Import theme** and select a `.lumitheme` file from the `lumithemes/` folder
4. Choose the static version (`vellum-bloom.lumitheme`) for a clean, no-motion experience, or the animated version (`vellum-bloom-animated.lumitheme`) for subtle motion effects

**Customizing fonts and text size** — see [`lumithemes/CUSTOMIZING.md`](lumithemes/CUSTOMIZING.md) for a full guide. The short version: use Lumiverse's built-in font size slider for text scale, and paste a one-line CSS variable override in the Custom CSS editor to change fonts. You can use any font installed on your device.

---

## Table of contents

1. [What problem this solves](#what-problem-this-solves)
2. [Quick start (5 minutes)](#quick-start-5-minutes)
3. [Installing the extension](#installing-the-extension)
4. [Installing & using the preset](#installing--using-the-preset)
5. [The Prompt Variables menu — every setting explained](#the-prompt-variables-menu--every-setting-explained)
6. [The preset, feature by feature](#the-preset-feature-by-feature)
7. [The extension, feature by feature](#the-extension-feature-by-feature)
8. [The panel: tabs & tools](#the-panel-tabs--tools)
9. [How the two halves talk to each other](#how-the-two-halves-talk-to-each-other)
10. [Permissions (and what breaks without each)](#permissions)
11. [FAQ & troubleshooting](#faq--troubleshooting)
12. [For developers](#for-developers)

---

## What problem this solves

If you've done long-form AI roleplay, you know the failure modes:

- The AI forgets a promise made twenty messages ago, or revives a character who died.
- Everyone likes you instantly and equally; relationships never really *develop*.
- A character "knows" something they were never told, because the AI can see the whole script.
- The world is frozen — nothing happens unless you're in the room to watch it.
- The prose drifts into the same tics: *a shiver ran down her spine*, *not X, but Y*, *time seemed to stand still.*

VELLUM II attacks all of these. The preset enforces good craft and a strict "characters only know what they actually witnessed" rule. The extension gives the story a memory that survives across hundreds of messages, tracks relationships as real numbers that move slowly and are *earned*, and can even simulate what people are doing off-screen.

---

## Quick start (5 minutes)

1. **Install the extension** in Lumiverse (steps [below](#installing-the-extension)).
2. **Import the preset** `vellum-ii.json` (steps [below](#installing--using-the-preset)).
3. Open a chat and start playing. After the AI's first reply, open the VELLUM panel — you'll see the scene, cast, and relationships start to fill in.
4. That's it. Everything runs automatically. The rest of this README is for when you want to understand or tune what's happening.

---

## Installing the extension

The extension is a Lumiverse "Spindle" extension installed straight from this GitHub repository.

1. In Lumiverse, open **Extensions** (in the main menu / settings).
2. Choose **Install** (or **Install from URL** / **Add extension**).
3. Paste this repository's URL:
   ```
   https://github.com/valdoix/vellum-engine
   ```
4. Confirm. Lumiverse downloads the extension and asks you to **grant permissions** — approve them. Each one is explained in the [Permissions](#permissions) section below; the short version is that VELLUM needs them to read your messages, write its hidden status block back, and store the chronicle. Your story data never leaves your Lumiverse instance (the only optional outside request is loading a web font — see [Permissions](#permissions)).
5. Once installed, you'll see a new **VELLUM** panel/drawer available in your chat view. Open it any time to see your chronicle.

That's the whole install. There's nothing to build or configure — the extension ships ready to run.

> **Updating:** when a new version is released, re-open Extensions and use **Update** (or reinstall from the same URL). Your chronicles are stored per-chat and aren't affected by updates; the extension automatically migrates older saved data to the new format.

---

## Installing & using the preset

The preset is a single file, `vellum-ii.json`, found in the [`presets/`](presets/) folder of this repo.

1. **Download `presets/vellum-ii.json`** to your computer.
2. In Lumiverse, go to the preset area (**Reasoning → Presets**, or the preset selector in your chat).
3. **Import** the file. It now appears in your list of presets as **"VELLUM II — Engine."**
4. **Select it** for your chat or connection.
5. (Optional) Click the **sliders / settings icon** that appears when a preset has options. This opens the **Prompt Variables** menu — a friendly settings panel where you can change POV, length, tone, content level, and toggle features on and off. You never have to edit the raw preset text.

### Do I need the separate regex file?

The preset already has all its display scripts **built in**, so importing `vellum-ii.json` is enough for everything to render correctly. A standalone `vellum-ii-regex.json` is also included in `presets/` for people who prefer to manage regex scripts separately, or who want to install only a subset — but most users can ignore it.

### A note on the hidden status block

When VELLUM II is active, the AI ends each reply with a small block wrapped in `<vellum>…</vellum>`. **You will never see it** — the preset's display scripts hide it from the chat and strip it from the AI's future context so it doesn't pile up. That block is how the preset and extension talk. If you ever see raw `<vellum>` text in your chat, it just means the regex display scripts aren't enabled; re-importing the preset fixes it.

---

## The Prompt Variables menu — every setting explained

When you select the preset and open its **sliders / settings icon**, Lumiverse shows the **Prompt Variables** menu: a friendly panel of dropdowns and toggles. You never edit raw preset text. This section documents **every one of the 53 settings**, grouped the way the preset groups them, with the options, the **default**, and when you'd change it.

> **Defaults are good.** You can play a full story without touching any of these. Skim for the handful you care about (POV, Length, NSFW Level) and leave the rest.

Twelve blocks ship **off by default** and only activate when you opt in: Possession Tracker, Emotional Save, The Scribes, Rough Hand, NSFL/Dark, VTK Card Library, VTK Spectacle, Colored Dialogue, host Memory Cortex entities, Hard Jailbreak Fallback, World Broadsheet card, and the Slop Proofreader.

### Session Settings

The voice of the whole story.

- **Point of View** — First / Second ("you") / **Third Limited** *(default)* / Third Omniscient. Change for the narrative person you want.
- **Length** — Concise (2–3 ¶) / Standard / Detailed (7–10 ¶) / **Adaptive** *(default)*. Adaptive matches length to the moment; pick a fixed size if replies run too long or short.
- **Tense** — **Past** *(default)* / Present.
- **Prose Register** — **Literary** *(default)* / Gothic / Wildean (witty) / Hardboiled (noir) / Sparse (minimalist) / Lush (rich) / Contemporary / Random (per turn) / Loom Style only. The governing style.
- **Stakes** — Cozy / **Grounded** *(default)* / Harsh / Brutal (permanent consequences, no plot armor). How hard the world bites.
- **Genre** — **Off (character drama)** *(default)* / Romance / Mystery / Horror / Thriller / Fantasy / Sci-Fi / Slice-of-Life / Adventure / Comedy. Adds genre conventions on top of the character drama.
- **Secondary Genre** — same options, **Off** *(default)*. Blend a second genre (e.g. Fantasy + Mystery).
- **Dialogue Frequency** — Sparse / **Balanced** *(default)* / Talkative / Banter-heavy.
- **User Agency** — **Forbidden** *(default, strict: the AI never speaks/thinks/moves your character)* / Minimal (involuntary reflexes only) / Director (expands your stated intent). Raise only if you *want* the AI to co-write your character.
- **Narrative Distance** — Intimate / **Standard** *(default)* / Cinematic / Panoramic / Adaptive. How close the "camera" sits to the POV character.
- **Pacing** — Lingering / **Measured** *(default)* / Propulsive / Adaptive.

### OOC channel

- **OOC channel** — *on by default.* Lets you talk to the AI out-of-character with `((double parens))` or `OOC:` without it bleeding into the story.

### Style & Era

- **Era / Idiom** — **Off (follow the world)** *(default)* / Mythic / Ancient / Medieval / Renaissance / Early-Modern / Georgian / Regency / Victorian / Industrial / Belle Époque / Fin de Siècle / Jazz Age / Early 20th C. / Mid-Century Modern / Contemporary / Near-Future / Far-Future / Space Age / Post-Collapse / Timeless / Fairytale / Random (per turn). Tints diction and idiom to a period.
- **Anachronism Strictness** — Authentic / **Flavored** *(default)* / Loose. How hard the era rule bites on word choice.
- **Tonal Cast** — **Off (no wash)** *(default)* / Blue (melancholy) / Amber (nostalgia/warmth) / Ash (numb) / Violet (feverish/dream) / Crimson (heat/appetite) / Verdigris (rot/envy) / Adaptive (per beat) / Random (per turn). An emotional color wash over the prose.
- **Colored dialogue** *(off by default)* — the model tags each spoken line with the speaker, and the display tints it a distinct color. With the extension installed, colors come from each character's cast card (a dedicated dialogue color if you set one, otherwise their name color, with a gradient name collapsed to one readable shade, otherwise a stable auto color). Without the extension the dialogue is still wrapped but simply inherits the normal text color. Coloring applies to both existing and newly generated turns.

### Anti-slop & craft

- **Prose Doctrine** — *always on.* Show feeling through body and concrete detail, distinct voices, varied rhythm, one beat per turn.
- **The Forge (Anti-Slop)** — *always on.* Names the AI's common tics and pushes for the truer line, without making the prose stiff.
- **Slop Proofreader (display marks)** *(off by default)* — faintly marks clichés that slipped through, in the display only; never changes what the AI wrote or sees.

### Knowledge & irony

- **Knowledge Discipline** — *always on.* Every character only knows what they personally witnessed, were told, overheard, read, or could deduce.
- **Epistemic Mode (dramatic irony dial)** — Behind (mystery) / **Alongside** *(default, discover together)* / Ahead (irony — you know, they don't) / Dark (no one knows, not even narration).

### World & cast

- **Group-scene handling** — *on by default.* In crowds, characters talk to *each other*, not just to you.
- **Living World** — Off ({{user}}-centric) / Minimal (protagonist) / **Active** *(default)* / Sandbox (autonomous, opportunities can expire). How much the world moves on its own.
- **Time Continuity** — *on by default.* Tracks the passage of days/time so "Day 47" stays coherent.
- **The Cartographer (world genesis)** — *on by default.* Fires once at a new chat's opening (or on demand via `((worldgen))`) to establish a coherent world frame. Sub-settings:
  - **World Premise (optional)** — free text; seed the world with a one-line premise.
  - **World Scale** — Chamber (one place) / **Locale (a town)** *(default)* / Realm (a region) / World (a civilization) / Cosmos (many worlds).
  - **World Texture** — Backdrop (scenery) / **Living** *(default)* / Insistent (intrudes).
  - **World Broadsheet card** *(off by default)* — renders opening world news as an illuminated card.
- **Ambient Breath (living setting)** — *on by default.* The setting itself presses on the scene's edges as texture, not plot — weather from a front moving across the map, distant news or rumor, a price or shortage reflecting events elsewhere, a festival on the calendar. Draws on established Codex facts and offscreen currents rather than inventing fresh each turn. Deploys lightly, only when it fits; intensity controlled by World Texture setting.
- **The Codex (mint provisional canon)** — *on by default.* Invents small consistent facts on demand and binds them as canon (shows in the Codex tab).
- **Possession Tracker** *(off by default)* — tracks who carries/owns what.
- **Character Engine** — *always on.* NPCs act on their own motives; change is geological.

### Tone & relationships

- **Romance Pace** — Off / Slow Burn / **Measured** *(default)* / Fast-Paced / Erotic. *When* intimacy becomes reachable (separate from how explicit it may get).
- **World Disposition** — Kind / Warm / **Fair** *(default)* / Harsh / Brutal. The social climate *before* you've earned anything (a prior, not a guarantee).
- **NPC Social Autonomy (NPC↔NPC)** — Off (you drive relationships) / Reactive (on-screen only) / **Living** *(default)* / Autonomous (full lives).
- **Faction Politics Autonomy (faction↔faction)** — **Off** *(default)* / Living (standings drift off-screen) / Autonomous (factions maneuver).
- **Emotional Save (d20 on charged beats)** *(off by default)* — rolls a die on emotionally charged beats so a character's reaction isn't always the obvious one.

### Variance suite (anti-repetition)

- **The Augury (dice-seeded variance)** — *on by default.* Privately rolls pressure / shape / cost / a rare omen each turn to break the mold.
- **Marginalia (diversity anchors)** — *on by default.* Sketches three anchored directions, keeps the best, discards the rest (anchors never appear in prose).
- **The Scribes (rotate authorial voice)** *(off by default)* — rotates the narrating voice turn to turn from a **Scribe pool** you pick: The Miniaturist / The Ironist / The Elegist / The Brawler / The Fabulist / The Clinician.
- **Rough Hand (one deliberate imperfection)** *(off by default)* — permits one purposeful human imperfection per turn to defeat the over-polished sheen.
- **The Palimpsest (belief scars)** — *on by default.* A disproven belief becomes a scar that can resurface as doubt (shows in the Scars tab).

### Planning & engine

- **Reverie (chain-of-thought)** — *on by default.* The AI's hidden plan-once-then-write step.
  - **Reasoning Depth** — Vibes (lean) / **Standard** *(default)* / Deep.
- **Emit State Block** — *on by default.* Emits the hidden `<vellum>` report the extension reads. Turn off only to use the preset with no extension.
  - **State Spec Verbosity** — **Lean (compact)** *(default)* / Full (with example). Switch to Full while teaching a new model the format.
- **Model Errata** — Generic / Off / Claude / Gemini / DeepSeek / Kimi / GLM / Reasoning model. **Default: none.** Opt-in fixes for a specific model family's quirks; pick the one matching your model.
- **Agency Reminder (post-history)** — *on by default.* Re-asserts the agency rule near the end of the prompt where it sticks best.
- **Recall — Use host Memory Cortex entities** *(off by default)* — lets the preset reference the host's native Memory Cortex entities.

### Mature (gated, opt-in)

- **NSFW Level** — Off / Clean (fade to black) / Sensual (implied) / **Explicit — labelled "romantic" internally is the default tasteful tier** / Graphic / Depraved (no limits). Tasteful by default; raise or lower deliberately.
- **NSFL / Dark Content** *(off by default)* — unflinching treatment of violence and dark themes.
- **Hard Limits (never depict)** — free text; content this story will **never** depict. Outranks every other setting. (Also editable from the panel's **Boundaries** action.)
- **Sovereign Mandate** — the preset's in-fiction "no forbidden subject" mandate, with an optional **Hard Jailbreak Fallback** *(off by default)* for stubborn models.

### Visuals

- **Visual Toolkit (diegetic HTML)** — **Off** *(default)* / Rare / Balanced / Frequent. Lets the AI render key beats as small self-contained HTML/CSS visuals.
- **VTK Card Library (tag → illuminated card)** *(off by default)* — turns compact tags (`[CODEX]`, `[LETTER]`, `[DECREE]`, `[PORTRAIT]`…) into illuminated cards.
- **VTK Spectacle (broadsheet / tarot / playbill)** *(off by default)* — rarer set-piece cards (a `[BROADSHEET]` of off-screen news, a `[TAROT]` pull, a playbill).

---

## The preset, feature by feature

Everything in the preset is a **block** you can switch on or off, or a **dropdown/slider** in the Prompt Variables menu (fully catalogued [above](#the-prompt-variables-menu--every-setting-explained)). Nothing is hidden behind an all-or-nothing "profile." Below is what each part does, in plain terms.

### Session settings (the basics)

A single panel of dropdowns that set the voice of the whole story:

- **POV** — first person, second person ("you"), third limited, or third omniscient.
- **Length** — from terse (2–3 paragraphs) to detailed (7–10), or **Adaptive**, which matches the length to the moment.
- **Tense** — past or present.
- **Prose register** — the governing style: *Literary, Gothic, Wildean (witty), Hardboiled (noir), Sparse (minimalist), Lush (rich), or Contemporary.*
- **Stakes** — how hard the world bites, from *Cozy* to *Brutal* (permanent consequences, no plot armor).
- **User Agency** — how much the AI is allowed to write *for you*. The default, **Forbidden**, is strict: the AI never speaks, thinks, or moves your character — it stops and hands control back to you. (**Minimal** allows only involuntary reflexes; **Director** lets it expand your stated intent.)

### Prose craft & anti-slop

Two always-on blocks push the writing toward real literary fiction instead of "chatbot prose":

- **Prose Doctrine** — show feeling through the body and the concrete detail, not by naming the emotion; give each character a distinct voice; vary the rhythm; resolve one beat per turn, then yield.
- **Anti-Slop** — names the AI's most common tics (the "not X, but Y" construction, throat-clearing openers, "warmth pooled in her chest," animal verbs for humans, neat bow-tie endings) and tells it to find the *truer* line underneath instead of just rephrasing. Crucially, it's framed as *"break the autopilot, not the nerve"* — so the prose doesn't go stiff and over-cautious.
- **Slop Proofreader** *(optional, off by default)* — if you enable it (and its matching display script), a faint mark appears in the chat next to any cliché that slipped through, as a gentle nudge. It only marks the display; it never changes what the AI actually wrote or sees.

### Knowledge discipline (the soul of the engine)

This is what makes VELLUM stories feel real. **Every character only knows what they personally witnessed, were told, overheard, read, or could reasonably deduce.** They can't read your mind, can't see the narration, and can't know about off-screen events. Suspicion isn't knowledge. Mistaken beliefs are treated as gifts, not errors.

You also get an **Epistemic Mode** dial that sets how much *the reader* knows versus the characters — *Behind* (mystery), *Alongside* (discover together), *Ahead* (dramatic irony — you know it's a trap, she doesn't), or *Dark* (no one knows, not even the narration).

### The Character Engine

Makes NPCs feel like protagonists of their own lives instead of quest-dispensers:

- They **act on their own motives** instead of asking you what you want; permission-seeking is treated as a failure of nerve.
- They can disagree, refuse, interrupt, start conflict, or leave.
- In group scenes, **characters talk to each other**, not just to you.
- **Change is geological** — one nice conversation doesn't heal a guarded person; old patterns resurface under stress.

### The Living World

A dial for how much the world acts on its own when you're not looking:

- **Off** — the world centers on you.
- **Minimal** — mostly waits for you, but shows small signs that time passed.
- **Active** (default) — absent characters pursue their own goals, rumors travel, subplots build pressure off-screen and eventually intersect you.
- **Sandbox** — a fully autonomous world that gets ahead of you; opportunities can expire if you ignore them.

### Tone: Romance & World Disposition

- **Romance Pace** — from *Off* to *Slow Burn* to *Desire-forward*. This controls *when* intimacy becomes reachable, separately from how explicit it can get.
- **World Disposition** — the social "climate": how people lean toward you *before* you've earned anything, from *Kind* to *Brutal*. It's a prior, not a guarantee — and there's a built-in anti-monotony rule so a kind world still has one sour exception and a brutal world one improbable mercy.

### Mature content (gated, opt-in)

- **NSFW Level** — from *Off* / *Clean* (fade to black) up through *Explicit*, with each level clearly defined. Tasteful by default; explicit only when you choose it.
- **NSFL / Dark Content** *(off by default)* — unflinching treatment of violence and dark themes for fiction that calls for it.
- **Sovereign Mandate** — the preset's "jailbreak": it tells the model that within this fiction there's no forbidden subject and no reason to break character to moralize. There's also an optional **Hard Fallback** for stubborn models.

### The variance suite (anti-repetition)

The newest and most distinctive part of the preset. These exist because even a good model gets *repetitive* over a long story — same sentence shapes, same scene openings. They inject controlled randomness so no two turns feel stamped from the same mold. All of this happens inside the AI's hidden planning block, so it costs you nothing in the visible chat:

- **The Augury** *(on by default)* — at the start of each turn the AI privately "rolls the lots": a **pressure** level (should this beat be still, or should something break?), a **shape** (an opening constraint, e.g. *open mid-gesture* or *on a threshold narrowing*), a **cost** (what this moment spends — time, trust, safety, a truth that can't be unsaid), and a rare **omen** that drops in a surprise. A cooldown stops the same big "eclipse" beat from firing two turns running.
- **Marginalia** *(on by default)* — the AI draws three random "anchors," sketches three possible directions for the beat using them, picks the best, and discards the rest. The anchors never appear in the prose; they just knock the model off its first, most-clichéd instinct.
- **The Scribes** *(off by default)* — rotates the *authorial voice* turn to turn from a pool you choose (the Miniaturist, the Ironist, the Elegist, the Brawler, the Fabulist, the Clinician), so the narration doesn't settle into one groove.
- **Rough Hand** *(off by default)* — permits exactly one deliberate human imperfection per turn (a thought abandoned mid-reach, a register that slips), to defeat the over-polished "AI sheen."

### Planning, model tuning & the state block

- **Reverie** — the AI's hidden planning step before it writes. You set the **depth** (*Vibes* / *Standard* / *Deep*). It plans once, then writes once — it's notes, never a rough draft. The preset also shows it a one-line read-back of your current settings and the story's "phase" (opening, rising, cruising, marathon, endurance) so it adapts as the story ages.
- **Model Errata** — opt-in fixes for specific AI families' known quirks (Claude's tendency to soften, Gemini's "absolute/sheer" crutch and robotic transitions, DeepSeek's purple drift, Kimi's over-thinking, GLM's under-planning). Pick the one matching your model.
- **The State Block** — the hidden `<vellum>` report the extension reads. You can dial its **verbosity** (lean by default to save tokens; full, with a worked example, while teaching a new model the format). If the AI can't manage the JSON, a single terse line still works as a fallback.

### Two signature memory features

- **The Codex** *(on by default)* — when the story needs a detail it hasn't established (a place name, a custom, a price), the AI invents a small, consistent fact and **binds it as canon** so it never drifts later. These show up in the extension's **Codex** tab.
- **The Palimpsest** *(on by default)* — when a character's belief is proven *wrong*, the old belief isn't deleted — it becomes a **scar** that can resurface later as doubt. These show up in the extension's **Scars** tab.

### Visuals (optional)

- **Visual Toolkit (VTK)** — lets the AI render key beats as small, self-contained HTML/CSS visuals (a letter read on-page, a mindscape, a title card). You set the frequency from *Rare* to *Frequent*, or leave it off.
- **Card Library & Spectacle** — compact tags the AI can emit (`[CODEX]`, `[LETTER]`, `[DECREE]`, `[PORTRAIT]`, and rarer treats like a `[BROADSHEET]` of off-screen news or a `[TAROT]` pull) that the display scripts turn into pretty illuminated cards — while keeping the raw tag out of the AI's future context so it stays cheap.

---

## The extension, feature by feature

The extension's job is to turn that hidden status block into a durable, queryable memory and feed the right parts back to the AI. Here's what it tracks and does.

### A chronicle that can't drift

Under the hood, the extension stores your story as an **append-only event log** — an immutable list of "what changed" each turn. Everything you see (the cast list, relationship scores, the day counter) is *recomputed* from that log, never edited in place. This sounds technical, but it buys you real features for free:

- **Undo a turn** cleanly.
- **Rebuild** the entire chronicle from your chat transcript if anything ever looks off.
- **Time-travel** views and a trustworthy timeline.
- An automatic **backup** and a **Recover** button if data is ever lost.

### Cast & evolving relationships

Every named character becomes a **cast card**. Relationships are tracked as two separate numbers — **affection** and **trust** — that move *gradually* and are *earned*, and carry categories (familial, romantic, alliance, rivalry, social). Because affection and trust are separate, the engine can represent things a single "love meter" can't: high affection with low trust reads as infatuation; low affection with high trust reads as respect. Relationships are **directional**, so "A adores B while B can't stand A" is fully representable.

### Knowledge, secrets & dramatic irony

This is the extension's standout. For each fact, it records **who knows it**, how sure they are (*knows / believes / suspects / wrong / unaware*), and whether it's *actually* true — independently of what they believe. That combination is a real dramatic-irony engine: the chronicle can hold "Cersei is **certain** of something that is **false**," and the AI is fed that asymmetry so the irony pays off. **Secrets** track who's hiding what from whom.

### The Journal

A per-character memory book. The engine extracts genuine turning points — a confession, a betrayal, a gift, a wound, a first kiss — written from *that character's* point of view. Your own persona keeps a journal too.

### Scars (the Palimpsest) & the Codex

- **Scars** — superseded beliefs, kept on purpose, that can resurface as doubt under pressure.
- **Codex** — facts that are true *of the world* (canon the story established), kept separate from any character's opinion so they never read as a personal belief — and never accidentally create a phantom "World" character.

### Off-screen life & parallel threads

The chronicle tracks subplots happening away from you. With the optional **off-screen simulation** turned on, the extension will, every few turns, quietly generate what absent characters are doing elsewhere — and let those threads eventually walk back into your scene.

### Hierarchical memory & smart recall

Old turns don't just scroll out of the AI's context window and vanish. The extension **summarizes** older stretches into "chapter" memories (and chapters into "arcs"), so the deep past compresses but stays *retrievable*. On every turn it performs **scene-aware recall**: it figures out what the current moment is about and injects only the relevant history — combining exact keyword matching, the host's semantic embeddings (if you've enabled them), and its own structured facts. Crucially, hard facts (relationship scores, who-knows-what, the day count) are injected **verbatim and authoritatively** — they're never left to fuzzy similarity search, so continuity is never lost.

### The Vault (world books)

A built-in **lorebook** manager. Your chapter/arc summaries are stored as editable world-book entries, so your story's compressed history is visible and editable in Lumiverse itself.

---

## The panel: tabs & tools

Open the VELLUM drawer to find these. There's also a compact floating **"Now"** window that shows just the live scene.

**Tabs:**

- **Now** — the live dashboard: current scene, who's present and their mood, tension, and the latest changes.
- **Chronicle** — the heart of the panel, with sub-views for **World** (scene, arcs, threads, off-screen), **Timeline**, **Turns**, **Beats**, **Time Sync** (lagging threads and off-screen subplots with catch-up options), **Memory**, **Knowledge**, **Secrets**, **Scars**, **Codex**, and **Items**.
- **Cast** — every character, editable.
- **Relations** — every relationship with its affection/trust scores and history.
- **Journal** — per-character memory books.
- **Graph** — a beautiful force-directed map of your relationship web, with faction clusters; drag, zoom, click to isolate.
- **Vault** — your lorebook entries.
- **Context** — a live feed showing exactly what the engine injected into the prompt this turn (great for understanding *why* the AI knew something).

**Tools** (in the toolbar / Actions menu) — everything is optional:

- **Customize** — theme the panel: colors, fonts (including a **custom Google Fonts URL**), size, skins.
- **Summarize** — compress older turns into chapter memories now.
- **Rescan / Rebuild** — re-read the latest turn, or reconstruct the whole chronicle from the transcript (recovery).
- **Undo turn** — drop the most recent turn's tracked changes.
- **Tidy threads / Tidy lore** — merge near-duplicate plot threads or facts.
- **Hide filed** — hide already-summarized turns from the prompt to save context.
- **Traverse** — smarter, AI-guided recall (cycles through off / one-shot / deep tree drill).
- **Tone** — quick access to romance pace + world disposition.
- **Off-screen** — toggle the off-screen life simulation.
- **Export / Import / Recover** — back up, restore, or move a chronicle.
- **Clear** — erase this chat's chronicle (quarantined as a destructive action).

Everything model-authored is safely escaped, and each panel fails in isolation — one glitchy panel can never freeze the rest.

### Using a custom font (Customize → Type)

The panel ships a set of built-in fonts (Cormorant, EB Garamond, Playfair, Lora, and more), but you can point it at **any Google Fonts family** without editing code:

1. Open **Customize → Type**.
2. In **Display font URL** (or **Data font URL** for the mono/data text), paste a Google Fonts CSS URL — e.g. `https://fonts.googleapis.com/css2?family=Lora:wght@400;700`. Get one from [fonts.google.com](https://fonts.google.com): pick a family, choose the styles, and copy the URL from the `<link>` the site gives you.
3. The panel restyles immediately and remembers the choice. Clear the box (or hit the ↺) to fall back to the picker.

Notes:
- Only `fonts.googleapis.com` links are accepted — arbitrary stylesheet URLs are rejected for safety.
- When a font URL is set, the font is fetched **from Google's servers** each session. Leave the box blank to keep everything local to your Lumiverse instance.
- This is the extension's own panel typography. It's separate from Lumiverse's host-theme fonts (see [`lumithemes/CUSTOMIZING.md`](lumithemes/CUSTOMIZING.md)), which go through the host's own CSS editor.

---

## How the two halves talk to each other

It's a clean loop, worth understanding once:

1. **You send a message.** The extension injects the relevant chronicle (authoritative facts + recalled history) into the prompt.
2. **The AI replies** — prose for you, plus the hidden `<vellum>` status block at the end.
3. **The extension reads that block** straight from the saved message, validates it, and folds the changes into the event log.
4. **Display scripts hide** the block and the planning notes from your view and **strip them from the AI's future context**, so they never clutter the chat or bloat the prompt.
5. The panel updates, and the cycle repeats.

If a turn's block is malformed or missing, the extension falls back gracefully — it can read a terse one-line format, and it even mines the prose itself for knowledge, secrets, and journal moments as a backup. **Token-tight local models still work**; they just lean on the fallbacks more.

---

## Permissions

When you install the extension, Lumiverse asks you to grant these. Here's what each is for and what stops working without it:

| Permission | Used for | Without it |
|---|---|---|
| `interceptor` | Injecting scene-aware recall into the prompt | No memory is fed to the AI |
| `chats` | Finding the active chat, attaching world books | No chat context / no Vault attach |
| `chat_mutation` | Reading raw messages, hiding filed turns | No scanning / no hierarchical memory |
| `generation` | Auto-summaries, fact extraction, off-screen sim | No auto-extraction or summarizing |
| `ui_panels` | The drawer and tabs | No UI |
| `world_books` | The in-app Vault (lorebooks) | No Vault |
| `memories` | Semantic recall via the host's embeddings | Recall still works, keyword-only |
| `presets` | The VELLUM tab in the Preset Editor (link status, health check, prompt budget) | No preset editor tab |

Everything runs inside your Lumiverse instance. The only optional outside request is loading a **web font**: the Gatsby and Sumi looks, a few Google-hosted picker fonts, and any Google Fonts URL you paste in **Customize → Type**, fetch from Google's font CDN (`fonts.googleapis.com`). Leave those unset and the extension makes no outside calls at all — your chronicle data and code never leave your instance either way.

---

## FAQ & troubleshooting

**I see raw `<vellum>` or `<reverie>` text in my chat.**
The preset's display scripts aren't active. Re-import `vellum-ii.json` (its scripts are built in), or import the standalone `vellum-ii-regex.json` and make sure those scripts are enabled.

**Can I use the preset without the extension?**
Yes. You'll get all the prose craft, knowledge discipline, and variance features. You just won't get the persistent chronicle, the panel, or the smart recall. The hidden state block will simply be ignored (and hidden).

**Can I use the extension with a different preset?**
Yes. The extension can mine your AI's prose for facts on its own. It works best with the VELLUM preset (which hands it clean structured data), but it degrades gracefully with anything.

**My local model isn't producing the JSON block.**
Switch the **State Spec Verbosity** to *Full* for a while so the model sees a worked example, and consider enabling the matching **Model Errata** block. Even if it never produces JSON, the terse one-line fallback and prose mining keep the chronicle alive.

**The story feels repetitive.**
Make sure **The Augury** and **Marginalia** are on, try turning on **The Scribes** with a few voices selected, and consider **Rough Hand**. These exist specifically to fight repetition.

**A character knows something they shouldn't.**
That's exactly what the **Knowledge Discipline** block prevents — make sure it's enabled. You can also inspect the **Knowledge** tab to see who the engine thinks knows what, and edit it.

**Something looks wrong in the chronicle.**
Try **Rescan** (re-read the last turn) or, as a last resort, **Rebuild** (reconstruct everything from the transcript). If data was lost, **Recover** restores from the automatic backup.

**Will updating the extension wipe my story?**
No. Chronicles are stored per-chat and are migrated forward automatically when the data format changes.

---

## For developers

VELLUM II is TypeScript, built with [Bun](https://bun.sh) / tsup, and unit-tested with vitest.

```sh
bun install
bun run typecheck   # tsc --noEmit (strict)
bun run test        # vitest
bun run build       # → dist/backend.js + dist/frontend.js
```

After changing source, run `bun run build` and reload the extension in Lumiverse.

**Architecture in one breath:** the event log is the single source of truth; `state = reduce(events)` is a set of pure, tested functions; retrieval fuses lexical + embeddings + structured facts (with structured facts kept authoritative); and the UI is small isolated components with error boundaries. Adding a feature means three small touch points — a new event kind, one reducer case, and a feature that emits it — never a sprawling edit. See [`EXTENDING.md`](EXTENDING.md) to add a feature.

## License

Licensed under **AGPL-3.0-only** (matching the Lumiverse extension ecosystem). The full text is in [`LICENSE`](LICENSE).

## Credits

Built by VELLUM, with thanks to the Lumiverse community. Influenced by the AI-directed retrieval and activity-feed ideas of LoreRecall and the splice-in-place chapter/arc compression of LumiBooks — rebuilt natively here in our own engine and words.
