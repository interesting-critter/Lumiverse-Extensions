# Bundling scripts into character cards

LumiScript scripts can ride **inside a character card**. Bundle a few scripts into a card, export the card the normal Lumiverse way, and whoever imports it is offered those scripts — reviewed, consented to, and installed **disabled** for safety. It's the zero-friction way to ship behaviour *with* a character: a card that comes with its own tracker, its own dice roller, its own scene-tempo logic.

This is a sibling to `.lumiscript.zip` [script packs](#vs-script-packs). Packs are a standalone file you hand someone; card-embedded scripts travel inside a character you were already sharing. Same script subset under the hood — a card entry is a pack entry plus a stable bundle id.

There is no `api.*` for this — it's a UI feature (an authoring modal + an import consent flow), so this guide is a walkthrough, not an API reference.

## At a glance

| | |
|---|---|
| **Author** | Script-manager toolbar → the **bundle** button (the box icon) → pick scripts + a target character → the scripts are written into that character's `extensions.lumiscript`. Export the card normally; the scripts ride along. |
| **Edit in-place** | The character editor's **LumiScript** tab views the card's bundled scripts (with installed-status badges), removes them, adds more from your library, or imports them — all on the character you're editing. Needs a recent Lumiverse (the editor-tab host API); the tab is simply absent on older hosts. |
| **Import** | On card import the **consent modal** lists the bundled scripts. The importer reviews + chooses which to install. Installs land **disabled** + `allowDangerous: false`. |
| **Trust model** | Importing a card is the trust act. Nothing auto-enables, auto-runs, or self-grants a permission. The importer reviews the code in the panel before enabling. |

## Authoring — bundle scripts into a card

In the script manager, the toolbar has a **bundle** button (a box icon, next to Import/Export). It opens a modal:

1. **Pick the scripts** — a checklist of every script you have (triggers and libraries both). Check the ones to bundle.
2. **Pick the target character** — a dropdown of your characters (shown with portrait thumbnails, so two characters that share a name are easy to tell apart). The scripts get written into *that* character's card.
3. **Confirm** — the backend serializes the chosen scripts and writes them into the character's `extensions.lumiscript`. Then **export that character** through Lumiverse's normal character-export flow and the scripts are inside the exported card.

(Whether each script runs only for this character or everywhere is **not** decided here — that's the importer's call, via a toggle in the consent modal. See [Scoping](#scoping--run-only-for-this-character) under Importing.)

The write is backend-authoritative: the modal sends only the chosen script ids + target character; the backend loads the real script bodies from storage and builds the bundle. It writes **only** the `lumiscript` key (the host shallow-merges, so other extensions' data on the card is untouched), and reuses the card's existing bundle id on a re-export so updates land as updates, not duplicates.

### What travels — and what doesn't

A bundled script carries the **shareable subset** — the same fields a script pack carries, nothing local:

| Travels | Stripped (the import regenerates it with a safe default) |
|---|---|
| `name`, `code`, `type` (trigger / library) | The installed script's `id` (fresh UUID on import) |
| **`triggers`** — the event hooks (which Lumiverse events it listens to) | `enabled` — imports land **disabled** |
| `folder`, `metadata` (`description` / `author` / `version` / `tags`) | `allowDangerous` — imports land **`false`** |
| | Timestamps |

The standout is **`triggers`**: a bundled trigger script's full set of event hooks travels and is applied on import, so the importer never has to read your comments to know which events to wire it to. (The script installs disabled, so the hooks are *set but dormant* until the importer enables it — then it listens to exactly the events you bundled.)

**Bindings are a special case.** They *are* written into the card (they're part of the shareable subset), but they're **discarded at install** — a binding's character UUID is dead on the importer's machine, so the import ignores the bundled bindings and the scope toggle generates a fresh one instead. So they "travel" in the file but never reach the installed script. See [Scoping](#scoping--run-only-for-this-character).

### Upfront warnings

The modal warns before you bundle when:

- More than **64** scripts are selected (the import side caps at 64 — the rest won't fit).
- A name / folder / metadata field exceeds the import-side length clamp (it'd be trimmed).
- A selected script has character/chat bindings (a reminder they won't carry to the importer — they'll re-scope it in the consent modal, see [Scoping](#scoping--run-only-for-this-character)).

## The character-editor LumiScript tab

Open any character in Lumiverse's character editor and there's a **LumiScript** tab — a richer home for a card's bundled scripts than the one-shot toolbar modal. It lists the scripts bundled in *that* card, read live from the editor's **draft** `extensions.lumiscript` (so it reflects edits you haven't saved yet). Each script shows its type (trigger / library), version, event hooks, code size, folder, a bindings note, and an **installed-status badge**:

- **In your library** — you already have this script (matched by `(bundleCardId, bundleId)`, or by identical code — so a script you just bundled *from* your library reads as present rather than tempting a duplicate import).
- **Not in your library** — the card carries it but you haven't installed it.
- **Update available** — the card's copy is a newer version than your installed one.
- **Library is newer** — your installed copy is ahead of the card's.

From this tab you can:

- **Remove a script from the card** — the trash icon, behind a confirm. This edits the **card** only; a copy already installed in your library is left untouched. The change **auto-saves** — the character editor persists it immediately (no separate save step), and removing the last script drops the whole `lumiscript` key, leaving a clean card.
- **Add scripts from your library** — the **Add** button opens an inline picker of your scripts (ones already in the card are filtered out), each with a trigger/library badge. Check some, **Add to card**, and they're bundled into the character you're editing. Same backend-authoritative write as the toolbar modal — the backend builds the entries from your real stored scripts — but it targets the open character instead of a dropdown pick, and reuses the card's existing bundle id (or mints one for a card that had none).
- **Import to library** — when the card carries scripts you don't have, an **Import to library** button opens the same [consent modal](#importing--the-consent-flow) (scope toggle, permission warnings, per-script selection) so you can install them on the spot — **disabled**, backend-authoritative (it reads the saved card, never the tab's view) — without leaving the editor.

This tab and the toolbar **bundle** button coexist: reach for the toolbar modal to bundle into *some other* character; reach for the editor tab to manage the card you're already editing.

## Importing — the consent flow

When a card with `extensions.lumiscript` is imported, the host fires `CHARACTER_CREATED`; LumiScript reads the bundle and, if there's anything actionable, shows a **consent modal**. Per script you see:

- **Name** + an **action badge** — `Install`, `Update v1.0.0 → v1.1.0`, `Up to date`, `Older — skipped`, or `Unchanged`.
- **Description / author / version** (from the bundled metadata).
- **Event hooks** — the events the script will listen to once enabled.
- A **permission warning** when the script appears to need a Spindle permission you haven't granted (a warning, **not** a gate — the script just degrades until you grant it).
- The **"Run only for ‹character›"** scope toggle.
- An overwrite warning on updates that would replace an **enabled** script's code or your **local edits**.

Check the ones you want and confirm. Everything installs **disabled** with `allowDangerous: false` — review the code in the LumiScript panel, then enable.

### Scoping — "Run only for this character"

Bindings (a script's "only run for character X / chat Y" constraint) match by **raw character UUID**, and the host regenerates a character's UUID on every import — so the author's own bindings are **dead on your machine** and aren't carried. Instead, the consent modal gives each trigger script a **"Run only for ‹character›"** toggle:

- **On** → the installed script is bound to *this* imported character (it runs only when that character is active).
- **Off** → the script runs globally (in every chat).

The default is **on** for a trigger script the author bundled with a character binding (a signal of "scope this"), off otherwise. Libraries get no toggle — bindings don't gate `require()`. Scope is an **install-time** decision: re-importing an *update* never re-scopes or clobbers a binding you set by hand — re-scope via the script's normal Bindings panel.

### Updates on re-import

A card's scripts are matched against what you already have by `(bundleCardId, bundleId)` (see [Identity](#identity--how-re-imports-match)). For a match:

- Both sides carry a **strict semver** `metadata.version` → incoming **greater** ⇒ offered as an update; equal or older ⇒ skipped (no downgrades).
- No comparable version → **code-hash** fallback: identical code ⇒ skipped (unchanged); different ⇒ offered as an update.

An update preserves your `enabled` + `allowDangerous` state and your manual name/bindings; it overwrites code, triggers, and metadata.

### The chat-open reminder

You won't always have the panel open at import time (or you might decline, or you imported the card before installing LumiScript). So when you **open a chat** whose character bundles scripts you don't have, a non-blocking **banner** appears in the Manage tab: *"This character bundles N script(s) you don't have — Review · Dismiss."* **Review** opens the same consent modal; **Dismiss** hushes it **for the rest of the session** — it returns after a restart, because you may well want those scripts later, and the [editor tab](#the-character-editor-lumiscript-tab)'s *Import to library* is always there for a deliberate grab. The hush is in-memory only: there's no dismissals file to accumulate or clear, and deleting a bundled script no longer suppresses its banner.

### Deleting a card

Delete a character whose card installed scripts, and you're **offered** — default **keep**, never automatic — to remove the scripts it installed. Bundled scripts are often generic, and they're your data once imported, so removal is always opt-in.

## Identity — how re-imports match

Two ids form the per-card de-dup key:

- **`bundleCardId`** — author-assigned, minted by the authoring UI and embedded in the card. It's **not** the host character UUID (that's regenerated on every import, so it can't anchor identity). Re-exporting the same character reuses its existing `bundleCardId`, so re-imports update in place.
- **`bundleId`** — per-script, stable across re-bundles. For a script you authored it anchors on the script's own stable id; for a script you imported from a card it preserves that card's `bundleId` (lineage carries across a re-share).

The key is **per-card**: the same `bundleId` in two different cards is two distinct installs. The host character UUID is recorded only as provenance (for "came from card X" purposes), never as the identity.

## Security posture

- **Importing a card is the trust act.** There is no separate signing/verification step (that's a possible future enhancement, not in scope today).
- **Nothing auto-runs.** Installs land disabled + `allowDangerous: false`. No path auto-enables, auto-grants a permission, or runs the code before you review it.
- **Permission warnings are warnings.** A bundled script that needs a permission you haven't granted is flagged, not blocked — it degrades gracefully until you grant it.
- **The card is untrusted JSON.** The import side validates and clamps the bundle defensively (script count, field lengths) and never throws on a malformed/hostile card.

## vs script packs

| | Character-card scripts | `.lumiscript.zip` packs |
|---|---|---|
| Carrier | Inside a character card you share | A standalone file |
| Best when | Behaviour belongs *to* a character (its tracker, its dice) | Behaviour is character-independent, or you're sharing scripts without a character |
| Import UX | Per-script consent modal on card import + a chat-open reminder | A confirm dialog when you open the `.zip` |
| Updates | Re-import offers update-if-newer per `(bundleCardId, bundleId)` | Re-import creates fresh copies |
| Event hooks | Travel + apply | Travel + apply |

Both install disabled and carry the same shareable script subset. Reach for cards when the scripts are part of a character you're already handing over; reach for packs otherwise.

## Version notes

Character-card-embedded scripts are **new in the v1.4.0 release**. The two original surfaces — authoring (the bundle modal) and importing (the consent modal, the chat-open reminder, the card-delete offer) — are self-contained in LumiScript; they need no Lumiverse host changes beyond the `CHARACTER_CREATED` / `CHARACTER_DELETED` events and `spindle.characters.*`, which the host already exposes. The **character-editor LumiScript tab** (also v1.4.0) is the one part that needs a newer Lumiverse — the host's editor-tab placement API (`ctx.ui.registerCharacterEditorTab` + `ctx.ui.characterEditor`); on a host without it the tab is simply absent and everything else still works. Script packs (the sibling format) have carried the same script subset since much earlier.
