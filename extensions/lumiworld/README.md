<!-- Mirror of upstream documentation. Everything below the metadata table is the original file, unmodified. -->

| Field | Value |
| --- | --- |
| Extension | LumiWorld |
| Source repository | https://github.com/Archkr/Lumiverse-LumiWorld |
| Original link (from Lumiverse-Extensions README) | https://github.com/Archkr/Lumiverse-LumiWorld |
| Upstream path | `README.md` |
| Retrieved | 2026-09-29 @ `main` (`5923eeb`) |

---

<div align="center">

# LumiWorld

**A private Director for your next Lumiverse reply.**

[![Version](https://img.shields.io/badge/version-0.4.0-8b7cf6)](./spindle.json)
[![Lumiverse](https://img.shields.io/badge/Lumiverse-%E2%89%A5%201.0.6-d4a35a)](https://github.com/prolix-oc/Lumiverse)
[![License](https://img.shields.io/badge/license-Lumiverse%20Community%202.0-6f9f78)](./LICENSE.md)

*Give the world a reason to move forward.*

</div>

LumiWorld prepares a short piece of direction before the main model writes its next reply. It considers the recent conversation and the context you enable, then suggests how the environment, NPCs, consequences, or hidden pressures should develop.

The resulting note is added to the main model’s prompt. Your chat model still writes the scene.

**Version 0.4.0 focuses entirely on the Director.** Connection setup, reply types, context controls, private notes, and advanced settings live in one compact drawer that follows your Lumiverse theme. The floating widget and World Agent simulation have been removed.

> **Private means prompt context:** Director notes are intended to stay out of the visible story. They are sent to the selected models and can be inspected in Prompt Breakdown. This is not an encryption or secrecy guarantee.

---

## Table of contents

- [At a glance](#at-a-glance)
- [How it works](#how-it-works)
- [Compatibility](#compatibility)
- [Installation](#installation)
- [Quick start](#quick-start)
- [The Director drawer](#the-director-drawer)
- [Settings reference](#settings-reference)
- [Prompt templates](#prompt-templates)
- [Testing and Prompt Breakdown](#testing-and-prompt-breakdown)
- [Generation time and usage](#generation-time-and-usage)
- [Permissions](#permissions)
- [Privacy and storage](#privacy-and-storage)
- [Upgrading to 0.4](#upgrading-to-04)
- [Troubleshooting](#troubleshooting)
- [Development](#development)
- [License](#license)

---

## At a glance

| Feature | What it does |
|---|---|
| **Direction before each reply** | Prepares a fresh note about what should change, how NPCs should respond, and what should remain unresolved. |
| **Your choice of model** | Uses a saved Lumiverse connection with an optional model override. |
| **Selective triggers** | Runs before new replies, continuations, regenerations, swipes, and impersonations only when selected. |
| **Context controls** | Includes recent chat history plus optional character, persona, and activated World Info context. |
| **Private guidance** | Accepts your own notes for the Director without adding them as a visible chat message. |
| **One drawer** | Keeps setup and everyday controls together, with notes and advanced settings expandable in place. |
| **Automatic saving** | Saves edits as you make them and retains the draft for retry if a save fails. |
| **Inspectable output** | Attributes the injected system block as **LumiWorld Director** in Prompt Breakdown. |
| **Graceful failure** | Continues the normal generation without a Director note if the Director fails or times out. |

## How it works

```text
You request a selected reply type
                 │
                 ▼
Recent history + enabled context + your notes
                 │
                 ▼
Director connection prepares a short directive
                 │
                 ▼
“LumiWorld Director” system block is added to the prompt
                 │
                 ▼
Your chat model writes the visible reply
```

1. **A selected generation begins.** LumiWorld checks whether the Director is enabled for that reply type.
2. **The Director receives context.** It uses recent chat messages from the assembled prompt, the additional context sources you enabled, and any private notes.
3. **It writes a directive.** The built-in prompt asks for concrete changes, environmental pressure, NPC intent, consequences, and things that should remain unresolved. The note is limited to 2,200 characters.
4. **The main model receives the note.** LumiWorld prepends a system block named **LumiWorld Director** to the existing prompt.
5. **The reply proceeds.** The main model uses that direction while writing the scene. If no usable directive is returned, the original prompt passes through.

No macro needs to be added to your character card or preset for the Director to work.

## Compatibility

| Requirement | Value |
|---|---|
| Lumiverse | `1.0.6` or newer |
| Extension version | `0.4.0` |
| Director connection | An explicitly selected Lumiverse connection profile with a usable model |
| Essential permissions | `interceptor` and `generation` |
| Build output | Committed `dist/backend.js` and `dist/frontend.js` |

Settings are saved per Lumiverse user and apply across that user’s chats. The Director uses the context of the generation currently being processed.

## Installation

1. Copy the repository URL:

   ```text
   https://github.com/Archkr/Lumiverse-LumiWorld
   ```

2. Open **Lumiverse → Extensions → Install**.
3. Paste the repository URL and choose **Install**.
4. Enable **LumiWorld** and grant its permissions.
5. Open **LumiWorld** in the drawer.

The repository includes its built bundles, so a normal installation does not require a local build.

For updates, use the update action on LumiWorld’s entry in Lumiverse’s Extensions panel and reload the extension if prompted.

## Quick start

| Step | Action |
|---|---|
| 1 | Open the **LumiWorld** drawer. |
| 2 | Select a **Connection**. Leave **Model** blank to use that connection’s default, or choose an override. |
| 3 | Choose **Test Director** and check the result. Testing works while the Director is disabled. |
| 4 | Select the reply types under **Run before**. |
| 5 | Choose whether to include **Character**, **User persona**, and **Activated World Info**. |
| 6 | Optionally expand **Director notes** and add guidance for the scene. |
| 7 | Turn on the switch beside **Director** and wait for **All changes saved**. |
| 8 | Generate a reply, then inspect **LumiWorld Director** in Prompt Breakdown. |

For example, your Director notes might say:

```text
Let the storm complicate travel. Keep the observatory's purpose uncertain,
and give nearby NPCs practical reasons to disagree about entering it.
```

Director notes remain in your user settings until you change or clear them. Clear scene-specific guidance before moving to an unrelated chat.

## The Director drawer

- **Director switch and status:** enable or disable automatic direction and see whether setup is complete.
- **Connection and Model:** select the model that prepares the note.
- **Test Director:** try the current draft settings and read the success or error feedback.
- **Run before:** select which reply types trigger a Director call. Unchecking every type stops automatic Director calls.
- **Include in context:** control the additional character, persona, and World Info context sent to the Director.
- **Director notes:** expand to edit your private guidance.
- **Advanced settings:** expand for response limits, history, retention, and prompt templates.
- **Save status:** edits save automatically. If saving fails, your draft stays in the open drawer and **Retry save** becomes available.

## Settings reference

### Connection, triggers, and context

| Setting | Default | Behavior |
|---|---|---|
| Director | Off | Enables automatic calls before selected generations. |
| Connection | None | Required. Select a saved Lumiverse connection; there is no automatic fallback to your chat connection. |
| Model | Blank | Uses the selected connection’s default model unless overridden. |
| Run before | All five types | New reply, Continue, Regenerate, Swipe, and Impersonate. Quiet and background generations are excluded. |
| Character | On | Adds the active character’s context when available. |
| User persona | On | Adds the active persona’s context when available. |
| Activated World Info | Off | Adds activated World Info entries, rather than entire World Books. |
| Director notes | Empty | Sends extra guidance as a separate system message to the Director. Supports `{{user}}` and `{{char}}`. |

Context switches control what LumiWorld adds to the Director’s input. They do not remove context already present in the main model’s prompt.

### Advanced settings

| Setting | Default | Range / meaning |
|---|---|---|
| Temperature | `0.35` | `0–2`; controls variation in the Director’s response. |
| Max tokens | `420` | At least `64`; output budget for the Director call, subject to the selected provider’s limits. |
| Timeout (ms) | `45000` | `1000–300000`; maximum wait for the Director call. Lumiverse caps interceptors at five minutes. |
| History messages | `12` | Most recent chat messages included in Director context. `0` excludes chat history. |
| Prompt cap (chars) | `60000` | `4000–500000`; caps the serialized history and enabled context, before custom templates and notes are added. |
| Run log limit | `12` | `0–50`; number of Director run records retained in extension storage. `0` disables retention of Director records. |

The prompt cap is measured in characters, not tokens. Large histories and custom templates still need to fit the selected model’s context window.

## Prompt templates

Open **Advanced settings → Prompt templates** to customize the system and user instructions sent to the Director.

The built-in templates ask for one forward-looking directive: concrete world changes, pressure on NPCs, what to show next, and what to leave unresolved. They discourage recaps and visible dialogue. The preferred response is:

```json
{"director_note":"Let the power fail in the west wing, forcing the caretaker to choose between protecting the visitors and concealing the locked archive."}
```

Plain text is also accepted. Only the final response content is used; a reasoning-only response does not produce a Director note.

<details>
<summary><b>Template variables</b></summary>

| Variable | Value |
|---|---|
| `{{prompt}}` | Formatted, size-limited Director context. Keep this in a template to include the selected context. |
| `{{generationType}}` | `normal`, `continue`, `regenerate`, `swipe`, or `impersonate`. |
| `{{user}}` | Resolved user/persona name, with `User` as the fallback. |
| `{{char}}` | Resolved character name, with `Character` as the fallback. |
| `{{chatId}}` | The current chat ID when available. |
| `{{connectionId}}` | Connection ID supplied by the intercepted generation; during a test, the selected Director connection ID. |
| `{{timestamp}}` | ISO timestamp for the Director request. |
| `{{maxDirectiveChars}}` | Maximum directive length: `2200`. |

Director notes are always sent separately. The legacy `{{additionalNotes}}` variable expands to an empty string to avoid duplicating them.

Saving an empty system or user template restores its built-in default.

</details>

## Testing and Prompt Breakdown

**Test Director** sends a short sample scene about an ancient observatory during a storm. It uses the current draft connection, model, response settings, prompt templates, and Director notes. It does not load the active chat’s history, character, persona, or World Info, and it does not add or change chat messages.

A successful test confirms that the connection can return a usable directive. To verify the complete chat flow:

1. Enable the Director and select a reply type.
2. Wait for **All changes saved**, then generate that type of reply in a chat.
3. Open Lumiverse’s **Prompt Breakdown** for the generation.
4. Look for the **LumiWorld Director** system block and inspect its note.

The drawer shows test feedback directly. It does not include a Recent activity section.

## Generation time and usage

Each eligible reply makes one extra model call before the main reply starts. **Test Director** also makes a model call. Both use the provider and billing associated with the selected Director connection.

The Director’s response time adds to the wait before the visible reply. History length, model choice, output budget, and enabled context affect usage and latency.

If another Director call is already running for the same user and chat, a duplicate request proceeds without an additional Director call. Separate chats can run independently. Failed, empty, or timed-out Director responses leave the original generation prompt unchanged.

## Permissions

| Permission | Used for |
|---|---|
| `interceptor` | Inspect the generation context and inject the Director system block. |
| `generation` | List saved connections and make Director calls. |
| `chats` | Resolve the chat’s character when routing information is needed. |
| `characters` | Read character context and identity. |
| `personas` | Read persona context and identity. |
| `world_books` | Read activated World Info metadata and entry content. |

The drawer warns when a required permission is missing. Character, persona, and World Info access supports the corresponding context options. The drawer does not require `ui_panels`, and LumiWorld no longer requests `chat_mutation`.

## Privacy and storage

LumiWorld uses Lumiverse’s connection profiles and does not read or store API keys.

- **Sent to the Director provider:** the selected history and context, prompt templates, and your Director notes.
- **Added to the main model’s prompt:** the resulting Director note. The private notes field is not copied directly into the injected block, but it can influence the result.
- **Stored by the extension:** user settings, custom templates, private notes, and retained run records.
- **Run records:** timestamps, statuses, connection/model details, timing, errors, World Info diagnostic counts, and a directive preview of up to 360 characters. That preview may contain story details.

Run records do not separately archive full input prompts, raw provider responses, or World Info entry bodies. A short directive can fit entirely inside its preview. Retained data is ordinary extension storage; “private” does not mean encrypted.

## Upgrading to 0.4

Version `0.4.0` keeps the Director and removes the former World Agent feature:

- The floating widget, World view, and duplicate settings modal are gone.
- World Agent scheduling, commands, simulation, and prompt injection no longer run.
- The public `agent_world.state.current` LumiState endpoint is no longer published.
- Existing World Agent files, settings, and historical run records remain stored for possible recovery with an older release. There is no deletion or migration of that data.
- Existing Director settings and custom prompt behavior are retained.

The technical extension identifier remains `agent_world` so existing installations retain their storage identity.

## Troubleshooting

| Symptom | What to check |
|---|---|
| **Test Director is disabled** | Select an available connection, ensure it has a default or overridden model, and grant `generation`. The hint below the button explains what is missing. |
| **Setup needed** | Check the connection/model, `interceptor` and `generation` permissions, and that at least one reply type is selected. |
| **No Director block in Prompt Breakdown** | Confirm the Director is enabled, settings have saved, and the current reply type is selected. Run Test Director to check the connection. A failed, empty, timed-out, or duplicate call skips injection. |
| **Saved connection unavailable** | The saved profile may have been removed or become inaccessible. Select an available connection and test it again. |
| **The Director returns no final note** | Test the chosen model and review custom templates. Reasoning-only output is ignored; the Director must return final text or a `director_note` response. |
| **Replies take too long** | Reduce history or the Director output budget, choose a faster model, or shorten the timeout. |
| **Context is missing** | Check the context switches and permissions. World Info must be activated for the chat. The history limit and prompt cap can reduce the included context. |
| **Save failed** | Keep the drawer open and choose **Retry save**. Your unsaved draft remains available there. |
| **Old scene guidance appears in another chat** | Director notes are shared across your chats. Clear or replace scene-specific notes when switching stories. |

## Development

```bash
bun install
bun run typecheck
bun test
bun run build
```

### Project layout

```text
src/
  backend.ts        Director calls, interception, settings, and storage
  frontend.ts       Director drawer and shared Lumiverse controls
  shared.ts         Defaults, context selection, templates, and response parsing
  types.ts          Frontend/backend message contracts
  backend.test.ts   Backend and interceptor tests
  frontend.test.ts  Settings normalization and autosave queue tests
  shared.test.ts    Context, generation selection, and prompt behavior tests

dist/
  backend.js        Backend bundle loaded by Lumiverse
  frontend.js       Frontend bundle loaded by Lumiverse

spindle.json        Extension manifest and permissions
```

Commit rebuilt `dist/` files with source changes: Lumiverse loads these bundles directly. Use the main Lumiverse repository’s `developer-docs/` as the API and shared-component reference.

## License

LumiWorld is distributed under the [Lumiverse Community License, Version 2.0](./LICENSE.md).
