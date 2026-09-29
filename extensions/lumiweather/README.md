<!-- Mirror of upstream documentation. Everything below the metadata table is the original file, unmodified. -->

| Field | Value |
| --- | --- |
| Extension | LumiWeather |
| Source repository | https://github.com/Archkr/Lumiverse-LumiWeather |
| Original link (from Lumiverse-Extensions README) | https://github.com/Archkr/Lumiverse-LumiWeather |
| Upstream path | `README.md` |
| Retrieved | 2026-09-29 @ `main` (`e659e44`) |

---

# LumiWeather

LumiWeather is a Lumiverse extension that adds a draggable weather HUD and animated scene ambience to chat.

It is built for story-driven use, not live forecast data. The model controls the scene by emitting a hidden `<weather-state>` tag, and the extension turns that into HUD updates, layered effects, and per-chat weather state.

<img width="364" height="211" alt="image" src="https://github.com/user-attachments/assets/a7baf3e4-b8fa-4d8b-b6f1-a7c903f632c8" />

## Features

- Compact draggable HUD with dynamic styling based on story time, palette, and weather
- Mobile weather launcher with a touch-friendly fullscreen HUD and controls
- Animated ambience that can render behind the chat, in front of the chat, or both
- Story sync mode driven by hidden inline weather tags
- Manual lock mode for overriding the current scene per chat
- Optional multi-day forecast strip projected from the hidden weather tag
- Scene changes blend between palettes and effects instead of swapping instantly
- Per-chat persistence for story state and manual overrides
- Prompt macros for reliable prompt-side weather tag generation
- Clear waiting status when a chat has not emitted its first weather tag

## Prompt Setup

To make the main model emit weather tags consistently, add this to the active character or preset system prompt:

```text
{{weather_tracker}}
```

Optional reference macros:

```text
{{weather_state}}
{{weather_format}}
```

`{{weather_tracker}}` is the canonical integration macro. The older aliases remain available for existing prompts; the current state is injected directly into generation, so `{{weather_state}}` is retained only as a compatibility marker. LumiWeather no longer makes extra generation calls when a tag is missing.

Supported aliases:

- `{{weather_tracker}}`
- `{{story_weather_tracker}}`
- `{{story_weather}}`
- `{{weather_state}}`
- `{{story_weather_state}}`
- `{{weather_format}}`
- `{{story_weather_format}}`

The legacy `story_weather_*` aliases and technical extension ID remain in place so existing installs and prompts update cleanly under the LumiWeather name.

## Hidden Tag Contract

The assistant should keep all visible prose natural, then end the message with exactly one hidden weather tag:

```html
<weather-state location="Example Location" date="2026-01-15" time="3:00 PM" condition="rain" summary="Steady afternoon rain" temperature="60F" intensity="0.65" wind="breezy" windDirection="west" palette="storm" season="winter" forecast="2026-01-16: snow, 30F, heavy flurries | 2026-01-17: cloudy, 34F"></weather-state>
```

Supported conditions:

- `clear`
- `cloudy`
- `rain`
- `storm`
- `snow`
- `fog`

Condition names are case-insensitive and common aliases such as `sunny`, `overcast`, `rainy`, `thunderstorm`, `snowy`, and `mist` are normalized to these six values. Dates must be real calendar dates, and times must be valid 12-hour or 24-hour values.

### Optional attributes

`season` and `forecast` are optional and were added in v1.4. Tags written before v1.4 keep working unchanged.

- `season` is one of `spring`, `summer`, `autumn`, or `winter`. When it is omitted, the season is derived from the story date.
- `forecast` projects at most five later days. Separate entries with a pipe and format each as `date: condition, temperature, summary`. Condition, temperature, and summary are each optional, so `2026-01-17` alone is a valid entry. Unparseable entries are dropped rather than failing the whole tag, and the projection is sorted by date.
- When `forecast` is omitted, the projection already in play is kept. Send `forecast=""` to clear it deliberately.

Effect placement is user-controlled in settings:

- Back only
- Front only
- Front and back

Supported palettes:

- `dawn`
- `day`
- `dusk`
- `night`
- `storm`
- `mist`
- `snow`

### Scene transitions

Scene changes blend rather than swap. Palette colors and layer opacities are registered CSS custom properties, so they interpolate, and rain density is expressed as a per-particle opacity multiplier so intensity changes read as a swell instead of particles popping in and out.

Blending is skipped when it would fight something else: reduced motion, paused effects, a hidden tab, or an FX layer still fading in. It can also be turned off entirely with **Blend scene changes** in settings.

### HUD clock

The HUD clock follows story time by default. **HUD clock** in settings chooses between:

- **Story time, live in manual lock** — the default; a manual lock shows real time until you resume story sync
- **Always follow real time**
- **Always follow story time**

## How It Works

1. The prompt includes `{{weather_tracker}}`.
2. The model writes its normal reply, then appends one final `<weather-state>` tag.
3. The frontend hides the tag from visible chat.
4. The HUD updates only after the assistant message is complete, so streaming does not mutate the scene mid-reply.
5. The backend stores the normalized weather state per chat.
6. If the model does not emit a weather tag, the current scene remains unchanged and the HUD reports that it is waiting for LumiWeather.

Only completed assistant tags are accepted. Streaming tags and user-authored tags are ignored, and duplicate tags from the same chat message do not update state twice. When a tagged message is deleted, edited, or swiped, LumiWeather rebuilds story state from the latest remaining assistant tag.

## Story Sync vs Manual Lock

### Story Sync

- Uses the AI-provided location, date, time, weather, and scene metadata
- Story time stays fixed to whatever the AI last set
- The scene only changes when the AI sends a new weather tag

### Manual Lock

- Lets you override the current scene for the active chat
- Keeps the override saved until you resume story sync
- The HUD clock can display live real time while manual mode is active

The manual scene editor covers every field a weather tag can carry, including the v1.4 additions:

- **Season** offers `Derived from date` by default, so editing the story date updates the season instead of pinning whatever the old date implied. Pick a season explicitly to override it; selecting `Derived from date` again clears the override. The choice survives saving and reloading. For older saved scenes without override metadata, a season matching the date is treated as derived; a different season is retained as an override.
- **Outlook** takes one day per line using the same format as the tag attribute, for example `2026-01-16: snow, 30F, heavy flurries`. Leave it empty to clear the projection. A line that cannot be parsed is reported instead of being silently dropped.
- All seven scene presets match the tagged form exactly, so a preset you apply by hand shows as active and produces the same scene as a tag would.

## Installation

1. Copy the repository URL:

```text
https://github.com/Archkr/Lumiverse-StoryWeather
```

2. In Lumiverse:

- Open the `Extensions` tab
- Click `Install`
- Paste the repo URL into the repo URL field
- Click `Install`

3. Enable the extension and grant `interceptor`, `chats`, `chat_mutation`, and `ui_panels` permissions. LumiWeather uses the broadly named `chat_mutation` permission only to read remaining message history after deletes, edits, and swipes; it does not append, update, delete, hide, or swipe chat messages.

4. Open the extension settings panel and confirm the HUD/settings panel loaded correctly.

## Setup

To use story-driven weather generation:

1. Open the character or preset system prompt you want to use.
2. Add this line somewhere in the prompt:

```text
{{weather_tracker}}
```

3. Save the prompt.
4. Start or continue a chat.
5. The assistant should write its visible reply first, then append the hidden `<weather-state>` tag at the end of the message.

If you do not want the model driving the scene, you can skip prompt setup and use `Manual lock` from the HUD or settings panel instead.

On the home screen, the HUD intentionally shows an empty waiting state and effects remain off until a chat is active and that chat has emitted a weather tag.

## Mobile Use

On phones, narrow windows, and coarse-pointer devices, LumiWeather appears as a draggable 40-pixel weather launcher so it does not cover the chat. Tap the launcher to open the existing HUD and scene controls in a fullscreen, scrollable view. Tap **Close** to return to the launcher.

The mobile panel follows the device safe areas and Lumiverse's visual viewport, including the virtual-keyboard inset. Rotating the device or opening settings keeps the current scene and launcher position intact. Desktop continues to use the original compact and expanded floating HUD.

## LumiState interoperability

LumiWeather publishes its normalized visible scene through the public, read-only `lumi_weather.state.current` endpoint. Compatible extensions receive the active chat ID, source-local revision, freshness, scene location, calendar date and time, and weather conditions with provenance.

As of v1.4 the weather condition also carries `season` and a serialized `forecast` in its `attributes` map, and `lumi_weather.contract.v1` advertises the additional `forecast` and `solar_time` capabilities. The protocol stays `lumi_state.v1` at `schemaVersion: 1`; every addition is of the same shape, so existing readers are unaffected.

Story history is rebuilt with each tag stamped by the timestamp of the message that carried it, so `updatedAt` and `freshness` reflect when the scene was actually written rather than when the history happened to be replayed.

Manual-lock and story-sync transitions both increase the per-chat revision. Returning to story sync also creates a new revision, even when the restored story state is older than the removed manual override.

Publishing is an in-memory state update and never makes an additional generation call.

## Project Layout

```text
src/
  backend.ts        Backend state, macros, prompt interception, chat persistence
  frontend.ts       HUD, message interception, FX mounting, scene updates
  shared.ts         Normalization, defaults, preference bounds
  time-utils.ts     Story date/time parsing, solar arc, palette derivation
  forecast-utils.ts Multi-day projection parsing and serialization
  scene-tokens.ts   Pure palette/opacity token math for the FX layers
  lumi-state.ts     Public LumiState v1 snapshot mapping
  story-history.ts  Story state rebuild and message-timestamp handling
  tag-dedupe.ts     Stable identity for intercepted weather tags
  version.ts        Release metadata and advertised capabilities
  presets.ts        Quick scene presets
  types.ts          Shared types
  ui/
    settings.ts     Extension settings panel
    styles.ts       HUD, settings, and FX styles

dist/
  backend.js
  frontend.js
```

`dist/` is committed because Lumiverse loads those bundles directly. Run `bun run build`, then `bun run check:dist` to confirm the committed bundles match a fresh build. CI runs the typecheck, the test suite, and that drift check.

## Notes

- This extension does not use a live weather API or perform fallback generation.
- The weather is intentionally narrative/state driven.
- The HUD is meant to stay fairly small and readable over chat.
- The settings page exposes both quick prompt guidance and full manual scene controls.
