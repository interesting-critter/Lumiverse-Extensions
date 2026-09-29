# Changelog

Every released version of Auto Refine, newest first.

Versions follow [Semantic Versioning](https://semver.org). A new major version means a reinstall rather than an update, a minor version adds something, and a patch version only fixes things.

---

## 1.25.0

_2026-09-27_

### Added

- **A key is kept for each host.** Switching between Jev and Span, or between hosts, no longer means pasting a key again.
  - Picking a host uses the key saved for it. The key box is named after the host, such as **Key for OpenRouter**.
  - A key belongs to the host, not the model. An OpenRouter key is used for Jev and for Span on OpenRouter.
  - **Keys are saved for**, under the box, lists the hosts that have a key.
  - **Forget key** deletes the key for the host shown and keeps the others.
  - **Another address** keeps a key for each address. A different address, such as one from imported settings, has no key until you save one for it.
  - The key you already saved is kept. It is moved to the host you have picked the first time it is used.

### Fixed

- **A host that repeated your key in an error message could put it in the Log.** From there it could go into a bug report. The key is now taken out of anything a host says, and shows as [your key].
- **Another address sent your key over plain http:// when the address used it.** Over http the key could be read on the way.
  - The key is now only sent to an https:// address, or to an http:// address on the same computer: localhost, 127.0.0.1, [::1] or host.docker.internal.
  - For another http:// address you run yourself, such as another Docker container, switch on **Let the key go over http://**, under the address. It is off by default.
  - Any other address gets a line in the Log and under **Test** that says why.

### Changed

- **Refine when a check reaches is lower for both second models.** Both often give a low score to a check they find true in the reply, and the old lines left many of those replies alone. A check they find false scores close to 0. See [What the second model checks](docs/two-models.md#what-the-second-model-checks).
  - **Jev: 30 by default, not 50.** Jev often scores a check it finds true at only 30 to 35 percent.
  - **Span: 15 by default, not 25.** Span often scores a check it finds true at only 15 to 25 percent.
  - With two models on, and your line still at the old default for the model you picked, a line above the tabs offers the change. Press **Take it**, or **Keep mine**.

---

## 1.24.0

_2026-09-27_

### Added

- **Refine when a check reaches is kept for each second model.** Span has a line of its own, 25 by default, since Span gives lower scores than Jev on the same reply. The panel shows the line for the model you picked. A line you set yourself before this version is used for Span too, until you change it. See [What the second model checks](docs/two-models.md#what-the-second-model-checks).

### Changed

- **Jev's line is 50 by default again.** The line of 40 in 1.22.0 was set from Span's scores, which run lower than Jev's. With two models on, Jev picked, and Jev's line still at 40, a line above the tabs offers the change. Press **Take it**, or **Keep mine**.

---

## 1.23.0

_2026-09-27_

### Added

- **Every fixed text sent to a model can be read and changed in the panel.** Each box has a button beside it that puts the built-in text back.
  - **How it asks**, under **Also check for worn-out phrases**: the check sent with the worn-out phrases.
  - **What the refine model is told about the checks**, on the Model tab: the lead-in to `{{checks_found}}`. `{{second_model}}` and `{{checks_line}}` in it become the model's name and your line.
  - **What the model is told about the stand-ins**, on the Limits tab: the text `{{protect_notes}}` puts in.
- **What it compares**, a box under **Also compare with the reply before it**, with its own **Use the built-in checks** button. It holds the checks that compare a reply with the one before it, so you can change them. See [Comparing with the reply before it](docs/two-models.md#comparing-with-the-reply-before-it).

### Changed

- **The checks that compare a reply with the one before it are clearer, and there are two more.** They say what repeating looks like, each with an example. The new ones ask whether a reply opens the same way, or ends the same way, as the one before it.

---

## 1.22.0

_2026-09-26_

### Added

- **A guide to writing checks for the second model.** It scores whether the statement you wrote is true of the reply, and does not stretch a check to cover what you meant. The guide says how to write checks it can answer. See [How the second model reads a check](docs/two-models.md#how-the-second-model-reads-a-check).
- **The panel says the second model can be wrong.** The hint under **How many models** says so, and so does the README. When a reply is left alone, the decision card on the Log tab says the second model can be wrong, and how to refine that reply anyway. The docs have a section on the mistakes it makes and what to do about them. See [It can be wrong](docs/two-models.md#it-can-be-wrong).

### Changed

- **Refine when a check reaches is 40 by default, not 50.** At 50, a reply with a problem the second model was not sure of was left alone. A clean reply still scores well under 40. With two models on, and the line still at 50, a line above the tabs offers the change. Press **Take it**, or **Keep mine**.
- **The built-in checks are rewritten for how the second model scores them.** Each one names what to look for on the page, with examples. The check for the negation trick now names both forms, so a line about an action, such as "she didn't just leave, she ran", is caught, not only "it wasn't a request, it was a command". The two checks that asked for a judgement, about feelings the actions already show and about strained comparisons, are gone. Two new ones match rules in the built-in reply prompts: an action followed by a comment on how it came out, and an action started then taken back. With two models on, and the checks you started with, a line above the tabs offers the new ones. Press **Take it**, or **Keep mine**.

### Fixed

- **Update selected said it updated when nothing had changed.** This is on saved presets and saved model setups. When the one picked already holds the settings on the panel, the line beside the buttons now says so, and nothing is saved. It works the same way as a name already in use.
- **Put it back said it put something back when nothing had changed.** When what is on the panel is already what was there before the load, the line now says so.

---

## 1.21.1

_2026-09-26_

### Fixed

- **Span on OpenRouter answered with a 400.** OpenRouter takes Span's state as a conversation: the reply as the assistant's turn, and the reply before it, when it is sent, as the turn ahead. It was sent the named fields Jev takes. The checks are now also put in words for it, the same as on Respan's own API.

---

## 1.21.0

_2026-09-26_

### Added

- **Span, a second choice for the second model.** Span is a scoring model from Respan. Pick it under **Which second model**, in **One model or two**. It answers the same checks as Jev. See [Two models](docs/two-models.md).
- **Which Span.** Span-01 Lite, free, Span-01 Lite, paid, and Span-01. On Respan's own API these are called span-01-free and span-01-pro. See [Which Span](docs/two-models.md#which-span).
- **Respan as a host**, under **Where it is reached**, while Span is picked.
- **Another address takes Respan's scores format.** An address ending in `/scores` is sent the kind of request Respan's own API takes. See [Another address](docs/two-models.md#another-address).
- **What is Span?**, a link on the Model tab to Respan's page introducing Span.
- **Also compare with the reply before it**, in **One model or two**, off by default. On, the second model is also sent the reply before the one it reads. Three checks are added for you: whether the reply has the same events in the same order, has the characters speak in the same order, or describes the surroundings again with the same details. Your own checks can name it too, as `previous_reply`. See [Comparing with the reply before it](docs/two-models.md#comparing-with-the-reply-before-it).
- **`{{checks_found}}`**, the macro for what the checks found. It works the same whichever second model is picked, and its lead-in names that model.

### Changed

- **The two-model settings are no longer named after Jev.** **Where Jev is reached** is now **Where it is reached**. **What Jev checks** is now **What the second model checks**. **Let Jev check refines you start yourself** is now **Let it check refines you start yourself**. **Have Jev check the rewrite** is now **Have it check the rewrite**. The key box, the status lines, the Log and the **What Jev decided** card use the name of the model you picked.
- **The block What Jev Found is now called Checks Found**, and its tag is `<checks_found>`. The Prompt tab says the built-in prompts have changed. Load a built-in prompt for replies again to get it.
- **`{{jev_found}}` is replaced by `{{checks_found}}`.** A block you saved with `{{jev_found}}` is switched to `{{checks_found}}` when it is loaded, so it keeps working.
- **The Two models page is now [docs/two-models.md](docs/two-models.md)**, and covers both models.
- **A refused call now says more.** The Log and the Test line give the status the host sent and the host's own message. A wrong key reads as "the key was refused (401: ...)". A host that turns your account away, such as Respan before Span-01 is switched on for it, reads as "turned the call down (403: ...)".

- **Making your own prompt explains where thinking steps go.** It says why they work best at the end, after the passage, and why the built-in prompts that think put them in **Hand It In**. See [More ways to make it work better](docs/rules.md#more-ways-to-make-it-work-better).

### Fixed

- **The macro list did not say which permission `{{memories}}` needs.** It said the block is empty where "the permission" is not granted. It now names the Chats permission.
- **The hint under How many models said a refine you start yourself is never held back by Jev.** With **Let Jev check refines you start yourself** on, Jev can leave such a reply alone. The hint no longer says this. The switch has its own hint.

---

## 1.20.0

_2026-09-26_

### Added

- **Two more checks for Jev.** One finds the negation trick, as in "it wasn't a request, it was a command". The other finds a reply that ends by turning to you with a question, such as what you do next. Both match rules the built-in reply prompts carry. See [What Jev checks](docs/two-models.md#what-the-second-model-checks).

### Changed

- **Jev's first three checks are narrower.** "Repeats a word" and "stock phrases that turn up in many stories" were true of almost any reply, so Jev sent nearly every reply to be refined. They now ask about a repeated phrase, or three or more sentences in a row that start the same way, and give examples of stock phrases. The check about feelings now counts only a feeling the character's actions already show.
- **With two models on, and the checks you started with, a line above the tabs offers the new checks.** Press **Take it** to use them, or **Keep mine**. A space at the end of a line or an empty line does not count as a change. Checks you wrote yourself are not touched, and nothing is said about them.

### Fixed

- **Use the built-in list gave no sign it was pressed when the box already held the built-in list.** This is on the two boxes for where the input box is and your next / swipe button. A line beside the button now says it is already the built-in list, and nothing is changed. It works the same way as a name already in use when you save a preset.
- **Use the built-in checks said so in a pop-up.** When the checks are already the built-in ones, it now says so on the line beside the button. A space at the end of a line or an empty line no longer counts as a change.

---

## 1.19.4

_2026-09-26_

### Fixed

- **Give up waiting after did not work while the tab was in the background.** On a phone, a tab left idle has its timers paused. So a refine could show "Thinking, 1175s" with the limit set to 1000. The limit is now checked against the real time while the panel runs, and again as soon as you come back to the tab. If the tab was in the background, the message says so and asks you to check the reply, since a reply that finished while the tab was asleep can miss the panel.

### Changed

- **A new page says what to use.** [What to use](docs/recommended.md) lists the prompt, model, thinking and sampler settings that work well for most people. It also says when to turn thinking on, and what to do when a refine is slow or changes too much.

---

## 1.19.3

_2026-09-26_

### Fixed

- **The Finish did not say whose choice the ending is.** It said "the writer's call", and "writer" is used nowhere else in the prompts, so a model could read it as you or as itself. It now says whoever wrote the passage picked the ending.

### Changed

- **The paragraph about adults is reworded.** It now says that sex scenes are between adults, 18 and older. A sexual scene involving anyone younger, or anyone written to seem younger, is handed back unedited with a note saying why. It no longer uses alarm words, which can make a model refuse to edit a scene between adults. What it protects is the same. See [The one thing they do not edit](docs/prompt.md#the-one-thing-they-do-not-edit).
- **The built-in prompts use the word "call" far less.** It was in most blocks. Two blocks in the prompts for your own messages are renamed: **Not Your Call** is now **Hands Off**, and **The Call** is now **The Ruling**.
- **The two plain built-in prompts no longer ask the model to check its work.** A model that does not reason writes its answer once, from start to end. It has no step to score in its head, look again or reread. So **A judge** and **A line judge** now say what to change and what to leave, as rules it can follow while it writes:
  - Their **Scorecard** block says to change only a line that clearly breaks a rule and could be quoted. A line in doubt stays. It no longer gives scores out of 100.
  - **Clean Run** says the rewrite comes out about as long as the original, or shorter. It no longer says to look again.
  - **Echoes** and **Rhythm** no longer ask for a second read.
  - **A line judge** ends its list of faults with one fix shown before and after.
  - The two that think keep their scores and their review steps. **Roll Call**, which all four share, now opens without "before you start cutting".
  See [The scorecard](docs/prompt.md#the-scorecard).
- **The guide to making your own prompt has a new section.** [More ways to make it work better](docs/rules.md#more-ways-to-make-it-work-better) covers using one name for each thing, describing a limit by what is allowed, watching your own repeated words, and giving the model a bar to clear. **Match your model** now says how to write for a model that does not reason.

---

## 1.19.2

_2026-09-25_

### Fixed

- **Worn Out counted colour tags as a worn-out phrase.** A reply with dialogue in a `<font color>` tag gave phrases like "font color font he said". The quote marks inside the tag also made the dialogue check miss a spoken line. HTML tags are now taken out before anything is counted. Reported by a Discord user.
- **Worn Out broke accented words and skipped other alphabets.** Only the letters a to z were read as letters. So "café" was counted as "caf", and a chat in Russian or Greek was never counted at all. Letters in any alphabet are now read as letters. The check that catches a softened rewrite reads them the same way, including words you add to it.
- **Worn Out counted a reply's own thinking.** Reasoning written in the message was counted along with the story. It is now left out.
- **A refine was dropped when the thinking was moved out of the reply.** Refine checks that a reply did not change while it was being rewritten. Moving the thinking into the Reasoning box while a refine ran counted as a change, so the refine was dropped. Now a change to the thinking alone is allowed. The refine is saved without the thinking that was moved out.
- **Worn Out counted trackers and status lines.** Lines a card prints in every reply, such as `Mood: tense`, `HP 10/10 | Coins 12` or `[Time: evening]`, came back as worn-out phrases. Table rows, lines split by `|`, lines in brackets or braces, `【】` brackets and short label lines are now left out of the counting.

## 1.19.1

_2026-09-25_

### Fixed

- **Saying yes to a refine could fail with "that message is gone".** Reported by a Discord user. With **Ask before saving a refine** on, a refine started with **Refine the latest reply** could not be saved. When the panel had no id for the latest reply, the question you were asked held no id either, so the yes looked for a message with no id and found none. The question now holds the id of the reply that was refined.
- **Thinking that was cut off could be refined as the reply.** A reply that was only reasoning, written in the message and cut off before its closing tag, was sent to the refine model as the passage. A thinking tag with nothing closing it is now treated as the model working, and that reply is left alone.
- **Fixed again: a refine you start is said once in the Log.** 1.19.0 fixed this for a refine started on a message, and missed one case. A refine started with **Refine the latest reply**, when the panel had no id for that reply, was still said twice. The result of that refine now names the reply it refined, so the two lines are matched.

## 1.19.0

_2026-09-25_

### Added

- **The built-in prompts score a passage before they change it.** All four ask the model to score each area out of 100, and each score has to rest on a line it could quote. Only areas under 85 are changed. A line the model is not sure about is left alone. The two for a model that thinks write the scores down, and they are kept under **What the model worked out** on the Log tab. The two plain ones score silently and send back only the rewrite. See [The scorecard](docs/prompt.md#the-scorecard).
- **Roll Call**, a block in all four built-in prompts. It keeps every line with its speaker, and each character's way of talking, in a scene with one character or several. In the prompts for replies it also keeps a name or a plain speech tag where it is the only thing saying who is talking, and uses a name where a pronoun could mean two people.
- **A step-by-step guide to making your own prompt.** It shows how to start from a built-in prompt, find what annoys you, group it by kind, keep it short, match your model, add your own taste, and test it. See [Making your own prompt](docs/rules.md). It starts with what every prompt must have to work.
- **The docs warn about prefills and samplers.** Many newer models no longer accept an Assistant block at the end of a request, or some sampler settings. The docs say when to avoid both, and what to do if a refine fails because of them. See [Roles](docs/prompt.md#roles) and [Sampler settings](docs/prompt.md#sampler-settings).
- **The Log says when what Jev found went to the refine model**, with how many checks, such as "what Jev found went to the refine model: 2 checks". It also says so for the second refine that **Have Jev check the rewrite** can start. A real request is not shown anywhere else, so this is how to tell it was sent. See [Passing on what Jev found](docs/two-models.md#passing-on-what-the-checks-found).

### Changed

- **The Prompt tab tells you the built-in prompts have changed.** Your own prompt is not touched. To get the new blocks, load a built-in prompt again.
- **The four built-in prompts have new names.** The line edit is now **A judge**, and The copy edit is now **A line judge**. The versions for a model that thinks are **A judge that thinks** and **A line judge that thinks**. A prompt you had picked, or named as a pass, follows the new name.
- **The built-in prompts are written around a scorecard.** The model is the judge on a reply, and the line judge on your own writing. Each block is named after the tag it holds, and several were renamed:
  - Your Role is **The Judge** in the prompts for replies, and **Line Judge** in the prompts for your own messages.
  - Phrases to Cut is **Instant Penalties**, Words to Cut is **Dead Weight**, Repetition is **Echoes**, Speech is **Dialogue**, Bodies and Feeling is **Body Language**, and How It Ends is **The Finish**.
  - What to Leave is **Clean Run** in the prompts for replies, and **Not Your Call** in the prompts for your own messages.
  - The Standard is **The Bar**, Where to Look is **Hot Spots**, Restraint is **No Extra Credit**, and Before You Answer is **Review the Tape**.
  - What to Mend is **Clear Faults**, The Test is **The Call**, and The Way They Write is **Their Voice**.
  - Copy These Exactly is **Out of Bounds**, and How to Answer is **Hand It In**.
  - Your Characters is **Cast Sheet**, The User's Characters is **Player Sheet**, What Is True is **World Facts**, Earlier Pages is **Earlier Turns**, What Has Happened is **Memories**, Already Worn Out in This Chat is **Worn Out**, The Reply Around It is **Reply Around It**, and Passage to Refine is **Passage**.
  - What each block asks for is the same, apart from the changes listed here.
- **The built-in prompts say plainly that the tone is yours.** Gentle, sexual or violent, a passage comes back at the strength it went in. A gentle scene gets no heat added, and sex, gore, swearing and insults keep their plain words.
- **The built-in prompts are written the way a person talks.** They use contractions, some swearing, and sentences of different shapes, so they do not read like the writing they exist to fix. What they ask for is the same. The lead-in before what Jev found now says to treat each check as a lead to check.
- **Instant Penalties is shorter and catches more.** It lists kinds of stock writing with a few examples each, instead of every phrase. New kinds: fancy stand-ins for plain words, office and maths talk in fiction, the negation trick, do-then-undo, and saying the same thing twice. **Dead Weight** also catches hedges like "something like fear" and two adjectives stacked on one noun. **Body Language** also cuts filters like "she noticed". Truly and utterly join **Dead Weight**.
- **A plain speech tag that only says who is talking is kept** in a scene with two or more people. The line edit, now A judge, took out tags such as "she asked" before, which could leave nothing to say who spoke.

### Fixed

- **Passes and phrases typed into their boxes did nothing.** **The passes, in order** and **Phrases to leave alone** saved what you typed as one piece of text, and the refine only reads a list. So typed passes never ran, and typed phrases were still reported. Both now save a list, one line to an entry, and anything already saved as text is read back as a list.
- **A group chat labelled every reply in the run-up with the chat's first character.** Each reply is now labelled with the character who wrote it. The card sent with a reply is also the card of the character who wrote it, not always the chat's first one.
- **A refine you start was said twice in the Log.** It wrote "refined a reply in" and "refined a reply on request in" for the same reply, and could show two toasts. It now says it once.
- **Show me the request left out the worn-out phrases.** It never worked out `{{overused}}`, so the **Worn Out** block was always empty there and left out, while a real refine sent it. The preview now works the phrases out the same way a refine does.
- **Show me the request showed the wrong prompt with several passes on.** It built the prompt on the Prompt tab, which a refine with several passes never sends. It now shows the first pass as it would go out, and names every pass in order.
- **Show me the request left out What Jev Found without saying so.** A preview never asks Jev, so the block was empty and left out, and the request looked as if the findings were never sent. The card now says the block is left out here, in the plain view and in **Raw**, and that the Log says when a real refine sent it. The README and the docs no longer call the preview exactly what is sent. See [Seeing what gets sent](docs/prompt.md#seeing-what-gets-sent).

---

## 1.18.0

_2026-09-24_

### Added

- **What Jev Found**, a block in both built-in prompts for replies, switched off. With two models, it hands the refine model the checks Jev found in the reply, strongest first, with their scores. A short lead-in says each one is a lead to check, not an order, since Jev can be wrong. The block is left out whenever Jev did not read the reply. See [Passing on what Jev found](docs/two-models.md#passing-on-what-the-checks-found).
- **`{{jev_found}}`**, the macro that block uses, for a prompt of your own.
- **Have Jev check the rewrite**, in **One model or two**, off by default. On, Jev also reads what the refine model wrote. If a check still reaches your line, the reply is refined once more, with what Jev found in the rewrite. It happens once, and costs one more Jev call per refine. See [Having Jev check the rewrite](docs/two-models.md#having-it-check-the-rewrite).
- **The One model or two card says when nothing takes what Jev finds**, and where to switch it on.

### Changed

- **The Prompt tab tells you the built-in prompts have changed.** They now hold What Jev Found. Your own prompt is not touched. To get the new block, load a built-in prompt for replies again.
- **Use the shipped list is called Use the built-in list.** The button on **Where the input box is** was renamed in 1.13.0, and that version did not say so. It does the same thing.

### Fixed

- **The Jev switch did not say it covers Refine every reply here.** **Let Jev check refines you start yourself** has always covered it, but the hint and the docs named only **Refine the latest reply** and the button on a message. Both now say so. With the switch on, Jev reads the replies one at a time, and only the ones it picks out are refined. See [When Jev is asked](docs/two-models.md#when-it-is-asked).
- **The docs did not say why Refine when a check reaches stops at 99.** They now explain both ends: at 100 nearly every reply would be left alone, and at 0 every reply would be refined, with a Jev call paid for either way. See [What Jev checks](docs/two-models.md#what-the-second-model-checks).

---

## 1.17.0

_2026-09-24_

### Added

- **Let Jev check refines you start yourself**, in **One model or two**, while two models are on. Off by default, so a refine button goes straight to the refine model as before. On, Jev reads the reply first when you press **Refine the latest reply** or the button on a message, and a reply it finds nothing wrong with is left alone. A selection and your own messages are never sent to Jev. See [When Jev is asked](docs/two-models.md#when-it-is-asked).

### Fixed

- **Fixed again: a built-in prompt stays locked.** 1.14.0 fixed this for switching tabs and lists, and missed one case. Picking one of your own presets under **For replies** also changed **For your messages**. It loaded the prompt for your messages saved in that preset, and it named your preset on that list too. So the prompt for your messages was no longer greyed out, even when it was a built-in one, and could be typed over. Now picking a preset names it only on the list you are on, and loads only that list's prompt. **Put it back** shows on that list only. See [Presets](docs/prompt.md#presets).
- **Two preset saves close together could leave the older one in your account.** Saves of your presets and model setups could finish out of order, so the next load from your account brought back the older copy. They are now written one at a time, in the order you made them, the same as your settings.
- **The panel could miss your account's settings for a whole visit.** If it opened before the extension's server side had started, its request for your account's copy got no answer, and it was not asked again. The panel then ran on this browser's copy, which can be older, and the next change you made saved that older copy to your account. Now the panel asks again as soon as the server side is ready. See [How settings are saved](docs/settings.md#how-settings-are-saved).

---

## 1.16.1

_2026-09-24_

### Fixed

- **The hint under How many passes a refine makes** no longer says that two cheap passes often beat one expensive one. That was never measured. It now says what the setting does and what it costs.

---

## 1.16.0

_2026-09-24_

### Added

- **Test shows its answer in What Jev decided**, on the Log tab, marked as a test. See [Reading a test](docs/two-models.md#reading-a-test).
  - The score for the test question, and which Jev answered.
  - The address the test went to, the format it was sent in, and the model name it asked for.
  - What the host said the test cost, or that it reported no cost.
  - If the test failed, why, and where it was sent.
  - A test is not counted as a reply Jev read.
- **The Expand editor comes up smoothly.** The dimmed background fades in, and the editor rises a little and grows into place as it fades in. It is the same entrance Auto Retry's dialogs make. If your device is set to reduce motion, it just appears.

### Fixed

- **An older prompt could come back.** Opening Lumiverse on a device saved that device's copy of your settings to your account before your account's copy was read. A phone opened after you changed a prompt on a computer put the old prompt back, on every device. Now only a change you make is saved to your account, saves land in the order you made them, and a tab you come back to after a while loads your settings from your account again. See [How settings are saved](docs/settings.md#how-settings-are-saved).
- **The refine buttons lit up on the home screen.** A reply or a refine finishing in a chat that was not open, such as one you had just left or one in another tab, made the panel act as if you were in that chat. The buttons now stay greyed out until a chat is open.
- **A deleted block stuck and then jumped shut.** The space it leaves now starts closing straight away. The same goes for every row that closes when a switch turns it off.

### Changed

- **Folding a block on the Prompt tab has no animation.** A block opens and closes at once. Deleting a block still closes its space over a moment.

---

## 1.15.0

_2026-09-23_

### Added

- **Which Jev.** In **One model or two**, under **Where Jev is reached**. Pick **The latest Jev**, **The preview Jev**, **Jev 1.13 exactly**, or **A name I type**. See [Which Jev](docs/two-models.md#which-jev).
  - The latest moves to each new Jev by itself, so its answers can change. Jev 1.13 exactly keeps them steady.
  - The preview runs ahead of the latest when TypeSafe has a preview build. Only TypeSafe names one, so it is only offered when the host is TypeSafe.
  - A name you type is sent as you typed it, for a host that renames Jev.

- **What Jev decided**, a card on the Log tab while two models are on. It shows the last reply Jev read: each check with its score as a bar, the line marked on each bar, whether the reply was refined, which Jev answered, and the cost. It also counts, since the page opened, how many replies Jev left alone, which is how many refines you did not pay for. See [Reading what it decided](docs/two-models.md#reading-what-it-decided).

- **What is Jev?**, a link on the Model tab to TypeSafe's own introduction to Jev.

- **The Log names the Jev that answered**, for example "Jev (jev-1.13.0) says leave it". **Test** names it too.

- **Another address takes OpenAI and Claude format addresses.** An address ending in `/chat/completions`, `/responses` or `/messages` is sent the kind of request that format takes. The address and model name boxes show an example, and [Another address](docs/two-models.md#another-address) lists the addresses known to work.

### Changed

- **Two models use the latest Jev by default.** Until a new Jev comes out, that is the same Jev 1.13 as before. Pick **Jev 1.13 exactly** to stay on it.

---

## 1.14.0

_2026-09-23_

### Added

- **Two models, a beta.** On the Model tab, under **One model or two**. With two, a small model called Jev reads each new reply first and checks it against a list you write. The reply is refined only if a check reaches the line you set, so replies that are already fine cost nothing. See [Two models](docs/two-models.md).
  - Jev is reached through OpenRouter, NanoGPT, TypeSafe, or another address.
  - Your Jev key is kept in Lumiverse's secure store, never in your settings or an export.
  - If Jev cannot answer, the reply is refined as normal, and the Log says why.
  - A refine you start yourself never goes to Jev.
  - **Use the built-in checks** puts the starting checks back, and changes nothing else.
  - It needs a new permission, `cors_proxy`. Refusing it only affects two-model mode.

- **Seconds between automatic refines**, on the Limits tab. A reply that arrives too soon waits for the gap, with a countdown, and is then refined. **Stop** ends the wait. It is 0, no gap, by default.

- **`{{charGroupFocused}}` in the list of macros.** In a group chat it is the name of the character whose turn it is.

### Changed

- **The built-in prompts name their one exception.** They now tell the model not to edit sexual content involving anyone under eighteen, or anyone written as a child, and to hand that passage back unchanged and say why. Your reply is then left as it was. A younger character in a scene with nothing sexual in it is edited like anyone else. The panel tells you the built-in prompts have changed. Your own prompts are not touched.

- **Each preset keeps its own folds.** Folding a block in one preset no longer folds it in presets saved from it. A copy you save keeps the folds you had, and a rename keeps them too.

- **The Fold all button fits on one line.** It says **Fold all** or **Open all**, with a count of how many blocks are folded beside it.

- **A passage handed back with a reason says so.** When the model returns a passage unchanged and explains why, the Log says that, instead of "it already read well". The model's words are under **What the model worked out**.

- **Clearer empty boxes.** An empty block asks "What do you want this block to tell the model?" and its name box asks "What is this block called?". The preset name box says "A name for this preset" instead of "A name for this setup".

- **A snip says it cost nothing.** **Take out what I selected** never calls a model, and the message after one now says so.

- **Only a switch flips a switch.** Pressing the words next to a switch, or the space beside the automatic switch, no longer changes it. Only the box does.

- **The README says exactly how the one exception works.** The code does not judge your story, keeps no list of words about it, and blocks nothing on its own. The exception is one paragraph in the built-in prompts, quoted in full in the docs.

- **Shorter descriptions in the panel.** Card intros and the longest descriptions behind **?** are cut to a sentence or two. The detail is in the docs.

- **The docs are written in plain language:** shorter sentences, lists and numbered steps, with the README kept to a short front page.

- **The comments on the refusal checks say what the words about age are for:** a model that misreads an adult character as a minor.

### Fixed

- **Take out what I selected works every time.** Reported by a Discord user. With refines added as swipes, a snip was also added as a swipe you could not see, and after that every snip looked broken. A snip now always edits the text on screen.

- **Thinking that starts in the prompt is kept out of the refine.** Some presets open the reply inside the thinking tag. That thinking was sent to be rewritten like prose. It is now held back and put back unchanged.

- **A built-in prompt stays locked.** Changing drawer tab, leaving the panel alone for a while, or switching between **For replies** and **For your messages** could unlock it, so you could type over it. Each list now remembers its own prompt.

- **With the automatic pass off, new replies no longer show up in the Log.** It said "left a reply alone" for every reply, which looked like it had tried. It also stopped the spinner of a refine you started yourself if another reply arrived.

- **Renaming a preset or setup to its current name says so**, instead of "Renamed.".

- **Tab names are no longer cut short.** On a phone, **Prompt** and **Context** showed as "Pro…" and "Cont…", and **Context** could be cut short in the drawer on a computer too. Each tab now starts at the width of its name.

- **An install shared by several accounts uses each account's own settings.** It used one set for everyone, from whichever account's panel sent settings last. Now a refine uses the settings of the account that asked, or whose chat it is. Refines from different accounts take turns, and the panel says when one is waiting.

---

## 1.13.0

_2026-09-19_

### Added

- **Your next / swipe button**, a box under **Add the refine as a swipe instead of writing over the reply** on the Limits tab. It is built the way **Where the input box is** is built: CSS selectors separated by commas, tried in the order you write them, with **Test**, **Use the built-in list**, and every selector listed under the box with the one in use highlighted.

  The refine is written into the message either way, and Lumiverse is told the new swipe is the one to show. Where a build draws it on its own, nothing here is pressed and the box never matters. Where a build leaves the reply you already had on screen instead, the arrow is now pressed for you, so what you are looking at is the refine rather than the writing it was put in beside. That press is what the box aims, and an update that renames those arrows is something you can correct on the day rather than wait out.

  It is pressed once, and only while the writing the refine replaced is still on screen. The same arrow asks Lumiverse for a fresh reply when there is no swipe ahead of it, so it is never pressed on the chance that it helps.

### Changed

- **Reroll is called swipe.** The switch, its explanation and the page about it all said reroll for something Lumiverse itself calls a swipe. Nothing about what it does has changed.

### Fixed

- **Every part's switch in a built-in prompt stays live.** Loading one locked the whole card, switches included, and two of the parts come switched off. **What Has Happened** is one of them, so putting your memories into a built-in prompt meant saving a copy under your own name to reach a switch that was sitting right there. A switch chooses which parts go to the model, and it does not rewrite a word of what they say, so it is not what the lock is for.

- **Back to the default greys the boxes out again.** Pressing it put the built-in words back and left the boxes open to type in, which read as a prompt of your own and was not one. The picker is what the lock reads, and it was being set after the card had already been drawn.

- **The automatic switch keeps its place while you search.** It sat in the same row as the buttons above it, so where it landed depended on how much room those buttons had left, and a search that found nothing moved it. It now has a line of its own under them.

- **A refine gives up by the second you set it.** Waiting out a provider that would not take the call gave the watchdog a fresh deadline each time it waited, so a run that waited several times could pass the limit and keep going. The limit is now counted from the moment the refine started, and the waits are added to it once.

- **The eyes in the panel read in step with the ones in the chat.** Each mark started its animation from the moment its own element was built, so a mark drawn later ran behind the rest and the panel's looked as though it were moving the other way. Every mark now joins the sweep already in progress.

- **No button stands under a message you are editing.** Opening a message for editing takes its row of actions away and leaves the footer behind, so the button fell back to the footer and sat on its own under the editor. There is nothing for it to do there either, since what it would work on is the reply as it stands saved rather than the text being typed over it. It comes back when the editor closes.

- **The button comes back with the rest of the row.** Closing an editor put Lumiverse's own buttons back at once and this extension's a quarter of a second later, so it arrived beside controls that were already sitting there. Everything else the page redraws is still answered at the old pace, since a chat that is streaming changes constantly and none of that is worth answering that quickly.

- **The mark on the drawer tab keeps blinking while you change tab.** That one is drawn by Lumiverse rather than by this extension: it is handed over once and written out again whenever the sidebar is redrawn, which is every time you change tab. The copy that came back was resting shut in the middle of a refine the rest of the marks were still reading through, and nothing told it otherwise until the refine ended.

  Any mark redrawn while a refine is running is now caught as it arrives, before the browser has drawn it, and put back into the sweep at the point the others have already reached. So there is no frame where it sits still and none where it jumps.

---

## 1.12.0

_2026-09-18_

### Added

- **Where the input box is**, a new card on the **Setup** tab. Refining a draft is the one thing here that reads Lumiverse's own layout, so a release that moves the input box stops that and nothing else. Until now the only fix was waiting for a new version.

  The box holds the list itself, so what is being tried can be read and edited in place. Separate the selectors with commas. They are tried in the order you write them, and emptying the box falls back to the list this came with.

  Every selector is listed under the box in the order it is tried. The one finding the box is highlighted, and the rest are shown plainly, so you can see where yours sits in the order.

  A selector the browser cannot read is marked in red. **Test** covers the box as a whole, and a box with one typo and four good selectors still passes, so the mark goes on the line instead.

  **Use the shipped list** puts the box back, and **Where the input box is** is its own part in the reset and transfer lists, so putting it back does not take the widget and the buttons with it.

- **Tap a running refine to stop it.** Suggested by a Discord user. The buttons in the chat already turned into a spinner while one was working, and tapping the spinner did nothing but say a refine was already running.

  Now the first tap starts it and the next one calls it off. While it runs the button names itself **Stop this refine**, so a screen reader says the same thing the mark shows.

  This is the two buttons in the chat: the one in the toolbar and the one under a message. The panel and the floating button already had a stop of their own.

- **Two buttons for the part you selected.** Suggested by a Discord user.

  **Refine the part I selected** does what the panel entry of the same name does, without opening the panel.

  **Take out what I selected** deletes the selection. There is no model call, so it costs nothing and is as quick as pressing it.

  Both appear only while text is selected, so neither is ever a button sitting there doing nothing. With **A button on every message** on they appear under the message holding the selection, and nowhere else. With **A button in the chat's row of controls** on they appear in the toolbar too, wherever in the chat the selection is. Either switch on its own is enough, and both are under **Ways to reach it**.

  In both places the button that starts a plain refine steps aside while something is selected, so what is in front of you is what to do with the text you picked. It comes back when you put the selection down, and the other messages keep theirs.

  Neither needs a button in the chat switched on at all. Both are in the chat input's Extras menu as well, and in the floating button's menu while that button is on screen, and in both places only while something is selected.

  Taking text out closes the gap the way a person would. One space between two halves rather than two, no space left in front of a full stop, and a paragraph taken out leaves one blank line rather than two. Nothing else in the message is touched.

  Selecting the whole message is refused rather than emptying it. **Put it back** works on a snip exactly as it works on a refine.

- **The before and the after, side by side.** Suggested by a Discord user. Every card showing what a refine changed marked the two versions up as one piece of text, with what was taken out struck through and what was put in next to it. That is the quicker read when a rewrite moved a word here and there, and the harder one when whole sentences were replaced and the result is more colour than writing.

  **Read them side by side** puts the original in one column and the rewrite in the other. Each column reads as the whole text it stands for, and each still carries its own marks: what was taken out on the left, what was put in on the right. **Read them together** puts it back.

  The button sits above every before and after there is: the card that comes up on the page, the **Put it back** card in the panel, and each pass under **What each pass changed** on the Log tab. Pressing it changes all of them at once, and the view you were last on is the one the next card opens on.

  On a narrow screen the two columns stack one above the other rather than squeeze, because two columns of twenty characters each is a worse read than a scroll.

### Fixed

- **A rewrite is now told not to take things out.** Every block guarded against the model adding to your writing, and none of them guarded against it removing a line that read plainly. Instructions hold a model back from adding much better than from cutting, so the cutting is now named on its own.

- **A built-in prompt no longer claims to be loaded after you have changed it.** Loading one set the picker and nothing cleared it, so editing a block afterwards left the box naming a prompt your list no longer matched.

  The card now says so once the two part company, and says where to keep the change: a prompt that comes with the extension cannot be written over, so it goes under a name of your own with **Save as new**.

  The blocks are not locked while a built-in prompt is picked. Loading one and changing it is how you are meant to start.

- **The thinking setting on a built-in prompt was never applied.** Each of them carried one, and loading a prompt walks past it, because that setting belongs to a model setup rather than to a prompt. It has been taken off them.

  Nothing changes in what a refine does. Which model a prompt is written for is still said in its name and its description, which is advice about what to pick rather than a switch thrown on your behalf.

- **A built-in prompt cannot be typed into.** Picking one from the list locks its blocks: the text, the switch, the role, the reordering, the delete, and **Add a block**. A line above them says why and says what to do instead.

  Those four cannot be written over, so editing them was typing into something the panel was about to refuse to save. The rule is now one sentence: the ones built in are there to read and to start from, and the ones you save are yours to change.

  To change one, put a name in the box under **Presets** and press **Save as new**. The copy is yours and opens for editing straight away.

  A fresh install is not affected. The picker starts on nothing, so somebody who has never opened the list can type into every block. The text stays selectable while locked, so a line can still be copied out of a prompt you want to borrow from.

- **The wait before a refine is given up on is four minutes, not ninety seconds.** Two of the four built-in prompts are written for a model that reasons, and such a model can think for minutes before it writes a character. A local model can spend that long loading. Both were being cut off mid-thought by the number they came with, and the settings page told you to raise it yourself.

  Four minutes is the slow end on purpose. A fast model answers in seconds and never reaches it, so all the number really decides is how long you wait before being told a refine that was never coming back has been given up on.

  If you were still on ninety seconds, the panel says so and offers to move you. If you had set your own, it says nothing, because nothing of yours changed.

- **A connection that refuses one of the fields sent with a refine no longer stops the refine.** A strict OpenAI-compatible endpoint turns the whole request down over a single field it does not take, rather than ignoring it, and the panel showed a 400 that read like a fault in your rules. NVIDIA's build does this with the context size and the thinking setting.

  The refusal is read and the refine is asked again without the fields it named, once. Only fields that actually went out are dropped, so a message mentioning something the extension never sent changes nothing, and a second refusal is treated as real.

- **A hold that a thumb moved during was dropped with no message.** Holding the floating button opens its menu, and the hold was called off as soon as the finger moved more than six pixels. A thumb resting on glass moves further than six, so the menu simply never opened and nothing said why. Ten pixels now, which is the same allowance Auto Retry's button gives.

- **The tab strip no longer slides sideways under a finger.** It scrolled, and the padding put its contents a few pixels over the box, so a drag meant for the panel moved the tabs instead. It is one row now and does not scroll. On a narrow screen the tabs get narrower rather than the last one dropping to a second line.

### Changed

- **The extension's mark is an eye.** Everywhere: the drawer tab, the panel heading, the menus, the button in the chat's row of controls, the button on each message, the floating button and the row in the input bar.

  What this does to a reply is read it and put it back in better words, and an eye is the part of that anybody can see happening. It also gives one shape three things to say, which a row of lines and a sparkle could not.

  Every eye rests shut and opens when a refine starts. Pointing at a button that carries one opens it too, so a mark you can press answers before you press it. The floating button is the exception there: it is shut because nothing is running, which is a fact about the extension rather than about where your pointer is.

  **Refine the part I selected** carries the same eye with a bracket at either side of it, so the pair reads as "all of it" and "this much of it". **Take out what I selected** is still a pair of scissors, because taking text out is a different thing from reading it.

  The floating button shows the mark for whatever it is running: the selection mark while it refines a selection, the eye for a plain refine. Taking a selection out is not on that list, since it is over the moment it is pressed.

- **Every mark moves together.** The drawer tab, the heading on the panel, the buttons in the chat and the floating button all read while a refine is running, and all rest shut when nothing is.

  A refine that finishes closes the eye on one long blink. One you stop closes it quickly and without the blink, since a blink is what an eye does when it has read something.

- **The buttons in the chat stand in Lumiverse's own rows, dressed as its own buttons.** The one under a message sat in a row of its own, centred under the text, with the app's Copy, Edit and the rest in their row above it. Two rows of buttons on one message, one of them holding a single button, and nothing about the lower one said it belonged to the same message as the upper one.

  Every display mode draws a row of buttons on a message, in a different place and under a different name, so the row is found rather than assumed. The buttons go in it. A display mode that draws no row at all gets the old row under the message, which is what that row was for.

  How they look is no longer decided by this extension at all. Each one takes the classes off the button standing next to it, and its mark takes that button's size and line weight. Those classes carry a build hash, so they could never have been written down in here; read off the button they are already on, they are right on every build and follow a Lumiverse update rather than break on one.

  That is also what puts these buttons inside your own styling. A theme reaches them, and so does CSS you wrote yourself, because to a stylesheet they are Lumiverse's buttons.

- **A refine no longer spins a ring. The eye reads.** The pupil crosses from one side to the other at the pace of somebody scanning a line, snaps back to the start the way your eye does at the end of one, and blinks on the way back.

  A turning ring says only that something is happening. This one says what is happening, and it is the same shape the button already had rather than an unrelated mark swapped in for the duration.

  Somebody who has asked their device for less movement gets the eye open and still, and the button's tooltip still says it is working.

- **A ring fills around the floating button while you hold it.** Holding the button opens its menu, and nothing on screen said a hold was under way, so the half second before the menu appeared read as a tap that did nothing.

  The ring starts at the top and closes a moment before the menu opens. Letting go early wipes it back. A press also dips the button a little, so a tap answers whether or not it changed anything: a dip on its own is a tap, a dip with the ring running is a hold.

  Auto Retry's floating button draws the same ring the same way, at the same length, so the two behave alike. Under less movement the menu opens on the same hold with no ring drawn.

- **Every built-in prompt now opens by telling the model which job it has.** The first block used to describe the work without ever naming the worker. A line editor and a copy editor are two different jobs with a real boundary between them, and handing the model one of those names carries more than a paragraph of instructions does.

  **The line edit** puts the model on how a reply reads and keeps it off what happens in it. **The copy edit** puts it on what went wrong on the way to the page and keeps it off your style.

  The blocks also say plainly what they only implied before: change a sentence the moment it matches something named, without weighing whether this one is borderline, and leave every sentence that matches nothing. A passage coming back changed in every line is a worse edit than one carrying four good changes.

- **Eight built-in prompts are now four.** The quick and close pairs were two sizes of the same prompt, and the smaller of each was a saving that did not amount to much on a prompt this size. What is left is one line edit and one copy edit, each once for any model and once for a model that reasons, which is the split that genuinely changes the prompt.

  Each description says which of those it is for and stops there. They used to rank themselves by size, offering the shorter of two, which is not a thing anybody was choosing on and is not a choice that remains.

  A fresh install starts on the fuller one of each pair, which is what the old close read was.

  Your own prompt is untouched. If you had loaded one of the eight, the panel says the built-in set has changed and leaves it to you to load a new one.

- **Typing in the search box no longer stalls between letters.** Every keystroke rebuilt the whole tab and then re-measured every line on it against your theme, which on the **Prompt** tab is about forty milliseconds a character. Typing one word cost nearly three tenths of a second of that, and it was felt as the field being slow to take letters.

  It waits for a gap in the typing now and repaints once. Measured on the **Prompt** tab: 42ms a keystroke before, under 1ms after.

- **The tabs share the row evenly.** Each was sized to its own label, so the gaps between them all differed and the selected one looked cramped next to the wide ones.

- **A clearer line when a refine is already running.** It said "Press it again to stop that one", which was only ever true of the panel's own button, and that button is replaced by **Stop this refine** while one runs. It now says to stop it first or wait for it to finish.

- **Plainer words in the built-in prompts and the panel.** A rewrite is returned rather than handed back, a last line sets up what comes next rather than reaching for it, a voice is short rather than clipped, and a refine finishes rather than lands. **When a refine lands** is **When a refine finishes**.

  Your own prompt is not touched. To take the new wording, load a built-in prompt from the **Prompt** tab.

## 1.11.0

_2026-09-14_

### Added

- **Four more reasoning formats are recognised, so the working inside them is no longer rewritten and saved over the reply.** Harmony, which gpt-oss writes, Gemma 4, Cohere Command A Reasoning, and Seed-OSS.

  None of these could be reached by adding a name under **Extra reasoning tag names**. The three wrappers already recognised are matched by tag name, with the same name closing the block. In all four of these the tag is not what names the reasoning: the word sits in the content, and the block closes on a different token again. Harmony has no closer of its own at all, and runs to the next control token.

  The `final` channel is the reply, not working, and is refined like any other passage. A pattern that had taken it would have deleted the answer rather than the reasoning in front of it.

  Adding a name to the list is still the answer for an unusual tag. These four are matched whatever is in that list, since their tag is not what names the reasoning.

- **The markers that open and close a turn are held back with the reasoning and put back around the rewrite.** A local backend can pass `<|turn>model`, `<turn|>`, `<|im_start|>`, `<|eot_id|>` and the rest through into the message text. They are not prose, and a refiner handed one either drops it or rewords it.

  This holds whether or not **Keep the reply's own reasoning out of the refine** is on. That switch governs the model's working, which is writing of a kind; a turn marker is not, and letting one through would leave the reply framed differently from the one Auto Retry reads.

- **The markers come off the refiner's answer whether or not the reasoning switches are on.** Both switches govern the model's working, which is writing of a kind. A turn marker is not, and one left in the answer is saved into your chat as text.

- **A hard line break in a rewrite survives.** Two spaces before a line break are a hard break in markdown. Tidying up after a marker was removed swept the whole answer, so a break the model meant to write was deleted on a rewrite that had no markers in it at all. The tidy now acts only where something actually came out.

- **A marker with no role after it keeps the first word of the reply.** The markers that name a speaker take the name with them, and a bare one sitting straight in front of the reply was taking the first word of it instead. That word is your writing.

- **A message that is nothing but the model working is refused rather than refined.** It has no prose in it to rewrite, and the refiner used to be handed an empty passage and asked to improve it.

### Fixed

- **A reasoning block sitting behind a turn marker was treated as prose.** The wrapper had to be the very first thing in the message, so one `<|turn>model` in front of it was enough to defeat every wrapper and every name in the list. This is the shape a local backend actually hands over.

- **The refiner's own working, when it answers in one of these formats, no longer reaches the chat.** It was already caught for the three tag-name wrappers and went through for the rest.

Cloud connections are unaffected by all of this. They hand reasoning back in a field of its own, so it never reaches the reply text and there has never been anything to cut off. This is what a local backend needs.

### Removed

- **Everything about prompt caching is gone from the panel and the pages.** The line under the block list that counted blocks below the passage, the paragraphs on the prompts page, and the clause on the cost note.

  The order the built-in prompts use has not changed and neither has anything the extension sends. What is gone is the commentary about it. The prompts page still says what the order is and why the shape of the answer sits at the bottom.

## 1.10.0

_2026-09-14_

### Added

- **The softening check now reads what the model put in, not only what it took out.** Counting the strong words that went missing only sees half of a toned-down rewrite. The other half is the word that arrives in their place: killed becomes unalived, blood becomes crimson liquid, sex becomes intimacy. A list of those pairs comes with the extension and is read on every refine.

  It only counts when **both halves happen**: the soft word is in the rewrite and was not in your reply, and the blunt word it stands in for was in your reply and is gone. Either on its own proves nothing, since a model can write eliminate about a possibility and a refine can drop the word kill while tightening a line. Needing both is what lets ordinary words like defeat, intimacy and shaft sit on the list without it firing all day.

  One confirmed swap is enough, with no fraction to clear. That is what this half is for: a reply whose only killed came back unalived was softened, and the counting check cannot see it, because one word going is below the floor it needs before it will speak.

  A refused rewrite is refused the same way as any other: your reply is left exactly as it was, and **Ask again when a check fails** covers it if you have that on.

- **Swaps of your own to watch**, under the same switch as the words. One per line as `soft => blunt`. An exported find-and-replace list pastes in as it stands, and a line with nothing on the right is skipped, since a deletion has no pair in it.

## 1.9.2

_2026-09-14_

### Fixed

- **Picking a preset loads it now, so a save can no longer land on the wrong one.** Picking one put its name in the name box and left your rules exactly as they were. The panel then showed one preset while the picker named another, and pressing **Update selected** wrote what was on screen over the preset you had just picked. The preset you overwrote was gone, with nothing to get it back from.

  Choosing one in the list loads it straight away. What you are looking at always matches what the picker says, which is what makes **Update selected** safe to press.

  **Put it back** appears beside the buttons after a pick has loaded over something. One press restores exactly what was there before, the picker included, so looking inside a preset cannot cost you work you had not saved yet. It only shows when there is something to put back.

  **Load** is now **Load it again**, since picking already loads. Its job is reloading the preset you are already on, which is how you throw away edits and get the saved wording back.

  The **Model setups** card had the same fault and got the same fix.

### Changed

- **The two chat buttons no longer say where on screen they end up.** Their names and descriptions said one goes at the end of Lumiverse's row of chat buttons and the other sits under the message text. Both were true of a stock Lumiverse, and neither is something this extension decides. A theme is CSS and can put anything anywhere, so anything naming a position is wrong for whoever moved it.

  **A button in the row above the input box** is now **A button in the chat's row of controls**, and **A button under every message** is now **A button on every message**. Both descriptions name the place the app hands the button and stop there.

  Only the wording changed. Neither button has moved, both are still where you left them on the Setup tab, and switching either on or off works exactly as it did.

## 1.9.1

_2026-09-13_

### Fixed

- **The description of A button under every message gave the wrong reason for where it sits.** It said Lumiverse leaves no room inside its own row of message buttons. That is not the reason. The button goes under the message text because that is where it is wanted, and [Settings](docs/settings.md) says the same now.

  Nothing about the button has changed. It is in the same place, and it works the same way.

- **A problem report now says which version each half of the extension is running.** Auto Refine is two files. Lumiverse loads one of them in your browser and runs the other on the server. Updating restarts the server side and leaves an open tab alone, so a tab you had open before the update goes on running the old panel against the new server side, until you reload it.

  The report only ever printed the panel's version and presented it as the version of the whole thing. It now prints both, and says plainly when they do not match. The Log says so too, as soon as it finds out.

  Nothing about this makes the halves agree. It makes a report that would have sent somebody looking in the wrong file say where to look instead.

## 1.9.0

_2026-09-13_

### Changed

- **The note about protected formatting has a block of its own now, called Protected Formatting.** It used to hang off the end of **How to Answer** with no tag around it, which made `{{protect_notes}}` the one macro in the list not sitting in one.

  A tag needs a block rather than a line. A block whose content comes out as nothing but tags is dropped before it is sent, and that macro is empty on any refine where nothing needed protecting, which is most of them. Left inside **How to Answer**, that check could never fire, because that block always has writing in it, so an empty pair would have gone out on every refine.

  It sits last, under **How to Answer**, which is where the macro already was. What a model is sent when protection does fire is what it was sent before, with the tag around it.

  It is not counted by the line about blocks below the passage, for the same reason the block carrying the reply around a part-refine is not: its macro answers to the passage, so it was never going to be reused.

A prompt you already have is yours and is not touched by this. To take the new arrangement, load a built-in prompt from the **Prompt** tab, which overwrites the list you are on. A prompt that still has the macro on the end of **How to Answer** goes on working exactly as it did.

## 1.8.1

_2026-09-13_

### Fixed

- **A setting's description is sized to the setting now, not to the screen.** Tapping a `?` opened a box capped at the width of the screen, and the tab is narrower than the screen on a phone, so the description came out wider than the panel and hung off the side of it. It is capped at the width of its own row instead, which lands it in the same column as the setting with the panel's own gutter either side.

  On a wider screen the row is wider than the cap the descriptions already had, so nothing about them changes there. Auto Retry had the same thing wrong with its descriptions and got the same fix.

## 1.8.0

_2026-09-13_

### Added

- **Two optional buttons that put a refine in the chat itself.** Both are under **Ways to reach it** on the Setup tab and both are off until you ask for them.

  **A button in the row above the input box** goes at the end of Lumiverse's own row of chat controls. One tap refines the latest reply.

  **A button under every message** refines that message. It is the only way to refine a message that is not the latest without selecting all of it first, and it works on your own messages as well as on replies.

  Both sit in the places Lumiverse hands out to extensions, which is the difference between these and the button on every message that came out in an earlier version: that one reached for a row by class name, and those names change whenever Lumiverse rebuilds its CSS. The message one sits under the message text, above the app's own row of buttons rather than among them.

### Changed

- **The built-in prompts no longer assume one character on each side.** A card can hold a cast and a chat can be a group, so **Your Character** is now **Your Characters** holding `<your_characters>`, and **The User's Character** is **The User's Characters** holding `<the_users_characters>`. Four lines inside the prompts said the other person where they meant anybody else in the scene: something hanging between two people, a hand finding the other's, eyes tracing the other's face, speech repeating back what the other person said. All of them read for any number of characters now.
- **One prompt still called you the other author.** That framing came out of the rest of them in 1.6.0 and this line was missed. A passage ending by pointing at what happens next is asking the user to do the work.

A prompt you already have is yours and is not touched by any of this. To take the new wording, load a built-in prompt from the **Prompt** tab, which overwrites the list you are on.

- **It says when the prompts that come with it have changed.** One line on the **Prompt** tab, with a **Got it** that puts it away for good. Your own prompt is never touched by it and it never loads anything for you: loading a built-in prompt writes over the list you are on, so that stays your choice.

  Only for somebody who has loaded one of the eight before. A prompt that is entirely your own is not affected by the built-in ones changing, so there is nothing to tell you. A fresh install is marked as up to date, so this release is quiet for everybody and the first thing it says is about the next change.

- **How to Answer is the last block now, and it goes out as you rather than as the setup.** It used to sit near the top with the rest of the rules. A rule about the shape of an answer is followed most closely when it is the last thing read: put it at the top and the model has the whole prompt between that rule and the answer, and some hand back a rewrite with the tags missing or wrapped around the wrong thing. That rule is also the one you cannot work around, since a rewrite without its tags is dropped rather than saved. The role is **User** because it is your instruction about what you want back rather than part of the setup, and it lands in the same message as the passage.

  It costs a little where prompts are cached, since it used to sit in the run that never changes and now sits under the part that changes every turn. It is a short block and the trade is the point. The line on the Prompt tab that counts blocks below the passage does not count this one, because every built-in prompt puts it there.

- **The tabs are a tray with the one you are on filled in.** They were a row of labels with a line under the selected one, which is the tab strip every panel has, and telling two of them apart at a glance is the one thing a tab strip has to do. The tab you are on is filled and its label is brighter; the rest are plain. Auto Retry marks its tabs the same way, so the two read as a pair. No label changes weight when you pick it, since a label that goes bold gets wider and shifts the row under the finger that just tapped it.

### Fixed

- **Dragging the tabs up and down moved them.** The titles shifted by a pixel and the line under the strip looked like it thickened. A strip that scrolls sideways scrolls up and down too unless it is told not to, and the tabs were pulled a pixel down over that line, so the strip was one pixel taller than it could show. Dragging scrolled that pixel and stacked the selected tab's underline onto the line. The line is gone with the redraw above, and the strip is locked to sideways now.
- **A token count Lumiverse called a guess was reported as a count.** It counts with a real tokeniser where it has one for the model and says so when it has none and fell back to characters over four. That flag was not read, so a guess came back through the panel as an exact figure, and every price on the Model tab is worked out from it. The flag is read now: a guess says roughly, the same as a build with no counter at all.
- **The `{{overused}}` macro never showed what it puts in.** The macro list described it and left you to run a refine to find out what the shape was. It now shows a line the way `{{protect_notes}}` shows its wording, and [How the prompt is built](docs/prompt.md) has the block as it arrives, counts and all.
- **Refining part of a reply was documented inside the floating button's paragraph.** It has three ways in and none of them is the floating button, so it has its own paragraph in [Settings](docs/settings.md) now, and the README says it exists.

## 1.7.0

_2026-09-12_

### Fixed

- **The run-up called you Co-author instead of using your character's name.** Every one of your messages in **See what gets sent** was labelled that way, while the character you are writing with got their name off the card. A job title is not anybody in the story, and a model reading it has to work out who is meant. Both sides are named now, so the run-up reads the way the chat page does. A chat with no persona set, or a build that will not resolve one, falls back to **You** rather than labelling a blank.

### Added

- **Name the speakers in the run-up**, on by default, under **Context**. The run-up goes out as one block of text, so the labels are the only thing separating your lines from the character's in it. A chat whose messages already begin with a name, which is what a group chat looks like, gets that name twice on every line, and switching this off sends the messages as they were written. It also skips the persona lookup, since the label is the only thing that reads it.

## 1.6.0

_2026-09-11_

### Fixed

- **The preview never said which of your two prompts it built.** There are two lists, one for replies and one for your own writing, and the preview builds whichever fits the message it is previewing. Loading a preset writes to one list and leaves the other alone, so loading a reply preset while your own message is the newest one built a preview of the list you had not touched, and it read as a preset that failed to load. It says which list it used now, and that a preset loaded into the other one will not show there. The backend had been working this out and sending it all along; nothing on the panel read it.

### Changed

- **Refining the same words twice is refused everywhere now, not only on the automatic pass.** The setting is **Refine something that has been refined before**, still off by default, and it covers the buttons and your own messages as well. Pressing refine on a reply still holding its refine used to go ahead and buy a second opinion on a passage nobody had changed. Swiping, regenerating, deleting a swipe or editing puts different words behind the message, and those are refined either way. The setting no longer hides itself when the automatic pass is off, because it applies whether or not that is running.
- **The phrase list the prompts cut from is half again as long.** It covered bodies and a few sentence shapes. It now covers feeling handed over in a container, faces and voices running stock business, the room doing the characters' work, pauses named instead of filled, and softened double negatives, alongside what it already had. Named as shapes wherever a shape catches more than an example does.
- **The Speech block knows about register.** Lines that announce themselves before arriving, lines that only grade what was just said, counselling talk from characters who do not counsel for a living, and a soft name dropped in to take the edge off. None of it touches how a character is allowed to sound: somebody who speaks badly goes on speaking badly, and a character who already talks that way keeps it.

- **The prompts that come with it now name you and the user plainly.** They called the person writing with you your co-author, which put a word in the prompt that means nothing to a model and reads as a role it has to work out. The user is the user. Their character is **The User's Character**, yours is **Your Character**, and both are written with they and them throughout, so nothing in the prompt assumes a gender for either.
- **The rhetorical framing is gone.** Two prompts opened on a question asked for effect and four descriptions repeated it. A rule reads as a rule; the standard and the test say what they want outright now.
- **Every block is named after the tag it holds.** The block called The Job holds `<your_job>`, What to Cut holds `<what_to_cut>`, Copy These Exactly holds `<copy_these_exactly>`. Reading the Prompt tab and reading the prompt are now the same thing, rather than two sets of names to hold in your head.
- **The examples inside the prompts use they and them.** They were written with he and she, which reads as an instruction about who is in the scene.

## 1.5.0

_2026-09-11_

### Fixed

- **Refine what I am typing stayed lit on the home screen.** The two buttons beside it grey out when no chat is open, and this one did not, so the odd one out read as the one that still worked. There is no input box out there to read, so pressing it could only ever fail. It greys with the others now and its tooltip says why, and the entry in the floating button's menu comes off on the same terms.

### Changed

- **Every cost figure now says what it is worth.** A line under the price boxes on the Model tab, and only once a price is typed. The tokens are counted from the prompt this extension builds rather than taken from your provider, so their tokeniser may disagree and whatever they wrap around the prompt is missing from the figure. It also prices every token sent at the full rate, so on a model with prompt caching switched on a refine usually costs less than the panel says, sometimes a lot less.

## 1.4.0

_2026-09-11_

### Fixed

- **Exporting only your saved model setups was refused.** The check that decides whether a file would be empty never looked at the setups, so ticking that part on its own and nothing else said there was nothing to put in a file, with your setups sitting right there. It counts what is going in now, setups included. An empty list counts as nothing too, so ticking a part you have never saved anything under says so plainly instead of writing a file holding an empty list and reporting it as exported.

### Added

- **Import takes more than one file at a time.** Two files one after the other reported each on its own, so a pair carrying a preset each said one preset, twice, and left you adding them up. Picked together they are one import with one total. They go on in the order you picked them, so where two files name the same preset, setup or setting the last one is what stands, and a file that cannot be read stops the whole import and names which one rather than taking half.

## 1.3.0

_2026-09-11_

### Fixed

- **Picking a model setup for a prompt that comes with the extension said nothing about where the pick was going.** The eight built-in prompts are fixed, so **Update selected** is greyed out on them and there is nowhere to write the link. **Save as new** does write it, onto your copy, which is the way to do it, but nothing said that: the box took the pick and then switching the preset list put it back to what the next preset carried. The card now says the copy is what keeps it, and only while a built-in prompt is the one selected.
- **A name clash with a built-in prompt counted wrong.** Saving a preset under a name one of them already has is refused, and the refusal said there were two of them. There are eight.

### Added

- **A measurement mark is no longer read as somebody speaking.** Dialogue is left out of the worn phrase counting, and a straight quote after a digit, the kind in `the gap was 6" wide`, opened a line of dialogue that never closed. Every word after it in that reply was thrown away, so a habit sitting past one was never found. Curly quotes are read by their shape, which says which end they are without guessing.
- **A chain stops when the reply is on its way out.** A chain is several model calls with real time between them, so the window for Auto Retry to swipe the reply, or for you to press regenerate, is as many times wider as there are passes. It was only checked before the first pass and after the last, which meant a reply replaced during the second of six bought four more calls that were never going to be saved. Checked between one pass and the next now, and the Log says which pass it stopped after.
- **A chain runs at most six passes.** Each pass is a model call, so a list of fifty lines would have fired fifty calls off one button press, and the bill would have arrived before anybody noticed. Six is past any chain worth building.
- **The Log tab says what each pass of a chain changed.** One before and one after, with three calls somewhere between them, tells you a chain came out worse and never which link did it. **What each pass changed** on the Log tab is a fold holding every pass in order, what each was handed, what it gave back, and how much it changed the length. A chain turned down on its total reports them too, which is when they matter most: the refusal says which limit was hit and this says which pass hit it. Nothing is there for a single pass, which has nothing to break down.
- **The built-in prompts carry the two new macros.** A setting with nowhere to put its answer does nothing, and says nothing about why, so every prompt that comes with the extension now has a block for `{{overused}}` and one for `{{whole_reply}}`. Both are switched on, which costs nothing: each is empty until the thing that fills it happens, and a block whose macros came back empty leaves the prompt, tags and all. A prompt you saved before this version does not gain them, so switching **Find phrases this chat has worn out** on without a block for it now says so instead of filling nothing.
- **A pass can name a prompt that comes with the extension.** Those are a separate list from the presets you save, and a pass naming one resolved to nothing and was skipped without saying why. Yours still wins where the two share a name.
- **A refine can make several passes instead of one.** **How many passes a refine makes** on the Limits tab, set to one pass by default, which is what every refine did before this. Several walks a list of your saved presets in order, each pass handed what the one before it wrote, so cutting the filler and fixing the rhythm can be two prompts rather than one doing both. **The passes, in order** takes one preset name per line. A name matching nothing is skipped, and so is a preset with no block carrying `{{message}}`; with no usable line the Prompt tab's own list runs as a single pass. Each pass is judged against what it was given rather than against the reply, since a pass that tightens by a fifth has not shrunk the reply by a fifth, and the end of the chain is judged once more against the reply it started from, because three passes each tightening by a third leave a reply half its length with no single pass doing anything the limits object to. A refusal stops the chain where it happens. It costs one call per pass, and the Log adds them up.
- **This version's settings travel in an export, and so does the button's size.** Six of them did not: the passes, the mode, and the four for worn phrases. An export ran, a file came down, and the settings were not in it, with nothing to say so, which turns restoring a backup on another device into finding out later. **How big it is** for the floating button was missing too, and had been for longer. A check holds every setting to it now, so one added later is either in a part or named as belonging to this screen.
- **The words your story is made of are not counted as a habit.** A place, a job, an institution, a thing the chat is about, all repeat because the chat is about them. The phrases are checked against your character card, and against your lorebook where a block was already asking for it, and anything written into the story is left out of the report. Telling somebody their own setting is a habit is telling them to stop writing their story.
- **Eight more shapes in the built-in prompts.** Shapes rather than particular phrases, because a model reaches for these whatever the scene is: a thing said by what it is not and then corrected, the same thing twice with the weaker one kept, three of anything in a row, a pause named instead of filled, actions strung onto one sentence with "as", a sound that escapes somebody, a sentence opened on a participle, and an action given and then graded.
- **It can find the phrases this chat has worn out.** A refine judges one reply at a time, so a phrase reads as fine every time it is met; used in eleven of the last fifteen replies it is the model's crutch, and nobody notices because nobody reads fifteen replies at once. **Find phrases this chat has worn out** on the Limits tab fills `{{overused}}` with them, so a block can name them and ask for something else. Counted across replies rather than within one, because five times in a single reply is that reply's choice. Dialogue is left out, since a character repeating a phrase is characterisation. So are your own messages, the character's name, runs of only common words, and anything in backticks. The longest phrase wins, so a finding is not also reported as the shorter runs inside it. **Phrases to leave alone** takes one per line for the times repetition is the point. It reads replies the refine already has, so it costs no extra call, and it sends it off.
- **Refine part of a reply instead of all of it.** Select a sentence or a paragraph in a message, then press **Refine the part I selected**. It appears on the panel beside the other buttons, in the floating button's menu, and in the chat input's Extras menu on the same terms as the other rows there. The panel one is there whatever the button and the row are doing, so nobody who switched those off is shut out of it. The rest of the reply is left exactly as it was. The model is given the part and nothing else, so every check on the answer, its length included, is measured against the part rather than the reply around it. The entry is only in the menu while something is selected. Selecting inside your own message works too and uses your own prompt list, the same as pressing the button on one does, and the greeting is refused the same as ever. A selection that grew out of italics takes the whole italic run with it, since leaving one marker behind would turn the rest of the reply into emphasis.
- **The length limits leave room on a short passage.** **How much longer** and **how much shorter** are shares of what was sent, and a share of one sentence is a handful of characters: sixty per cent of "it was fine" is six, which turns down every rewrite of it that is not nearly the same length. Each limit now gives the share or a floor in characters, whichever is larger. A reply long enough for the share to matter is judged by the share exactly as before. The two floors are not the same number, because a short passage coming back longer is the rewrite working and one coming back much shorter is a model answering with a stub.
- **`{{whole_reply}}` shows the model what surrounds the part it was given.** The reply as it stands, with the part being rewritten wrapped in `<<<` and `>>>`, for a prompt that wants a fragment read in context. Empty on an ordinary refine, which leaves the block carrying it out of the prompt rather than sending an empty heading.

## 1.2.0

_2026-09-10_

### Fixed

- **Sizing the floating button sent it back to the corner it started in.** The host sets the size when the button is made, so changing it builds the button again, and the rebuild was handed a fixed spot near the top of the screen rather than wherever you had dragged it to. Where you put it is written down now and the rebuild starts from there. It grows around its middle as well, since the position a host is given is a top-left corner: carrying that across unchanged pins the corner and lets the button grow down and to the right, which against an edge is a jump to somewhere you never put it. [Auto Retry](https://github.com/starlitcode/Lumiverse-Auto-Retry) already worked this way, so the two behave the same. Where it sits is kept in the browser rather than in your settings, so it does not travel in an export or land on your phone at a position a phone has no room for.
- **The automatic pass could not refine anything on some installs.** A reply finishing said which chat and which message and never which account, and a Lumiverse installed for an operator refuses a model call that names no account. So every automatic refine came back saying Lumiverse could not tell whose it was, while the buttons carried on working, because pressing one says who pressed it. The panel hands its account over with its settings, and that is the account the automatic pass now runs as. On a server with several accounts it is whichever panel loaded last, the same as the rules themselves.
- **The spinner could turn forever after a run through a chat.** A reply refined inside a run sends its own ending and that stopped the spinner; a reply the run left alone sends nothing, because the run counts those itself. So a run whose last reply was left alone finished with the panel still marked busy, and with **Give up waiting after** set to 0 there was no timer left to notice. The run's own ending stops it now, and with the wait switched off the panel stops waiting at an hour rather than never, which is the ceiling every other setting is already held to.
- **The panel said the backend was not answering when it was.** It turns the spinner on the moment a reply lands rather than waiting for this extension to answer, so a turn does not open with a second of nothing. Several ways out of the automatic pass returned in silence, and five seconds later the panel reported a backend that was not installed and counted a refine it never made as turned down. Every way out says so now, in words, on the Log tab.
- **A run through a chat gave up partway through itself.** The timer that gives up on a refine was armed once at the first reply and covered the whole run, so on a long chat it came due while the run was working.
- **Put it back could write over the wrong swipe.** The text from before a refine is the only copy there is, and it went back into the message whatever the message was holding by then. A reply swiped, regenerated or edited since the refine keeps what it has, the row comes off the tab, and it says why.
- **Ask before saving a refine could save a rewrite of the swipe before it.** The card waits for a yes, and a reply swiped while it waited had the rewrite of the old one written over the new one. The save is refused if the reply moved while the card was up.
- **Not every row under a switch faded in and out.** Four were built only while their switch was on, so switching it rebuilt the panel around them and they arrived between two frames: the sound rows, **Refine what I am typing**, and the two folds under the protection switches. All four are built either way and hidden now, the same as every other row.
- **A line on the Log tab rebuilt the whole panel.** A run through a chat writes a line per reply, which was forty rebuilds of every box on the tab, back to back, while somebody watched it.
- **A block switched off still cost what it would have cost switched on.** The lorebook, the chat memory and the run-up are each a call to Lumiverse, and all three were made on every refine whether or not any block being sent asked for them. A macro nobody is going to see is a call nobody has to make.

### Added

- **`{{memories}}` puts what Lumiverse remembers of the chat into the prompt.** Everything from further back than the run-up, and Lumiverse's own answer rather than one assembled here: the same pieces it would have retrieved, as many as your chat memory settings ask for, written out with your own header and chunk templates. Every built-in prompt carries it in a block of its own, **What has happened before now**, under the setting and above the pages before this one. That block starts switched off, because its size is decided by your chat memory settings rather than by anything on the Prompt tab and it would be sent on every refine; switch it on there when you want it. It needs no new permission, and a chat with memory switched off or nothing vectorised sends no block at all rather than an empty heading.
- **Refine a reply that has been refined before**, off by default, on the Limits tab under the automatic pass. Off means "still holding the refine it was given", not "has an id this pass has seen": a reply you swiped, regenerated, deleted a swipe from or edited is holding different words, and different words are a reply the pass has never seen, so it is refined either way. That is what keeps the pass working alongside [Auto Retry](https://github.com/starlitcode/Lumiverse-Auto-Retry), since every reply Auto Retry re-rolls is a new one. On, the same reply goes through again even when nothing about it changed. Either way a build that announces one generation twice buys one refine, not two, and pressing a button always refines what you pressed it on.
- **It waits out a provider that will not take the call.** A "too many requests", a shared key at its limit, or a local server answering 503 while it loads a model is not a bad answer, because there was no answer and nothing was spent. **Wait out a provider that will not take the call** on the Limits tab does this twice by default, waiting longer each time, and where the provider says how long to wait, that is what it waits. It never waits on a wrong key or a prompt too long for the model, since waiting cannot fix either. The status line counts down while it waits, and Stop ends the wait as well as the call.
- **Add the refine as a reroll instead of writing over the reply**, on the Limits tab, off by default. On, the rewrite goes in beside the reply as another reroll and the original stays one swipe back. That is Lumiverse's own way back, it survives a reload, and the arrows for it are already on the message, where **Put it back** is held in memory and gone the moment you close the tab. **Put it back** then takes that reroll off again rather than writing the original over it, and it refuses where you have rolled another one since. A build that does not give a message rerolls writes over the reply as before.
- **A ceiling on how many times the automatic pass will refine one reply**, at twelve. A swipe, a regenerate or an edit is new writing and is refined, which is the point above. What that cannot see is a loop: another extension rewriting what this one wrote, or a build announcing the same generation under new ids. Twelve is far past anything anyone does by hand, so it is a stop on something going wrong rather than a budget. Pressing the button still refines it, and the Log says why the pass stopped.
- **Blocks fold shut on the Prompt tab.** A caret on each one, and **Fold them all** for the list. A folded block is its switch and its name, so a prompt of twenty is a list you can read at once rather than twenty text boxes to scroll past. Which are folded is remembered, and it is not part of a preset: it is where you were looking, not part of the request.
- **A row that hangs off a switch closes rather than vanishing.** The same movement a deleted block makes, so the panel below travels instead of jumping by the row's whole height in one frame.

### Changed

- **Nothing is refined while the next reply is being written.** Auto Retry swiping a refusal, or you pressing regenerate, means the reply in front of it is on its way out. The pass stands down before the call where it can, and refuses to save where the call was already out.
- **The floating button keeps its own mark.** A refine that had landed turned it into an arrow, which put a control over your chat you had not asked for and took the extension's mark off the screen for as long as there was something to put back. The way back is a line in the menu behind the button instead, always, and **One tap puts the last refine back** is gone with the state it switched between.
- **Refining the latest reply has left that menu**, because a tap on the button does it. Two ways to one thing, one of them behind a hold, is one more than anybody needs.

## 1.1.0

_2026-09-06_

### Added

- **The request is measured in tokens, and priced.** **Show me the request** gives the whole request in tokens, with Lumiverse's own tokeniser where it will answer and an estimate where it will not, saying which of the two you are reading. **Where the tokens go** breaks it down block by block, largest first with a share each, which counting the messages cannot do because blocks with the same role are joined before they are sent. Put your provider's input and output prices on the Model tab and the same card says what that request would cost and what a hundred replies at that size come to. Both prices start at 0, and with both at 0 no cost is shown anywhere. Fill in one and the line says which half of the sum it is pricing. Price lists write these as `$0.075/M`, so type the number on its own or paste the line and the number is taken out of it, and a saved model setup carries the prices of the model it names. No currency is shown either, because nothing here knows which one you are billed in.
- **Every built-in prompt holds the point of view and the strength of what it is given.** A reply written in first person, present tense, from inside one head could come back in polished third with another character's thoughts in it, and nothing about that read as an error. And a model rewriting roleplay softens it: the heat comes down, the violence goes vague, the crude word turns polite. Limits could already refuse a rewrite that sanitised a reply, but that is a call already paid for, so the prompts ask first. Both are said in what the passage keeps rather than in what the model is forbidden.
- **There is one prompt, and it is the one on the Prompt tab.** A second copy sat in the backend to fall back on before your settings had crossed over, and it had drifted: four blocks fewer, including the one that keeps markup away from the model, and an opening paragraph the panel stopped using. A refine that ran on it would have reported your prompt and sent that. The copy is gone, and a refine asked for in the moment before your settings arrive says so and sends nothing.
- **The built-in prompts stop using the phrases they exist to cut.** Two opened on "you are the second pair of eyes", which is both the stock way to open a prompt and a stock phrase. "A beat" and "a fragment lands" used writing-workshop metaphors in the same breath as asking for plain writing, and "their hand" for the way somebody writes said nothing to anyone reading the block list. A check holds them to it, built from what was actually taken out rather than a guess at what might creep in.
- **The preset menu offers the four for the list you are editing.** All eight were listed under two headings, so loading a **For your messages** one while editing replies changed the prompt you were not looking at and left the one in front of you alone, with nothing saying so. Your own are still offered whichever list you are on, since one of yours can carry either.
- **Importing the same file twice changes nothing.** Presets and model setups went in beside what you had, so a second import handed you the same preset again under a "(copy)" name, and with a cap on the list it pushed your own oldest one out to make room for it. They go by name now, the way [Auto Retry](https://github.com/starlitcode/Lumiverse-Auto-Retry)'s always have: one that matches a name you have replaces it, one that matches it exactly is left alone, and the panel says which of the three happened to each.
- **The macro list puts each meaning behind a ?**, the same one every other row on the panel uses. They were spelled out under each macro, which made a list you scan for a half remembered name into ten paragraphs; `{{protect_notes}}` alone ran longer than the other nine together.
- **The Prompt tab says when your block order costs you prompt caching.** Reuse runs from the front of a prompt up to the first thing that changed, so a block below the passage or the run-up is sent as new on every refine. The line counts how many are down there. It is a line rather than a warning because a rule below the passage is followed more closely, so it is a trade you may want to make.
- **The Log tab says what the last refine really used**, input and output, and what it cost. It covers the whole refine rather than its last ask, so a rewrite that was dropped is on that line and so is every extra try **Ask again when a check fails** made: the calls were paid for whether or not anything was saved.

## 1.0.0

_2026-09-05_

First release.

---

[Back to the README](README.md)
