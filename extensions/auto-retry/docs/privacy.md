# Privacy

This page explains what Auto Retry can reach, what it keeps, and what it cannot get at. To report a security problem, see [SECURITY.md](../SECURITY.md).

## It works entirely on your own device

- **It has no networking.** It never opens a connection, never contacts a server of mine or anyone else's, and has no analytics. Everything happens inside your copy of Lumiverse.
- **You can check this.** Search the two source files for `fetch(`, `XMLHttpRequest`, `WebSocket`, `sendBeacon` or `EventSource`. None of them appear, and those are the only ways a browser extension can send anything out.
- **It never runs text as code.** There is no `eval` and no `new Function`, so nothing in a reply, a character card, or your settings can run.
- **There is one link**, in the warning before the crisis-support setting. It points at the safety page in this repository. Showing it fetches nothing. Tapping it opens GitHub in your browser.

## What it can reach, and why

It asks Lumiverse for five permissions. All five are privileged: on a shared server, each one does nothing until an admin approves it. None is needed for the extension to start.

| Permission | What it is for | Without it |
| --- | --- | --- |
| `generation` | Seeing replies start, stream and end. | It cannot see replies at all. This is the one it needs. |
| `ui_panels` | The floating on/off button, and the on-screen panel when it floats over the chat. It gives screen space, not access to your data. | Everything works, with no floating button. A panel in the sidebar drawer needs no permission. |
| `chats` | Asking which chat you are looking at. | Everything works. **Turn off here** waits to be told which chat you are in. |
| `characters` | Turning a chat's card id into a name, so the panel can say who a chat is with. It reads one card, for the chat you are in. | Everything works. The panel says "This chat" instead of a name. |
| `interceptor` | Adding the retry note to a prompt, and showing the whole prompt on the panel's **Prompt** tab. | Everything works. No retry note is sent, and the Prompt tab stays empty and says so. |

**About `chats`:** Lumiverse puts reading, creating and deleting chats into one permission, with no narrower one to ask for. Auto Retry only uses one call from it: which chat is open. It never creates, deletes or changes a chat.

**About `characters`:** it never creates, edits or deletes a card.

### The Lumiverse features it uses

Behind those permissions it uses eight Lumiverse features and nothing else:

- reading the messages in a chat
- updating a message
- asking which chat is open
- reading the name on one character card
- counting the tokens in a prompt
- reading its own settings
- saving its own settings
- adding the retry note to a prompt

### The note sent with a refusal retry

The note is the only thing in the extension that changes what the model is asked. It is off by default.

- It carries only the text you typed. Your prompt is not read, copied or stored, and nothing is written to your chat.
- Because it is not a message in your chat, it may not appear in Lumiverse's Prompt Breakdown. The on-screen panel writes a line naming the note on the retry that carried it.

It is added just before the extension presses your retry button, and five things keep it to that one reply:

- It belongs to one chat, and is never added to a reply in another chat.
- It is used once, then cleared, whether it was used or not.
- It expires after 45 seconds.
- If the retry press started nothing, it is taken back straight away. If there is no retry button to press, no note is set up at all.
- Optionally, **Only send them on a regenerate or a swipe** also requires Lumiverse to call the reply a regenerate or a swipe. It is off by default, because most versions of Lumiverse call every reply "normal", and the note would then never be sent.

## This page can change

A new feature can need a new permission. If that happens, this page says so before the version that needs it is released. `spindle.json` lists the permissions, and the changelog says when one is added and why.

A new permission is a real cost, because every user has to approve it again, so it is only asked for when a feature truly needs it.

Two things will never change: the extension will not make network calls, and it will not send anything anywhere.

## What it cannot reach

**Your API keys, passwords and account details.** Lumiverse does not give these to extensions, and nothing in Auto Retry looks for them. Search the source for `apikey`, `secret` or `credential` to check.

What it reads, and for how long:

- **Each reply**, to run its checks. It does not keep a copy.
- **The text of a reply while it streams**, because some versions of Lumiverse do not include the finished reply when it ends. This copy is dropped when the reply ends or is stopped.
- **The reply on screen when a reply starts.** It keeps only a fingerprint: a length and a number worked out from the characters. That is enough to tell if it changed, and useless for anything else. It is dropped when the reply ends.
- **The address in your browser**, to check one thing: does it still contain the id of the chat you are in. The answer is yes or no. The address is not stored or sent anywhere.

**One case sends a reply to the extension's own backend.** With the panel open on the **Prompt** tab and a price set under **What a retry costs**, the finished reply is counted by Lumiverse's own tokeniser, so the panel can show what the reply part of a retry costs.

- Only the number comes back. The text is not stored or used for anything else.
- It stays inside your Lumiverse.
- Close the panel, leave the Prompt tab, or leave the prices at 0, and this never happens. The panel then estimates at four characters a token and says **roughly**.

## What it keeps

**In your browser and your account:** your settings and saved presets. They follow your account between devices. This is the only thing that reaches your account.

**In this browser only:**

- where you left the on-screen panel and the floating button, their size, and which tab was open
- the chats you switched Auto Retry off in, as chat ids only: no titles and no text

Neither of these is in an export, because a screen position or a chat id means nothing on another device.

**In memory only, gone when you close the tab:**

- the last twenty lines of what it did
- counters for the session: good replies, retries, their reasons, retries per chat, and notes sent and skipped. Per-chat counts use the chat id, never anything a reply said.
- **replies a retry replaced**, if **Keep the reply a retry replaced** is on. You can read them back on the **Replaced** tab. At most eight chats' worth is kept, newest first. Turn it off under **How it retries** and nothing is kept.
- **the last prompt**, only while you have the panel's **Prompt** tab open. It is the text of your chat, so it is only kept after you open that tab. One prompt at a time. Close the panel and nothing is kept.

None of these is written to disk, synced, or sent anywhere.

## One thing to be careful with

Nothing here leaves your device by itself. The risk is only in what you copy and where you paste it.

- **Debug info** gathers your settings, button selectors, browser string, session counters and recent activity. The activity log can contain short pieces of a reply. Read what you copied before pasting it in public, and untick any section you would rather keep private.
- **Copy**, on the on-screen panel, copies everything the tab shows. On **Replaced**, that is a whole reply. On **Log**, the whole activity log. On **Prompt**, the whole prompt, which is most of your chat. **Stats** is the safe one: only counters and reasons.
- **The refusal tester**, in the settings, only reads the text in its box. **Use my last reply** fills the box with the reply on the page. The check runs on your device, and nothing is sent or stored.

## Checking any of this yourself

Lumiverse loads two files, named in `spindle.json`: `dist/frontend.js` and `dist/backend.js`. They are readable code, not minified or bundled. What you read is what runs. If you are checking this extension, or pointing a scanner at it, those two files are all of it.

The rest of the repo is for working on the extension and never reaches your browser:

- `src/` is the TypeScript the two files are built from. A scanner that only reads JavaScript cannot read it. The `dist/` files are plain JavaScript.
- `test/` only runs when a developer runs `bun run check`.
- `setup.sh` prepares a developer's machine. Nothing runs it when you install.
- `docs/` is these pages.
- `.github/workflows/` runs the checks on pull requests. Its actions are pinned to exact versions, and it keeps no credentials in the build.

The checks rebuild `dist/` from `src/` on every pull request and fail if it differs, so the files you read always match the source.

---

[Back to the README](../README.md)
