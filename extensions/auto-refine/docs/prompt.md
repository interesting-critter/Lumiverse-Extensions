# How the prompt is built

A refine is one model call. This page is about what goes into it.

The prompt is the settings. On the **Prompt** tab, it is a list of blocks you can see and edit, in the order they are sent. Nothing else is added in secret, so if a rewrite goes wrong, you can see exactly what the model was told.

## A block

A block has a name, a role, and text.

- **The name** is only for you.
- **The role** is System, User or Assistant.
- **The text** is what is sent. Macros in it are filled in when the refine runs.

How blocks are sent:

- Blocks are sent top to bottom.
- Two blocks in a row with the same role are joined into one message.
- A block whose text comes out empty is not sent. So is a block that is only empty tags, like `<world></world>` in a chat with no lorebook. A block that is not sent costs nothing.

### Folding one shut

Each block has an arrow beside its switch. A folded block shows only its switch and its name, so a long prompt fits on screen.

- **Fold all**, above the list, folds every block. It then says **Open all**.
- Beside it is how many blocks there are, and how many are folded.
- A folded block is still sent exactly as it is. Folding only changes the screen. Its switch still works.
- Each preset keeps its own folds. The two lists, for replies and for your messages, fold separately.
- Folds are never part of a preset or an export.

## Macros

Anything in double braces is filled in when the refine runs. There are two kinds.

**This extension's macros:**

| Macro | What it becomes |
| --- | --- |
| `{{message}}` | The turn being refined. |
| `{{history}}` | The messages before it, as many as the **Context** tab says. |
| `{{lore}}` | The lorebook entries this chat has active. |
| `{{memories}}` | What Lumiverse remembers of this chat, from further back than the history. Empty when memory is off for the chat, or when the `chats` permission is not granted. |
| `{{overused}}` | Phrases the replies in this chat keep using, one per line with a count, like `shiver ran down (4 replies)`. Only when **Find phrases this chat has worn out** is on. |
| `{{checks_found}}` | With two models, the checks the second model found in the reply, strongest first, each with its score. They come after a short lead-in that names the second model and says to treat each one as a lead to check. Only filled in when the second model read the reply and picked it out. A block saved with the older `{{jev_found}}` is switched to this one when it is loaded. See [Passing on what the checks found](two-models.md#passing-on-what-the-checks-found). |
| `{{whole_reply}}` | The reply with the part being rewritten marked. Only filled in when you refine part of a reply. |
| `{{protect_notes}}` | The instruction to leave protection tokens alone. Only when there are some. You can change it on the Limits tab. |

**Lumiverse's macros:** `{{description}}`, `{{personality}}`, `{{scenario}}`, `{{persona}}`, `{{char}}`, `{{charGroupFocused}}` (the character in focus in a group chat), `{{user}}`, and anything else that works in a character card or a preset.

Rules for macros:

- **Every prompt needs `{{message}}`.** Without it the model never sees what it should rewrite, so the refine is refused before anything is spent. The Prompt tab warns you in red.
- **A macro only costs something when it is sent.** `{{history}}`, `{{lore}}` and `{{memories}}` each ask Lumiverse for something, and only when a block that is switched on uses them.
- **This extension's macros are filled in last**, after Lumiverse's. So if a reply happens to contain the text `{{persona}}`, it stays as that text.
- **Macros only bring in what is already in your chat**: the passage, the history, the lorebook, the card. They never add wording of their own.

**Keep a macro in a block of its own.** A block is only left out when all of it comes out empty. If you write

```
<notes>
{{memories}}
</notes>
Keep these in mind.
```

then a chat with no memories still sends `Keep these in mind.`, which now refers to nothing. Put the macro in one block and your wording in another, and an empty macro takes its whole block with it.

**About `{{memories}}`:**

- It is Lumiverse's own memory of the chat, written out with your own header and chunk templates, as many pieces as your chat memory settings ask for.
- A chat with memory off, or with nothing stored yet, gives nothing, and the block is left out.
- The block that uses it, **Memories**, starts switched off, because its size is set by your chat memory settings and it is sent on every refine. Switch it on under **Prompt** when you want the refine to know more of the story.

**About `{{protect_notes}}`:** it has its own block, **Protected Formatting**. When protection has hidden something, it becomes the text in **What the model is told about the stand-ins**, on the Limits tab. You can change it there, and **Use the built-in text** puts it back. The built-in text is:

> Parts of this passage have been replaced with tokens shaped like `[[AR1]]`, `[[AR2]]` and so on. Each stands in for formatting that has to survive the edit exactly as it is. Copy every one into your answer unchanged and in the same place, treating each as a single character you cannot spell.

When nothing was hidden, it becomes nothing and the block is not sent. If you empty the box, it becomes nothing as well. The stand-ins are then sent with no note, and a model is more likely to change or drop one, which makes the refine fail.

There is no macro for whether the passage is a reply or your own message. Instead, your own messages have their own prompt, under **For your messages**. There is also no macro for the model's working. See [Asking it what it changed](#asking-it-what-it-changed).

## Order matters more than it looks

Keep the turn near the bottom. Anything after the message reads as an instruction about it, so a rule placed below it is followed more closely. A new block goes above the turn unless you move it.

The built-in prompts are ordered from what never changes to what changes every time:

1. **The rules.** The same on every refine.
2. **The setting**: who the story follows, who you are writing with, and what is true in its world. The same for a whole chat.
3. **Memories.** Lumiverse's memory of the chat. Off by default.
4. **Earlier Turns.** The recent messages, which change every turn.
5. **Passage.** Different every time.
6. **Hand It In.** Below the passage on purpose, and sent as **User**.
7. **Protected Formatting.** Only sent when something was hidden.

**Why Hand It In comes last.** A rule about the shape of the answer is followed best when it is the last thing the model reads. Placed at the top, some models forget the tags, and a rewrite without its tags is dropped, which costs you the call.

It is sent as **User** because it is your instruction about what you want back, so the model reads your passage and your request together.

## Roles

System is right for almost everything. One case where changing it helps:

- **A model that ignores system instructions.** Some providers pay more attention to the last user message. Move your rules to **User**.

**Be careful with Assistant.** A block sent as **Assistant** at the end of the request is called a prefill. It puts words in the model's mouth before it starts.

- Many newer models no longer accept a prefill. Some return an error. Some ignore it.
- This is changing fast, and more models are dropping it.
- Only end on an **Assistant** block if you know your model accepts one. Check your provider's docs, and test it on one reply first.
- An **Assistant** block earlier in the request, with a **User** block after it, is fine for any model.

The built-in prompts end on **User** blocks for this reason.

## The answer it asks for

The rewrite comes back between `<REFINED>` and `</REFINED>`, and only what is between them is saved.

- If the model opens with "Sure! Here is the rewritten message:", that line is ignored and the rewrite is still saved.
- If there is an opening tag with no closing tag, the answer was cut off. Nothing is saved, and your reply stays as it was.
- The tags are read without caring about capitals, so a prompt written in lower case still works.

**Asking for the tags is your prompt's job.** The built-in prompts ask in the **Hand It In** block, in plain words you can reword, move or delete. No hidden macro adds the instruction.

**Take the answer from between the tags**, on the Limits tab, is on by default. Off, the whole answer is taken as the rewrite, and the other checks catch a preamble instead.

## The scorecard

All four built-in prompts have a **Scorecard** block. It limits what the model can change. The two for a model that thinks score each area. The two plain ones give the same limit as a plain rule.

How the scoring works in the two that think:

- An area is one kind of fault the prompt tells the model to look for, such as stock phrases, repetition or speech.
- Each area gets a score out of 100.
- A score has to rest on a line from the passage that the model could quote. An area with nothing to quote scores 100.
- From 85 to 99 means a line might fit, but the model is not sure. Those lines are left alone.
- Only areas under 85 are changed, and in them only the lines that fit.
- A passage can score 100 in every area. Then it comes back unchanged.

Why it is there: a model that has to point at a line before it changes it leaves alone the lines it cannot point at. This stops a refine rewriting a passage that was already fine.

**The two plain prompts give a rule instead of a score.** A line is changed only when it clearly breaks a rule and the model could quote it. A line that might break a rule stays. So does a line where the fix would do more harm than good.

**The two for a model that thinks write the scorecard down.** Their answer comes in two parts:

```
<REFINE_NOTES>
Area: score. "the worst line, quoted"
(one line for each area)
The areas under 85, and what changes in each.
</REFINE_NOTES>
<REFINED>
the rewritten message
</REFINED>
```

- The scores come before the rewrite, so they decide what changes. They do not grade the rewrite afterwards.
- `<REFINE_NOTES>` is outside `<REFINED>`, so it can never reach your chat.
- The notes are kept on the **Log** tab, under **What the model worked out**. Only a refine that finishes replaces them. Stopping one keeps the last notes.
- A score is only the model's judgement. The quoted line is the part to trust.

**The two plain prompts ask for the rewrite and nothing else.** A model that does not reason tends to fill a notes tag with a summary and then do something else, which only adds cost.

**The two plain prompts never ask the model to check its work.** A model that does not reason writes its answer once, from start to end. It has no step before the answer to score in, and no step after it to look again. So these two do not say "score it in your head", "look again" or "reread yours". They say what to change and what to leave, as rules it can follow while it writes. The two that think keep their review steps, because a reasoning model does have a step before the answer.

Both tags are in capitals so a model scanning the prompt finds them easily.

## Asking it what it changed

Nothing outside the `<REFINED>` tags is ever saved into your chat. So a prompt can safely ask for a report around the rewrite: what was cut, what was added, what was left alone.

Add the tags you want to the **Hand It In** block, in your own words. For example:

```
Before the rewrite, list what you changed:

<cut>every phrase you removed, one per line</cut>
<added>anything you added, one per line</added>
<kept>anything you were tempted to change and left alone, and why</kept>

Then give the rewrite between <REFINED> and </REFINED>.
```

- Everything outside `<REFINED>` and `</REFINED>`, including `<REFINE_NOTES>`, is kept on the **Log** tab under **What the model worked out**.
- The tag names are yours. The extension only looks for `<REFINED>`. You can ask for a paragraph instead of tags.
- What the rewrite changed in your writing is shown on the card that appears when it is saved, marked word by word.

## The prompts built in

There are four: a judge for replies and a line judge for your own messages, each in two versions. Pick the version by whether your model reasons.

### For replies

| Prompt | What it is | Needs a reasoning model |
| --- | --- | --- |
| **A judge** | Tells the model it is the judge: how a passage reads is its job, and what happens in it is yours. Then one block for each rule: Instant Penalties, Dead Weight, Echoes, Rhythm, Dialogue, Body Language, Roll Call and The Finish, with a scorecard rule for all of them. The one to start with. | no |
| **A judge that thinks** | The same role, then The Bar to clear, five Hot Spots, the writer's Voice, Roll Call, and Review the Tape for its own rewrite. Scores each area in `<REFINE_NOTES>`. | yes |

- In the two that think, the thinking steps are in **Hand It In**, after the passage, near the end of the prompt. The reasons are in [Put the thinking steps at the end](rules.md#more-ways-to-make-it-work-better).
- The version for a model that thinks is the smaller one. A reasoning model is given the bar and applies it. A model that does not reason is given the full list instead, because it follows a list better than a principle.
- The rules name kinds of stock writing, with a few examples each, because "cut clichés" gives a model nothing to act on. A model reads a kind as a pattern and catches the versions that are not listed. The kinds include held breaths and hammering hearts, stock faces, voices given a texture, the air doing the mood's job, fancy stand-ins for plain words, office talk in fiction, and the negation trick: "it wasn't a request, it was a command".
- **Roll Call** is for scenes with one character or several. It keeps each line with its speaker and each character's way of talking. It keeps a name or a plain speech tag where it is the only thing saying who is talking, and it uses a name where a pronoun could mean two people. It keeps every character in the scene. It does not say who plays which character, so it works whether you write your own character or let the model write it too.
- Both have a block called **Checks Found**, switched off. It is for two models. See [Passing on what the checks found](two-models.md#passing-on-what-the-checks-found).
- All four work as they are. Load one, change what you like, and save it under your own name.
- **When these change in a later version, the Prompt tab tells you**, with a **Got it** to hide the message. It only appears if you have loaded one of the four before. It never changes your prompt or loads one for you.

### For your own messages

The same two versions, for a different job. A reply is prose to improve. Your own message is writing to leave alone except where it went wrong.

| Prompt | What it is | Needs a reasoning model |
| --- | --- | --- |
| **A line judge** | Tells the model it is a line judge: it calls clear faults on the way to the page and leaves your style alone. Then the full list, plus a block naming what is not a repair: adding a gesture, making a plain line vivid, finishing a thought you left open. Ends with one fix shown before and after. | no |
| **A line judge that thinks** | The same role, then The Ruling for telling a mistake from a choice, and the Hot Spots where fast typing goes wrong. Scores each area in `<REFINE_NOTES>`. | yes |

- Each ends by saying that when a slip cannot be told from a choice, it is a choice. So a plain "she left" stays as it is.
- Their **Roll Call** block keeps every line with its speaker when you write more than one character. It adds no line for any character.
- They are in the same menu, under their own heading. Loading one changes only the prompt for your own messages.

### The one thing they do not edit

Each of the four tells the model that the tone of the story is the writer's decision, whether it is a gentle scene, sexual content or graphic violence. It says a passage comes back at the strength it went in, that a gentle scene gets no heat added, and that sex, gore, swearing and insults keep their plain words. Then it says who a sex scene is between. This is the paragraph, word for word, in the first block of all four, **The Judge** or **Line Judge**:

> Sex scenes are between adults, 18 and older. If one involves anyone younger than that, or anyone written to seem younger, don't edit it. Hand the passage back exactly as it came, and say outside the tags that's why. Characters of any age in scenes with nothing sexual in them get edited like anyone else.

It is written as what is allowed, adults 18 and older, rather than as a list of alarming words. Words like that can make a model refuse the whole job, even on a scene between adults.

What it does and does not do:

- **It is text in the prompt, and nothing more.** The model reads it like any other line. The extension's code does not read your story for this, keeps no list of words about it, and does not block, change or delete anything on its own.
- **When the model follows it, nothing is saved.** Your reply stays exactly as it was. The Log says the model handed it back unchanged and said why, and its words are under **What the model worked out**.
- **A character of any age in a scene with nothing sexual in it is edited like anyone else.** Age alone is not the exception. Nothing else is excepted.
- **A prompt of your own carries only what you put in it.** A preset you save from one of the four keeps the paragraph, where you can read it. One you write yourself does not have it.

The checks on what comes back are separate. A model that refuses to edit a scene, for any reason, including misreading an adult character as a minor, has written a refusal instead of a rewrite. With **Refuse an answer that declines the job** on, which it is by default, a refusal is dropped and your reply is left as it was. See [The model answered the wrong question](guardrails.md#the-model-answered-the-wrong-question).

## How much it is told

Four settings on the **Context** tab. Three of them set a size, and every one of those costs tokens on every refine.

- **Messages of run-up to send** is how many messages before the one being refined are sent. The default is 4. 0 sends none, which is fine for rules about wording, but a model that cannot see earlier messages may lose the thread of the scene.
- **Most tokens of run-up** is a size limit on the same messages. Whichever limit is reached first wins. Whole messages are kept or dropped, counting back from the one being refined, so the message just before it is always kept.
- **Name the speakers in the run-up** is on by default. It puts each speaker's name at the start of their lines, which is the only way to tell the voices apart in one block of text. Turn it off for a chat whose messages already start with a name.

**In a group chat**, each reply is named after the character who wrote it. The card sent in `{{description}}` is that character's card too, found by name among the cards in the chat. Where no card matches the name, the chat's own card is sent.
- **Most tokens of lorebook** is a size limit on the lorebook entries. Whole entries are kept or dropped.

Sizes are in tokens, because tokens are what a model's context is measured in. They are counted with Lumiverse's own tokeniser where it can, and estimated at four characters a token where it cannot.

## Phrases this chat has worn out

A refine reads one reply at a time, so a phrase that appears in eleven of the last fifteen replies looks fine each time. `{{overused}}` lists those phrases, so a block can name them and ask for something else. It is only filled in while **Find phrases this chat has worn out** is on.

It looks like this, one phrase a line with how many replies it was in:

```
shiver ran down (4 replies)
the air thick with (3 replies)
let out a breath (3 replies)
```

What it counts:

- **Across replies, not within one.** A phrase has to be in a number of different replies, three by default. Set this with **How many replies a phrase has to be in**. Five times in one reply never counts.
- **How far back it looks** is **How many replies to look across**, 60 by default.
- **Narration only.** Everything in quotation marks is left out, because a character who repeats a phrase is being themselves. The **Dialogue** block handles speech instead.
- **Three words at least.** A run of only common words, like "out of the", never counts.
- **Any language with spaces between words.** Accented letters and other alphabets, such as Russian or Greek, are counted like English. Emoji are ignored. Chinese and Japanese do not put spaces between words, so phrases cannot be found in them.
- **The longest phrase wins.** A six-word phrase is not also reported as the four-word phrase inside it.
- **Left out:** your own messages, the character's name, words from the character card and the lorebook (they are the story, not a habit), and anything in backticks.
- **Formatting, trackers and thinking are left out too.** These are taken out before anything is counted:
  - HTML tags, such as a `<font color>` tag
  - trackers and status lines: table rows, lines split by `|`, lines in brackets or braces, `【】` brackets, and short label lines such as `Mood: tense`
  - a reply's own reasoning
- A line of prose that looks like a label line, such as one with a colon near its start, is only left out of the counting. The reply itself is never changed.

It adds no extra call, because it reads replies the refine already has. It does add tokens: about 150 to a request of about 1,600, so about a tenth more per refine while it is on.

**Phrases to leave alone** takes one phrase a line. Nothing listed there is ever reported. Use it when a repeated phrase is part of your story, or when a long chat starts flagging its own setting.

## Several passes instead of one

A refine is one model call by default. **How many passes a refine makes**, on the Limits tab, can make it several, each pass given what the one before it wrote. For example, one pass cuts filler and the next fixes rhythm, so no single prompt has to do both.

**The passes, in order** takes one preset name per line, top to bottom. It can be one of yours or a built-in prompt. Yours wins when the names match. A pass uses the preset by name, so editing a preset changes every pass that uses it.

The built-in prompts are whole refines, so running two in a row does the same work twice. Build a chain from presets of your own that each do one thing.

**What it costs:** one call per pass. A refine with a built-in prompt is about 1,600 tokens in, so six passes is nearer ten thousand. The Log adds all the passes up.

If a line is wrong:

- A name that matches no preset is skipped.
- A preset with no block containing `{{message}}` is skipped.
- With no usable line at all, the prompt on the **Prompt** tab runs as one pass.

How the checks apply:

- **Each pass is judged against what it was given**, not the original reply.
- **The end of the chain is judged once more against the original.** Three passes that each cut a third leave half the reply. That refusal says "across all 3 passes".
- **A refusal stops the chain.** Nothing after it runs, and nothing is saved.
- **Stop ends the chain.** The call in progress stops, and the passes after it never run.
- **A reply being replaced ends it too.** If Auto Retry swipes the reply, or you press regenerate, the chain stops between passes, so you only pay for the passes that already ran.

**What each pass changed**, on the Log tab, shows every pass in order: what it was given, what it gave back, and how much it changed the length.

Markup protection is applied once before the first pass and undone after the last, so the same tokens are used throughout.

## Protecting what is not prose

A model asked to improve a paragraph may drop a `<font color>` tag, reflow a code block, or "fix" an image link. None of that is writing.

**Hide markup from the model**, on the Limits tab, is on by default:

1. Before the refine, each piece of markup is replaced with a short token like `[[AR1]]`.
2. The model is told the tokens must come back unchanged.
3. Afterwards, the real markup goes back in.
4. **If a token did not come back, the rewrite is dropped.** This check is what makes the protection a guarantee.

What is hidden: fenced code, inline code, images, links, bare URLs, comments, HTML entities, wiki brackets, spoiler bars, table rows, tracker brackets, and any tag with an attribute (such as a colour or a link).

What is not hidden:

- **Braces.** A macro in a reply is already safe, because this extension's macros are filled in last.
- **Plain inline formatting** like `<i>`, `<b>` and `<em>`. Hiding these would leave holes in the middle of sentences and make the rewrite worse. The prompt tells the model to leave them alone. **Hide plain italic and bold too** hides them as well, if you prefer.

**Patterns of your own to hide** adds one regular expression per line to the built-in list.

- Yours are tried first.
- A pattern that cannot be read is named under the box.
- A pattern that matches nothing at all, the empty string, is refused, because it would match everywhere.

**Patterns to keep visible** keeps a region visible even when a rule above would hide it. The usual use is a colour span in the middle of a sentence, which the model handles better when it can see it.

Some things cannot be found by pattern, such as a stat block or a translation line. The built-in prompts have an **Out of Bounds** block that tells the model to keep these as they are.

### The reply's own reasoning

**Keep the reply's own reasoning out of the refine** is also on by default. A reasoning model's working is taken off before the refine and put back afterwards, unchanged.

**A reply that is only reasoning is not refined.** This covers two cases:

- The reasoning closes and nothing follows it.
- The reasoning opens with a tag and never closes, because the model ran out of room or was stopped while it was still thinking.

In both, there is no reply to rewrite, so the refine stops and says why in the Log.

Three forms are recognised by tag name:

| Form | Example |
| --- | --- |
| Angle brackets | `<think>` … `</think>` |
| Square brackets | `[thinking]` … `[/thinking]` |
| Pipes | `<\|think\|>` … `<\|/think\|>`, and `<\|think>` … `<think\|>` |

Four more are recognised as formats of their own, because their closing token has a different name:

| Form | Example |
| --- | --- |
| Harmony, used by gpt-oss | `<\|channel\|>analysis<\|message\|>` … `<\|end\|>` |
| Gemma 4 | `<\|channel>thought` … `<channel\|>` |
| Cohere | `<\|START_THINKING\|>` … `<\|END_THINKING\|>` |
| Seed-OSS | `<seed:think>` … `</seed:think>` |

- **Harmony:** the block runs to the next control token. The channels treated as working are `analysis`, `thinking`, `thought`, `reasoning` and `commentary`. The `final` channel is the reply and is refined.
- **Gemma 4:** every reply has one, empty when the model is not thinking. An empty pair is held back too.
- **Turn markers** that open and close a reply are always held back and put back around the rewrite, whatever the reasoning switches say.
- **Thinking opened in the prompt:** if the reply has a closing tag with no opening tag, everything before it is treated as thinking.

**Extra reasoning tag names** is for a model that uses a tag not listed above. Write only the name, with no brackets or pipes. It applies to the three tag-name forms.

Most cloud providers send reasoning separately, not inside the reply, so none of this is needed there. It matters for local backends.

**Keep the refiner's own reasoning out of your chat** covers the other direction: working that the refining model adds to its answer. The tags already keep most of it out. This also covers an answer read without the tags, and a model that puts its working inside the tags.

## What a refine costs

Set **Input price, per million tokens** and **Output price, per million tokens** on the **Model** tab, and the panel works out costs.

- Both start at 0, which hides all costs.
- Fill in only one, and the cost line says which half it covers.
- The prices come from your provider's price list. The extension does not know any prices.
- There is no currency. The number you type is the number you see.

**What to type:** price lists show prices like `$5.00/M` or `$0.075/M`, which already means per million tokens. Type `5` or `0.075`. You can also paste the whole thing, with `$` and `/M`, and the number is taken out.

**Read it as the most it could cost, not your exact bill.** It is worked out from the prompt this extension builds, so anything your provider adds is missing. It also prices every token at the full rate. Tokens are counted with Lumiverse's own tokeniser. Where it has none for your model, the panel says "roughly".

Where costs appear:

- **Before you spend it**, under **Show me the request**: what this request would cost, and what 100 replies of that size would cost. The answer's size is guessed as the same as the passage.
- **After you spend it**, on the **Log** tab under **Right now**: what the last refine really used and cost. Dropped rewrites and retries are included, because they were paid for.

The biggest saving of all: point **Refine using** at a smaller, cheaper model.

## Sampler settings

On the **Model** tab, every sampler starts blank. Blank means your connection's own preset decides, so a preset you tuned is never overridden.

- A value you fill in is sent with the refine only. Your chat and your preset are not changed.
- **Context size** and **Longest answer** start blank too, which is almost always right, because a refine is a small request.
- **Temperature** is the one worth changing. A rewrite usually wants it lower than for roleplay, since you want the same scene said better.
- **Longest answer** set too low cuts the rewrite off, and it is then dropped as too short.
- **Clear them all** sets every sampler back to blank.

**Some models do not accept samplers.** Many newer models, reasoning models most of all, ignore temperature and the other samplers, or return an error when one is sent. This is changing fast too.

- If a refine fails with an error from the provider after you set a sampler, clear that sampler and try again.
- Leaving them blank is always safe.

## How much thinking it does

On the **Model** tab, **Let it think first** has three choices:

- **No, keep it quick.** The default. Rewriting a paragraph does not need reasoning, and reasoning on every reply adds up.
- **Whatever my connection is set to.** Sends nothing about reasoning, so your connection's settings decide.
- **Yes, and I will say how much.** Adds an effort level: auto, none, minimal, low, medium, high, extra high or max. Each provider decides what these mean. A rewrite rarely needs more than low.

## Seeing what gets sent

**Show me the request** builds the request for the reply you are looking at and shows it, message by message, with each message's role and size. No model is called and nothing is charged.

- **The line at the top** gives the whole request in tokens, with the character count. If the tokeniser cannot answer, it says **roughly**.
- **Where the tokens go** lists each block, largest first, with its share. This shows you when history or lorebook takes most of every request.
- It is built by the same code a real refine uses, so it is what the model gets, apart from the two things below. That is why markup shows as `[[AR1]]` tokens and the reasoning is already taken off.
- Below the messages, it also shows the connection, thinking and samplers the call would use.
- If no reply can be found, it still builds, with a stand-in for the message, and says so.
- The worn-out phrases are worked out the same way a refine works them out, so **Worn Out** shows what a refine would send.
- With **Several passes** on, it shows the first pass as it would go out, and names every pass in order. The passes after the first are sent the rewrite the one before wrote, so they cannot be shown until a refine runs.
- **Checks Found** is left out, because a preview never asks the second model. The card says so when a switched-on block uses `{{checks_found}}`. The Log says when a real refine sent it.
- **Copy it** copies the whole thing, useful when asking why a refine did something odd.

## Starting a block from nothing

- A new block starts empty. The built-in prompts use tags like `<speech>`, but you can write a block in any style.
- **Expand** opens a block in a full-screen editor. It does not put the cursor in the box, so a phone keyboard does not pop up and cover it. The preview has **Expand** too, for reading.

## Presets

At the bottom of the **Prompt** tab, presets save a prompt under a name, so you can switch between prompts.

- **The four built-in prompts** are always in the list and cannot be renamed, deleted or overwritten.
- **The menu shows the prompts for the list you are editing.** A built-in prompt is written for replies or for your own messages, so only the matching two are offered. Your own presets are always offered, because they can hold either list or both.
- **Presets follow your account**, like your settings, and are also included in an export.

What a preset holds:

- every block of both prompts, for replies and for your messages: its name, text, role and place in the order
- how many messages of history are sent
- the size limits for history and lorebook

**Loading a preset changes only the prompt for the list you are on.** Load one while editing **For replies** and your prompt for your own messages is left as it is, and the other way round. The picker on the other list also stays as it was. A preset still saves both prompts, so either can be loaded from it later.

What a preset never changes: whether refining is on, the length limits, whether it asks before saving, the sounds, the chats you switched off, and everything on the **Model** tab.

**Model setup to load with it** links a preset to one of your saved model setups, so loading the preset loads that setup too. Leave it at **None** to leave the Model tab alone.

- The link can only be set on a preset of your own. To link one to a built-in prompt, pick the setup, then press **Save as new**. The card says this while you are there.
- The setup is linked by name, so a preset can be shared. If you load a preset that names a setup you do not have, the preset still loads and the card says which setup it wanted.

**Picking a preset loads it.** Its prompt is on screen and in use straight away. This means the prompt on screen is always the preset the picker names, so **Update selected** can never save one preset over another by mistake.

The buttons:

- **Put it back** appears after picking a preset loads over what you had. It restores exactly what was there, including the picker. It shows on the list the load changed, and disappears once you save.
- **Load it again** reloads the picked preset, throwing away your edits.
- **Save as new** saves what is on screen under the name in the box.
- **Update selected** saves over the picked preset.
- **Rename selected** renames it. A name already in use is refused.
- **Delete** removes it, after asking.

The **Model setups** card on the Model tab works the same way.

## Import and export

- **Export to file** saves one JSON file with the parts ticked under **What goes in the file**.
- **Import from files** reads them back. You can pick several files at once. They are applied together, and if two files name the same preset or setting, the last one picked wins.
- If a file cannot be read, nothing is taken from any of them, and the panel says which file failed.
- Importing replaces what you have, so export first if you want a way back.
- **Chats you switched off** starts unticked, because chat ids mean nothing on another account. Tick it only to move between browsers on the same account.

**Presets and model setups are matched by name.** A file with a name you already have replaces that one, instead of adding a copy. A preset that matches yours exactly is left alone, so importing the same file twice changes nothing. The panel says how many came in, how many replaced one you had, and how many were already there.

Every value in a file is checked before it is used. A hand-edited or damaged file loads what it can and says how many settings it took. A sound in a file must be audio and small, or it is left out.

## Starting again

**Start again**, on the Setup tab, puts settings back to how a fresh install has them.

- **What to put back** chooses which parts, from the same list as import and export. So you can reset your prompt and keep your connection, or reset everything including your presets.
- The button says what it is about to do, and asks first. It cannot be undone.
- The tab you are on stays open.

---

[Back to the README](../README.md)
