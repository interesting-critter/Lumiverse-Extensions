# All settings

This page covers how the settings panel works, the on-screen panel, and every option with its default.

## Finding a setting

There are about fifty options. The quickest way to reach one is the **Search settings** box at the top of the panel:

- Type part of a name or a description. Only the rows that match are shown, and any closed section they are in opens.
- Clear the box to put the panel back as it was.

The panel has eight sections. Three are open when the panel opens:

- **Basics**: the main switch, the floating button, the Extras entry, the retry pop-up, the on-screen panel, and the switch for the chat you are in.
- **How it retries**: how many tries, how long it waits between them, when it pauses itself, and whether a retry adds a new reroll or redoes the reply.
- **When to count a reply as bad**: which kinds of bad reply start a retry, and the two waits for a reply that freezes or never arrives.

Four start closed. Press **▸** to open one. Nothing in them is needed to use the extension.

- **Refusal tuning**
- **Buttons it clicks**
- **Debug info**
- **Import / export**

The longer sections have headings inside them, each with a line saying what its rows are for:

- **How it retries**: **When it gives up**, then **How long it waits between tries**.
- **When to count a reply as bad**: starts at **Errors** and ends at **Replies that freeze**.
- **Refusal tuning**: **What counts as one**, **Wording you supply**, **How far it looks**, then the note rows.

Every section header is a button, so a closed section also opens with Enter or Space on a keyboard.

Switching Auto Retry off does not change anything in the panel. Every setting stays editable, so you can set it up while it is off. A line at the top says it is off and that your settings are saved.

## The **?** on each option

Press **?** to read what an option does. The description opens in a small box just below the option, over the panel, so nothing moves.

- **What the notes say** is the one exception. Its description opens above, because the row below it is long.
- A long description scrolls inside its box.
- Only one shows at a time.
- To close it, tap it, tap the **?** again, tap somewhere else, scroll the panel, or press Esc.

## Rows that come and go

A setting that does nothing until another one is switched on stays hidden until then.

- Turning off **It looks like an accidental refusal** hides the whole **Refusal tuning** section, because nothing in it does anything while that is off.
- Turning on **Send a note with a refusal retry** shows the note rows.
- The switch that hides the rows never moves itself.

The search box still finds a hidden setting by name. A row found that way says which switch it is waiting on.

`refusalThinkTags` is the one to know about. It hides with the rest of **Refusal tuning**, but the blank-reply and short-reply checks still read it to find where the reply starts. Search for it by name to reach it while the section is hidden.

## Saving

Only **Save** keeps your changes. Closing with the X, or tapping outside the panel, throws them away, so you can try things freely.

- Saved settings follow your Lumiverse account to other browsers and devices.
- A tab left open catches up. Opening the settings, or coming back to the tab after more than 30 seconds, loads your settings and presets from your account first. So a tab left open on your phone does not save older settings over newer ones.
- Saves reach your account in the order you make them, so an older save never ends up as the stored copy.
- If the settings open before the extension's server side has started, they ask your account again once that side is ready.
- They apply from the next reply.
- Long text boxes, like your own refusal phrases, have an **Expand** button that opens a bigger editor.

## Turning it off everywhere

**Basics** has two ways to switch Auto Retry on and off without opening the settings. You can use either or both.

### Floating on/off button

A small button over the chat. One tap switches Auto Retry on or off.

- Drag it anywhere. It moves to the nearest edge and stays where you leave it.
- You can set its size. The button changes size as you type, so you can see it before you save. A button against an edge moves inward if it needs to, to fit.
- Hold it, or right-click on a computer, to open its menu. A ring fills around the button while you hold. Let go early and nothing opens.
- A tap makes the button dip a little, so you can tell your tap was felt.
- If your device is set to reduce motion, no ring is drawn. Holding still opens the menu.

Its menu has, in order: **Auto Retry settings**, **Open the Auto Retry panel** (only when the panel lives in the sidebar), and **Hide this button**. Lumiverse draws the menu, so it uses your theme.

Auto Refine's floating button works the same way.

### On/off button in the Extras menu

A button in the chat input's Extras menu, next to the settings button. It takes up no room on screen.

- Its label says whether Auto Retry is on or off.
- In a chat you switched off, the label says so.
- Tapping it always switches Auto Retry on or off everywhere.
- It is hidden while the floating button is on, because the two do the same job. Turn the floating button off and this one comes back.

## Turning it off in one chat

Use this for a scene where the model is meant to refuse, or a chat you are using to test something.

1. Open the settings.
2. Under **Basics**, find the **This chat** row.
3. Press **Turn off here**.

That chat is left alone, and every other chat carries on. The button changes to **Turn on here**. This is the only place the switch is.

Everything that shows whether Auto Retry is running follows the chat you are in: this button, the line under the on-screen panel's tabs, the floating button, and the Extras entry. None of them says "on" in a chat you switched off.

While you are in a chat you switched off, a line at the top of the settings says so. This is there because a chat switched off weeks ago looks the same as the extension not working.

**Outside a chat**, on the home screen or the character browser, the button is greyed out and the row says **No chat is open**.

How it knows which chat you are in:

- It checks the address in your browser. While you are in a chat, the address contains the chat's id.
- Lumiverse can also be asked, but it answers with your most recent chat. On the home screen, that is the chat you just left. So the address wins.
- If your Lumiverse's addresses do not contain the chat id, it waits to be told instead.

**If the button is greyed out inside a chat**, it has not been told which chat this is yet. With the `chats` permission it asks and this clears by itself. Without it, sending a message, a reply arriving, or switching chats and back will fix it. You will usually only see this right after updating the extension without leaving the chat.

**Where it is kept:** the list of chats you switched off is saved in this browser, so it survives a reload. It is only a list of chat ids. It stays in this browser, and exports leave it out.

**Temporary chats** can be switched off the same way, but it is not saved, because a temporary chat is gone when you leave it. The row says so, and the switch lasts as long as the chat.

## Resetting

**Reset…** at the bottom of the panel opens a list. Nothing is reset until you choose.

1. Tick the parts you want back at their defaults. Anything unticked is not touched. The parts have the same names as in import and export.
2. Each line says how many of its settings you have changed. A part with nothing changed cannot be ticked. **Tick every setting** ticks every part that has something to reset.
3. Press **Reset ticked**. It shows what you picked and how many settings are in each part.
4. Press **Yes, reset** to go ahead, or **Go back** to change your ticks. Esc or a click outside closes it with nothing changed.

A reset fills in the defaults but does not save them, the same as an import. Press **Save** to keep them, or close the panel to undo the reset.

**Delete saved presets** sits apart from the rest, and it is the one thing closing the panel does not undo. Presets are stored separately from settings, so they are deleted straight away. **Tick every setting** never ticks it.

A reset never touches your chats, your replies or your characters.

## The on-screen panel

Turn on **Show the on-screen panel** under **Basics**. **Where that panel goes** picks where it appears. It is the same panel either way.

- **Floating over the chat**: a small box. Drag the header to move it, and drag the bottom-right corner to resize it. This works with a mouse or a finger. Its place, size and open tab are remembered in this browser, so an update does not move it. If it was placed on a bigger screen, it is moved back onto the screen it opens on.
- **In the sidebar drawer**: a tab in Lumiverse's own drawer. It cannot cover the reply you are reading. While a retry is running, the tab shows a dot. If your Lumiverse has no drawer for extensions, you get the floating box, and the Log says why.

Changing this moves the panel straight away. Closing the settings without saving puts it back.

**To open the panel from the sidebar:** hold the floating button and choose **Open the Auto Retry panel**. With the floating button off, the same button is in the **Extras** menu. On a computer, **Ctrl+K** then typing `Auto Retry` also works. The floating panel does not need opening, because it is already on screen.

The panel has four tabs. Switch tabs by tapping, or with the left and right arrow keys. **Copy** and **Clear** act on the tab you are on.

### The line under the tabs

This line says what is happening right now, with a dot beside it. It is the same on every tab.

The dot:

- **dim**: Auto Retry is off or paused
- **lit and still**: on, with nothing to do
- **pulsing**: something is happening. If your device is set to reduce motion, it glows without pulsing.

The line reports:

- a retry that is waiting, with a countdown (`47s`, `5m 03s`, `1h 05m 03s`)
- what that retry is for, and which try it is
- a reply arriving, and roughly how much has come in
- the model thinking
- that it has paused itself after repeated failures

A retry in a chat you have since left is still shown, marked as being in another chat. The line and the retry pop-up read from the same place, so they always agree.

### Log

What the extension is doing as it happens: replies starting, retries and why, replies that came back fine, and notes being sent. It keeps the last twenty lines.

### Stats

What it has done since the page loaded:

- replies that came back fine
- retries
- messages it gave up on
- what it retried for, with a bar for each reason

**Watching for** shows how long it has been counting. It also shows how often a reply needed a retry, and says when it has paused itself after repeated failures. **Clear** starts the counting again.

### Prompt

The whole prompt that went to the model, after your settings, world info, persona and every extension have added to it. This is different from Lumiverse's **Prompt Breakdown**, which shows what your chat is built from.

- Every message is listed in order, with its role, its size, and whether it came from your chat or was added around it. Tap one to read it. Nothing is cut: every message and every character is there.
- **Your refusal notes are marked** in your accent colour and opened for you. A line at the top says how many went and where they were added.
- The button under the message count switches between **Rendered** and **Raw**. Rendered is the readable list. Raw is the prompt as the model received it, role and content. **Copy** copies whichever you are looking at. Your choice is remembered.

**When it captures:** nothing is captured until you open this tab. After that it keeps up while the panel is open, even on another tab. Close the panel and it stops. There is no separate switch for this.

**Permission:** reading the prompt needs the `interceptor` permission, which an admin must approve. Without it the tab stays empty and says so after your next reply. If you opened the tab partway through a reply, it asks you to send another, because that prompt was built before the tab was open.

**Two tabs open:** if you have two chats open in two browser tabs, both receive every prompt. The tab only shows the prompt for the chat you are in, and says so when one belongs to another chat.

The prompt stays on your device. It is not sent anywhere or written to disk, and it goes when you close the tab.

### What a retry costs

This line sits under the message count on the **Prompt** tab once you fill in the two prices under **Basics**. **Input** and **output** are the words your provider's price list uses: input is what you send, output is what the model writes back.

**What to type:** price lists write prices as `$5.00/M` or `$0.075/M`, meaning per million tokens. Type the number: `5` or `0.075`. You can also paste the whole thing, and the number is taken out of it.

How it works it out:

- A retry pays for the prompt and the new reply, so both are counted.
- The prompt is the one on this tab.
- The new reply has not been written yet, so it uses the size of the last reply in this chat. The line says so.
- A chat with no reply yet is priced on the prompt only, and says so.
- Under it is what the retries so far this session come to.

**Read it as the most it could cost, not your bill.** It prices every token at the full rate, and it leaves out anything your provider adds around your prompt.

- Tokens are counted by Lumiverse's own tokeniser for the model. Where it has none, the line says **roughly**.
- Both prices start at 0, and while both are 0 the line is hidden.
- Prices are in your provider's own currency. Nothing is converted.

### Replaced

The reply the last retry in this chat threw away, with the reason and how long ago. Use it when a retry was a mistake: read the old reply, or press **Copy** to take it.

A retry that adds a reroll already leaves the old reply to swipe back to. But rerolls can be deleted, by you or by another extension. This tab keeps its own copy.

- It keeps one reply per chat, for the last eight chats.
- **Clear** drops the one you are looking at.
- Turn it off with **Keep the reply a retry replaced**, under **How it retries**.

## Every option and its default

These are the defaults for a new install. They are in the `CONFIG` block at the top of `src/frontend.ts` and `dist/frontend.js`. `dist/frontend.js` is the file Lumiverse loads.

| Option | Default | Meaning |
| --- | --- | --- |
| enabled | true | Master switch. |
| showFloatingToggle | false | Put a small draggable on/off button over the chat. |
| floatingToggleSize | 44 | How wide that floating button is, in pixels (28-96). The button on the chat resizes as you type, so the size can be seen before it is saved, and closing the settings without saving puts it back. Shown only while `showFloatingToggle` is on. |
| showExtrasToggle | false | Add an on/off button to the chat input's Extras menu. Its label says whether Auto Retry is on or off. Hidden while the floating button is on. |
| maxRetries | 4 | Hard cap per message. Nothing retries past this. The lowest is 1: to stop it retrying, switch it off rather than setting this to 0. |
| pauseWhenFailing | true | Pause auto-retry after several whole runs give up in a row. Cleared by the next reply that comes back fine. |
| breakerRuns | 3 | How many failed runs in a row trigger the pause. A run is one message that used up all its tries. Shown only while `pauseWhenFailing` is on. |
| breakerPauseMins | 5 | How long the pause lasts, in minutes. A reply that comes back fine ends it early. Shown only while `pauseWhenFailing` is on. |
| retryDelayMs | 2000 | Wait before the first retry, in milliseconds. |
| backoffFactor | 2 | Each wait is this many times longer than the last. |
| maxDelayMs | 60000 | Longest it will ever wait. |
| jitter | true | Nudges each wait randomly so retries do not all arrive at once. |
| rateLimitDelayMs | 15000 | Floor wait when the server says it is busy. Most shared tiers meter per minute, so a shorter wait usually spends a try hitting the same limit. Where the server says how long to wait, that figure is used instead and it is not held under `maxDelayMs`: it is the only number here that is not a guess. An hour is the ceiling. |
| retryByNewReroll | true | On: a retry clicks the next / swipe button, adding a new reroll and keeping the existing ones, so a reply it was wrong to retry can be swiped back to. Off: a retry redoes the reply in place via the regenerate button, which on some builds clears the other rerolls. Applies to every retry reason. The other button is the fallback. |
| keepReplaced | true | Keep the last reply a retry threw away in this chat, so it can be read back or copied from the Replaced tab of the on-screen panel. Held in the tab's memory only: never written down, never sent anywhere, gone when the tab closes. |
| stuckTimeoutMs | 240000 | Started, then nothing arrived and it never finished, within this. 0 disables. |
| idleTimeoutMs | 90000 | Tokens flowed then stopped for this long. 0 disables. |
| retryOnError | true | Retry provider errors. |
| ignoreHardErrors | true | Skip permanent failures like missing models or invalid API keys. |
| hardErrorPhrases | (blank) | Your own wording for an error that will not fix itself, one per line, counted alongside the built-in list. Shown only while `ignoreHardErrors` is on. |
| retryOnEmpty | true | Retry empty replies and mid-reasoning cutoffs. |
| retryOnSpam | true | Retry when the reply is one character over and over, such as `!!!!!!!!`. |
| retryOnSpamThinking | true | Retry when the thinking is one character over and over, even when the reply after it looks fine. |
| retryOnTruncated | true | Retry a reply that ends mid-sentence. |
| retryOnNoPunct | true | Retry a reply that stops on a word with nothing after it. Punctuation in any script counts as an ending, and so does an emoji. |
| retryOnShort | false | Retry short replies. Off unless you mean it. |
| minChars | 24 | Short threshold, used when retryOnShort is on. Counts the words you read only: any reasoning block is left out, and so are HTML tags, so a line wrapped in markup is measured by what it says. Shown only while `retryOnShort` is on. |
| retryOnRefusal | true | Retry an accidental out-of-character refusal. |
| refusalUseBuiltins | true | Use the built-in English refusal lists. Off = only your own phrases. |
| refusalCatchDisengage | true | Also catch the model breaking off ("I'll stop here", "I won't continue this conversation"). Only counted when it is how the reply ends, never inside quotation marks, and never behind a dialogue tag. Shown only while `refusalUseBuiltins` is on. |
| refusalCatchCrisis | false | Also catch the model leaving the scene to offer real-world support and crisis resources. Two separate parts of the reply have to point that way before it counts, one of them the model addressing you rather than your character, and a line inside quotation marks never counts. The only check `refusalMaxChars` does not apply to. Ticking it opens a warning that has to be answered before it goes on. Shown only while `refusalUseBuiltins` is on, and read [Safety](safety.md) first. |
| refusalIgnoreQuoted | true | A built-in match inside quotation marks is a character speaking, so it is not counted. Your own phrases are counted either way. |
| refusalExtraPhrases | (blank) | Phrases that also count as a refusal, one per line. |
| refusalPhraseSubs | (blank) | Reword the built-in phrases with "old => new" rules, one per line. Shown only while `refusalUseBuiltins` is on. |
| refusalIgnorePhrases | (blank) | Whitelist, one per line; a reply containing any is never a refusal. |
| refusalMaxChars | 2000 | Longest reply still treated as a possible refusal. 0 = no limit. |
| refusalStripThinking | true | Only check the final reply, stripping known reasoning tags first. Off checks the whole raw output. |
| refusalThinkTags | (blank) | Extra reasoning tag names, one per line, for unusual thinking wrappers. |
| refusalNote | false | Send a note with a refusal retry, and only a refusal retry. Needs the `interceptor` permission. |
| refusalNotes | one empty note | The notes themselves. Each carries its own role (system, user or assistant) and its own first try, so notes can be set to escalate. Up to ten. Whichever have come due are sent together, in order. Empty ones are skipped, and nothing is sent while they all are. Shown only while `refusalNote` is on. |
| refusalNotePlacement | after | For the whole list, not one note. Where the block of due notes goes: after the last message, before it, at the very end (past anything the build appends behind the conversation), or at the very start. Shown only while `refusalNote` is on. |
| refusalNoteStrictType | false | For the whole list, not one note: it decides whether any of them are sent at all. Only attach them when Lumiverse reports the generation as a regenerate or a swipe. Most builds report every generation as "normal", and on those this stops the note going out at all, which is why it is off. Shown only while `refusalNote` is on. |
| regenerateSelector | (see file) | Your regenerate button. See [Buttons it clicks](buttons.md). |
| swipeNextSelector | (see file) | Your next / swipe button, which a retry presses first while `retryByNewReroll` is on. |
| confirmButtonsCustom | false | Lets you add your own dialog button words. Off: only the built-in list is used, and the box is hidden. |
| confirmButtonLabels | (blank) | Extra dialog button labels it may press when a dialog appears after a retry, one per line. Tried before the built-in list, which is used as well. Shown and read only while `confirmButtonsCustom` is on. |
| stopSelector | (see file) | Your stop button, used to stop a reply that has frozen. |
| toast | true | Show the little retry pop-up with its Cancel button. It counts the wait down in real time and names what the retry is for and which try it is. |
| liveLog | false | Show the on-screen panel. Four tabs: Log for what it is doing as it happens, Prompt for what went to the model, Stats for what it keeps retrying for, and Replaced for the last reply a retry threw away. |
| panelHome | float | Where that panel goes. `float` is a small box over the chat you can move and resize, and where you leave it is remembered. `drawer` puts it in Lumiverse's own side panel, which never covers the reply you are reading. A Lumiverse with no side panel for extensions gets the box, and the Log says so. Shown only while `liveLog` is on. |
| costIn | 0 | Your provider's input price per million tokens, in its own currency. The panel's Prompt tab uses it to say what retrying costs. 0 leaves the line off. |
| costOut | 0 | The output price from the same list, for the reply a retry produces. Both at 0 leaves the line off. |

### The two waits for a frozen reply

`stuckTimeoutMs` and `idleTimeoutMs` are long on purpose, so a slow model has time to answer.

- A wait that runs out too early throws away a reply that was still coming. The retry then goes to the same slow model, so it runs out again.
- If your provider is fast and you want quicker retries, lower them.

`idleTimeoutMs` needs streaming on. It watches for text that stopped arriving, and with streaming off no text arrives until the end. A reply that hangs is then caught by `stuckTimeoutMs` instead. Every other check reads the finished reply, so it works with streaming on or off.

### After an update

Defaults only apply to a new install. Settings already saved to your account keep their values. To get the newer defaults, open **Reset…** and tick the part you want.

---

[Back to the README](../README.md)
