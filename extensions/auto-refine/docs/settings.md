# Settings

Auto Refine lives in a tab in Lumiverse's sidebar drawer. Open it from the drawer, or find it with `Ctrl+K`. It has no settings window and is not in the Extras menu.

It is a tab so you can keep it open while you write, and see what the last refine changed.

## How settings are saved

- **There is no Save button.** Everything saves as you change it.
- **Only deleting a preset and starting again throw anything away**, and both ask first.
- **Your settings follow your account.** They are saved to Lumiverse under your account, so a different browser or device finds the same setup. This browser also keeps a copy, so the panel opens instantly and keeps working if your account cannot be reached.
- **The account copy wins.** When the panel opens, it uses this browser's copy until your account's copy arrives, then switches to the account's. Only a change you make is saved to your account. Opening Lumiverse on another device never saves that device's copy over yours. If the panel opens before the extension's server side has started, it asks your account again once that side is ready.
- **A tab left open catches up.** Come back to a tab after more than 30 seconds away and it loads your settings and presets from your account again. So an old tab on your phone does not bring back an older prompt.
- **If a save to your account fails, the panel says so.**
- **On a server with several accounts**, each account's settings and presets are stored separately. No account can read another's.

**One install for several accounts.** An admin can install Auto Refine once for everyone on a server. Each account's refines still use that account's own settings:

- A refine you start uses your settings.
- The automatic pass uses the settings of the account the chat belongs to. If Lumiverse cannot say whose chat it is, the account whose panel last sent its settings is used, and Lumiverse's server log says so.
- Refines take turns between accounts. If another account's refine is running, yours waits for it to finish, and the panel says "Waiting for another account's refine to finish".
- Your own refines never wait for each other.

## Finding a setting

The search box above the tabs searches every tab, not only the one you are on.

- Results are grouped by tab.
- It also searches inside folded sections and inside each setting's description.

## Above the tabs

These stay in place whichever tab is open:

- **The switch** turns everything on or off. Off, nothing is refined and no model is called.
- **Refine the latest reply** refines one reply now. While it runs, the buttons change to **Stop this refine**.
- **Refine every reply here** goes through the chat you are in, oldest first, one model call each. It asks before it starts.
- **every reply, automatically** is the automatic pass. It is off by default.
- **Refines you can put back** lists every refine in this chat, newest first, each with **Put it back**. The tab shows a badge with the count.

The greeting is never refined. Your own messages are only refined when you press the button on one.

## Prompt

- **Your prompt** is the whole request, block by block. You can rename, reorder, switch off, and add blocks, and choose the role each is sent as. **Expand** opens a block in a full-screen editor. See [How the prompt is built](prompt.md).
- **Fold all** folds every block down to its name, and **Open all** opens them again. Each preset remembers its own folds.
- **Macros you can use** lists what a block can contain. Tap one to copy it, and press **?** to see what it turns into.
- **Presets** save a prompt under a name, and can name a model setup to load with it. Four come built in. See [Presets](prompt.md#presets).

## Context

- **How much it is told** sets how many messages of chat history are sent, whether each line names who said it, and the size limits for history and lorebook. See [How much it is told](prompt.md#how-much-it-is-told).
- **See what gets sent** builds the request a refine would send and shows it, without calling a model. What the checks found and the passes after the first are named rather than shown. **Raw** shows it as the JSON that is sent. **Expand** opens it full screen.

## Model

Not sure what to pick? [What to use](recommended.md) has settings that work well for most people.

- **Refine using** picks the model that refines, from your connection profiles. Leave it on the default to use the model you chat with.
- **Let it think first** is off by default. You can leave it to your connection's setting, or pick an effort level. See [How much thinking it does](prompt.md#how-much-thinking-it-does).
- **Give up waiting after** stops a refine that has not come back.
  - The default is four minutes. The most is an hour. 0 means never give up.
  - Four minutes is long on purpose. A fast model answers in seconds. A reasoning model on a high setting, or a local model loading, can take minutes, and stopping it early throws that work away.
  - With it off you are never stuck: **Stop this refine** is always there.
  - The time counts while the tab is in the background too. On a phone, a refine that ran out while you were away ends as soon as you come back to the tab. A reply that finished while the tab was asleep can miss the panel, so check the reply itself.
- **Input price, per million tokens** and **Output price, per million tokens** are your provider's prices.
  - Input is what you send. Output is what the model writes back.
  - Both start at 0, which hides all costs.
  - There is no currency. The number you type is the number you see.
  - See [What a refine costs](prompt.md#what-a-refine-costs).
- **Samplers** start blank, which means your connection's preset decides. See [Sampler settings](prompt.md#sampler-settings).
- **Saved model setups** keep everything on this tab under a name: the connection, thinking, timeout, samplers and prices.
  - Save one for a cheap model and one for a careful one, and switch between them.
  - A setup never contains your prompt, so loading one only changes what runs the refine.
  - Setups are kept in this browser and your account.
  - A preset can name a setup, so the two load together.
  - If a setup's connection has been deleted, the card says so.
- **One model or two** is a beta. With two, a small second model, Jev or Span, checks each new reply first, and only the replies that need it are refined. It has its own host, version, key and checks, and every text it sends is in a box you can change. See [Two models](two-models.md).

## Limits

**Protecting what is not prose.** See [Protecting what is not prose](prompt.md#protecting-what-is-not-prose).

- **Hide markup from the model** and **Hide plain italic and bold too** keep formatting out of the rewrite.
- **What the model is told about the stand-ins** is the text `{{protect_notes}}` puts in. You can change it. **Use the built-in text** puts it back.
- **Keep the reply's own reasoning out of the refine** keeps the model's thinking out.
- **Extra reasoning tag names** is folded under it. Thinking in these tags is already recognised: think, thinking, thought, thoughts, reasoning, reflection, scratchpad and analysis. Only add a name if your model uses a different one. Thinking that is not recognised is rewritten like prose and saved over the reply.
- **Patterns of your own to hide** is folded under the markup switch.

**Reading the answer.**

- **Take the answer from between the tags** takes only the rewrite from the model's answer, so a preamble is ignored. See [The answer it asks for](prompt.md#the-answer-it-asks-for).
- **Say how much has come back** shows the answer arriving, so the status line counts characters. It does not change what is saved.

**What it refuses to save.** See [What it refuses to save](guardrails.md).

- **Longest a rewrite may get** and **Shortest a rewrite may get** are the two length limits.
- **Refuse an answer that declines the job**, **Refuse an answer that talks about the edit** and **Refuse a rewrite that sanitised the reply** each have a switch. If all three are off, the panel warns you.

**Before it writes.**

- **Refine something that has been refined before** is off by default.
  - Off, a reply that still holds its refine is not refined again, by any button or by the automatic pass.
  - A reply you swiped, regenerated or edited holds different words, so it is refined either way. This is what lets it work alongside [Auto Retry](https://github.com/starlitcode/Lumiverse-Auto-Retry).
  - One reply announced twice is still only refined once.
- **Wait out a provider that will not take the call** waits and tries again, twice by default, when the provider is busy or a local model is loading. See [Waiting out a provider that will not take the call](guardrails.md#waiting-out-a-provider-that-will-not-take-the-call).
- **Seconds between automatic refines** puts a gap between automatic refines, for a provider that limits calls per minute. A reply that arrives too soon waits, with a countdown, and is then refined. It is 0, no gap, by default. Refines you start yourself never wait.
- **Ask again when a check fails** retries a refine that failed a check a second try could fix. It is 0 by default.
- **Add the refine as a swipe instead of writing over the reply** is off by default.
  - On, the rewrite is added as a new swipe, and the original stays one swipe back.
  - This is the only way back that survives a reload.
  - **Put it back** then removes that swipe. It refuses if you have added another swipe since.
  - **Your next / swipe button** is under it. You only need it if a Lumiverse update renames the swipe arrows. It works like **Where the input box is**, below.
- **Keep what a refine replaced** keeps the original, so you can put it back. It is kept in memory until you reload.
- **Ask before saving a refine** holds every refine until you decide.
  - It appears in a **Waiting for you** card at the top of the tab, with **Accept it** and **Turn it down**.
  - Where Lumiverse can show one, the question also opens as a window. Answering either one answers both.
  - The floating button's menu can answer it too. A tap on the button only opens the tab.
- **Your own messages** are only refined when you press a button on one. They use their own prompt, under **For your messages** on the Prompt tab.

## Log

- **Right now** shows what is happening: asking, thinking, writing (with a character count), checking, or waiting.
  - A clock runs while it works. After eight seconds it also shows how long is left before the timeout.
  - Below, it shows what the last refine used, in input and output tokens, and what it cost if you set prices. This includes dropped rewrites and retries, because those were paid for too.
- **What the model worked out** keeps the model's notes from the last refine that finished. It has **Copy**, **Expand** and **Clear**. Stopping a refine leaves the last notes alone.
- **What it has been doing** lists what it did, newest first.
- **Reporting a problem** copies everything needed for a bug report.
  - **What it carries** chooses what goes in: your settings, the shape of your prompt, the counts, the recent log, where you are, and your browser.
  - What your blocks say is never included, only their names, roles and macros.
  - **Read and edit it first** lets you check it before it is copied.
  - The first two lines always name the panel's version and the backend's version. They differ if you updated without reloading.

## Setup

**This chat** switches Auto Refine off in the chat you are in, and leaves other chats alone.

- It is saved in your browser, as a list of chat ids.
- In a **temporary chat**, one with no character card, the switch works while the chat is open but is not saved, because the next temporary chat has a different id.

**When a refine finishes** chooses how you are told.

- **Show the before and after on screen** puts a card on the page with the before, the after, and a button to put it back. It is on by default.
- **Read them side by side** or **Read them together** switches every before-and-after on screen between two columns and one text with the changes coloured. The next card opens the way you left it. On a narrow screen, the columns stack.
- **Show a brief message** is the one-line note at the edge of the screen.
- The sound is off by default. With no file chosen, it is a short built-in sound.

**Ways to reach it** are four optional ways in, all off by default.

**A floating button** puts a round button over the chat. It needs the interface panels permission.

- A tap refines the latest reply. That is all a tap does.
- Hold it, or right-click it, to open its menu: the tab, refining every reply, stopping a refine, putting one back, refining your draft, hiding the button, and the main switch.
- Holding fills a ring around the edge. The menu opens when the ring closes. Let go early and nothing happens.
- Its mark is an eye. It is shut when nothing is running, and reads while a refine runs. It blinks once when a refine finishes, and closes without a blink when you stop one.
- If your device is set to reduce motion, the eye stays still and there is no ring.
- Drag it where you want it. It stays there, in this browser.

**A button in the chat's row of controls** adds a button to Lumiverse's own row of chat buttons. A tap refines the latest reply.

**A button on every message** adds a button to each message's row of buttons. A tap refines that message.

- It is the only way to refine an older message without selecting all of it.
- It works on your own messages too.
- On the greeting, it says the greeting is never refined.
- While a message is open for editing, its button is hidden, like Lumiverse's own.

Both chat buttons copy the look of the Lumiverse button next to them, so your theme and your own CSS apply to them. While a refine runs, each turns into **Stop this refine**.

**Refining part of a reply** needs no setting. Select part of a reply and **Refine the part I selected** appears:

- on the panel
- in the chat input's Extras menu
- in the floating button's menu, when it is on screen
- on the chat buttons, when they are on

It rewrites only what you selected. It works in your own messages too, with their prompt.

**Take out what I selected** appears in the same places. It deletes the selection without calling a model, so it costs nothing.

- It tidies the gap. It leaves one space where a phrase was cut, and one blank line where a paragraph was.
- It will not delete the whole message.
- **Put it back** undoes it.

**Refining the draft in your input box** is off by default, because it writes into the box you type in. On, **Refine what I am typing** appears above the tabs and in the Extras menu, or in the floating button's menu when that is on screen. It is greyed out when no chat is open, since there is no input box to read. **Your draft, refined** then shows at the top of the tab, with **Put it back**, **Dismiss** and **Read it in full**.

While the floating button is on screen, its menu holds these extra actions. With the button off, they are in the chat input's Extras menu instead, which is how you reach them on a phone.

**Where the input box is** is only needed if a Lumiverse update moves the input box.

- **The selectors it looks under** is the list it tries, in order, separated by commas.
- Emptying the box goes back to the built-in list.
- The list below the box shows which selector found the box. A selector the browser cannot read is marked in red.
- **Test** checks the page when you press it, and says whether the box was found and can be typed in.
- **Use the built-in list** puts the box back to the built-in list.

**Your whole setup** exports your settings to a file, imports a file, or resets to defaults.

- All three use the same list of parts:
  - Your prompt
  - Context
  - Model and thinking
  - Samplers
  - Limits and protection
  - Alerts and sound
  - Buttons and the widget
  - Where the input box is
  - One model or two (never the key for the second model)
  - The on and off switches
  - Saved presets
  - Saved model setups
  - Chats you switched off
- **What goes in the file**, **What to take from a file** and **What to put back** each choose their own parts. So you can export only your prompt, or reset your prompt and keep your connection.
- See [Import and export](prompt.md#import-and-export) and [Starting again](prompt.md#starting-again).

---

[Back to the README](../README.md)
