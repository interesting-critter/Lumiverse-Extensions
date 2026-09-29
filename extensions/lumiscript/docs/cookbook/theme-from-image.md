# Theme the app from a character's image

Recolor Lumiverse to match whoever you're talking to — pull the dominant color from the active character's avatar and hand it to the host's palette engine, which derives a coherent ~80-variable theme. The whole app shifts to suit the character, and reverts cleanly when you switch away or disable the script.

## What you'll use

- [`api.theme.*`](../guides/theme.md) — `extractColors(imageId)` (palette from a stored image) + `applyPalette({ accent })` (accent → coherent theme) + `clear()`. Requires `app_manipulation`.
- `api.chats.getActive()` + `api.characters.get(id)` — to reach the active character's avatar `imageId`. Require `chats` and `characters` respectively.
- `CHAT_SWITCHED` as the trigger — fires whenever the active chat opens, switches, or closes. ([Trigger model](../concepts/trigger-model.md))

The idea up front: **you give the host one accent color; it generates the rest.** `applyPalette({ accent })` derives backgrounds, borders, hover states, glass tints — a coherent set — from a single HSL accent. So the work is just: find a good color (here, the avatar's dominant) and hand it over. The host re-applies your layer across the user's own theme/mode switches, and clears it automatically on disable.

## The script

```js
// @triggers CHAT_SWITCHED
// Recolor the app to match the active character's avatar.
// Permissions: app_manipulation (theme) · chats + characters (look up the avatar).

// CHAT_SWITCHED also fires on return-to-home, where chatId is null. The
// characterId isn't on the event payload (and resolves a beat after the event),
// so read it from the host's live state — the canonical pattern.
const chat = await api.chats.getActive();
if (!chat?.characterId) {
  await api.theme.clear();          // no character (home screen) → drop our recolor
  return;
}

const character = await api.characters.get(chat.characterId);
if (!character?.imageId) return;    // character has no avatar — leave the theme alone

// Extract a palette from the avatar, apply its dominant color as the accent.
const palette = await api.theme.extractColors(character.imageId);
await api.theme.applyPalette({ accent: palette.dominantHsl });
```

Enable it and open chats with different characters: the accent, backgrounds, and borders shift to match each avatar. Return to the home screen and your recolor drops away. Disable the script and the app reverts to the user's own theme — no cleanup code required.

## How it works

**`extractColors` works on an `imageId`, never a URL.** The id is Lumiverse's canonical image-table handle — exactly what `character.imageId` (the avatar), `api.images.upload(...)`, and `api.imageGen.generate(...)` all hand you. `extractColors` returns a [`ColorExtractionInfo`](../guides/theme.md#extractcolorsimageid): a `dominant` color, per-region samples (`top` / `center` / `bottom` / …), `flatness` scores, `isLight`, and `dominantHsl` — the dominant in HSL, ready for `applyPalette`.

**`applyPalette` takes HSL, and derives the rest.** `applyPalette({ accent: { h, s, l } })` is the whole recolor — the host generates ~80 coordinated variables from that one accent. Passing a hex string is a type error; `dominantHsl` is already in the right shape, so the two pair directly. Palettes are most-recent-wins across scripts (they don't merge), so the last script to call `applyPalette` owns the accent.

**Resolve the character off live state, not the payload.** `CHAT_SWITCHED` fires with `{ chatId }` only — no `characterId`, and the host resolves the character a beat *after* the event fires. `await api.chats.getActive()` reads the host's settled state, so `chat.characterId` is populated reliably. On the home screen `chatId` is `null`; the guard calls `clear()` so the previous character's colors don't linger.

**Cleanup is automatic.** When the script is disabled or deleted, the host walks your contribution out and re-derives the merged theme — there's no teardown obligation. `clear()` is only for *mid-life* reverts, like the home-screen branch above.

## Make it yours

- **Skip flat backgrounds.** If the avatar's top region is a flat sky or wall, its dominant can be a dull background color. `extractColors` returns `flatness` per region — when `palette.flatness.top > 0.8`, reach for `palette.regions.bottom` instead (you'll need a small `rgbToHsl` helper, since regions come back as RGB).
- **Stay in the user's mode.** Read `await api.theme.getCurrent()` for the user's `mode`, `accent`, and whether they've disabled glass (`enableGlass`), and blend toward the avatar rather than overriding wholesale — e.g. keep their hue, nudge lightness.
- **Theme from a generated image.** `api.imageGen.generate(...)` returns an `imageId` too — generate a scene image per chapter and theme from that instead of the avatar.
- **Follow avatar edits.** Also trigger on `CHARACTER_EDITED` so a changed avatar re-themes the app, not just chat switches.

## Gotchas

- **`applyPalette` wants HSL, not hex.** `{ accent: '#9370DB' }` is a type error. Use `dominantHsl` (already HSL) or convert with your own hex-to-HSL helper.
- **`extractColors` needs the image stored.** It only takes an `imageId`. If you have a URL, fetch it (`api.utils.http.get(url, { responseType: 'arraybuffer' })`) and `api.images.upload(...)` first to get an id.
- **Theme writes can transiently fail on chat-open storms.** Mutating theme calls retry transparently around the host's macro-resolution race, but a pathological `CHAT_SWITCHED` storm can exhaust the retries. If you theme heavily on every open, defer the apply or precompute with `generateVariables` (read-only, no race). See [Macro-race resilience](../guides/theme.md#macro-race-resilience).
- **`getCurrent()` returns the BASE theme, not the merged result.** There's no public read of the fully-merged applied state — track what *you* contributed if you need it.

## See also

- [Theming the host UI](../guides/theme.md) — the full `api.theme` surface, per-script attribution, mode-keyed variants, and the [token reference](../guides/theme.md#token-reference).
- [Permissions](../concepts/permissions.md) — `app_manipulation` (theme) plus `chats` / `characters` (the avatar lookup).
- [Storage model](../concepts/storage-model.md) — `api.images` provenance for `imageId`, if you theme from uploaded or generated images.
