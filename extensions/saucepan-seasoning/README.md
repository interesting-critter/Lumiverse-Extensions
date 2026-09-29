<!-- Mirror of upstream documentation. Everything below the metadata table is the original file, unmodified. -->

| Field | Value |
| --- | --- |
| Extension | Saucepan Seasoning |
| Source repository | https://github.com/bumyann/Lumiverse-Saucepan-Seasoning |
| Original link (from Lumiverse-Extensions README) | https://github.com/bumyann/Lumiverse-Saucepan-Seasoning |
| Upstream path | `README.md` |
| Retrieved | 2026-09-29 @ `main` (`e29829a`) |

---

# Saucepan Seasoning — Lumiverse

<img src="https://i.imgur.com/dKkOyfc.png" alt="SaucepanSeasoning">

Port of SaucepanAI's response instructions and write-for-me UI for **Lumiverse**, built as a native Spindle extension. Made this because I kept wanting Saucepan's in-chat commands while using Lumiverse and figured porting them was less painful than context-switching forever.

Disclaimer: Vibe-coded with Claude. Tested before publishing because I'm not a rat.

---

## Features

### ✦ Response Instructions

Inject a steering instruction into the prompt before every AI reply — no character limit, no fuss.

- **Simple mode** — structured chip selectors for length, style, intimacy, pacing, narration balance, and more. picks compose into a clean natural language instruction automatically. good for quick setups without thinking too hard.
- **Custom mode** — raw freeform textarea. write exactly what you want, no restrictions.
- Toggle on/off at any time without clearing your text
- Dot indicator on the toolbar button stays visible when active, even with the panel closed
- Preset library — save, load, and delete named instruction sets

### ✦ Write For Me

AI drafts a reply for you using your active connection and the current chat context.

- Add an optional direction (`act nervous`, `change the subject`, `confess feelings`…) or leave it blank
- Hit **Generate** — uses whatever connection you have set up in Lumiverse
- Browse drafts with prev/next navigation, generate more if you don't like what you got
- Save drafts you like for later from the **Saved** tab
- Hit **Use this** to drop the draft into the chat input, then send as normal

---

## Install

1. Lumiverse → Extensions → Install Extension → paste this repo's URL
2. Enable the extension and grant `interceptor` and `generation` permissions
3. Open any chat — the toolbar appears just above the message input

---

## Usage

Both features live in a row just above the chat input.

**Response Instructions:**
- Click the 📋 button to open the panel
- Pick your settings in Simple mode, or switch to Custom for freeform input
- Toggle **ON** — dot indicator appears on the button, instructions inject on next send
- 📁 → preset library · 🗑 → clear · ✕ → close panel

**Write For Me:**
- Click the ✦ button to open the panel
- Optionally add a direction, hit **Generate**
- Browse drafts with ← → · bookmark one with 🔖 to save it
- **Use this** → drops the draft into chat input ready to send

### Simple Mode — What does each field inject?

| Field | Option | Injected instruction |
|---|---|---|
| Length | Short | Keep your response brief and concise. |
| | Medium | Write a moderate length response. |
| | Long | Write a long, detailed response. |
| | Essay | Write a lengthy, essay-style response with thorough detail. |
| | Ramble | Write a lengthy, rambling response — don't cut yourself short. |
| Style | First Person | Narrate in first person. |
| | Second Person | Narrate in second person, addressing the user as "you". |
| | Third Person | Narrate in third person. |
| | Text Messaging | Write in a casual text messaging style — short messages, no prose. |
| Speak For | Companion only | Only write dialogue and actions for your character. Do not write for the user. |
| | Both | Write dialogue and actions for both your character and the user. |
| Intimacy | Platonic | Keep the tone platonic. Avoid romantic or sexual content. |
| | Romantic | Keep the tone romantic and emotionally intimate. |
| | Sexual | Sexual content is permitted. |
| | Explicit | Explicit sexual content is permitted. Do not fade to black. |
| Story Pacing | Slow | Use a slow pace — linger on details, emotions, and atmosphere. |
| | Fast | Use a fast pace — keep things moving, minimize dwelling. |
| Narration vs Dialogue | Narration | Focus on narration and description over dialogue. |
| | Balanced | Balance narration and dialogue equally. |
| | Dialogue | Focus on dialogue over narration and description. |

Default on any field = nothing injected for that field. A live preview at the bottom of the panel shows the exact string that'll be sent.

---

## Permissions

| Permission | Why |
|---|---|
| `interceptor` | Inject the active instruction into the prompt before every LLM call |
| `generation` | Write For Me AI drafting |

Storage is free-tier — no extra permission needed. Everything (instructions, presets, Simple selections, saved drafts) persists across sessions.

---

## Credits

Feature design by [SaucepanAI](https://saucepan.ai). Lumiverse port by [bumyann](https://github.com/bumyann).

---

## License

MIT
