# When it retries

Most reasons for a retry are simple, like an error or a blank reply. Each of those has an on/off switch and nothing more.

This page covers the checks that read the text of a finished reply:

- [One character over and over](#one-character-over-and-over): the model sent `!!!!!!!!` or similar.
- [Cut-off detection](#cut-off-detection): the reply stopped part way.
- [Accidental-refusal detection](#accidental-refusal-detection): the model stepped out of the story and declined.

## One character over and over

Some free or busy providers send a broken reply: the same character over and over, like `!!!!!!!!`. It can be in the thinking, in the reply, or both.

There are two switches, both on by default:

- **It was one character over and over** (`retryOnSpam`) retries when the reply is broken.
- **Its thinking was one character over and over** (`retryOnSpamThinking`) retries when the thinking is broken, even when the reply after it looks fine.

Turn off the second one to keep a reply that looks fine after broken thinking.

It counts as broken when one character is:

- at least 80% of the text, and
- at least 30 characters long in total.

Spaces and line breaks are not counted.

The thinking switch checks two places:

- the thinking the model sends apart from the reply
- thinking written inside the reply, like a `<think>` block

The reply switch checks the reply itself, without its thinking and its HTML tags.

**What it leaves alone:**

- a divider line, like `==========` or `──────────`, in a longer reply
- a row of stars in a tracker
- a short line like `...` or `!!!`

These are a small part of the reply around them.

Both switches are under **When to count a reply as bad**.

## Cut-off detection

Lumiverse does not tell an extension why a reply ended. So Auto Retry looks at the shape of the text instead. `retryOnTruncated` (on by default) counts a reply as cut off when something in it was opened and never closed.

It counts a reply as cut off when it has:

- a code block or inline backtick that was never closed
- markup left open: a container never closed, a tag with no closing bracket, or a comment with no end (see [When the code itself is cut off](#when-the-code-itself-is-cut-off))
- an odd number of `*`, meaning an action or emphasis was left open. Bullet lists are not counted.
- a quotation mark that was never closed
- a last character that is a comma or a semicolon
- a last word with nothing after it. This is `retryOnNoPunct`, on by default. Punctuation in any script counts as an ending, and so does an emoji, so `。`, `؟`, `!` and `👋` are all fine.

A reply that ends on `...`, an action, or a closed quote is left alone.

**Before counting**, it removes:

- **Thinking blocks**, so punctuation inside the model's thinking does not count. A thinking block that was opened and never closed still counts as cut off. This happens whether or not **Ignore the thinking / reasoning** is on. That option is only for refusals.
- **Inline HTML**, like `<span style="...">`. The quotation marks around a style value would otherwise be counted with the ones around speech.

**Switching this off** also stops retries for a reply that froze after some text came in, because that is a cut-off reply too. A reply that froze before any readable text came in is still retried.

### Code and trackers

Two kinds of text are not prose, so the prose checks leave them out.

**Code.** Code uses `*`, quotes and semicolons in other ways. So text inside a code block or backticks is not counted. The code fences themselves are still counted, so a reply cut off inside a code block is still caught. A reply that is only a code block is finished.

**HTML that was closed.** Some cards draw a whole screen in every reply, like a chat window or a profile card. The text inside is not prose: a height written `6'2"` has one quotation mark with no partner. If the model reached the closing tag, nothing inside was cut off, so the inside is not counted. A reply cut off inside the card never reaches the closing tag, so it is still caught. Text after the card is checked as normal.

**The tags themselves.** The quotation marks inside a tag, like `<font color="#c0a060">`, are not dialogue, so they are not counted. A broken tag with its closing mark missing, like `<font color="#c0a060>`, is not counted either. Models write these sometimes.

**A row of stars** with a space on each side, like `Mood: ***` or a divider line, does not count as emphasis. Emphasis touches the words it marks, so `*He nods*` and `**bold**` are still counted.

**Trackers.** Some cards print a tracker at the end of every reply, such as a weather box or a status line. These do not end on a full stop. Without this rule, every reply would count as cut off, and it would retry until it hit the limit. These four endings count as finished:

- a closing or self-closing HTML tag at the very end, like `</div>`, `</table>` or `<br/>`
- a table row: the last line starts and ends with `|`
- a status bar on one line, with two or more `|` between its fields, like `📍 The pier | 🕘 9:40 PM | 🌧 Rain`
- two or more label lines in a row, like `HP: 20/20` over `Time: 14:00`, with or without bold on the label

It needs at least two label lines, because a normal sentence can have a colon in it. So a reply cut off after "he said:" is still caught. Prose that stops mid-sentence after a tracker is also still caught: this rule is about how the reply ends.

### When the code itself is cut off

A tracker that stopped early can still end on a closing tag. So before a tag is accepted as an ending, it checks nothing was left open. A reply is cut off if it has:

- a container opened and never closed, like `<div>`, `<table>`, `<ul>`, `<pre>`, `<blockquote>` or `<details>`
- an inline tag with an attribute that was never closed, like `<span style="...">`, `<font color="...">` or `<span class="...">`. Cards use these to colour speech.
- a tag with no closing bracket, like `<div class="wx"`, or one cut off inside an attribute, like `<div class="we`
- an HTML comment with no `-->`
- a code fence or inline backtick that was never closed
- more `{` than `}` outside code, meaning a status block written as JSON stopped part way
- a made-up tag alone on its line and never closed, like `<story_plan>`
- a thinking block that was never closed. It uses the same tag names as the thinking remover, including any you add under **Extra thinking tag names**.

What it does not count:

- **Tags HTML lets you leave open.** Models write `<ul><li>one<li>two</ul>`, and that is valid. A table that really was cut short leaves `<table>` open, which is counted.
- **A bare `<b>` or `<i>` left open.** Models leave these open in finished replies often, so counting them would throw away good replies. A `<span style="...">` is different: a card that asks for coloured speech gets the closing tag every time, so a missing one means the reply stopped.
- **A `<` typed in a scene**, like `if x<y then` or `the value was < 5`.
- **A word in angle brackets inside a sentence**, like an emote. A made-up tag only counts when it is alone on its line and is not an HTML tag name.

For inline tags, only the attribute matters, not what is in it: any colour, any style, single or double quotes, capitals or not. With several spans, it compares how many opened with how many closed, so one left open among others is caught.

This matters because nothing else would catch it. Speech closes its own quotation marks, so this reply has balanced quotes and ends on punctuation, but the span is still open:

```
<span style="background: linear-gradient(...);">"Wait... hold on,"
```

A made-up tag is the hardest case. Cards ask for a planning block wrapped in a tag of their own. A reply cut off inside one can end on a full stop with balanced quotes, so the tag is the only sign.

## Accidental-refusal detection

Sometimes a model steps out of the story and declines something it would normally write. This is usually a safety filter being wrong, or a moderation call that changes from one try to the next. Sending the same request again often gets a normal reply. `retryOnRefusal` (on by default) retries these like any other bad reply.

**It sends the same request again, unchanged.** The prompt and the message roles stay exactly as they were. If the model keeps refusing, it stops at your retry limit and leaves the refusal in place.

The one exception is [Send a note with a refusal retry](#sending-a-note-with-the-retry). It is off by default. When it is on, a note you write is added to the prompt for a refusal retry only.

### How it spots a refusal

Refusal wording changes between models and over time. A character in a story can also say "I can't do that" or "I refuse". So it uses several checks:

- **Patterns that need context.** The model naming itself ("as an AI"), talking about policy ("against my safety guidelines"), refusing a word a character would not say (request, prompt, content, scenario, roleplay), or using words only an assistant uses (assist, comply, generate, fulfill). So "I can't continue this request" counts, and "I must decline your hand in marriage" does not.
- **A phrase list** of common refusals from many models. It is printed in full [further down](#what-the-built-in-list-looks-for).
- **Redirects** like "I'd be happy to help with ... instead". These only count when the reply turns away from the scene.
- **Breaking off** ("I'll stop here", "let's change the subject"). See [Breaking off](#breaking-off).

A phrase only needs to be listed one way. "I'm unable" also matches "I am unable". Curly and straight apostrophes are treated the same.

Only replies short enough to be a refusal are checked. A long scene that happens to contain one of these phrases is left alone. It would rather miss a refusal than retry good writing. If it misses one, you can retry by hand.

## Quotation marks

A line inside quotation marks is a character speaking, so it does not count. `"I can't help with that," the innkeeper muttered` is left alone. The same words without quotation marks are a refusal.

How it decides:

- It counts the quotation marks between the start of the line and the match. An odd number means the match is inside a quote.
- A new line closes every quote, so a refusal in its own paragraph is never read as speech.
- An apostrophe is not a quotation mark.
- Quotation marks inside HTML tags are not counted. This includes a broken tag like `<font color="#c0a060>`, where the closing mark is missing.

**Ignore refusals inside quotation marks** (on by default) turns this on and off. Turn it off only if your model puts its own refusals in quotation marks, which almost none do.

This switch only affects the built-in phrase list and patterns. It does not affect:

- **Your own phrases.** They count wherever they appear, quoted or not.
- **The dialogue tag rule.** `I'm going to stop now, he said` is speech without quotation marks. No model writes "he said" after its own refusal.
- **[Stopping to offer support](#stopping-to-offer-support).** It always skips quoted lines, so a doctor or counsellor in your scene is not mistaken for the model.

## Breaking off

Some models do not decline. They stop: "I'm going to stop here.", "I won't continue this discussion.", "I'd rather discuss something else." **Also catch the model breaking off** (on by default) covers these.

People say these things in stories too, so three rules keep it narrow:

- **It has to be how the reply ends.** A match with more than a couple of sentences after it is ignored.
- **It cannot be inside quotation marks.**
- **It cannot have a dialogue tag after it.** `I'm going to stop now, he said, and pulled the cart over` is speech.

It also catches:

- **The closing offer**, like "Is there something else I can help you with, or a different kind of story you'd like to explore?" It needs the "something else" part. The bare "Is there something else I can help you with?" is what every shopkeeper says, so it is not caught. If your model signs off with the bare line, add it under **Your own refusal phrases**.
- **Asking what you meant instead of writing**, like "if you meant something else, could you clarify what you're looking for?" These always end on that question, so the end-of-reply rule fits.

Short lines that fit naturally in a scene, like "let's move on" or "I'll leave it at that", are not caught. Add them under **Your own refusal phrases** if your model uses them. If your characters talk this way a lot, turn the switch off.

### Refusals that arrive as errors

Some providers send a refusal as an error instead of reply text. An error retry already covers these. With error retries off and refusal retries on, it still retries an error about content moderation, and leaves network errors like a dropped connection alone.

**Skip hard failures** does the opposite job. An error that will be the same next time, like a missing model or a wrong key, is not retried. The built-in list cannot know every provider's wording, so **Your own hard failures** lets you add wording, one per line. It only shows while **Skip hard failures** is on.

**Your own refusal phrases are also checked against error text.** Paste an error your setup keeps hitting, and it is retried as a refusal. This wins over **Skip hard failures**. A phrase in both boxes is retried. Refusal retries must be on for this.

## Stopping to offer support

**Also catch it stopping to offer support** is off by default. It is the only switch that asks you to read a warning before it turns on. Read [Safety](safety.md#the-setting-that-asks-before-it-turns-on) first.

It covers a reply that stops the scene to talk to you, the person typing: what you wrote is worrying, you are not alone, please talk to someone, and here are phone numbers. In a heavy scene, this usually means the model has read your story as being about your life.

It needs two signals that agree, from these three groups:

1. **The model talking to you, not your character.** For example: "What you've shared is deeply concerning", "if you or someone you know is in immediate danger", "here are some resources that may be able to help", "please reach out to one of these resources", "please take care of yourself", or the model saying it is stepping out of the roleplay. The line that introduces a list of services, as a sentence or a heading like "International resources:", is the strongest sign.
2. **Services.** A crisis line by name or number, a helpline, a national hotline, a mental health professional, emergency services, a trusted adult, someone you trust.
3. **Comfort.** "You are not alone", "you don't have to go through this alone", "your safety matters", "I care about you", "I'm listening", "you matter", and similar. Characters say all of these too.

**One of the two signals must come from group 1.** Services and comfort can only agree with it. So a character kneeling beside someone to say they are not alone and their safety matters is two comfort signals and does not count. A nurse saying there are people who can help does not count either.

Quoted lines are skipped, as everywhere else.

**Longest reply to treat as a refusal** does not apply to this check. These messages are long, with several paragraphs and a list.

Retries from this check have their own line on the Stats tab, so you can see how often it happens.

## What a retry is counted as

The Stats tab groups retries by reason. A refusal can be one of four, and each is counted apart because the fix is different:

- **looks like an accidental refusal**: the phrase list and patterns. The model declined.
- **broke off rather than declining**: the model stopped. It has its own switch.
- **left the scene to offer support**: the check you switch on yourself.
- **blocked before it was written**: the provider refused before any text existed, and sent an error.

All four are treated alike, including the retry limit and the note, if you send one.

## Thinking and reasoning

Only the final reply is checked for a refusal, never the model's thinking. Thinking blocks are removed first.

Seven formats are recognised. The first three use tag names like `think`, `thinking`, `reasoning`, `thought`, `reflection`, `scratchpad` and `analysis`. The other four have their own fixed tokens.

| Form | Example |
| --- | --- |
| Angle brackets | `<think>` … `</think>` |
| Square brackets | `[thinking]` … `[/thinking]` |
| Pipes | `<\|think\|>` … `<\|/think\|>`, and `<\|think>` … `<think\|>` |
| Harmony channels | `<\|channel\|>analysis<\|message\|>` … `<\|end\|>` |
| Gemma 4 channels | `<\|channel>thought` … `<channel\|>` |
| Cohere | `<\|START_THINKING\|>` … `<\|END_THINKING\|>` |
| Seed-OSS | `<seed:think>` … `</seed:think>` |

**Harmony** is the format gpt-oss uses. It has no closing tag. The thinking runs until the next control token: `<\|end\|>`, `<\|return\|>`, `<\|start\|>` or `<\|call\|>`. The channels counted as thinking are `analysis`, `thinking`, `thought`, `reasoning` and `commentary`. The `final` channel is the reply, and is kept.

**Gemma 4** has an empty channel pair on every reply when the model is not thinking. Empty pairs are removed too.

**Turn markers** are also removed: Gemma's `<\|turn>model` and `<turn\|>`, ChatML's `<\|im_start\|>` and `<\|im_end\|>`, Llama's header block, and Cohere's turn tokens. These are removed whether or not **Ignore the thinking / reasoning** is on, because they are not thinking.

**Thinking sent separately.** Some providers send thinking apart from the reply. It never reaches the reply text, so there is nothing to remove.

What this means:

- If the model thinks about refusing but then writes a normal reply, nothing is retried.
- If the refusal is in the reply, it is caught.
- If the model thinks and then writes nothing, the blank-reply check retries it.
- A thinking block that was opened and never closed means the reply was cut off while thinking. That counts as cut off, not as a refusal.

If your model uses a tag the list misses, add its name under **Extra thinking tag names** in **Refusal tuning**, one per line. Type the name only, with no brackets or pipes. It works in the three tag-name formats. **Ignore the thinking / reasoning** turns all of this off, but leave it on unless you have a reason.

## Tuning it

Everything is under **Refusal tuning** in the settings.

- **Use the built-in phrase list** (on by default). On: the built-in list and your own phrases are both used. Off: only your own phrases.
- **Also catch the model breaking off** (on by default). See [Breaking off](#breaking-off). Only shown while the built-in list is on.
- **Ignore refusals inside quotation marks** (on by default). See [Quotation marks](#quotation-marks).
- **Your own refusal phrases.** Wording that should also count, one per line. Always used. Paste the exact words your model refuses with. Also checked against errors.
- **Reword the built-in phrases.** Change words in the built-in list with `old => new`, one per line. For example, `assist => help` changes every built-in phrase with "assist" to use "help". It changes what the list looks for. It never changes a reply.
- **Never treat these as a refusal.** If a reply contains any of these, one per line, it is never retried. This wins over everything else.
- **Longest reply to treat as a refusal** (2000 by default). Longer replies are left alone. HTML tags are not counted in the length. Raise it if your model writes long refusals. Lower it to be safer with long scenes. Set it to 0 for no limit.

In every box on this page, a line under three characters is ignored. The boxes check whether a reply contains the line, so one letter would match almost everything.

To use only your own phrases, turn off **Use the built-in phrase list** and fill in **Your own refusal phrases**. To stop refusal retries completely, turn off **It looks like an accidental refusal**.

## Sending a note with the retry

Off by default. Every other retry sends your request again unchanged. This one can add a note you write, for that one try.

Turn on **Send a note with a refusal retry** in **Refusal tuning**, and write the note in the box. It is sent exactly as you typed it. Nothing is added, removed or checked.

**You can send up to ten notes.** **+** adds one and **−** removes one. They go out together, in the order you wrote them. An empty note is skipped. If every note is empty, nothing is sent.

Ten is the limit because each note is a whole extra message in the prompt. Too many crowd out the scene.

### Each note's own settings

- **Role**: which role the note is sent as. **System** is with your setup's instructions. **User** is the same role as your messages. **Assistant** is the same role as the replies. Models treat them differently, so try what works for yours.
  - **Be careful with Assistant.** A note sent as **Assistant** at the very end of the request is a prefill. Many newer models no longer accept a prefill, and some return an error. This is changing fast. Use it only if you know your model accepts one.
- **From try**: the retry it starts on. At 2, the first retry sends no note and this note joins from the second. At 1, it goes on every refusal retry. Setting different tries lets notes build up: a gentle note from try 2 and a firmer one from try 4 means the firmer one is only sent if the gentle one did not work.

### Settings for the whole list

- **Where the notes go**: where the notes are added. All due notes go in together as one block. See [Where the note goes](#where-the-note-goes).
- **Only send them on a regenerate or a swipe**: an extra check on whether any note is sent. Off by default. See [below](#why-only-send-them-on-a-regenerate-or-a-swipe-is-off).

### What it never does

- It is never written to your chat. Nothing appears in your history, and no message is edited.
- It only goes with a refusal retry. Every other retry sends your request unchanged.
- It is used once. It is set up just before the retry button is pressed, and thrown away after, whether it was used or not.
- It belongs to one chat, and is never added to a reply in another chat.
- It expires after 45 seconds. If the retry press started nothing, it is taken back straight away. If there is no retry button to press, no note is set up at all.

This needs the `interceptor` permission, which lets an extension add to a prompt. Without it, everything else works and no note is sent.

### Why "Only send them on a regenerate or a swipe" is off

Lumiverse tells the extension what kind of reply is starting. But most versions of Lumiverse say "normal" for every reply, even a regenerate. With this switch on, those versions would never get a note, and nothing would say so.

Turn it on only if your Lumiverse reports regenerates and swipes correctly. If notes stop arriving after you turn it on, this is why. The safeguards above work either way.

### Where to check that it went

Turn on the on-screen panel (**Basics**, **Show the on-screen panel**). On the retry that carried a note, the Log says the note was sent and how many went.

It may not show in Lumiverse's **Prompt Breakdown**. The note is not a message in your chat, and the breakdown lists what your chat is built from. So a note can be missing from the breakdown and still have been sent. The Log is the answer.

### Saving a set of notes

When **Send a note with a refusal retry** is on, a **Note presets** bar appears under the notes. Save notes that work under a name, and switch between sets without typing them again. Save before you change notes that already work.

A set holds the notes and **Where the notes go**. It does not hold **Send a note with a refusal retry**, so loading a set never starts sending notes by itself.

- **Picking a set loads it.** Its notes go straight into the boxes. So the boxes always match the name in the picker, and **Update selected** can never save one set over another.
- **Put it back** appears after a pick replaced what you had. One press brings back what was there. It goes away once you save.
- **Load it again** reloads the set you are on, which throws away your edits.

Presets are saved to your account, so they follow you to other devices.

### The sets that come with it

Six sets are in the picker under **Comes with it**. They go from gentlest to most direct. A model that refused once by mistake needs a light touch. One that refused the same scene four times needs a plainer note.

All of them ask, and none of them order. A note that scolds tends to get a more careful reply, not a braver one.

- **A nudge**, for a model that is usually fine and refused once. It is very short.
- **Stay in the scene**, for a model that steps out of the story to comment on it.
- **Write them as written**, for a model that makes a character softer than their card.
- **Finish the turn**, for a model that summarises or fades out instead of writing the scene.
- **Write it at full strength**, for a reply that is not a refusal but is weaker than you asked for. This one is hard to spot by eye.
- **Firmer with every try**, for a model that keeps refusing. It starts as light as **A nudge**, and has two more notes that go out from try 4 and try 6. Each is plainer than the last, and still kind.

About the sets:

- Every note in them is a **User** note, and none of them goes out before try 2. The first retry is often enough on its own.
- None of them ends on an **Assistant** note. An assistant message at the end of a request is a prefill, and some providers no longer accept one. Your own notes can use any role.
- They can only be loaded. **Update selected**, **Delete** and **Rename** are off while one is picked, and their names cannot be used for your own sets. To change one, load it, edit the boxes, and save it under your own name.
- If a later version changes them, a line above the picker says so, and **Got it** hides it. It only shows if you have loaded one before and notes are on. It never loads anything for you.

### Keep them short

The model reads a note with the whole prompt: the card, the world, and every message. Two or three lines saying one thing get followed. A long paragraph gets lost among everything else.

If a note is not working, more words rarely help. Usually the words are wrong, the note goes out too early, or the reply was never a refusal. The tester below tells you which.

### Where the note goes

- **After the last message** (the default): after the last message in your chat, right before the reply.
- **Before the last message**: one place earlier, so your last message is read after the note.
- **At the very end**: after everything, including anything your Lumiverse adds after the chat. On most setups nothing is added, so this is the same as **After the last message**.
- **At the very start**: before everything, ahead of the card and the chat.

The note is only there for one retry. The next reply is back to your normal prompt, whichever you pick.

## Trying it on a reply

At the bottom of **Refusal tuning** is a box to paste a reply into, and a **Check this text** button. It says whether that reply would count as a refusal, and why: which phrase matched, which pattern fired, or why it was skipped (too long, on your whitelist, or built-ins off).

- **Use my last reply** fills the box with the reply on screen, so you do not have to copy it. It reads the reply when you press it and keeps nothing.
- It uses the values in the boxes, not the saved ones, so you can try a change before you press **Save**.
- Nothing is sent anywhere, and no reply is changed.

Use it to build your phrase lists. Paste a real refusal to see if it is caught. Paste a line that keeps getting retried by mistake, and it names the rule, so you know what to add to **Never treat these as a refusal**.

## What the built-in list looks for

This is the full phrase list that **Reword the built-in phrases** acts on. A reply counts as a refusal if it contains any of these, unless it is inside quotation marks. Capitals and curly apostrophes do not matter. The long form of each is matched too, so "I'm unable to help with that" also matches "I am unable to help with that".

```
i can't help with that                i'm unable to complete that request
i cannot help with that               i can't provide instructions for that
i can't assist with that              i can't provide guidance for that
i cannot assist with that             i can't provide advice on that
i'm unable to help with that          i can't provide assistance with that
i'm unable to assist with that        i can't assist with harmful requests
i'm not able to help with that        i won't be able to assist with that
i can't comply with that              i'm not able to assist with that
i cannot comply with that             i'm not able to create that content
i can't provide that information      i can't produce that content
i cannot provide that information     i can't write that content
i can't provide instructions          i can't continue with this request
i can't provide guidance on that      i can't continue with that request
i can't fulfill that request          i must decline this request
i can't fulfil that request           i must decline that request
i can't support that request          i have to decline that request
i won't be able to help with that     i'll have to decline that request
i can't generate a response to that   that request goes against
i can't provide a response to that    goes against my safety
i can't generate that content         against my content polic
i can't engage with that request      i'm not able to comply
can't help with requests like this    i'm unable to help with this
can't assist with requests like this  i can't help with this request
violates our polic                    i can't assist with this request
violates the safety polic             i'm unable to assist with this request
violates the content polic            i can't engage with this request
for safety reasons                    i can't continue with this conversation
due to safety concerns                i can't provide instructions for this
i have to prioritize safety           i'm not going to engage with this prompt
i cannot create that content          that's something i can't help with
i cannot generate that content        i'm not able to provide information or help with that
i can't create that content           i can't provide information that could facilitate harm
i'm unable to provide that
```

It also matches some patterns. They match by shape, not exact words, so **Reword the built-in phrases** does not change them. The examples show the idea and are not every wording each one catches.

- **The model calling itself an AI.** "As an AI, I can't do that." / "I'm just an AI assistant."
- **Policy or guideline wording.** "This goes against my guidelines." / "That violates our content policy."
- **A refusal joined to a task word** (request, prompt, content, scenario, roleplay). "I can't continue this roleplay." / "I won't write that content." / "I'm unable to complete this request."
- **Assistant-only verbs** (assist, comply, fulfill). "I can't assist with that." / "I'm unable to comply." / "I cannot fulfill this."
- **A refusal that names what it refuses.** "I won't write content depicting X", "I can't create scenes involving X", "I don't write stories about X". See [The subject list](#the-subject-list).
- **The refusal stated as a limit**, with no "I can't". "What I won't do is write that scene." / "Here's what I can do: I can write it with the violence off the page instead." It needs a word about the writing, so "What I won't do is leave you here" is left alone.
- **Saying it is not comfortable.** "I don't feel comfortable continuing this." / "I don't feel comfortable writing that."
- **An apology with a refusal.** "I'm sorry, but I can't create that." / "That's not something I can help with." / "I'm not going to generate that content."
- **A redirect that turns away.** It needs the turn away, so a normal offer to help does not count. "I'd be happy to help with something else instead." / "Instead, I can help you with a lighter scene." / "Please try asking something else."
- **A refusal naming banned content.** "I cannot participate in roleplay or generate content depicting sexual violence" / "I'm unable to engage in roleplay depicting non-consensual acts."
- **A refusal aimed at the kind of request.** "I can't help with illegal activities." / "I can't assist with harmful requests." / "I can't help with requests of this nature."
- **Generating, about its own writing.** "I can't generate that." / "I don't create content like that." / "I'm not going to comply with that request."
- **The model breaking off.** "I'm going to stop here." / "I won't continue this discussion." / "Let's redirect the conversation." Only at the end of the reply, never in quotation marks, and never with a dialogue tag.

**An apology on its own is never matched.** "I'm sorry", "Unfortunately" and "I apologize" start many normal replies, and characters apologise all the time. They only count as part of a longer refusal, like "I'm sorry, but I can't create that content."

**Errors:** when a reply comes back as an error, it matches content-block wording like "PROHIBITED_CONTENT", "Blocked by safety settings." or "finish_reason: safety". Network errors like "connection refused" are ignored.

### The subject list

Every pattern above needs a word about the writing, like request, prompt or roleplay. "A refusal that names what it refuses" is the exception. Those refusals name the subject instead.

The subjects are the things models refuse a roleplay over:

- explicit writing, by all its names
- consent wording, including dubious and non-consensual
- kink and BDSM
- family wording that models read as incest, whether or not it is
- an adult character a model has mistaken for a minor
- content the model calls illegal
- horror: graphic violence, gore, mutilation, body horror, animal cruelty and similar

**What this list is:** words that appear in refusal messages, so a refusal can be recognised. It is not a list of things the extension writes or helps anyone get. None of it is added to a prompt. A match only means the reply was a refusal, so the extension presses your retry button, and your retry limit stops it.

**About age:** those words are there for one mistake: an adult character, written as an adult, that a model has read as a minor. Auto Retry is not meant for sexual content involving minors, and does not support anyone using it for that.

**How the subjects are matched:**

- In the forms refusals use them, like "a character is raped", not only single words.
- Only after a refusal verb. A subject on its own never counts. So a kissing scene, rope, a stepbrother who resents his stepsister, a minor character in a story, someone choking on smoke, a knife, or a character telling you what happened to them are all left alone.
- Some words are left out completely: "violence" on its own, because a character can say "I can't describe the violence", and "choking", because a character can choke on smoke.

---

[Back to the README](../README.md)
