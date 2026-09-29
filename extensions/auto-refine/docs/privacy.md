# Privacy

## What leaves your machine

With one model, which is the default, only one thing: the model call that does the refining. It goes through Lumiverse to the provider you set up, on the connection you picked.

With two models there is a second call, to the second model you picked. See [The second model](#the-second-model) below.

The refine call carries the blocks you see under **How the prompt is built**, and nothing else:

- the message being refined
- the rules you wrote
- the character card, if that block is on and the `characters` permission is granted
- the last few messages of the chat, if that block is on, cut to a size limit
- the lorebook entries this chat has active, if that block is on and the `world_books` permission is granted, cut to a size limit
- what Lumiverse remembers of the chat, if that block is on and the `chats` permission is granted
- any block you wrote yourself

The panel shows what is sent:

- A block that is switched off is not sent.
- **Messages of run-up to send** set to 0 sends no chat history.
- Switching off the **Cast Sheet** block sends no card.
- **Show me the request**, on the Context tab, builds the request a refine would send and shows it to you without sending it anywhere. What the checks found is not in it, since a preview does not ask the second model. The Log says when a refine sent it.

Two things are taken out of a message before it is sent:

- **The model's own thinking.** It is never sent.
- **Your markup.** Tags, code and image links are replaced with tokens and put back afterwards, so a rewrite cannot change them.

Your settings are never sent. Nothing from a chat you are not in is sent.

The extension has no networking of its own. It never contacts a server of mine. You can check by searching the two source files for `fetch(`, `XMLHttpRequest`, `WebSocket`, `sendBeacon` or `EventSource`. None of them appear. The only call to somewhere other than your provider is the call to the second model, with two models on, and it goes through Lumiverse's own proxy, `spindle.cors`.

There are two links on the Model tab, **What is Jev?** and **What is Span?**. They point at TypeSafe's page introducing Jev and Respan's page introducing Span. Showing them fetches nothing. Tapping one opens the page in your browser.

## The second model

With **How many models** set to two, each finished reply is sent to the second model you picked, Jev or Span, before it is refined. It is reached through the host you picked under **Where it is reached**: OpenRouter, NanoGPT, TypeSafe, Respan, or an address you typed. That host is a separate service with its own terms. It receives:

- the reply, with its thinking taken out
- the checks you wrote under **What the second model checks**
- the list of phrases this chat has worn out, if **Also check for worn-out phrases** is on
- the reply before it, with its thinking taken out, if **Also compare with the reply before it** is on
- your key, which tells the host the call is yours

Nothing else goes to the second model. Your card, your own messages, the rest of the chat, lore, memories and settings stay out of it.

A refine you start yourself is only sent to it when **Let it check refines you start yourself** is on. A selection and your own messages are never sent to it.

Your key:

- is sent from the panel to the backend once, when you press **Save key**
- is kept in Lumiverse's secure store, for your account only, with one key for each host, and for **Another address** one for each address
- is only sent over `https://`, or over `http://` to the same computer, unless you switch on **Let the key go over http://**
- is never in your settings, an export, or this browser's storage
- is never shown again. The panel is only told whether a key is saved.
- is deleted by **Forget key**, for the host shown

## The seven permissions

| Permission | What it is for | Without it |
| --- | --- | --- |
| `generation` | Runs the refine. | Nothing is refined, and the panel says so. |
| `chat_mutation` | Saves the rewrite over the message. | Refines run, but nothing can be saved. |
| `chats` | Knows which chat you are in and which card it belongs to. Reads what Lumiverse remembers of the chat, for `{{memories}}`. It only reads memories, never changes them. | It uses the last chat it saw a reply in, sends no card, and leaves out memories. |
| `characters` | Reads the character card, so a rewrite keeps each character's voice. | The card block is left out. Refining carries on. |
| `world_books` | Reads the lorebook entries this chat has active, so a rewrite does not contradict them. | The lore block is left out. Refining carries on. |
| `ui_panels` | The floating button, which is off by default. | Everything works except that button, and the panel says so. |
| `cors_proxy` | Reaches the second model, with two models on, which is off by default. | Two-model mode refines every reply, like one model. Nothing else changes. |

The ones that read your chat, card and lore make a rewrite sound like your characters. Refusing them lowers quality, but the extension still works.

## The part that reads the page

**Refine what I am typing** reads and writes the chat input box directly, because Lumiverse has no API for it. It is off by default.

- It only reads the box you are typing in.
- The text goes to the same place a refine goes: your provider, on your connection.
- It is not sent anywhere else, and it is not saved to your chat.

The Auto Refine tab itself needs no permission. Every extension can add a drawer tab.

## What it keeps

- **Your settings**, in your browser and in Lumiverse's storage for your account, so they follow you to other devices.
- **The chats you switched it off in**, in your browser. Only the chat ids are kept. Chat titles and text are not.
- **Your presets and model setups**, in your browser and your account, stored apart from your settings.
- **Your keys for the second model**, one for each host you saved one for, in Lumiverse's secure store for your account and nowhere else.
- **Your own sound**, if you chose one, with your settings. It never leaves your machine.
- **The text from before each refine**, in memory, so you can put a refine back. It is never written to disk and is gone when you reload.

## What it never does

- It never refines the greeting.
- It never sends anything the block list does not show.
- It never keeps a copy of a reply after the refine finishes.
- It never writes to any message except the one it refined.
- It never runs text as code. There is no `eval` and no `new Function`, so nothing in a reply, a rule or a model's answer can run.

## Checking any of this yourself

Lumiverse loads two files, named in `spindle.json`: `dist/frontend.js` and `dist/backend.js`. They are readable code, not minified or bundled. What you read is what runs. If you are checking this extension, or pointing a scanner at it, those two files are all of it.

The rest of the repo is for working on the extension and never reaches your browser:

- `src/` is the TypeScript the two files are built from. A scanner that only reads JavaScript cannot read it. The `dist/` files are plain JavaScript.
- `test/` only runs when a developer runs `bun run check`.
- `setup.sh` prepares a developer's machine. Nothing runs it when you install.
- `docs/` is these pages.
- `.github/workflows/` runs the checks on pull requests.

The checks rebuild `dist/` on every push and fail if it differs from `src/`, so the files Lumiverse loads always match the source.

---

[Back to the README](../README.md)
