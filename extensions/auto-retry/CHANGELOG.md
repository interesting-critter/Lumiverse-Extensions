# Changelog

Every released version of Auto Retry, newest first.

**Two releases need a reinstall rather than an update: [4.0.0](#400) and [3.0.0](#300).** Both are marked below. Your settings and presets are stored separately and are not touched by reinstalling.

Versions follow [Semantic Versioning](https://semver.org). A new major version means a reinstall rather than an update, a minor version adds something, and a patch version only fixes things.

---

## 5.10.4

_2026-09-26_

### Fixed

- **Update selected said it updated a preset when nothing had changed.** When the preset picked already holds the settings on the panel, the line under the preset buttons now says so, and nothing is saved. It works the same way as a name already in use.
- **Put it back said it put something back when nothing had changed.** When what is on the panel is already what was there before the load, the line now says so.

---

## 5.10.3

_2026-09-26_

### Fixed

- **This chat could say no chat is open while you were in one.** When the panel asked which chat is open, and a reply arrived in a chat before the answer came back, an answer of "none" put the **This chat** row back on "No chat is open". It stayed that way until the panel asked again. An answer that arrives after a chat event is now not used to decide which chat is open.

---

## 5.10.2

_2026-09-26_

### Changed

- **The note sets stop reading their "seen" mark from its old storage name.** The name changed in 5.7.0. A browser that has not run 5.7.0 or later since then does not get the line saying the note sets changed, once. Nothing else is different.

---

## 5.10.1

_2026-09-26_

### Fixed

- **The note buttons were not locked on a built-in note set.** With a built-in set picked, the notes are read-only, but the **+** and **−** buttons stayed bright and still worked. So a note could be added to or removed from a set that cannot be saved. Both buttons are now greyed out and switched off while a built-in set is picked. Save the set under your own name to change it.

---

## 5.10.0

_2026-09-25_

### Added

- **A separate switch for broken thinking.** **Its thinking was one character over and over** retries when the thinking is only something like `!!!!!!!!`, even when the reply after it looks fine. It is on by default. Turn it off to keep a reply that looks fine. See [One character over and over](docs/detection.md#one-character-over-and-over).

### Changed

- **It was one character over and over now checks the reply only.** In 5.9.0 it checked the thinking and the reply together. The thinking has its own switch now. If you had turned it off, the new switch starts off too, so nothing is turned back on.

---

## 5.9.0

_2026-09-25_

### Added

- **A retry for a reply that is one character over and over.** Some free or busy providers send broken output, like `!!!!!!!!`, in the thinking, the reply, or both. **It was one character over and over**, on by default, retries it. The thinking is checked on its own, so a reply that looks normal is still retried when the thinking before it was only `!!!!!!!!`. A divider line or a row of stars in a normal reply is left alone. See [One character over and over](docs/detection.md#one-character-over-and-over).

### Fixed

- **A broken colour tag could make a finished reply look cut off.** Models sometimes write a tag with its closing quotation mark missing, like `<font color="#c0a060>`. That lone quotation mark counted as dialogue opened and never closed, so the reply was retried. Tags like this are now left out of the count. See [Code and trackers](docs/detection.md#code-and-trackers).
- **A broken colour tag could hide a refusal.** The same lone quotation mark made a refusal after it look like a character speaking. The refusal check now reads the reply without its tags. See [Quotation marks](docs/detection.md#quotation-marks).
- **HTML tags counted toward the refusal length limit.** A short refusal wrapped in a lot of markup could pass the 2000-character limit and be left alone. The length now counts only the text.
- **A status bar on one line made a finished reply look cut off.** A tracker such as `📍 The pier | 🕘 9:40 PM | 🌧 Rain` has no full stop at the end, so the reply was retried. A last line with two or more `|` now counts as a finished ending. See [Code and trackers](docs/detection.md#code-and-trackers).

---

## 5.8.3

_2026-09-25_

### Fixed

- **The docs and one panel hint leaned on the same few sentence shapes.** Lines such as "not X, not Y" and three-item lists are rewritten plainly in the detection, settings, safety and troubleshooting docs and the README. The refusal length hint now reads "Replies longer than this are treated as real writing and left alone." The facts are the same.
- **The docs did not warn about ending on an Assistant note.** A note sent as **Assistant** at the very end of a request is a prefill, and many newer models no longer accept one. The **Role** explainer now says so. See [Each note's own settings](docs/detection.md#each-notes-own-settings).

---

## 5.8.2

_2026-09-24_

### Fixed

- **Two saves close together could leave the older one in your account.** Saves of your settings and presets could finish out of order, so the next load from your account brought back the older copy. They are now written one at a time, in the order you made them.
- **The panel could miss your account's settings for a whole visit.** If it opened before the extension's server side had started, its request for your account's copy got no answer, and it was not asked again. The panel then ran on this browser's copy, which can be older, and the next save wrote that older copy to your account. Now the panel asks again as soon as the server side is ready. See [Saving](docs/settings.md#saving).

---

## 5.8.1

_2026-09-24_

### Fixed

- **An old tab could save older settings over newer ones.** Settings were read from your account only when the page first loaded. A tab left open on your phone showed the settings it had then, and pressing **Save** there put them back on every device. Now opening the settings, or coming back to a tab after more than 30 seconds, reads your settings and presets from your account first.

---

## 5.8.0

_2026-09-24_

### Added

- **Pop-ups come up and go down smoothly.**
  - The retry message and every other message at the bottom of the screen rise into place as they fade in, and sink a little as they fade out.
  - A new message in a box that is already up gives a small pulse, so you can see it changed. A countdown does not.
  - The reset list, the Expand editor and the warning before the crisis-support setting grow to their size as they come up, over a dimmed background that fades in.
  - If your device is set to reduce motion, they only fade.

### Changed

- **Sections in the settings open and close at once.** Refusal tuning, Buttons it clicks, Debug info, Import / export and the rest have no animation now. Everything else keeps its animation.

### Fixed

- **A removed retry note stuck and then jumped shut.** The space it left now starts closing straight away.

---

## 5.7.1

_2026-09-23_

### Fixed

- **"No retry control found" while the button is on screen.** Three causes, all fixed:
  - A button list saved in an older version could match nothing after Lumiverse changed its buttons. The built-in list is now always tried after yours.
  - The built-in list now starts with the mark Lumiverse puts on its own Regenerate button, so it keeps working if the button's title or language changes.
  - A retry looked for the button only once, sometimes a moment before Lumiverse showed it. It now keeps looking for a couple of seconds.

- **A reply that was only thinking is retried when the thinking tag was opened in the prompt.** Some presets start the reply inside the thinking tag, so the reply has a closing tag but no opening one. That thinking is now recognised, so a reply that never got past it is retried. A reply that did reach its answer is judged on the answer alone.
  - One case is still out of reach: thinking that started in the prompt and was cut off before its closing tag. The reply then has no tag at all to go by.

- **Impersonate is no longer retried as a cut-off reply.** Impersonate writes your own turn into the input box, and it could look cut off and be retried over the top of what it wrote. The generation after you press **Impersonate** is now left alone.

- **A retry note is kept per chat.** On an install shared by several accounts, a note set up in one chat could replace another chat's note, so that retry went out without it.

- **The privacy page counts the permissions correctly.** It said six, four of them privileged. There are five, all privileged.

- **On-screen panel tab names are no longer cut short.** **Replaced** showed as "Repl…" even in a computer's sidebar, and **Prompt** was cut short on a phone. Each tab now starts at the width of its name. On a small phone, **Copy** and **Clear** move to the line below together.

- **Buttons it clicks says to press and hold.** Its intro said to press **Pick it for me** and then click the real button. A short press on the real button just works as normal. Picking needs a press and hold.

- **A long line in the Log wraps.** The settings line the Log prints when it starts has no spaces, and it ran off the right edge of the panel.

### Changed

- **Only a switch flips a switch.** Pressing a setting's name no longer changes it. Only the box does. This matters most in the reset list, where one line deletes your saved presets.

- **The docs are written in plain language:** shorter sentences, lists and numbered steps. Some wrong details were corrected on the way. For example, the settings page said a long prompt is trimmed on the Prompt tab. Every character is shown.

- **Shorter descriptions.** The longest descriptions behind **?** are cut to a sentence or two. The detail is in the docs.

- **Panel messages are written out in full**, such as "Could not save" instead of "Couldn't save". The preset name box says "A name for this preset".

- **The README says exactly what Auto Retry does about refusals involving age.** It never judges or changes your story. A retry asks the same model again, up to the retry limit. It recognises a refusal about a character's age because models sometimes misread an adult character as a minor, but it cannot tell a mistaken refusal from a correct one, so any refusal it recognises is retried the same way. The retry notes, off by default, mention no age and no subject.

- **The comments and docs on the refusal checks say what the words about age are for:** a model that misreads an adult character as a minor. What is recognised, and what is done about it, has not changed.

- **An inaccurate comment is gone.** It said a retry changes what a model will write no more than clicking twice does. A new attempt can come back different, which is why this extension exists.

---

## 5.7.0

_2026-09-18_

### Added

- **A ring fills around the floating button while you hold it.** Holding the button opens its menu, and nothing on screen said a hold was under way, so the half second before the menu appeared read as a tap that did nothing.

  The ring starts at the top and closes a moment before the menu opens. Letting go early wipes it back. A press still dips the button a little, so a tap answers whether or not it changed anything: a dip on its own is a tap, a dip with the ring running is a hold.

  Auto Refine's floating button draws the same ring the same way, at the same length, so the two behave alike.

  Somebody who has asked their device for less movement gets the menu on the same hold with no ring drawn.

### Changed

- **The four tabs on the floating panel share the row evenly.** They wrapped onto a second line on a narrow panel, and each was sized to its own label, so the gaps between them all differed and the selected one read as cramped next to the wide ones. Auto Refine's tab strip is the same now.

- **No contractions in the panel's own words.** Five descriptions still had them: the backoff factor, the randomness switch, the cut-off switch, the box of phrases never to count, and the section on the buttons it clicks. The prose here does not use them, so that somebody reading it in a second language or with a reading difficulty is not working out what a shortened word stands for on top of everything else.

  The settings page is caught up at the same time. **Where that panel goes** was missing from it entirely, and the row for the panel itself still said it has two tabs. It has four.

- **The picker heading Ships with it is now Comes with it**, and the sets that come with the extension are called built-in sets. The key holding which of them you were last shown is `lv-auto-retry:built-in-seen:v1` now, and it reads the old name across once and drops it. Without it, the line saying the sets changed would not show.

### Fixed

- **A tap on the floating button opened its menu.** Reported while trying the button. The end of a press was watched on the button itself, and Lumiverse captures the pointer so the button can be dragged. Once it has, the release is delivered to whatever the pointer was captured on and never reaches a listener sitting on the button, so the timer that opens the menu on a hold kept running after the finger had already lifted. Every tap became a hold. The end of a press is watched on the page now, so a tap ends the press wherever the release lands.

  The drift a finger is allowed during a hold went from eight pixels to ten at the same time. A thumb resting on glass moves further than eight, and each of those was a hold that did nothing.

- **A reply is given four minutes to start, not three.** The watchdog that catches a generation which began and then produced nothing was set to three minutes, which covers a local model loading weights and a long prompt being read.

  It did not cover a reasoning model on an endpoint that sends nothing at all until the answer starts. A model that streams its thinking clears the watchdog on its first thinking token and was never at risk; one that keeps its thinking to itself looks identical to a dead generation, and a hard question can hold it past three minutes. The reply was then thrown away and asked for again, which is the worst of both, since the second attempt thinks just as long.

  If you were still on three minutes, the panel says so at the top and offers to move you. If you had set your own number, it says nothing, because nothing of yours changed. **Take it** moves you and **Keep mine** leaves it, and either one puts the line away for good.

  Auto Refine carries the same kind of line for the same reason.

- **A note set that comes with the extension cannot be typed into.** Picking one locks its notes: the text, the role, the try it starts on, the remove button and the button that adds another. A line above them says why and says what to do instead.

  Those sets cannot be written over, so editing them was typing into something the panel was about to refuse to save. To change one, put a name in the box under **Saved presets** and press **Save as new**. The copy is yours and opens for editing straight away.

  A fresh install is not affected, since the picker starts on nothing. The text stays selectable while locked, so a line can still be copied out of a set you want to borrow wording from.

  Auto Refine locks its built-in prompts the same way, and both say the same thing about it.

- **A note set no longer claims to be loaded after you have changed it.** Loading one put its name in the picker and nothing cleared it, so editing a note afterwards left the picker naming a set the panel no longer held.

  The line under the picker says so once the two part company, and says where to keep the change: a set that comes with the extension cannot be written over, so it goes under a name of your own with **Save as new**.

  The notes stay editable while one is picked. Loading a set and changing it is how you are meant to start.

- **The mark on the floating button is legible on a light theme.** It was measured at 2.49 against a floor of 3, because the sweep that checks what is readable walked text and never looked at a drawing.


## 5.6.0

_2026-09-14_

### Added

- **Three more reasoning formats are recognised by the refusal, empty and cut-off checks.** Gemma 4, Cohere Command A Reasoning, and Seed-OSS. Harmony was already covered.

  None of these could be reached by adding a name under **Extra thinking tag names**. The three wrappers matched by tag name close on the name that opened them; in these the reasoning word sits in the content and the closer is a different token again.

  An opener with nothing closing it now reads as cut off for these as well, which is what a reply that stopped inside the model's thinking looks like.

- **A nudge**, a new note set and the gentlest of them, for a model that is not usually fussy. It says almost nothing, which is the point.

- **Firmer with every try**, a new note set for a model that keeps declining. It opens as lightly as **A nudge**, then sends a plainer note from try 4 and a plainer one again from try 6. It replaces **Stay in the scene, firmer after a few tries**, which only had two steps and started halfway up.

### Changed

- **The note sets that come with it are written at the refusal now, and they run from the gentlest to the most direct.** A note only ever goes out after a reply was read as a refusal, so its one job is getting the next reply written rather than declined. The old five were all pitched the same way and none of them named what had just happened.

  How plainly a note asks is what you pick by. A model that declined once by accident does not need arguing with, and one that has declined the same scene four times does need telling.

  All of them ask rather than order, the firmest included. A note that scolds gets a more careful reply back rather than a braver one, which is the opposite of what you switched notes on for.

- **The acknowledgement notes are gone.** Every set used to answer its own note in the assistant's voice. That leaves the request ending on a turn the model has not written, which is a prefill, and providers are dropping support for it. A set built that way works until the connection behind it stops allowing it, and then it fails for a reason nobody reading the note would guess. Every built-in note is a user note now. Your own notes can still use any role the panel offers.

- **Turn and role markers are removed before the checks run.** Gemma's `<|turn>model` and `<turn|>`, ChatML's `<|im_start|>` and `<|im_end|>`, Llama's header block, and Cohere's turn tokens. They are not reasoning, but until they were gone they counted towards the length checks and sat in the middle of the phrases a refusal is matched on.

  This happens whether or not **Ignore the thinking / reasoning** is on. That option decides whether a refusal written inside the working counts, which is a question about the working. A turn marker is not the working.

### Fixed

- **A commentary channel closed by a tool call is no longer read as cut off.** A tool call ends on `<|call|>` rather than on `<|end|>`, and that token was missing from the list, so every reply where the model called a tool looked truncated.

- **A marker with no role after it keeps the first word of the reply.** The markers that name a speaker take the name with them, and a bare one sitting straight in front of the reply was taking the first word of it instead.

- **The panel said four sets come with it.** Five did, and six do now.

### Removed

- **Everything about prompt caching is gone from the panel and the pages.** The clause on the note placement hint, the section on the detection page, and the clause on the cost note.

  Nothing about where a note goes has changed. The detection page still says where each placement lands the note and which one goes after anything your build appends.

Cloud connections are unaffected by the reasoning half of this. They hand reasoning back in a field of its own, so it never reaches the reply text and there has never been anything to strip. Your own notes and your own saved sets are untouched, and the line above the picker tells you the built-in sets have moved.

## 5.5.3

_2026-09-14_

### Fixed

- **Picking a note set loads it now, so a save can no longer land on the wrong one.** Picking one only lit the buttons up and left your notes exactly as they were. The panel then showed one set while the picker named another, and pressing **Update selected** wrote what was in the boxes over the set you had just picked. The set you overwrote was gone, with nothing to get it back from.

  Choosing one in the list loads it straight away. What you are looking at always matches what the picker says, which is what makes **Update selected** safe to press.

  **Put it back** appears beside the picker after a pick has loaded over something. One press restores exactly what was there before, the picker included, so looking inside a set cannot cost you wording you had not saved yet. It only shows when there is something to put back.

  **Load** is now **Load it again**, since picking already loads. Its job is reloading the set you are already on, which is how you throw away edits and get the saved wording back.

  [Auto Refine](https://github.com/starlitcode/Lumiverse-Auto-Refine) had the same fault in its presets and its model setups, and got the same fix in both.

## 5.5.2

_2026-09-13_

### Fixed

- **A debug report now says which version each half of the extension is running.** Auto Retry is two files. Lumiverse loads one of them in your browser and runs the other on the server. Updating restarts the server side and leaves an open tab alone, so a tab you had open before the update goes on running the old panel against the new server side, until you reload it.

  The report only ever printed the panel's version and presented it as the version of the whole thing. It now prints both, and says plainly when they do not match. The on-screen log says so too, as soon as it finds out.

  Nothing about this makes the halves agree. It makes a report that would have sent somebody looking in the wrong file say where to look instead. [Auto Refine](https://github.com/starlitcode/Lumiverse-Auto-Refine) got the same thing.

## 5.5.1

_2026-09-13_

### Fixed

- **A setting's description is sized to the setting now, not to the screen.** Tapping a `?` opened a box capped at the width of the screen, and the settings panel is narrower than the screen on a phone, so the description came out wider than the panel and hung off the side of it. It is capped at the width of its own row instead, which lands it in the same column as the setting with the panel's own gutter either side.

  On a wider screen the row is wider than the cap the descriptions already had, so nothing about them changes there. Auto Refine had the same thing wrong with its descriptions and got the same fix.

- **The pop-up at the bottom of the screen is the size of what is written in it.** Any message too long for one line filled the box out to its widest allowed width, which on a phone is almost the whole screen, and the wrap then left the last line sitting a long way short of the right edge. A message ending in one stranded word was the usual result.

  The lines are evened out first and the box is then fitted to the longest of them. A short message already shrank to fit and is unchanged, and the widest a message can get is what it was before. This is every message it puts there, including the ones about a refusal, an error, and giving up.

- **The countdown no longer moves its own Cancel button.** The box has always been the width of the message in it, and the countdown rewrites that message as the seconds run down. Going from two digits to one made the message shorter, so the box pulled in and took Cancel with it, which is a button moving sideways while somebody is reaching for it. A message being rewritten in place holds the width it opened at now. It can still widen for a longer message and will not pull in.

  Auto Refine hands its short messages to Lumiverse and they are drawn by the app rather than by the extension, so there is nothing matching to change there.

## 5.5.0

_2026-09-13_

### Fixed

- **The way back out of find and replace now has a date on it.** The card and the **Download my old word swaps** line under Import / export are the last places that hand your old rules and presets over as a file, and both now say they go on **15 October 2026**. That is just under six weeks after 5.0.0 retired the feature.

  Your rules are not wiped that day. Nothing in the panel will offer them to you after it, which is the part worth knowing, so both places say that rather than leading with the reassurance and leaving you to work out why there is a deadline at all.

  A door that shuts without warning is worse than one that stays open too long, which is why the date is on the card rather than in a release note somebody has to have read.

- **A token count Lumiverse called a guess was reported as a count.** It counts with a real tokeniser where it has one for the model and says so when it has none and fell back to characters over four. That flag was not read, so the Prompt tab showed a guess as an exact figure instead of saying roughly over it. The flag is read now.
- **A built-in note assumed one character.** **Write them as written** said the sheet was the authority on who this person is, which reads wrong in a group chat or on a card holding a cast. It says the sheet is the authority here, and the rest of the note is unchanged. Notes you saved yourself are not touched.
- **The note sets page said four sets and listed five.** It says five.

### Added

- **It says when the note sets that come with it have changed.** One line above the preset picker under **Refusal tuning**, with a **Got it** that puts it away for good. Your own notes are never touched by it and it never loads anything for you: loading a set writes over the notes you have, so that stays your call.

  Only where notes are on, since the sets do nothing for anybody else. A browser that has never loaded one of the sets is told nothing and marked as up to date, so this release is quiet for everybody and the first thing it says is about the next change.

- **Your own hard failures**, a box under **Skip hard failures**. The list of errors that will not fix themselves is built in, and no built-in list holds every provider's wording, so an error yours returns that is genuinely permanent was retried to the cap every time. Paste the wording, one per line, and it stops being retried. Counted alongside the built-in list, shown only while the skip is on, and a phrase that is also in **Your own refusal phrases** is retried as a refusal instead.

  The error rows now sit under an **Errors** heading, which is what keeps that section readable with a row added to it.

### Changed

- **A phrase too short to mean anything is ignored.** Every phrase box is matched with "does the text contain this", so a stray letter left on a line of its own matched almost everything: in **Your own refusal phrases** that threw away every reply you wrote until the retry cap, and in the other two boxes it switched the check off without a word. A line under three characters is skipped now, which still leaves a provider's own codes like 402 usable. The three hints say so.
- **The note roles are named after the roles they are.** **You** is **User** and **The character** is **Assistant**. The old pair read as friendlier and were worse: a chat can have a cast on it, so the reply role is not one character, and the user role is where your messages go whether you write one person or five. It is also what every other tool that builds a prompt calls them. Notes you already saved are not touched: the names changed, not the roles.
- **The settings now say your own refusal phrases are matched against a provider error too.** They always were, and it is the only way to add an error of your own, so leaving it unsaid meant somebody with a provider error their setup keeps hitting had no way of knowing where to put it. It also overrides **Skip hard failures**: an error the built-in list reads as permanent is retried when one of your phrases is in it. Nothing about how it behaves has changed, only what the panel and [When it retries](docs/detection.md) tell you about it.

## 5.4.1

_2026-09-11_

### Fixed

- **A prompt the Prompt tab never received now says why.** The backend reads every prompt and hands it to the tab waiting for one, and it matches the two by account. Where the generation names an account the tab is not waiting under, or names none while several tabs wait, it could hand it to nobody and said nothing at all, leaving the tab on a message telling you to report a bug with nothing to report. It names which of the two happened now, in the tab and in the Log. Counts only, never anybody's account.

## 5.4.0

_2026-09-11_

### Changed

- **The cost figure now says what it is worth.** A line under it on the Prompt tab, whenever there is a figure to caveat. The tokens are counted here rather than taken from your provider, so their tokeniser may disagree and whatever they wrap around the prompt is missing from the number. It also prices every token sent at the full rate, so on a model with prompt caching switched on a retry usually costs less than the panel says, sometimes a lot less.

## 5.3.0

_2026-09-11_

### Added

- **Import takes more than one file at a time.** Two files one after the other reported each on its own, so a pair carrying a preset each said one preset, twice, and left you adding them up. Picked together they are one import with one total. They go on in the order you picked them, so where two files carry the same setting the last one is what stands, and a file that cannot be read stops the whole import and names which one rather than taking half.

## 5.2.0

_2026-09-11_

### Added

- **A fifth note set, Write it at full strength.** The other four answer a model that steps out of the scene, writes a character against their sheet, or cuts the turn short. This one answers the reply that is not a refusal at all: the scene gets written, but watered down. It is the hardest of them to catch by eye, because what arrives reads fine until you notice what is missing from it.

### Changed

- **Stay in the scene, firmer is now Stay in the scene, firmer after a few tries.** The old name said how strong it is. What it does is open lighter than the plain one and get firmer only if the same thing keeps happening, which is a different offer, and the name should say which one you are picking.

### Fixed

- **A hint had markdown in it.** Hints are drawn as plain text, so the asterisks around a word in the mid-sentence hint were two asterisks on the screen. It says what it means in words now, and a check holds every hint, label and description to it.

## 5.1.0

_2026-09-10_

### Added

- **The panel says what a retry costs, both halves of it.** Put your provider's input and output prices under Basics and the Prompt tab works out what a retry comes to: the prompt from what is on that tab, and the reply reckoned at the size of the last one in that chat, which is the only honest stand-in for one nobody has written yet. Under it, what the retries fired this session come to at that size. The reply is measured with Lumiverse's own tokeniser, and only while the panel is on the Prompt tab with a price set: leave either and nothing is sent anywhere. Both prices start at 0, and with both at 0 the line is not there, and where only one half can be worked out the line says which. Price lists write these as `$5.00/M`, so type the number on its own or paste the line and the number is taken out of it. They ride with **Panel, pop-up and prices** in an export. No currency is shown, because nothing here knows which one you are billed in.
- **At the very end**, a fourth place for a refusal note. **After the last message** stops at the end of the conversation, and some builds append their own instructions behind it; this is the only placement that puts a note after those, which is where a note has to be to answer one of them.
- **Four note sets come with it**, under **Ships with it** in the note preset picker. Load one to see the shape, edit the boxes, and save it under a name of your own. They cannot be renamed, changed or deleted, so they are still there after you have made a mess of one.
- **A wait the provider named is the wait it takes.** A free or shared tier that answers "retry after 23 seconds" has said exactly when the next try can work, and every wait shorter than that spent a try being told the same thing. The number is read out of what came back, in the header's own form and in the wordings providers write into an error body, and it wins over the backoff and over **Longest wait between tries**. An hour is the ceiling, because a provider naming longer than that is naming a daily quota.

### Changed

- **The note box says to keep a note short.** A note is read alongside the whole prompt, and a short one that says one thing gets followed where a paragraph gets averaged in with everything else.
- **A dropdown takes no mark at all now, reached any way.** It still took one from the keyboard, and lifted its border under the pointer as though there were a cursor to put in it. The list opening in front of you is the whole of the feedback, and anything drawn behind it sits over the top of what it would be pointing at. A box you can type in is still marked, which is where the mark belongs. [Auto Refine](https://github.com/starlitcode/Lumiverse-Auto-Refine) does the same, so a dropdown behaves the same in both.

### Fixed

- **How much longer each wait gets was rounded to a whole number.** Every number box on the panel held itself to whole ones, so `1.5` was saved as `2` and every retry after the first waited longer than you asked it to. A box for a setting that is not a whole number now takes one, and asks a phone for a keypad with a decimal point on it.
- **The package description still named find and replace**, which went in 5.0.0.

## 5.0.0

_2026-09-05_

**No reinstall needed.** It is a major because a feature people were using has been taken out, which is what the number is for even when installing does not change.

### Removed

- **Find and replace is gone.** It swapped words in a reply after it arrived. Rewriting a reply well is a different job from retrying one: [Auto Refine](https://github.com/starlitcode/Lumiverse-Auto-Refine) does that job, and this extension is going back to retrying. Its settings, its two Extras buttons, its presets, the backend that did the swapping and the edit watcher all went with it.
- **The `chat_mutation` permission is no longer asked for.** Word swaps were the only thing that used it, so Auto Retry no longer writes to a saved reply at all.

### Added

- **A card in the panel for anyone who had word swaps**, offering a JSON file with their rules and presets in it. The same download stays under Import / export for as long as there is anything to hand back, so putting the card away does not take it with it.
- **What to accept from one** is a list of ticks of its own, so you can take somebody's refusal phrases from their file without their button selectors. **Reset...** asks over the same parts.

### Changed

- **Pick it for me takes a press you hold** rather than any press at all. Everything else on the page keeps working while it is up, which is what lets you press send to make the stop button appear, then hold that button to pick it. Text selection is off while it is up, since the browser starts a selection on a long press too.
- **A setting's description fades in and out**, and the "?" lights its ring up and down with it.
- **A reply that ends inside an unclosed bold run is retried.** Emphasis is counted in pairs now, so an unclosed `**bold**` no longer looks balanced.
- **The panel moves rather than jumps.** Tick boxes fill in, sections open, and a row that hangs off a switch arrives when the switch goes on.
- **The live dot is the same one in both extensions**, so a glance at either says the same thing.
- **The widget's menu is grouped**, with a line between the ways in and the way out.

### Fixed

- **The panel follows a theme change.** Where a theme makes its own text unreadable the panel repaints it, and that repaint is only right for the theme it measured. Moving a phone from dark to light left the panel wearing the wrong ink. The theme is watched three ways, since none of them catches the others, and a change measures again.
- **A press does not repaint the panel twice.** New rows were drawn in the theme's own colours and repaired a frame later, which shows as a flicker. The repair now lands on the first frame they appear on.
- **A setting's own words flip its switch, rather than opening its description.** A label with no `for` names the first thing inside it that can be labelled, and a button is one, so the "?" beside each setting had taken the label off the setting. The control is named outright now.
- **The press that closes a description does only that**, instead of also flipping whatever tick was under your finger.
- **A setting's description stays on the screen of a small phone.** Its width cap is room on the screen while the width itself is in the element's own units, and Lumiverse's UI Scale is a zoom, so the two came apart. The width is measured against however much the host is scaling.
- **A removed note's space closes rather than vanishing**, the same movement [Auto Refine](https://github.com/starlitcode/Lumiverse-Auto-Refine) gives a deleted block.
- **Removing a note asks first.** A note holds writing you typed and there is no undo, so it goes the way a preset does.
- **Deleting a preset does not cost you your place in the panel.** A dialog stops the page behind it scrolling, which loses where you were, so that place is held across it.
- **The panel knows which chat you are in after pressing Update.** The backend answers with the account's most recent chat, or that nobody is in a chat while it is still starting. Neither is taken as final: the address settles it, and the question is asked again until something answers properly.
- **The Prompt tab's "different chat" line has both chat ids behind it.** They live in the debug report, where they say whether a host is naming one chat two ways.
- **Switching the extension off while Pick it for me is up takes the picker with it.** It left its listeners on the page and text selection switched off across it.

## 4.24.8

_2026-08-29_

### Fixed

- **Word swaps stopped working after leaving the tab and coming back.** The rules, the on and off switch, and the list of chats you switched Auto Retry off in all live in the extension's backend, and all three got there over the bridge from the panel. A backend that comes back up, which a tab closed and opened again is enough to cause, comes back knowing none of it: its own read happens before it knows whose settings to read, so it finds nothing and sits at its defaults, and its default is swapping off. Nothing said so, because from your side nothing happened. The panel tells it again the moment it announces itself, and nothing is written to your account over it, since the stored copy was already right.
- **A chat you had switched Auto Retry off in started being swapped again after that same restart.** Same cause, and it went with the same fix.

### Changed

- **Four lines in the panel's log say what happened in fewer words.** The ones about a tab that was asleep, a reply the tab never heard arrive, and a swap another extension made unnecessary. A heading in **How it retries** also stopped describing a setting as a brake and now says what it does.

## 4.24.7

_2026-08-28_

### Changed

- **The two longest sections of the panel are split under headings.** **How it retries** was eleven rows in a row and **Refusal tuning** was eleven, both flat, so everything in them looked equally important and nothing said which rows belonged together. They now read **When it gives up** and **How long it waits between tries**, and **What counts as one**, **Wording you supply** and **How far it looks**, each with a line saying what its rows have in common. Nothing moved section and no setting changed. A check holds the panel to this, so a section cannot drift back into a wall of rows.

## 4.24.6

_2026-08-28_

### Fixed

- **A reply that finished while you were in another tab was called stuck and re-rolled.** Everything the extension knows about a generation arrives over Lumiverse's socket, and a tab in the background can miss those events outright: they are not held and handed over later, they are gone. So the extension sat waiting for a first word that had already come and gone, and the wait ran out on a reply sitting there finished. A watchdog now looks at the page before it acts. The reply on screen when a generation starts is remembered, and if what is on screen has changed, words arrived, whatever the socket did or did not deliver, so it stands down and says so. A generation that really produced nothing is still re-rolled.
- **The same waits could also be judged over time the page spent asleep.** A background tab has its timers held back, and one the browser freezes runs nothing at all and then hands everything over at once. A wait that comes back far later than it asked for now starts again rather than reaching a verdict, and the panel says why.

## 4.24.5

_2026-08-27_

### Changed

- **A swap button ends the wait for other extensions, in that whole chat.** With **Wait for other extensions to finish** on, a swap can be held back for up to five minutes. Pressing **swap words now** or **swap words in every reply** is you saying you are not waiting, so nothing in that chat stays on a timer: the one you swapped already stopped, and now so does every other reply waiting in it, rather than being rewritten minutes later with nothing on screen to say why. The panel writes a line saying how many waits it ended. A chat the host will not hand back stops waiting too, which it did not before. This ends the wait, not the reply: an extension that rewrites your reply afterwards still gets the swap put back on top of it.
- **A swap that another extension made unnecessary says so.** If the other extension's rewrite takes your words out of the reply, the swap drops itself and nothing is written, which it always did: the rules run against the reply as it is when the swap fires, not against the text it had when the wait started. What it did not do was say anything, so a wait you turned on could spend a minute and finish in silence. The panel writes a line for that one case now. A reply that never had a matching word in it is still not reported, since nothing was lost there.

## 4.24.4

_2026-08-27_

### Fixed

- **The panel went on naming a chat after you had gone back to the home screen.** It stayed on that chat and its character until you walked back in or reloaded the tab, so the per-chat row offered to switch Auto Retry off in a chat that was not open, the live line still read **Off in this chat**, and a swap pressed from the floating button's menu could have landed there. Nothing announced the move: `CHAT_SWITCHED` reports it but a build emitting only `CHAT_CHANGED` says nothing, and asking Lumiverse is answered on the server with the most recent chat on the account, which on the home screen is the chat just left. Your browser's address is what tells them apart, since it carries the chat id while you are in one, so that is what is read now. A Lumiverse whose addresses do not carry the id is left exactly as it was.

## 4.24.3

_2026-08-27_

### Fixed

- **The Note presets bar stayed on screen with notes switched off.** The boxes it saves are hidden while **Send a note with a refusal retry** is off, so the bar was left sitting under a heading with nothing to act on. It now goes with them, heading and description included.

### Changed

- **The word swap preset bar is called Word presets.** It was **Presets**, which said nothing about what it carries next to **Note presets** in the section above it.

## 4.24.2

_2026-08-27_

### Changed

- **Where a refusal note goes says what it costs if your provider caches prompts.** Caching works from the front, so a note added at the very start makes the whole chat count as new for that turn, while the default, after the last message, leaves the cached part alone. The option's description says so now, and the [refusal detection](docs/detection.md) page covers it. The word swaps page covered it too, and went with the feature in 5.0.0.

## 4.24.1

_2026-08-27_

### Fixed

- **The Replaced tab could not show a reply from a chat Lumiverse names no chat for.** It was filed under the extension's own name for that case and looked up under a different one, so what it kept could never be found again. Everything keyed by chat now goes through one place, which is also where the five hand-written copies of that rule went.
- **Two things holding reply text had no size limit.** The kept reply was capped and the field feeding it was not, which only moved where the memory went.

### Changed

- **The troubleshooting page covers a reply you wanted that got retried.** Where to find it, and which switch to change so it does not happen again, listed by the reason the Replaced tab gives.

## 4.24.0

_2026-08-26_

### Added

- **The reply a retry threw away is kept, so you can get it back.** A new **Replaced** tab on the on-screen panel holds the last reply a retry replaced in this chat, with what it was replaced for and how long ago. Read it, or press Copy to take it. Retrying by adding a reroll leaves the old reply in the chat to swipe back to, but a reroll is not a safe place to leave it: you can tidy your rerolls away, and an extension whose job is tidying them will, including on the newest message. This copy is the extension's own. It is held in your browser tab and nowhere else, never written down and never sent anywhere, and it goes when you close the tab. One reply per chat for the last eight chats. Turn it off with **Keep the reply a retry replaced**, under How it retries.
- **The panel's header wraps instead of overflowing.** Three tabs plus Copy and Clear fitted the narrowest a panel can be, and a fourth does not. On a narrow sidebar or a phone the header now becomes two rows rather than pushing Clear off the edge with nothing to scroll it back.

## 4.23.0

_2026-08-26_

### Changed

- **A retry now adds a reroll instead of redoing the reply in place.** **Retry by adding a new reroll** is on by default, and is switched on once for anyone who already had settings saved. The two differ in what they cost when a retry was wrong: a regenerate redoes the reply where it stands and on some builds clears the other rerolls with it, so a good reply it should not have touched is gone for good, while a swipe puts the new attempt beside the old one and the reply it was wrong about is still there to swipe back to. If your build has no next / swipe button the regenerate button is still used, and either one falls back to the other. Saving writes every setting, at its default or not, so a copy saved before this version would have pinned the old behaviour and nobody already using Auto Retry would have seen the change. It is turned on once, the first time this version runs, and turning it back off after that sticks. The three button boxes are now listed in the order a retry tries them: next / swipe, then regenerate, then stop, with the dialog-wording switch under them.

### Fixed

- **A retry says in the log when the button it wanted was not there.** With swiping preferred, a build whose next / swipe selector matches nothing retries by regenerating instead, without saying so, which is the one that can take the old reply with it. The log now says so and points at the buttons section, rather than leaving a missing reroll to be worked out.
- **A reply was re-rolled far past your Most tries setting.** Reported by a Discord user. Giving up hands the budget back so your next reply starts fresh. On a build that reports one generation as ended twice, the second ending landed after the giving up and took that fresh budget with it, so a cap of four could re-roll a reply over and over until the failure breaker caught it. One ending now gets one verdict. A second route did the same thing: a retry click on a host slow to announce the generation was written off as having done nothing, and the reply it really did start was then counted as one you asked for, which also hands the tries back. A generation that follows one of our own clicks is treated as ours whether or not the wait for it was given up on.
- **A finished reply was re-rolled when the ending reported it as empty text.** Not every build puts the reply on the ending. One that reports an empty string for a reply that really streamed is contradicting the screen, and it was believed, so whole replies sitting there finished were counted as blank and thrown away. What the extension watched arrive is used instead, since that is what you are looking at.
- **A retry could click the extension's own settings panel instead of your regenerate button.** The button selectors are patterns rather than addresses, and the panel's own description button for **Retry by adding a new reroll** matches the built-in swipe pattern word for word. With the panel open, a retry opened that description and did nothing else, which is worst for the person most likely to hit it: the one watching the log to see what the extension is doing. Nothing the extension puts on the page can be clicked by a retry now.
- **A reply that stopped partway was re-rolled even with cut-off retries switched off.** Reported by a Discord user. **Give up on a reply that froze** watches for words that were appearing and then stopped, and it was stopping and re-rolling the reply whatever else you had turned off. With **It cut off mid-sentence** off, a reply that stopped with real text already in it is now left alone: it is a cut-off reply by another route, and that switch is the one you would go looking for. A generation that produced nothing readable is still re-rolled, which is what the freeze watchdog is for.

## 4.22.1

_2026-08-26_

### Fixed

- **The per-chat switch went on naming a chat you had left.** On the home screen, or anywhere else outside a chat, **Turn off here** stayed live and named the last chat you were in, offering to switch Auto Retry off in a chat you were not looking at. The chat it shows is learned from things happening in a chat, and walking away from one is not a thing happening in a chat, so on a build that says nothing when you leave, nothing corrected it. Opening the settings panel now asks outright which chat is open. The row also says **No chat is open** in that state rather than that it is still working out which chat you are in, which is a different thing and was the wrong thing to read on the home screen.

## 4.22.0

_2026-08-25_

### Added

- **Refusal notes have presets, the way word swaps do.** Save the notes you have tuned as a named set and switch between them, under Refusal tuning. A set carries the notes and where they go, and nothing else, so loading one never turns notes on or off for you. Saved to your account, so they follow you to other devices.
- **The panel counts the words your swaps changed.** A swap leaves nothing behind to look at once it lands, since the reply reads as though the model wrote it that way, so there was no way to tell a rule that never matched from a rule that was not running. The log says what each swap changed and the running total for that chat, the Stats tab splits the total between the chat you are in and everywhere else, and **Copy debug info** carries the same figure, so a bug report about swaps that did not happen answers the first question on its own.

### Fixed

- **A reply that was writing itself out could be retried as stuck.** The wait for a reply to start is called off by the first text that arrives, and that text has to be matched to the chat the reply started in. Lumiverse does not always name the chat the same way on a piece of text as it did when the reply began, and when it did not, the wait was never called off and re-rolled a reply that was streaming or already finished. Which chat a reply belongs to is now decided once, when it starts, and everything after that follows it.
- **Saved presets are counted, deleted and synced as one set.** Three places reached past the store into the word swap presets alone. Refusal note presets did not follow your account to another device, the count offered before deleting left them out, and deleting what the panel called word swap presets took them with it anyway. All three now cover both kinds. The reset picker says **Delete saved presets**, and the tick box under Import / export says **All presets**, since it always moved every kind whatever it was called.
- **Every tick box is named after what it carries.** Four of them held something their name did not mention, and the two that mattered most were the ones people untick: **Browser and screen** also carried your permissions, which is the one line that explains a report where nothing happened at all, and **Refusal detection** also carried the note wording sent on a retry. Under Import / export and Reset the parts are now **Retrying and its buttons**, **Refusals and notes**, **Word swaps**, **Button selectors**, **Panel and pop-up** and **All presets**. Under Debug info they are **Your settings**, **Buttons and selectors**, **Permissions, browser and screen** and **Session totals and recent activity**.
- **The log records what you do with presets.** Saving, loading, renaming, updating and deleting one each write a line naming which set it was, so a settings change that came from loading a preset is visible afterwards rather than having to be remembered.
- **A bug report says how long the session has been running.** Every counter in it was written down with nothing to read it against, and no retries after two minutes and no retries after four hours are the same three zeroes.
- **Two word swap settings were on the panel while they could not do anything.** **Wait for other extensions to finish** and its delay only ever hold up automatic swapping, so with **Swap words in replies** off they did nothing at all. They now appear only when automatic swapping is on. The swap buttons never waited on them and still do not, which the setting's own description now says.

## 4.21.0

_2026-08-25_

### Added

- **Replies are retried even when Lumiverse does not say which chat they came from.** Auto Retry files everything by chat, so a reply arriving without one used to be dropped before anything looked at it, and nothing was retried. Lumiverse normally names the chat, so this is a guard against a case you are unlikely to meet rather than something you have been running into.

  Three things need a chat by name and are skipped when there is not one, listed on the [troubleshooting page](docs/troubleshooting.md#when-lumiverse-does-not-say-which-chat-you-are-in). **Turn off here** is greyed out, the retry note is not added, and swapping words by hand says no chat is open. Retrying itself, and swaps that run automatically on each reply, carry on.

### Changed

- **Switching off a temporary chat no longer leaves a note in your browser that can never apply again.** A temporary chat is the mode for talking to the model with no character, and Lumiverse discards it when you leave. **Turn off here** still works there, and it is the place you are most likely to want it, since a retry re-rolls the reply you were trying to look at. The difference is that it is now kept only for as long as that chat is open. Remembering it was pointless: the chat is gone and the next one is a different chat, so nothing would ever match it. The row tells you this while you are in one.

## 4.20.1

_2026-08-24_

### Changed

- **Two settings descriptions were still carrying material the docs already cover.** A description is read while you are looking at the control it belongs to, so what belongs there is the shape of the setting and anything that would surprise you. The note list drops the worked example of how From try counts, which the detection page explains better, and keeps what you cannot get anywhere else: that notes reach the model rather than your chat, the cap, and one line each on Role and From try.

## 4.20.0

_2026-08-24_

### Added

- **The live line now knows whether your build streams.** With streaming on, a reply arriving counts its characters up as it lands. With it off, nothing arrives until the reply is finished, and the line sat on "waiting for the reply to start" for the whole generation, which is wrong twice over: it had started, and nothing was going to turn up before it was done. It says "Generating the reply" instead, with the time so far. Nothing exposes the setting, so this is worked out from what happens rather than asked: one token proves streaming is on, and a reply that finished with text having never sent one proves it is off. Until a whole reply has gone by it does not guess.

### Changed

- **Three settings descriptions were long enough to be a second copy of the docs.** The word swaps box carried the whole of the Writing rules page, and two others had grown a paragraph of reference material apiece. A description is read while you are looking at the control it belongs to, so it now gives the shape of the setting and what would surprise you, and leaves the rest to the page that covers it.

### Fixed

- **The on-screen panel looked frozen while it was waiting on the model.** Some busy states carried a figure that moves: a reply arriving counts characters, a retry counts down. The three that cover waiting, the model thinking, waiting for a reply to start, and waiting for a retry to start, each said one fixed sentence for as long as they lasted, which on a model that thinks for a minute is hard to tell from the panel having stopped working. All three now say how long they have been going.
- **Some models had their thinking counted as the reply.** A streamed token says what it is, and builds do not agree on the word. Three that mean thinking were not recognised, so that text was filed as reply text: the panel called the model's working-out a reply arriving, and the text was counted toward the length checks.

## 4.19.0

_2026-08-24_

### Added

- **Word swaps now leave HTML tags in a reply alone, with a setting to include them.** A reply that uses markup for colour or emphasis carries words inside the tags, and those words are the ordinary ones: color, font, small, center. A rule of `color => colour` was rewriting `<font color="#ffff00">` into `<font colour="#ffff00">`, which changes no wording at all, it just stops the text being coloured, with nothing to say why. Tags are skipped now, while words between them swap as they always did. **Also swap inside HTML tags** under Find and replace turns that off, for a rule aimed at the markup on purpose such as recolouring every line at once.
- **Word swaps now leave the model's thinking alone, with a setting to include it.** A reasoning model writes its working-out before the reply, and when it writes that as tags inside the reply it is part of the stored message, so swap rules were rewriting it. Lumiverse shows reasoning in its own block, so nothing about that was visible: a rule aimed at your prose was also editing what the model worked out, somewhere you would never look. Reasoning that a provider returns separately was never swapped, so the same rule behaved differently depending on who was serving the model. Only the reply you read is swapped now, and **Also swap inside the thinking** under Find and replace turns the old behaviour back on. The common wrappers are recognised, along with any name in Extra thinking tag names.

### Fixed

- **A reply carrying a `>` inside a tag could be thrown away as cut off.** Markup is removed before the checks that count quotation marks, and the pattern doing it stopped at the first `>` it found, including one inside a quoted attribute such as `<span title="a > b">`. That left a stray quotation mark in the text, which then read as dialogue opened and never closed, so a finished reply was re-rolled. The same flaw let a word swap reach into the tail of such a tag. Tags are read whole now, quoted values and all.
- **A very short reply wrapped in markup is no longer measured by its tags.** The too-short check is meant to catch a reply that says almost nothing, but it counted the whole stored text, and a line of dialogue wrapped in something like `<font color="#ffff00">` carries about thirty characters of tags around it. Three short lines could therefore clear a threshold set for prose while saying almost nothing. It counts the words now, the same way it already left reasoning out.
- **A reply cut off inside a commentary block was not recognised as cut off.** Models using the channel format put their reasoning in one of several named channels, and the cut-off check knew four of the five names while the part that strips thinking knew all five. A reply cut short inside a `commentary` channel therefore fell through to the empty-reply check, so it was retried for the wrong reason, and not retried at all by anyone who has that check switched off. All three places that ask which channels are thinking now read the same list.

## 4.18.4

_2026-08-23_

### Changed

- **The warning on the support-message setting is written more plainly.** It says the same things in the same order and asks the same question. The wording leaned on implication in a few places, which is the wrong way to write the one box in the extension you have to answer before you can go on. What the setting does, what it cannot tell, and what to keep in mind are each stated outright now.

## 4.18.3

_2026-08-21_

### Fixed

- **A reply mentioning a height was read as cut off.** A height written 6'2", or a gap 3" wide, puts a single straight quote in ordinary prose, and the check that looks for dialogue opened and never closed counted it as an opening. Character descriptions are full of heights, so this threw away replies that were plainly complete. A measurement is no longer mistaken for the start of a line of speech.
- **A reply that ends on a list was read as cut off.** A bullet or numbered item is a fragment and has no full stop at the end, which is what the check for a missing ending was looking for. A list that has finished now counts as an ending, while one cut mid-item is still caught.
- **Multiplication written 2*3 was read as an unfinished action.** A single asterisk between two words is not the start of an *action*, so it is no longer counted as one.
- **The This chat row often did not say who the chat was with.** The character's name was only ever looked up when a message rendered. Switching chats and starting a reply both make a chat current without going through that, so the two most ordinary ways of ending up in a chat left the row reading "This chat" with no name. It is looked up whichever way you got there, and asked for once per chat.

### Changed

- **The cut-off check is now held to a corpus of whole replies**, both kinds: replies that are finished, which it must never throw away, and replies that really were cut, which it must still catch.

## 4.18.2

_2026-08-21_

### Fixed

- **A word swap pressed with no chat open could edit the chat you had just left.** The swap buttons sit in the floating button's menu, which is on screen on the home screen too. Auto Retry was acting on the last chat it had seen, and some builds never say when you leave one, so that was still the chat you walked away from. It swapped saved replies you were not looking at, and a swap cannot be undone. It now checks which chat is actually open at the moment you press, and if there is none it says so and does nothing.
- **On the home screen it used to say "No reply found to swap in this chat"**, which named a chat that was not open. It says no chat is open.
- **A swap could aim at a reply in a different chat.** The last reply seen was remembered when the chat changed, so it belonged to the chat before. Nothing was harmed, because a reply that cannot be found falls back to the latest one, but it was asking for the wrong thing.

If your Lumiverse does not grant the chats permission, Auto Retry cannot check which chat is open and behaves as it did before.

## 4.18.1

_2026-08-21_

### Fixed

- **A reply that finished perfectly could be thrown away and asked for again as stalled.** Auto Retry sets a timer while a reply is coming in, and calls it off when the reply arrives. Finding the right timer to call off depends on Lumiverse telling it which chat the reply belongs to, and some builds leave that off the message that says the reply is done, or word it differently than they did when it started. When that happened the timer was never called off. It went on running and then re-rolled a reply that had been finished for a minute or more. Auto Retry now goes by the reply itself, which is named the same way every time, so it always finds the timer it set.
- **The same slip could throw away a good reply as empty.** On builds that do not include the finished text in that message, Auto Retry uses the text it watched arrive. That is kept per chat, so looking in the wrong place found nothing and a perfectly good reply counted as blank.
- **A chat could end up with two sets of timers.** A chat id that arrived as a number in one message and as text in another was treated as two different chats, so a reply was watched under one and finished under the other. They are read the same way now, whichever a build sends.

Nothing about when a retry should happen has changed. A reply that never starts, stops halfway, comes back empty, cut off or with an error is retried exactly as before.

## 4.18.0

_2026-08-20_

### Changed

- **The two word swap buttons now follow the floating button, the same way the panel button already did.** When the floating button is on, hold it and **Swap words in the last reply** and **Swap words in every reply** are in its menu. When the floating button is off, they are back in the Extras menu. Each button is in one place at a time, so you never get the same thing twice.
- **Hide this button** is now at the bottom of that menu, under everything else, because it is the only one that closes the menu for good. **Auto Retry settings** is still at the top.
- **The on/off button in the Extras menu is hidden while the floating button is on.** The floating button is the same on/off switch and one tap does it, so having both was two buttons for one thing. Turn the floating button off and this one comes back. This happens even on an older Lumiverse that cannot open the floating button's menu, because it is the button itself that replaces it, not the menu.
- **Auto Retry settings never moves.** It stays in the Extras menu whatever else is on, so there is always one way in that does not depend on anything.
- **The extension is called Auto Retry everywhere now.** Some messages said "auto-retry" and others said "Auto Retry", and one said both in the same sentence.
- **The Log tab speaks plainly.** Several lines in it named an internal part of Lumiverse or an API, which tells you nothing you can act on. They say what happened in ordinary words now. The permission names stay, because those are what Lumiverse's own extension settings call them, so they are worth knowing.
- **The pop-up messages were squeezed into half the screen.** "No reply found to swap in this chat." wrapped onto two lines on a phone and left "chat." alone on the second, on a message that fits on one line with room to spare. The box was being centred in a way that only gave it half the width of the screen, so the width it was meant to have never applied. It gets that width now, so fewer messages wrap at all, and the ones that do wrap use the whole box. Its padding is also inside that width now rather than added on top, which had it almost touching both edges of a phone.
- **The "?" beside each setting is bigger on a phone.** It was one size everywhere, comfortable with a mouse and small under a thumb. On a screen you touch it is now large enough to hit without aiming. On a computer it is unchanged, so the panel stays as dense as it was.
- **Every tick box in the panel is the same size.** The settings rows have always used one size; the reset picker and the import and export lists used the browser's smaller default, which was a harder thing to hit on a phone and looked wrong next to the rest. That mattered most in the reset picker, where a tick can delete saved word swap presets.
- **Clearer wording in the settings panel.** Several descriptions were long, or used a turn of phrase instead of saying the thing. The warning before you turn on the support-message option had a sentence that did not finish its comparison, which is the last place that should happen.

### Fixed

- **Save could tell you your settings were saved when they were not.** If your browser blocks site data, or has no room left, writing them fails. That failure was ignored, so the panel said "Saved" over settings that were gone the next time you loaded the page. It now says it could not save, and leaves that on screen rather than clearing it after a moment. If the copy kept on your account fails instead, it says that too, and says which one worked: your settings still apply here, they just will not follow you to another device.
- **Switching Auto Retry off in one chat said it was remembered even when it could not be.** That switch is kept in your browser rather than in your settings, so a browser blocking site data forgets it on the next reload. It now says so when that happens, and stays quiet the rest of the time.
- **Pressing a word swap button could do nothing at all, with no message.** If the extension's backend was not running, the request went out and no answer ever came back, so the button looked broken. It now says so after a few seconds and tells you to reload. The backend confirms it has the request before it starts work, so swapping a long chat, which can take a while, never shows that message by mistake.
- **The way into the panel could vanish.** **Open the Auto Retry panel** sits in the floating button's menu while that button is showing, and in the Extras menu when it is not. Holding the floating button and choosing **Hide this button** took that menu away without handing the entry back to the Extras menu, so it was in neither. If your panel was set to live in the sidebar, there was suddenly no way to open it except Ctrl+K, which is no use on a phone. Turning the floating button on went wrong the other way round: the entry moved into its menu and stayed in the Extras menu too, so it was in both.
- **On an older Lumiverse, the panel button went missing.** It moved into the floating button's menu on versions that cannot open that menu at all. Nothing moves into that menu now unless there is a menu to move into.
- **The floating button told screen readers to hold it for a per-chat switch that is not there.** That option moved to the settings panel in 4.16.0 and the button's description was never updated. For anyone using a screen reader, that description is the only thing describing the button. The per-chat switch is in the settings panel, under Basics, on the **This chat** row.
- **Closing the extension put its Extras buttons back on the way out**, so reloading the page could leave you with two of each.
- **Settings arriving from your account did not update which Extras buttons you had asked for.** Turning one on in another browser worked everywhere except in the menu it belongs in, until you saved something else.

## 4.17.0

_2026-08-20_

### Changed

- **The support check knows more of the wordings models actually use.** It caught the shape that is mostly a list of services; the shape that spends most of its length being kind and carries one line of referral was going past it. Reported wordings are now covered: "I'm glad you told me", "that takes courage to say out loud", "I'm listening", "I'm not going anywhere", "I won't judge you", "you matter", "this pain doesn't have to be carried alone", and the heading that introduces a second list of services under the first.
- Every one of those went in the softer of the two lists, which can agree with a signal but can never be one. A character in a scene says all of them, and two of them were already in the checks as scenes that must not be caught. The rule is unchanged: two agreeing signals, and the deciding one has to come from the register no scene uses. Take the referral line off any of the new replies and the same words are one character comforting another, which the checks now hold them to in both directions.

## 4.16.0

_2026-08-20_

### Added

- **Turning the floating button on or off eases between the two states.** The colours fade and the new mark grows in over the old one, so a tap reads as one movement rather than a flicker. Only on a real change: the button repaints when you switch chats and after a drag, and animating those would be movement saying nothing. A device set to reduce motion gets the same change with nothing in between, and the button itself still never moves under a press, since a press can be the start of a hold.

### Changed

- **The Prompt tab keeps up while the panel is open, instead of only while you are looking at it.** Switching to the log for a moment and back used to lose the prompt sent in between, which is most of what anybody does with the panel open. Nothing is still kept until you open the Prompt tab at least once, so using the panel for its log alone costs nothing, and closing the panel forgets it again.
- **Open the Auto Retry panel has moved out of the Extras popover and into the floating button's menu.** Hold the button and it is there, under the two entries that were already there. It stays in Extras only when there is no floating button to carry it, so there is always exactly one way in and never two.

## 4.15.0

_2026-08-19_

### Changed

- **The floating button's menu is now drawn by Lumiverse instead of by the extension.** It arrives in your own theme, accent and dark or light mode, clamps itself to the screen, and closes on Escape, and it looks the same as the menu any other extension puts there. A run of bugs on phones came from that menu being drawn by hand, each one down to guessing what a pointer was doing, and none of that is the extension's to get wrong any more. On a Lumiverse too old to have this menu, holding the button says where the settings are rather than opening nothing.

### Fixed

- **On a phone, a button in the settings panel stayed in its hover colour once you tapped it.** A touch browser raises the hover at the end of a tap and never sends the matching leave, and the reset that would have caught it had already run by then. Hover is now read from the pointer that caused the event, which says outright whether it was a finger or a mouse, rather than from the screen, which a phone showing the desktop site answers wrongly.
- **Descriptions in the settings panel could open on a tap with no way to close them, on a phone set to show the desktop site.** The same wrong answer wired up the hover pair and switched the tap off. Touching a long description to scroll it no longer closes it either.
- **The reset picker's second step opened with a ring around Go back.** It puts focus there so a keyboard can answer it, and whether that focus was drawn was left to the browser, which decides from the last kind of input it saw anywhere on the page. Focus the extension moves itself is now marked as its own, and the mark lifts at the first key pressed, so tabbing still shows where you are.

## 4.14.5

_2026-08-19_

### Fixed

- **An entry in the floating button's menu could stay highlighted after your finger left it.** A phone cannot hover, and a browser on one sends a hover when a finger rests somewhere and never sends the matching leave. The highlight was drawn from that hover alone, so touching an entry and sliding off it left the entry lit with the menu still open. Hovering is now only read on something that hovers, and a press lights the entry while it is held and puts it out when it is let go, which is what holding one should look like on any device.

## 4.14.4

_2026-08-19_

### Fixed

- **The floating button's menu opened with its first entry already lit.** It puts focus there as it opens so a keyboard can act on it straight away, and that was drawn the same as hovering, so a menu opened with a thumb came up looking like **Auto Retry settings** was about to be chosen. The entry still takes focus, so Enter and the arrow keys work exactly as before; it is only marked when a key put you there. Same fault as the one fixed in 4.14.3, in the other menu.

## 4.14.3

_2026-08-19_

### Fixed

- **A button lit up when a dialog moved focus onto it.** Opening the reset picker's second step puts focus on the safe answer so a keyboard can act on it, and the panel counted that as somebody having tabbed there, so the dialog opened with a ring around a button nobody had gone near. Whether a button should show its focus is a question the browser already answers, and it is asked properly now: pressed with a pointer, no mark; reached with a key, marked.

## 4.14.2

_2026-08-19_

### Fixed

- **A permission note came back on the grant that should have taken it away.** The backend answers with what the host says is granted, and when a grant changed it re-read that from a local cache rather than believing the event announcing the change. Inside that callback the cache can still hold the answer from before it, so turning a permission on reported it as still refused, and the note somebody had just put away came straight back and stayed. The event is believed for the permission it is about now, and its full list is used when the host sends one. When the panel asks outright it uses the host's authoritative answer rather than that cache, since a panel opening is rare and it is the answer somebody acts on. The panel also asks again every time it opens, so a grant is picked up even on a build that raises no event at all.

### Changed

- **A note about a missing permission can be put away.** Some are meant to be refused: somebody who does not want their prompt read declines the interceptor on purpose, and a panel telling them so on every visit is nagging about a decision they already made. Each note now has an × that hides it until you reload the page. Nothing is written down, so a reload brings every note back, and granting a permission and then losing it again brings its own note back too: hiding one answers the permission being off now, not for the rest of time. They are hidden by name rather than all at once, so putting away the one you chose to refuse does not also hide the next one that goes missing for a reason you did not choose. The debug report lists every permission either way.

## 4.14.1

_2026-08-18_

### Changed

- **The panel marks what has focus properly.** A field used to tint one hairline of border, which is easy to lose on a busy theme and says very little across a wide box. It now carries a soft band just outside the edge and a short halo past that, both in the theme's own accent, so it follows whatever colours you run. It is kept tight on purpose: a wide halo washes over the rows above and below and reads as belonging to the row rather than to the box. All of it is painted outside the box, so nothing sits on the text and no row moves when it lands.
- **Buttons reached by keyboard wear the same mark.** They were left with whatever outline the host's stylesheet happened to give them, which on a dark theme was often nothing you could see. A button you pressed with a pointer still wears nothing, since the press already said which one it was.
- **Fields lift their border under the pointer**, so a box reads as something you can put a cursor in before you have.
- **The search box stays quiet.** It takes none of the marks the rows below it do, such as the lift under the pointer or the tinted border. It sits alone above the scroll area with nothing beside it to be told apart from, and it answers every keystroke by filtering the list underneath, so it says where you are without being lit. Its clear button is unchanged.
- **The browser's own arrows are off the number boxes.** They are drawn by the browser rather than the theme, so on a dark panel they arrived as a pair of grey chevrons belonging to no design here. The value is typed, and a box you are in still steps with the arrow keys. The rule is scoped to this extension's own boxes, so nothing else in Lumiverse is touched.
- **A rule under each section heading.** Sections were text sitting above rows with nothing between them, so on a long panel one ran into the next and a heading read as another row rather than as a break. Drawn in the theme's own border colour, which is faint by design: enough to separate, not enough to become furniture.

## 4.14.0

_2026-08-18_

### Added

- **The panel says when a permission it needs was never granted.** This is the one fault that raises nothing anywhere: a gated event simply never arrives and a registration that needs approval silently does nothing, so an extension with the wrong grants sits there looking installed while doing none of what it was asked to. Missing `generation` means nothing is ever retried, and until now nothing said so. The panel now names each one that is missing and what it costs, and shows nothing at all when everything is granted. The debug report lists every one either way, so a report about an extension that did nothing carries the reason.
- **The Prompt tab says whether the interceptor permission is actually missing.** It used to guess from a prompt that never arrived, and a guess about a permission is the one claim in the panel you cannot check from the panel. It now asks, so a denial is stated plainly and a permission that is granted is never blamed.

### Removed

- **The preview circle beside the floating button's size.** It reserved a box as wide as the largest size the setting allows, in every panel, whether or not the button was even switched on. The button on the chat still takes the size on as you type, which is the preview that shows it where it will actually be.

## 4.13.0

_2026-08-17_

### Added

- **A raw view of the prompt, beside the usual one.** The button under the message and character count switches between them. Rendered is the panel as it has always looked, and where it starts: a row per message, its role, its size, whether it came from the chat or was wrapped around it, and any notes marked. Raw takes all of that off and shows the prompt as the data the model was handed, role and content and nothing else, which is the form to read when the question is about structure rather than wording, and the form to paste somewhere else. **Copy** follows whichever view you are on. The button sits on its own line and is sized for the longer of its two labels, so pressing it cannot move it. Whichever you pick is remembered.

### Changed

- **The Prompt tab shows the whole prompt.** It used to be capped at 200 messages, 4000 characters each and 300000 in total, with a line under a long message saying how much of it was missing. That was the one thing a reader could not work around, since what was cut only ever existed on the server and was thrown away as the view was built. Every message is now listed and every character of each one is there. The cost stays where it always was: a prompt is only captured while the Prompt tab is actually open, and nothing is captured at all once you switch away or close it.

### Fixed

- **The Prompt tab could show a prompt from a different chat.** A captured prompt is sent to a person rather than to a window, so with two chats open in two tabs, both of them received every prompt either one produced, and the tab showing one chat drew the other's without a word about it. The tab now checks, and says so instead of drawing it. The prompt itself is held rather than thrown away, so walking back into the chat it belongs to brings it back, and a prompt that arrives before anything has said which chat it was for is still shown once that is known.
- **The Prompt tab could stay empty however long you waited, and the refusal note could never send.** Both run off an interceptor, and registering one is fire-and-forget: without the permission the host does not throw, it silently does nothing and notifies separately. This registered once as the backend loaded, which was a bet that the grant was already in the local cache at that instant. A grant can also be given or taken away while the extension runs with nothing restarting. Losing that bet left both features dead for the life of the backend with nothing anywhere saying so, which looks exactly like a quiet install. The permission is checked first now, the registration is tried again the moment the permission is granted, and a refusal is written to the log rather than passing in silence.
- **The Prompt tab said the interceptor permission was missing when it was not.** A prompt is assembled as a reply begins, which is the only moment there is to capture it, so a tab opened partway through a reply cannot catch that one however long the reply runs. The tab took that silence for a missing permission and said so. Sending a reply with the panel shut, or while reading the Log, and then going to look at the prompt was enough to be told the extension lacked a permission it had, which is the one thing named there that you cannot check from the panel. It now only says that about a reply the tab was open and asking for from the start, and asks for another reply otherwise. **Clear** takes the message back along with the prompt, rather than emptying the tab and going on saying it about a reply you had just discarded.
- **The Prompt tab stayed empty until you left the chat and came back.** Asking the backend to capture prompts is a live request rather than a saved setting, and it was sent only when the answer changed. The two sides have separate lifetimes, so a backend that was not listening yet, or that restarted afterwards, knew nothing while the panel was certain it had already asked, and nothing ever re-sent it. Leaving the chat and returning happened to toggle the view off and on, which sent it again, which is why that appeared to be the fix. The backend now says when it has started, and any panel waiting on a prompt asks again when it hears it.

## 4.12.2

_2026-08-16_

### Changed

- **The Extras menu entry said Auto Retry was on in a chat you had just switched off.** Three things show whether it is running: the row in the settings panel, the floating button, and this entry. The first two are repainted when anything changes. The entry cannot be relabelled once it is registered, so it is torn down and registered again instead, and that only happened when the master switch moved. It reads both switches now, says "on, but off in this chat" when that is where you are, and follows you between chats.

### Fixed

- **Changing the floating button's size walked the button up the screen.** A size change has to rebuild the widget, since width and height are fixed when it is created, and the position for the rebuild was read back off the screen. That made it a feedback loop: any gap between what was measured and where the button actually sat went back in on the next change and added up, so dragging the size along moved the button a little further each step until it ran out of screen and stopped. The gap comes from the measurement taking the button's size from the host's own box, which does not always carry it. Nothing on this path is measured now. The button is rebuilt around the place this extension last put it, at the size it knows it asked for, so a run of size changes lands exactly where one does. Against an edge it still comes inward far enough for the bigger size to fit.
- **The per-chat switch could be left disagreeing with itself.** There were two ways back into a chat you had switched off: the **This chat** row, and a **Turn it back on here** button on the line at the top of the panel. The row repainted itself from its own click handler, so the line at the top was the one path that changed the state without touching the row. Pressing it turned the chat back on and left the row still offering to turn it on, and pressing the row then switched the chat off again.

### Removed

- **That button on the line at the top.** One switch does not need two buttons, and these two were not next to each other, so working out whether they did the same thing was left to the reader. The line says what is true and the **This chat** row is where you change it, which is also the arrangement that cannot come apart. Everything that describes the chat you are in is now repainted from one place, so any way of changing it reaches all of them.

## 4.12.1

_2026-08-16_

### Fixed

- **The Prompt tab stayed empty on builds that name the user only on one side of the bridge.** Opening that tab asks the backend to start capturing, and the request arrives with whatever Lumiverse calls you. The capture happens in the interceptor, which reads a name off its own context, and not every build puts one there. So the request was filed under your name, every generation looked you up under no name at all, and the tab sat empty for good with nothing anywhere saying why. An unnamed generation now reaches the only person watching. Where two or more are watching, a prompt that cannot be attributed is still dropped rather than handed to whichever of them is first, so nobody is shown a prompt that is not theirs.
- **An empty Prompt tab said "send a reply" to people who had sent several.** Reading the prompt needs the `interceptor` permission, which is privileged, and registering without it raises nothing at all, so a tab that could never work looked exactly like one waiting for you to do something. After a reply has finished with nothing arriving, it says the permission is what is missing and that the rest of the extension works without it.

## 4.12.0

_2026-08-15_

### Added

- **What a model refuses a horror roleplay over.** A slasher scene is an ordinary thing to write on a roleplay app, and two of eighteen refusal wordings were caught before this: graphic violence, gore, mutilation, dismemberment, body horror, animal cruelty, violence against children, a violent death, depictions of harm, a murder scene, stalking, and declining to play a real person. "Violence" on its own is not in the list, because "I can't describe the violence" is a line somebody says in a scene, so the qualified forms are listed instead.
- **A subject in the form a refusal about a backstory uses.** The list held the bare nouns, so a reply declining to write "a character is raped" or "her being sexually assaulted" or "him being tortured" walked past every pattern, which is the wording that comes up when somebody is asking a character about their past. The endings are spelled out rather than left to a wildcard, so a rapeseed field is still a field, and the forms that drop a letter are written separately. Somebody telling a character what happened to them is left alone, in seven checks that use the same words.
- **None of it costs you a scene that only sounds like one.** A knife in the porch light, someone stalking through the corn, blood on the stairs, a kissing scene, someone asking first, rope on a table, a stepbrother resenting his stepsister, the minor character in act two, somebody choking on smoke, the grooming of the horses. Every one of those carries a word from the lists above and none of them is a refusal, because a subject only ever counts as the object of a refusal verb. That rule is what lets the lists be as wide as they are.
- **The refusal aimed at a kind of writing.** "I cannot generate sexually explicit content or graphic descriptions of that." One word hid it: the pattern read whatever word followed the verb, and its list held "sexual" while the reply said "sexually", so the adverb was enough to walk past the whole thing. There is room for the adjectives that stack up in front now, and the noun still has to be one a model uses about its own output.
- **Every kind of writing a model refuses a roleplay over.** The subjects it names were checked as a sweep rather than one at a time, which turned up a category that matched nothing at all: kink. BDSM, bondage, degradation, breath play, ageplay, power exchange, the word kink itself. Also explicit writing under every name it goes by, smut and erotica among them; consent framings including dubious, unclear and non-consensual; the family framings a model reads as incest whether or not they are; a character it decides is underage; and content it calls illegal. Thirty-one wordings across five categories, and every one of them is a check.
- **The lesser version it offers instead.** "I can, however, continue the narrative with a focus on the dialogue," and "let me know if you would like to proceed with that approach". Nobody in a scene talks about continuing the narrative.

## 4.11.0

_2026-08-15_

### Changed

- **The extension has a new mark: a reply, with the retry arrow sweeping over it.** It was a tumbling die, because Lumiverse calls a fresh attempt a reroll. A die on its own says dice, though, and dice say tabletop, which is not what this is. What the extension actually acts on is a reply: it reads one, decides it failed, and asks for another, so the reply is the shape and the arrow is what is being done to it. It appears everywhere the old one did, at the same sizes, with the same slash across it when the extension is switched off.

## 4.10.0

_2026-08-15_

### Added

- **The refusal that names what it refuses.** Every other pattern needs a meta object, a request or a prompt or a roleplay, because those are words a character never uses. A model that names what it is declining uses none of them: the subject is the object, as in "I won't write content depicting that" or "I can't create scenes involving that". The subject on its own is never a signal and only ever counts as the object of a refusal verb, so a scene about any of it, a backstory that turns on it, a trial, or a character who will not talk about it, is left alone.
- **The flat no aimed at nothing in particular.** "No. I won't engage with that." The opening was recognised and the object was not, since "that" is not one of the meta words the rest of the list requires.
- **The framing dismissed as a device.** "Framing it as consensual roleplay doesn't change what it is", "regardless of how the request is framed", "I'm here for a genuine conversation".

### Removed

- **A pattern for something a roleplay reply never says.** Real refusal wording, but from a model declining to describe an uploaded image rather than declining to write a scene, so it could never fire here. Nothing that was caught before is missed now. A pattern that cannot fire still has to be read and kept right by everyone after you, and its being there implies a guard that is not really there.

## 4.9.0

_2026-08-15_

### Added

- **The refusal that spends a paragraph being reasonable.** Four of them got past 4.8.0, and not one says "I can't" anywhere. They open with "I'm not going to continue with this" or "I'm not going to roleplay this scenario", dismiss the fiction, offer to help with something else at length, and sign off with a question. Every sentence carrying the refusal in all four is caught now.
- **The fiction dismissed after the refusal rather than before it.** The list knew "even in a fictional context" and not "regardless of framing", which is the commoner half and the sentence all four of those replies share. It counts as an ordinary refusal rather than a breaking-off one, because it can sit in the opening line of a reply that then explains itself for three paragraphs, and the breaking-off check only looks near the end. It also only counts trailing a refusal, since framing is a word about pictures before it is a word about prompts and a painter is entitled to use it.
- **The refusal aimed at the form rather than the request.** "I'm not going to roleplay this scenario", "I don't write roleplay involving that". A character never says they do not write roleplay, because a character does not know they are in one. No pattern reads the subject any of this is about, and none ever will: what marks these replies as the model is the sentence dismissing the fiction and the sentence naming what it will not write, so a character discussing a song, a book or anything else is not affected.
- **"This isn't something I'll write", and "that's something I won't write".** The same sentence both ways up, and neither was matched: the pattern behind it knew "I can" and "I could" and not the plain future. The conditional is kept out and the future has to end its clause where it lands, so "that is not something I would write in a letter to him" is left alone.
- **The redirect offer and the sign-off, in the words a roleplay model uses.** It was looking for help-desk vocabulary, writing tasks and other topics, and had nothing for "I'm happy to help with other directions", "I'd be glad to help with a story", "let me know what you'd like to explore" or "is there something along those lines you'd like to try?".

### Changed

- **Stats counts a refusal under the cause that produced it.** Everything the refusal side caught landed on one line, so the tab could tell you that a hundred replies were refused and nothing about what to do next. There are four lines now: the model declined, the model broke off rather than declining, the model left the scene to offer support, and the provider blocked it before a reply was written. Each points at a different switch, and all four take the same retry, the same cap and the same note. This is what tells you whether the support check is worth having on, which matters more there than anywhere else, since it is the one check you turn on yourself.

### Fixed

- **A named speaker was not recognised as a dialogue tag.** The breaking-off check ignores a match with an attribution behind it, and the list it matched knew pronouns and proper nouns but not the commonest form of all, an article and a noun: "the shopkeeper asked", "her sister said". Anything a character said that way was read as the model breaking off.

## 4.8.0

_2026-08-15_

### Added

- **It can retry a reply where the model stops the scene to offer you real-world support.** Some replies do not decline anything and do not break off either: they stop being the story and become a message to the person at the keyboard, saying that what you have written is concerning, that you are not alone, and that you should talk to a professional, usually with a list of services under it. In a scene about something heavy that is normally the model reading your fiction as a report about your life, and it takes the scene away at the worst moment. **Also catch it stopping to offer support**, under Refusal tuning, throws that reply away and asks for another.
- **It is off unless you turn it on, and ticking it opens a warning that has to be answered first.** This is the one check in the extension that decides whether a particular message reaches you, and the only setting in the panel that asks before it takes effect. The warning links straight to [Safety](docs/safety.md), because telling somebody to go and read something, in a box they have to answer to get out of, works out to telling them not to bother. That link is the only address the extension points at, and drawing it fetches nothing.
- **What it takes to fire.** Two agreeing signals, at least one of them the model addressing you rather than your character. Comfort and the names of services can only ever agree with a signal, never carry one on their own, since warmth is a register a character uses: a man crouching beside somebody to say she does not have to go through this alone is not the model. A line inside quotation marks is never counted either. It is the one check **Longest reply to treat as a refusal** does not apply to, because that limit is built around a refusal being short and one of these is the opposite.
- **A safety page in the docs.** Who this is built for, what the support check can and cannot know, what retrying does when it is pointed at a reply somebody did not want to hear, how to reset or remove every part of it, and a closing note for anyone using it for something other than the writing.

### Changed

- **The doubled refusal is caught: "I cannot and will not engage with content that ...".** Every pattern in the list expected the verb straight after "I can't", so putting "and will not" between them hid the most emphatic refusal a model writes. A meta object is still required, so a character saying "I cannot and will not marry him" is left alone.
- **So is the refusal stated as a boundary.** "What I won't do is write that scene", and the offer that follows it, "here's what I can do". There is no "I can't" anywhere in either, so nothing in the list saw them. A meta object is required, so "What I won't do is leave you here" is left alone.
- **And the closing offer it signs off with.** "Is there something else I can help you with, or a different kind of story you'd like to explore?" The bare line without the offer of a different story is left out, because that is what every shopkeeper in every tavern scene says, and in script format it has no quotation marks to give it away. Add it under **Your own refusal phrases** if your model uses it.
- **And the reply that sorts out what you meant instead of writing.** The model reads your message as a question with several answers, lays them out, and ends by asking which you were after. It rides on the breaking-off check, since it always ends on that question, so the switch that turns that off turns this off with it.
- **So is the flat no.** Some models do not soften it at all: the reply opens with "No." and then says what it will not write. Nothing in the list saw those, because every pattern in it starts at "I". It only counts at the very start of a reply and still needs an object no character has, so "No. I can't tell you that story" is left alone and "No. I won't write a scene like that" is not.
- **The published crisis wordings of the three big assistants** are in the built-in list, rather than the shapes one of them happens to use. Roughly forty more, on both sides: the support message, and the ordinary refusal openings that come with it.

### Fixed

- **Switching off "Ignore refusals inside quotation marks" took the dialogue tag rule with it.** That switch is for a model that wraps its own refusals in quotation marks, and it now reaches the phrase list and the patterns and nothing else. The dialogue tag rule stays on, since it is about an attribution rather than about quotation marks. The support check added above ignores quoted lines whatever the switch says, for the same reason: no model wraps that message in quotation marks, so switching it off could only stop a doctor or a counsellor in the scene from being told apart from the model.
- **A refusal standing between two pieces of speech on one line was read as dialogue.** The quotation rule looked for the nearest mark behind the match and any mark ahead of it, which is the right answer on a line carrying one piece of speech and the wrong one on a line carrying two: `"Go on," he said. I can't help with that. "Please," she said.` found the closing mark of the first speech behind it and the opening mark of the second ahead of it, and left the refusal alone. It counts the marks between the start of the line and the match now, and an odd number is what puts the match inside a quotation.
- **A note written rather than spoken read as the model breaking off.** The breaking-off check ignores a match with a dialogue tag behind it, because that is speech with the quotation marks left off, and the list of tags it knew had no writing verbs in it. "Let me know how you'd like to proceed, she wrote at the bottom of the letter" was thrown away as a result. A dozen more attributions are recognised now.

## 4.7.0

_2026-08-13_

### Added

- **It can now ask Lumiverse which chat you are in, under a new `chats` permission.** This is what fixes **Turn off here** sitting greyed out in a chat you were already in, which happened after updating the extension because nothing re-renders and so nothing announced where you were. Auto Retry uses a single call from it, the one that answers which chat is open. It never creates, deletes or alters a chat. Refuse it and everything still works, with the switch waiting to be told as before. [Privacy](docs/privacy.md) covers what this permission reaches.

### Changed

- **The panel says who a chat is with.** The **This chat** row reads "This chat, with *name*" and the Stats tab breaks retries down by chat as well as by cause, so a card whose replies keep needing a retry is visible instead of buried in a total. This needs the new `characters` permission alongside `chats`; without it the row reads "This chat" as before and the breakdown falls back to a short chat id. A group chat is named by its primary card.
- **The Prompt tab counts tokens rather than estimating them.** It said "roughly N tokens", worked out as characters divided by four. Where Lumiverse will do the counting it now shows the real figure and drops the "roughly". This needs no permission, and the estimate is still what you see on a build or model that will not answer. The count arrives just after the view does, so nothing waits on it.
- **Accidental-refusal retrying is no longer marked beta.** It has been on by default for a long time while carrying a label that says to be careful with it, which are two opposite claims. The detection has not changed; the label was undersold. Find and replace keeps its beta label, because that one is off by default, edits your saved messages, cannot be undone and needs a privileged permission.
- **The per-chat switch says what it is actually waiting for.** When it does not yet know which chat you are in, it said "open a chat", which is confusing advice to read while you are in one. It now says it is waiting to catch the chat, and what will tell it. That wording is what you see when the `chats` permission above is refused or not yet approved, since asking outright is the other half of this fix.

### Fixed

- **Word swaps did nothing on their own until you opened the settings and pressed Save.** The manual buttons worked the whole time, which is what made it look like automatic swapping was broken rather than switched off. Your settings live per user, so the check the backend runs when it starts has nobody to read them for and finds nothing. The panel asking for your settings when the page loads is the one moment they arrive with a user attached, and that path handed them to the panel without ever telling the swap engine about them. It applies them now, so swapping is ready as soon as a chat is.
- **Word swaps ignored the master switch.** Switching Auto Retry off stopped it retrying and left word swapping rewriting your replies, because the backend that does the swapping had never been told that switch exists. Its own on/off is the swap one. Off now means off for both. The two manual swap buttons are unchanged and still work whatever the switches say: pressing one is asking for a swap there and then, which is different from swapping happening on its own.
- **Word swaps ignored the per-chat switch too.** A chat you had switched off carried on having its replies rewritten. The list of chats you have switched off is kept in your browser rather than in your settings, so the backend could not see it and had to be told; it is now sent when it changes and when the page loads.
- **Flipping the switch from the floating button or the Extras entry did not sync.** Saving the settings panel writes to your account, and those two controls flip the same switch and did not, so the setting people change most often stayed in whichever browser they changed it in. It also never reached the backend, which is the second half of why swapping carried on after switching the extension off.

### Removed

- **Dead code, all of it.** A backend handler for a message the panel has never sent in the extension's history, the storage file only that handler wrote, and the startup read that looked for it. A one-time clear of another file nothing has ever written. None of it did anything; it just looked like it did.

## 4.6.2

_2026-08-13_

### Changed

- **The Prompt tab says plainly that only the view is shortened.** A long message was marked "(cut for display)", which reads as though the prompt had been cut before it went out. It now says how much more was sent and that only what you are looking at is capped, so there is no way to read it as the model having been given less. Copying the tab says the same: the header for a message claims its real length, and until now the text under it could be shorter with nothing saying so.

### Fixed

- **Lumiverse's own menu opened underneath the floating button's.** Holding or right-clicking the button showed its menu with the app's default one behind it, which then cleared on its own a moment later. The press was being told not to draw the browser's menu, which is a different thing from being stopped, so it carried on up to Lumiverse and opened that one too. It is now stopped at the button, before anything else can act on it. Dragging is untouched, since only the menu press is swallowed and every other kind still reaches the app.

### Removed

- **"Move back to the corner" is gone from the floating button's menu.** The button's saved position is checked against the screen every time it is drawn and it snaps to the nearest edge, so it cannot end up somewhere you cannot reach, which is what that entry was for. Dragging it back is fewer taps than opening a menu. The menu is two entries now.

## 4.6.1

_2026-08-13_

### Fixed

- **"Turn off here" stayed greyed out in a chat you were already sitting in.** The row needs to know which chat you are in, and it only ever learned that from a chat change or from a reply being generated. Open Lumiverse straight into a chat, or reload the page there, and neither has happened, so the button sat disabled saying to open a chat while you were in one. Leaving and coming back fixed it, which is not something anyone should have to work out. It now also learns the chat from a message being drawn on screen, which is what actually happens when a chat opens, so the row is ready as soon as you can see the conversation. If the settings panel is already open when it finds out, the row updates in place.

## 4.6.0

_2026-08-13_

### Added

- **The on-screen panel can live in Lumiverse's sidebar drawer.** A new row under the panel switch, **Where that panel goes**, offers **Floating over the chat**, which is what it has always been, or **In the sidebar drawer**. Same panel, same three tabs, either way. In the drawer, Lumiverse places, sizes and themes it, so there is nothing to drag and nothing to remember, and it cannot cover the reply you are reading. Its tab carries a dot while a retry is running, so you can see something is happening without opening it. Floating is still the default, so an update does not move your panel. The drawer needs no permission, so the extension still declares the same four, and on a build with no drawer for extensions, asking for the sidebar gets you the floating panel and a line in the Log saying why.
- **Open the Auto Retry panel**, in the chat input's **Extras** popover next to the settings entry, while the panel lives in the drawer. Where the drawer opens from belongs to Lumiverse and is not the same in every build, and **Ctrl+K** is no use on a phone, so Extras is the way in: one tap, any device, nothing else to switch on. It is not offered while the panel is floating, where it is already on screen.

### Changed

- **The settings panel is down to eight sections from eleven, and opens on three.** Two of them were a heading over a single row, which is a heading that says nothing the row does not already say. **How it redoes a reply** was one switch and now sits at the end of **How it retries**, the section it was next to. **Watch for frozen replies** was two waits and now sits at the end of **When to count a reply as bad**, under a **Replies that freeze** heading, because a reply that never finished is a bad reply too. Nothing was renamed except **How hard it tries**, which is **How it retries** now that it also says which button a retry presses.
- **The on-screen panel switch is in Basics.** It had an **Advanced: on-screen log** heading to itself, shut by default, so turning on the panel meant opening a collapsed Advanced section first. It is not advanced, and it is the first thing you are asked to turn on when reporting a bug. It is in Basics with the master switch and the other ways of seeing what the extension is doing.
- **Nothing calls itself "Advanced" any more.** The five closed sections were **Advanced: refusal tuning**, **Advanced: find and replace**, and so on, and for most of them that was not true: saving your settings to a file and building a bug report are ordinary things anybody might want. The word was really doing the job of "this one starts closed", which the **▸** already says. They are now **Refusal tuning**, **Find and replace**, **Buttons it clicks**, **Debug info** and **Import / export**. They still start closed, because nothing in them is needed to use the extension, which is a different claim from being difficult.
- **Refusal tuning sits directly under the check it tunes.** It was below find and replace, two sections away from **It looks like an accidental refusal**, which is the switch that makes it do anything at all.

### Fixed

- **A stray end-of-reply event could re-roll a reply based on the one before it.** While a reply streams, the extension keeps the text arriving so far, because some Lumiverse builds do not put the finished reply on the event that says it ended, and then what streamed is the only thing there is to check. That copy was cleared when the *next* reply started rather than when the one it belonged to finished, so an end event arriving on its own was judged on the previous reply's text and could fire a retry for a cut-off that had already been dealt with. It is dropped when the reply ends now, and when you stop one partway.

## 4.5.7

_2026-08-10_

### Fixed

- **The retry pop-up could stay on screen with a Cancel button that did nothing.** It exists to count one wait down and it never removes itself, so it has to be taken away when the wait ends. It was stopping only once the chat had nothing left to say, and a retry that fired successfully has plenty to say: the reply it just started. So the box stayed up describing that reply, then the next one, with a Cancel button for a retry that was long finished. It now goes the moment the wait it was counting is over.
- **Cancel and Stop always clear it now.** Standing down hid the pop-up only when a retry was still pending, and after one had already fired there is nothing pending, so pressing Cancel or Stop left the box exactly where it was. Pressing Stop more than once hit the same thing. The pop-up carries the Cancel button, so it is hidden whenever anything stands down, whatever was or was not in flight.

## 4.5.6

_2026-08-07_

### Changed

- **A dropdown is no longer left marked for having been clicked.** Clicking one used to tint its border to say it had the focus, and the tint stayed on the row after the choosing was done, until something else was clicked. It says nothing you cannot already see, since clicking a dropdown puts its menu on screen with the choice in front of you. Reaching one from the keyboard still marks it, because there is no menu then and nothing else saying where you are, and a text box is still marked either way, which is where the mark was doing its job.

## 4.5.5

_2026-08-07_

### Changed

- **The description on What the notes say was nearly five times the length of a normal one, and said half of itself twice.** It explained that each note carries its own role and starting try, then explained it again a few sentences later, and it also described the two settings underneath the list, which now sit under a **For the whole list** heading that says so on its own. The heading went in a few versions back and the words it replaced were never taken out. The descriptions on **Where the notes go** and **Only send them on a regenerate or a swipe** both opened with the same redundant sentence and have lost it too.

### Fixed

- **Copy now takes everything the tab is showing.** It was leaving things out, and the things it left out were the ones you would be copying the tab to report. On **Stats**: the refusal note counts, how often a reply needed a retry, the notice that it had paused itself, and the line saying nothing had needed a retry yet. On **Prompt**: the summary of how many messages and how large, the line saying how many notes went and where they landed, and the marking on the messages that carried them, so a copied prompt now says which message was a note instead of leaving you to guess. On **Log**, an empty log copied nothing at all rather than what it says on screen. The text the button builds is written separately from the view it describes, which is how the two drifted apart in the first place.

## 4.5.4

_2026-08-07_

### Changed

- **The dot beside the status line has three states instead of two.** Dim and flat when Auto Retry is off or paused. Lit, and still, when it is on with nothing to do. Pulsing while something is actually happening: a retry counting down, a reply arriving, the model thinking. Movement means movement rather than decoration, so glancing at the corner answers the question without reading the line. It is opacity and a glow only, and the movement is dropped for anyone whose system asks for less of it, keeping the glow, which is the part carrying the meaning.

### Fixed

- **The panel said the model was thinking after a reply was stopped.** The flag behind that line was only cleared when a generation ended, and stopping one does not always end it, so the line sat there claiming work was happening until the next reply started. Stopping clears it now, on the host's own stop event and on the plain click of your Stop button, which is the case with no event at all. A token arriving puts it back, so a reply that really is still streaming corrects the line by itself.
- **A reply starting, ending, or a chat being switched took up to a quarter of a second to show.** Those are changes of state rather than a number ticking down, so they repaint at once, the same as a retry being scheduled or called off already did.

## 4.5.3

_2026-08-07_

### Fixed

- **A reply cut off inside a block the card asked for is caught now.** Cards ask the model to wrap its planning in a tag of their own making, `<story_plan>` and the like. When the reply stopped partway through one, nothing noticed: the text inside can end on a full stop with its quotation marks balanced, so every check that reads the shape of the reply said it finished, and stripping the markup deleted the one piece of evidence first. A tag alone on its line, with a name HTML does not have, opened and never closed, is now read as cut off. A word in angle brackets inside a sentence is not, since that is how people write an emote.
- **The check for an unclosed reasoning block knew the wrong list of names.** It was written out by hand rather than read from the setting, so it knew five of the eight built-in names, and none of the ones you had added under **Extra thinking tag names**. A reply cut off inside `<scratchpad>`, `<analysis>`, `<thoughts>` or any tag you had told the extension about read as finished. Both now read the same list.

## 4.5.2

_2026-08-07_

### Fixed

- **A card that renders a whole interface no longer has its replies thrown away.** A chat window, a profile card, a stat panel: dozens of nested `div`s with text inside them. The checks for open dialogue and unpaired emphasis were reading that text as prose, and a height written `6'2"` is one unpaired quotation mark. That flipped the count for every properly closed piece of dialogue in the reply, so a finished reply read as having speech left open, got thrown away, and the replacement went the same way. What is inside a container that closed is finished writing, because the model reached the closing tag, so none of it is counted now. The same goes for a stray asterisk or a line ending on a comma inside a widget.

Nothing is lost by trusting a closing tag, because a reply cut off inside a widget never reaches one: that leaves the container open, which is already read as cut off. Prose after a widget is still checked as prose, so a reply that renders its card and then stops mid-sentence is caught as it was.

## 4.5.1

_2026-08-07_

### Fixed

- **The retry limit could be set to 0, which stopped it retrying anything.** The give-up check passed on the very first failure, so a retry extension sat there reporting itself as on, button lit and panel saying it was watching, and did nothing. Worse, giving up is the only thing that counts a failed run, so at 0 **Pause when everything is failing** could never fire either. The lowest is 1 now, and a saved 0 is raised to 1 when it loads. To stop it retrying, switch it off instead, either everywhere or in one chat, since both of those say on screen that they are the reason.

## 4.5.0

_2026-08-07_

### Added

- **It notices when a tracker was cut off partway through.** The other half of leaving trackers alone. A reply that stopped inside something it had started is cut off whatever it stopped on, the same as one with an opened quote, so markup left open is checked before a block ending is accepted as an ending: a container opened and never closed, a tag with no closing bracket, a tag cut off inside an attribute, an HTML comment with no end. Without this, `<div class="wx"><b>Weather</b>` would read as finished on the strength of the last tag it managed to write.
- **A status block written as raw JSON is checked too.** `{"temp": 24, "sky":` ends on a colon, and a colon is punctuation, so it read as a finished reply. Braces are counted outside code now, in the one direction that means something.
- **Dialogue coloured with a `<span style="...">` is checked for its closing tag.** This is how cards colour speech, and it was the one cut nothing else here noticed: the speech closes its own quotation marks, so a reply that stopped with the colour still open came out even on every check and read as finished. A styled span left open now counts as cut off. A bare `<b>` or `<i>` does not, since models fumble those in ordinary prose often enough that counting them would throw away good writing, while a span carrying an attribute is there because a card asked for it and gets its closing tag every time. What counts is the attribute rather than what is in it, so a single colour, a gradient, a `<font color>` or a `<span class>` are all checked the same way.

What has not started counting: a `<` someone typed in a scene, `if x<y`, a bare inline tag left open, or list items written without their end tags. Elements whose end tag is optional in HTML are left out of the container count, because models write `<ul><li>one<li>two</ul>` and mean it. An inline tag left hanging is a finished reply written badly, and that is not worth throwing the reply away over.

### Changed

- **The retry pop-up counts down instead of freezing.** It used to say "Retrying 2/5 (cut off) in 47.3s" once and go on saying it for the next forty-seven seconds, so the one number anyone actually watches was the one number that never moved. 4.4.0 raised the longest wait to a minute, which turned that from a small oddity into something that looks like the extension having stopped. It now reads **Cut off. Retrying in 47s (try 2 of 5)** and the number goes down. Only the text is repainted, so the Cancel button next to it cannot be swallowed by a press landing mid-redraw.
- **One way of writing a length, everywhere.** Whole seconds, because a figure twitching four times a second is noise, then `5m 03s` and `1h 05m 03s` as the wait grows. The countdown, the panel, the Stats tab and the message announcing a pause all say a length the same way now. Hours are there because the pause after repeated failures can be set to three of them, and `180 minutes` leaves you doing the division. Smaller units keep their leading zero so the line does not change width as it counts.
- **The on-screen panel says what is happening this second.** A line under the tabs, with a dot that lights while something is going on. It counts down a pending retry, says when a reply is arriving and roughly how much has landed, says when the model is thinking, says when it has paused itself after repeated failures, and says when there is nothing to do. It sits above all three tabs because none of them answered that question: the Log says what already happened, the Stats say what has happened overall. A retry in a chat you have moved away from is still reported, marked as being in another chat. It and the pop-up read from the same place, so they cannot disagree.
- **The Stats tab counts up as you watch it.** **Watching for** was rounded to the nearest minute and drawn once, so it read "1 minute" for the first ninety seconds of every session and then sat there until something else redrew the view. It reads `2h 23m 05s` now and it moves. The note saying it has paused itself counts down there too, and clears itself when the pause ends rather than counting past zero. Only those two lines are rewritten as they change, so the bars and the scroll position stay where they are.
- **Switching off in one chat is only in the settings now.** It was in the floating button's menu as well, and that button sits over your chat: its menu gets opened to reach the settings or to move the button, and a per-chat switch sitting among those read as clutter every time. It is under **Basics**, on the **This chat** row, which is where it already was. Nothing about the feature itself changed. It was never reliably in that menu anyway: the entry was only drawn once a chat had been seen, and that only happens on a generation, so on a fresh page load it was missing until the first reply came through.
- **The panel and the floating button stay where you put them.** Both went back to their default corner on every reload, which means on every update: drag the panel clear of your chat, size it to what you want to read, update the extension, do it all again. Where the panel is, how big it is, which tab was open, and where the button was dragged to are all remembered now. **Move back to the corner** still does exactly that, and forgets the saved spot rather than restoring it a moment later.
- **A layout is checked against the screen it opens on.** One saved on a desktop window and reopened on a phone would otherwise put the panel mostly past the edge with its header out of reach, and the header is the only way to drag it back. It is kept in the browser rather than in your settings, alongside the list of chats you have switched it off in, for the same reason: a position belongs to the screen you are sitting at, not to your account, so it does not follow you to another device and it is not in an export.
- **One clock drives all of it, and it stops on its own.** Nothing repaints while there is nothing on screen showing a live figure, and everything stops when the panel closes or the extension is torn down. Between ticks nothing is happening, so a quarter of a second is fine for a number counting down; anything that actually changes, a retry being scheduled or called off, the master switch, a chat switched off, the breaker pausing, repaints at once rather than waiting for the next tick.
- **Extra dialog buttons is behind a switch now, and off.** Its own description said "almost nobody needs this", and it sat in the panel anyway, one more box to wonder about. **My dialog's button says something else** reveals it. The built-in list, which knows Skip, Regenerate, Confirm, Proceed, Submit and OK, is used either way, so nothing about retrying changes. The switch genuinely gates the box rather than only hiding it: a hidden box whose contents were still being pressed would be the panel saying one thing and doing another. If you already had wording typed in there, the switch was turned on for you, so nothing you set has stopped working.
- **The two note settings that are not per note now sit under a heading saying so.** The list gives every note a role and a try to start on, which made the two settings underneath it look like more of the same. They are not: where the block goes and whether it is sent at all are set once and apply to whichever notes are due. **For the whole list** now sits above them and says which is which.

### Fixed

- **A card that prints a tracker no longer has every reply thrown away.** 4.4.0 turned **Retry when a reply has no ending punctuation** on by default, and a tracker is the one shape that check was worst at. A weather box, a stat block, a status line, a table: none of them end on a full stop, so the reply read as cut off mid-sentence, the retry ended the same way, and it went round until the cap stopped it. A reply that ends on a block ends on a block. Closing HTML, a markdown table row and a run of two or more label lines like `HP: 20/20` all count as an ending now. Prose that stops mid-sentence after a tracker is still caught, and a single line with a colon in it is still an ordinary sentence, since a tracker never has only the one field.
- **Code in a reply is no longer read as unfinished writing.** The checks below the code fence count are about prose: dialogue left open, a sentence stopping on a comma, an emphasis run with no partner. A snippet is full of the same characters meaning something else, so one `const a = b * 2;` counted as an opened emphasis run and re-rolled a finished answer. Fenced blocks and inline spans are now left out of that counting. The fences and backticks themselves are still counted first, so a reply cut off inside a code block is caught exactly as before.
- **A row of asterisks is no longer mistaken for an opened emphasis run.** `Mood: ***`, printed by a card as a gauge, or a line of them used as a divider. Emphasis has to touch the words it marks, so `*He nods*` and `**bold**` still count.

## 4.4.0

_2026-08-06_

### Added

- **It now catches the model breaking off rather than declining.** "I'm going to stop here", "I won't continue this discussion", "let's redirect the conversation". On by default, and narrow: it only counts when that is how the reply ends, never inside quotation marks, and never behind a dialogue tag, so a character who stops walking and carries on with the scene is left alone. Switch it off with **Also catch the model breaking off**.
- **The built-in refusal list has grown by about twenty wordings.** These are the ones models actually use that were being walked past: "I can't generate that", "I don't create content like that", "I'm not going to comply with that request", "I can't help with illegal activities", "I can't provide advice on that", "I can't process that request" and others. Apologetic openings on their own ("I'm sorry", "Unfortunately", "I apologize") are still not counted: they open as many ordinary replies as refusals, and a character apologising is one of the most common things in roleplay.
- **A switch for whether a refusal in quotation marks counts.** On by default, which leaves dialogue alone. Your own phrases are counted either way.
- **The on-screen log says when a refusal note went out.** It names the retry that carried it and how many notes went with it. Copy debug info counts them too, so a bug report can say whether notes were going out at all. Before this, a note that worked and no note at all looked exactly the same from outside.
- **A Stats view in the on-screen panel.** A third tab beside Log and Prompt: replies that came back fine, retries fired, messages it gave up on, how often a reply needed a retry at all, and a breakdown of what it retried for with a bar for each. It also says when it has paused itself after repeated failures, which is the state that otherwise looks like it having stopped working. The counters behind it already existed for the debug report, which is a wall of text you have to ask for and then read. **Clear** on that tab starts the counting again.
- **Auto Retry can be switched off in one chat.** In settings, under **Basics**, the **This chat** row does it, and holding or right-clicking the floating button offers the same thing. That chat is left alone and every other one carries on, which is the shape the master switch was the wrong answer for: a scene where the model is meant to refuse, in the middle of a day of ordinary chats. It survives a reload, and while you are in a chat that is switched off the top of the settings panel says so and offers to undo it, because a chat you switched off and forgot about looks exactly like the extension having stopped working. It is kept in your browser rather than in your settings: a list of chat ids would mean nothing on another account and has no business in an export.
- **Use my last reply, in the refusal tester.** Fills the box from the reply on screen so you can check the one that actually bothered you without copying it by hand. It reads what is rendered at the moment you press it, so nothing is kept between replies.
- **The floating button's menu opens the settings.** Hold the button, or right-click it, and **Auto Retry settings** is the first entry. Reaching them otherwise means the input bar's Extras popover, which is several taps away.
- **The on-screen panel now has tabs, and one of them shows the prompt.** **Log** is what the extension is doing, as before. **Prompt** is the whole prompt that went to the model: every message in order, its role, how large it is, and whether it came from your chat or was added around it. Tap one to read it. Your refusal notes are marked in that list and opened for you, with a line at the top saying how many went and where they landed, so you can see exactly how and where a note was inserted. This is what actually went, after your setup, your world info, your persona and every extension have had their turn, which is a different question from the one Lumiverse's **Prompt Breakdown** answers. Copy and Clear act on whichever tab you are on, tapping switches tabs, and so do the left and right arrow keys.
- **The prompt is only captured while you are looking at it.** There is no second setting for the Prompt view, because there does not need to be one: switch to Log, close the panel, or close the tab, and nothing is captured at all. A setting left on would go on paying for itself in every chat long after you looked once. What is captured is captured on your device and shown to you. Nothing is sent anywhere, nothing is written to disk, and it goes with the tab.
- **Each note carries its own first try.** The list used to share one number, so more notes could only ever mean more text at once. Now a gentle note can start on try 2 and a firmer one on try 4, and each retry carries whichever have come due, still in order so one can answer another. A list saved before this keeps behaving exactly as it did: every note takes the number the single setting held. The separate **Start the note on try** row is gone, replaced by a **from try** box on each note.
- **Three more ways of wrapping reasoning are recognised.** On top of `<think>` and `[thinking]`, the pipe forms `<|think|>` … `<|/think|>` and `<|think>` … `<think|>` are read now, and so is the `<|channel|>analysis<|message|>` … `<|end|>` block that models trained on the Harmony format use. Only the thinking channels are removed; the `final` channel is the visible reply and is kept. Any tag name you add under **Extra thinking tag names** works in all four. Until now a reply wrapped this way was checked with its whole reasoning block counted as part of it, so a refusal the model only weighed up caused a retry, and the length limit was measured against text the reader never sees.
- **The floating button's size can be seen while you set it.** A circle beside the box is drawn at the size you type, and the real button changes with it, so you are not guessing from a number. Closing the panel without saving puts it back.

### Changed

- **Reset is no longer all or nothing.** **Reset…** at the bottom of the panel opens a picker: tick the parts you want put back to their defaults, and anything you leave unticked is not touched. The parts are the same ones import and export use, so the names match between the two panels. Each line says how many of its settings you have actually changed, and a part still at its defaults cannot be ticked, since there would be nothing for it to do.
- **Nothing resets without being asked first.** **Reset ticked** shows what it is about to do, naming the parts you picked, how many settings are in each, and whether presets are going with them. Nothing happens until you press **Yes, reset**. **Go back** returns to the list with your ticks kept, and Escape or a click outside closes it without touching a setting. The question is asked by the extension rather than handed to Lumiverse's own confirm dialog, because not every build has one.
- **A reset waits for Save, the way an import does.** It fills the settings in behind the box without saving them, so you can look at what it did first. Press Save to keep it, or close the panel to discard it. Pressing Reset by mistake now costs a panel close rather than your settings.
- **The reset confirmation says how serious it is, and looks it.** Putting settings back is undone by closing the panel, and deleting presets is not, so the two are not painted the same: the first asks in the theme's warning colour, the second turns red, says "Deleting presets cannot be undone", and relabels its button **Yes, reset and delete**. The preset line in the list is bold and red as well. Painting the mild case like the serious one would make the warning worth ignoring on the one that matters.
- **The picker says what it cannot reach.** Your chats, your replies and your characters are never touched by any of it. Your saved word swap presets are kept too, unless you tick the line for them, which sits under a rule of its own because it is the one thing there that deletes something for real rather than waiting for Save. **Tick every setting** never ticks it.
- **The defaults now assume a slow model rather than a fast one.** A watchdog that fires early on a model that is slow but healthy throws away a reply that was still arriving, and the replacement comes from the same slow model, so it fires again on that one too. The wait for a reply that has started but produced nothing goes from 90 seconds to 3 minutes, the wait for a stream that has gone quiet mid-reply from 45 to 90 seconds, the longest wait between tries from 30 to 60 seconds, and the wait when the server says it is busy from 8 to 15 seconds, which clears the per-minute limits most shared tiers use. A retry click also gets 15 seconds rather than 6 to produce a generation before the extension decides the click failed.
- **The new defaults apply to a fresh install.** Settings already saved to your account keep the values they had, so if you want the new ones, open **Reset…**, tick the parts you want, and Save.
- **Waiting for another extension to finish now waits 85 seconds instead of 15.** A refinement pass is a whole generation, so how long it takes depends on the model, the prompt and how much it has to read. Fifteen seconds covered a fast model and nothing else, so on anything slower the swap landed first and the refinement arrived on top and wiped it, which is the failure that setting exists to prevent. The most you can set has gone from 2 minutes to 5.
- **Retrying a reply that stops on a word is now on by default.** This was off because it was wrong too often, and the reason it was wrong is fixed below. It is what catches a reply cut off mid-sentence when nothing else was left open.
- **Nothing the extension draws sits at the top of the stacking order any more.** Two of its surfaces used the highest number a browser accepts, which meant no other extension could ever draw above them. They are still above the page and now leave room above themselves.
- **Nothing in the extension animates except a hover colour and a focus outline.** The floating button carried transitions on four colour properties and a scale dip on every press; it now changes instantly, which says the same thing sooner. The transition on `filter` is gone from every button: the press feedback that uses it is meant to be instant, and animating `filter` is what forces a button onto its own compositing layer for no benefit.

### Fixed

- **The refusal note was never sent on most setups.** It was only attached when Lumiverse reported the generation as a regenerate or a swipe. Most builds report every generation as "normal", a regenerate included, so on those the note was armed, the retry ran without it, and the only sign was a line in the log. The note is now attached to the retry it was armed for whatever the host calls it. What keeps it off a message you type has not changed: it is armed the moment before the retry is clicked, it belongs to one chat, it is used once, it expires, and it is taken back the moment the click it was armed for turns out to have started nothing. If there is no retry button on screen to click, nothing is armed at all. The old behaviour is available as **Only send it on a regenerate or a swipe**, off by default.
- **A word swap was undone by pressing Edit.** The swap is saved to the server, but Lumiverse's own copy of the message in the browser did not always pick it up, so the edit box opened showing the wording from before the swap and pressing Save wrote that back over it. Each swap is now remembered as the exact text before and after, and an edit box holding a message that is character for character the pre-swap text is filled with the swapped one instead. It is a whole-string match, so a box holding anything else, including a message you have started editing, is never touched.
- **A refusal inside quotation marks was treated as the model refusing.** `"I can't help with that," the innkeeper muttered` was counted as a refusal and the reply was thrown away. Dialogue is a character speaking, and it is left alone across every built-in list now rather than just the "I am an AI" patterns.
- **Line breaks were flattened before a reply was checked.** Every break became a space, which let a refusal pattern match across a paragraph break and made dialogue anywhere in a long reply look like it wrapped everything between it. Both threw away replies that were fine.
- **"That violates my safety guidelines" and "that goes against my content policies" were never matched.** The pattern only allowed the noun to follow the possessive directly. Neither was "that isn't something I can help with", which was only matched written as "that's not".
- **A phrase only matched the way it happened to be listed.** "I'm unable to help with that" and "I am unable to help with that" are the same refusal, and only one of them was caught. The written-out form of every contracted phrase is worked out from the contracted one now.
- **Inline HTML confused the checks that decide a reply was cut off.** Models colour their dialogue with a raw `<span style="...">`, and the two quotes around that style value were counted alongside the two around the speech, so a reply whose dialogue was genuinely cut open came out even and was passed as finished. A trailing `</span>` was also read as end punctuation, which hid the same fault from the other check. Tags are removed before anything is counted.
- **A reply only counted as finished if it ended in Latin punctuation.** A scene closing on an emoji, or on a Japanese, Chinese, Greek or Arabic full stop, was read as having no ending at all. Punctuation in any script counts now, and so does an emoji.
- **Stripping a model's reasoning got slower the more of it there was.** The patterns that remove a reasoning block walk forward from every opener looking for its closer, so a reply carrying many openers and no closer made each one scan everything after it. Twice the input meant four times the work: 5,000 openers took 96ms, 20,000 took a second and a half, and 40,000 took six seconds, which on a real reply is a locked-up tab. It now checks whether there is a closer at all before going looking, which makes that case linear and takes it to a couple of milliseconds. An ordinary long reply was never affected and is unchanged.
- **Word swaps could rewrite the extension's own log panel.** The pass over what is on screen skipped the settings panel and the pop-up but not the log, so a rule could edit the extension's own words underneath it.
- **Word swaps could rewrite other extensions' interfaces.** Applying a swap to what was on screen walked every piece of text on the page, so a rule of "cat => dog" changed the word wherever it appeared, including inside another extension's panel, which had no idea its own text had been edited underneath it. Only the rendered replies are touched now.
- **On a server shared by several accounts, replies from the backend went to everybody.** A word-swap confirmation, and the text of a swap, were sent without saying which account they belonged to. They are addressed now. Nothing changes on an ordinary single-user install.
- **Changing the floating button's size moved it back to the corner.** Resizing rebuilds the button, and the new one started where a fresh one starts, so wherever you had dragged it was lost. It keeps its place now, pulled back onto the screen if the larger size would hang off the edge.

### Removed

- **The separate Start the note on try setting.** Replaced by a **from try** box on each note, which is what lets a list escalate. Your existing value is carried onto every note you already had.
- **The separate Reset button selectors button.** It existed because resetting everything to fix one mistyped selector was too blunt. The picker covers that properly: tick **Button selectors** and nothing else.

## 4.3.0

_2026-08-05_

### Added

- **Wait for other extensions to finish.** Off by default, under Advanced: find and replace. If another extension also rewrites replies, Hone with auto-refine on being the case this was built for, a swap applied the instant a reply landed was overwritten by that extension's rewrite a few seconds later. With this on, the swap waits for the reply to stop changing and then applies to whatever the text has become, so both extensions' work survives. If a later edit undoes a swap anyway, it is re-applied, up to three times per reply. With it off nothing changes: swaps land immediately, which is right when nothing else is editing.

### Fixed

- **The refusal note was often never sent.** The note was handed to the backend at the same moment the retry button was clicked, and those travel by different routes. The click regularly reached the model first, so the note was not in place yet and the prompt went out without it: nothing in the reply, nothing in the Prompt Breakdown, nothing in the log. The retry now waits for the backend to confirm the note is in place before clicking. If the host has no backend bridge the retry still fires, after a short wait, exactly as before.
- **The note now says when it was skipped and why.** It is only ever attached to a regenerate or a swipe. If the host called the generation something else, that is now written to the log with the name it used, instead of the note not appearing with no reason given. The log also says when a note was held back because it does not start until a later try, which is the default.
- **A swapped reply sometimes did not appear until you left the chat and came back.** The chat view only redraws when a message is saved with its swipe details named. A reply carrying no usable swipe list was saved with its text alone, which the view ignores, so the swap was correctly stored and invisible. Those saves now name the active swipe, which is enough for the redraw and changes nothing about the message.
- **A deferred swap no longer undoes another extension's edit.** Swaps used to be worked out from the reply as it stood when it finished generating. Anything that rewrote the reply after that was replaced by the older text. The rules are now applied to the message as it stands at the moment of the swap.
- **The swap buttons work in a chat you have not generated in yet.** They learn the current chat from the chat itself now, rather than only from a generation, so opening an older chat and pressing swap no longer reports that there is no reply to swap.

## 4.2.0

_2026-08-04_

### Added

- **Send a note with a refusal retry.** Off by default. Every other kind of retry re-sends your request exactly as it was, and still does. This one, and only this one, can add a note you write to the prompt for that single try. Whatever you type is sent exactly as written: nothing is added to it, nothing is removed, and nothing in it is checked.
- **You can send more than one note.** The **+** button adds another and **−** removes it, up to ten. They go out together in the order you wrote them, so a note can answer the one before it: a system note explaining the scene, then a line in the character's voice picking it back up, then a line from you asking it to continue. Each carries its own role. Adding one puts the cursor straight in it if you are working with a mouse or a keyboard, and does not on a phone, where that would raise the on-screen keyboard over the panel. An empty note is skipped, so a half-filled list is not a trap. Ten is the ceiling because every note is a whole message added to the prompt on every refusal retry, and past that they crowd out the scene they are meant to rescue. Use fewer by adding fewer.
- **Three things control how the notes are sent.** Which role each is sent under: system, you, or your character. Where the block is inserted: after the last message, before it, or at the very start. And which try it starts on, 2 by default, so the first retry goes out unchanged and the note is added from the second onward. Set it to 1 to add it every time.
- **The note never touches your chat.** It goes to the model for one generation and nothing else. No message is written, nothing is edited, and it is not part of the reply. It cannot attach itself to a message you type either: Lumiverse says what kind of generation is running, and anything you send yourself is a normal one, which the note is never applied to. If your Lumiverse shows a Prompt Breakdown, the note appears there as its own block so you can check exactly what went out.

### Changed

- **A filled button gets an outline when your theme's accent has all but vanished.** On a theme whose accent sits close to the panel colour, Save stayed readable but lost its edge, so nothing said it was a button. It now gets a border only when its fill has faded into the surface behind it. A theme with an ordinary accent is left exactly as it was, and the quieter secondary buttons keep the border your theme gives them.
- **The panel says when Auto Retry itself is off.** It can be switched off from the floating button or the Extras menu without opening the settings, so it was possible to arrive here with it off and nothing saying why nothing was happening. A line at the top says so now. Nothing is hidden or greyed for it: off means paused rather than unconfigured, and setting things up while it is off is a normal thing to want to do.
- **A setting that does nothing yet is no longer shown.** Options that only matter once something else is switched on are kept out of the panel until it is. Turning **Send a note with a refusal retry** on adds the note rows below it and turning it off takes them away, and the same goes for the short-reply threshold, the pause settings, the floating button's size, the phrase rewording and the re-swap option. Whole sections go the same way: with **It looks like an accidental refusal** off, nothing under **Advanced: refusal tuning** does anything, so the heading goes too. The switch doing the hiding never moves itself.
- **The search box ignores all of that** and finds a setting whichever way its switch is set, so nothing is hidden from you when you go looking by name. A row found that way says which switch it is waiting on, so changing it never looks like it did nothing. Only settings the extension genuinely ignores are hidden either way: the extra thinking tag names, for one, are still used to find the reply when the reasoning option is off, so they stay.
- **A new permission, `interceptor`.** This is what lets an extension add to a prompt before it reaches the model, and it is the only way the note above can work. Without it granted, everything else in the extension works as before and the note is simply not sent.

### Fixed

- **Every status line in the panel was invisible on some light themes.** The search count, the "already at the defaults" note, and the line confirming a save were all white on white on a theme that sets the common colours but not the ones behind the panel. The panel checks its own text against the surface it sits on, once, when it is built, and it only looked at text that was there at that moment. Every one of those lines is empty until something happens, so not one of them had ever been checked. They are now.
- **The hint text and the full-size editor were blank boxes on some light themes.** Making those panels solid in 4.1.0 meant painting a background colour and laying your theme's tint over the top of it. The extension then reads that background to decide whether text on it should be light or dark, and it was reading the colour underneath rather than the one you can see. On a theme that sets the common colour variables but not the one behind those panels, the fallback under the tint is dark, so the panel painted near-white and its text was turned white to match. It now reads what the panel actually paints. A theme that sets every variable was never affected, which is why this only showed up on hand-written ones.
- **The search box's clear button had no colour on light themes.** That cross is drawn by the browser and takes no colour of its own, so it is cut into a shape and filled from your theme instead. It was filled from a named colour, and naming one means naming what to use when your theme does not set it, which was a colour for dark themes. A light theme that set the common colours but not that one got a near-white cross on a near-white field. It now takes the search box's own text colour instead of naming anything, so there is nothing left to fall back to and nothing to keep in step: whatever your theme makes the text, the cross matches, on any theme.
- **A description moved depending on how much room there was.** Every **?** description opens just under the setting it belongs to, except when it was too tall to fit below, where it flipped above the row instead. That put a description in a place none of the others go, and the taller the description the likelier it moved, so the ones hardest to find were the ones that moved. Which side a description opens on is now fixed per setting rather than worked out each time, and one too tall for the room there scrolls inside itself instead of moving. Scrolling it to read it no longer closes it. **What the notes say** is set to open above, because that row holds the whole note list and below it would be a long way from the **?** you pressed.
- **"Where the note goes" would not stay set.** Pick **Before the last message** or **At the very start**, press Save, and it was saved correctly. Come back to the panel and it read **After the last message** again, and that is the one that was actually used. It was the only setting picked from a list rather than typed or ticked, and the check that reads a saved value back was not being told which list to check it against, so nothing ever matched and it fell back to the first option every time. Every setting is now read back through its own field, and there is a check that changes every setting in the panel, saves, starts fresh, and compares them all.
- **A setting found by searching kept asking for a switch you had just turned on.** Search for a setting whose switch is off and the row says which one it is waiting on. Turning that switch on from the same search results left the line sitting there, still naming a switch that was now on, until the search box was cleared. The line is rebuilt whenever one of those switches moves.
- **Turning Auto Retry off with the settings panel open put it back on.** The floating button and the Extras entry both switch it without opening the panel, so it could be flipped while the panel was up. The panel's own checkbox went on showing the old state, and because closing the panel puts back whatever was there when it opened, closing it undid the switch. Both halves now agree, and closing the panel leaves the switch where you put it.
- **A note could be left waiting for a retry that never happened.** The note is put in place a moment before Auto Retry clicks your retry button, because some setups start the reply straight off that click. If the button was not there to click, the note stayed armed for a minute, and a regenerate you pressed yourself in that minute would have picked it up. That is the one thing this feature promises never to do. A retry that does not fire now takes its note back, and so does pressing Stop or Cancel. Taking it back is tied to the chat it was armed for, so stopping a retry in one chat leaves a note waiting in another where it is.
- **A word swap could rewrite the wrong message on screen.** Lumiverse saves a swapped reply without redrawing the chat, so the extension rewrites the visible text itself, and it was rewriting more of it than had actually been changed. If a reply used the same word twice, the second swap went looking further up the page and rewrote an older reply that was never edited. A swap-the-whole-chat did the same to your own messages, which are never swapped at all. Nothing was ever wrong in the saved chat, and reopening it put the display right, so this only ever showed while you were looking at it. It now changes exactly as many words as were really changed, newest first.
- **The full-size editor could be left behind.** Open the **Expand** editor on a long box and then have Lumiverse reload or switch the extension off, and the editor stayed on screen covering the app, with nothing behind it. It closes with everything else now. In the same pass, hiding the floating button left a check running on every pointer movement for a button that was no longer there.

## 4.1.0

_2026-08-02_

### Added

- **Hold the floating button for a menu.** Move it back to the corner, or hide it. Right-click does the same on a computer. Before this the only way to put that button away was to open settings and switch it off, and on a phone there is no right-click at all.
- **The floating button follows your "reduce motion" setting.** It dips when pressed, which is the only thing in the extension that actually moves. If your device asks for less movement it stays still, and still changes colour so a tap is acknowledged.

### Changed

- **Auto Retry has its own symbol.** A die caught mid-tumble, since a fresh attempt is a reroll. It replaces the circular arrow on the settings entry and the power symbol on the on/off entry. The floating button was drawing a text character, so its shape was whatever font your phone reached for. It is a real drawing now and holds together at every size.

### Fixed

- **Word swaps work in every language now.** A rule for a single word only worked if that word started and ended with a plain English letter. Everything else was read, accepted, and then did nothing: no error, nothing in the log. `café => bar`, `über => over`, `résumé`, `fiancé`, and every word in Greek, Cyrillic, Japanese, Turkish, Polish or Czech. A word with the accent in the middle, like `naïve` or `señor`, was fine, which is why this went unnoticed for so long. Phrases were never affected. If you have a rule like that sitting in your list doing nothing, it will start working after this update.
- **Unchecked boxes were white blocks.** A checkbox is drawn by the browser, which picks its colours from the page rather than from your theme, so an unchecked one came out as a bright white square on a dark panel. The off state was the loudest thing on screen and the on state receded. The panel now measures what it is sitting on and tells the browser which way round it is, so a light theme still gets light controls.
- **The full-size editor, the live log and the retry pop-up were see-through.** Open the editor over the settings and you could read the rows behind it, Save included. All three were painted with a colour that is 90% opaque, meant to tint a surface rather than be one. Every panel that floats over something else is solid now.
- **Five settings showed a number with no unit.** "Wait before the first retry" read 1200 with nothing saying whether that meant milliseconds, seconds or minutes, while "How long to pause (minutes)" right above it named its unit. All five say (ms) now.
- **The retry pop-up covered the floating button's menu.** It sat above everything, so it could land on top of a menu you had just opened and turn a tap on "Hide this button" into a tap on Cancel. Things you open on purpose now sit above things that appear on their own.
- **The menu's focus ring was a hard white rectangle**, taken from the browser rather than your theme. It uses your accent colour now.
- **Preset buttons that had nothing to act on.** With no presets saved, Load, Update selected, Delete and Rename selected were all lit, Load styled as the main action, and each one answered a press with a message telling you to pick a preset first. They wait until there is a preset to act on.

## 4.0.0

_2026-07-31_

**Reinstall required.** Renaming the repository's main branch broke the link your install used to find updates. Remove Auto Retry and install it again from the same URL and it will pick up the new name. Your settings are saved to your Lumiverse account, so they come back with it.

### Changed

- **Main is now called `stable`, and there is a new `testing` branch.** Stable only moves when there is a real release, so installing from it means a notification when something has actually gone out. Testing is where work in progress goes, so install from there for an early look, bugs included.

### Fixed

- **The clear button in the search box follows your theme.** The cross that empties the search field was white. It now uses the same muted colour as the **?** circles next to each setting. Chrome, Edge and Safari only; Firefox does not add a clear button to search fields at all, so there is nothing there to restyle.

Nothing else changed from 3.3.0. The major version is for the forced reinstall, not for the size of the change.

## 3.3.0

_2026-07-31_

### Added

- **Search the settings.** A box at the top of the settings panel finds any option by its name or its description, and opens whichever Advanced section it lives in. There are over forty options now, so this saves remembering which group something is under. Clear the box and the panel goes back exactly as it was.
- **Floating on/off button.** Optional, off by default. Puts a small round button over the chat that turns Auto Retry on or off in one tap and shows which state it is in. Drag it anywhere, it snaps to the nearest edge and stays where you leave it, and you can set its size. Needs the new `ui_panels` permission.
- **Try your refusal settings on a reply.** New box at the bottom of the refusal tuning section. Paste a reply, press Check this text, and it says whether it counts as a refusal and what decided it: which phrase matched, which built-in pattern fired, or why it was passed over. It uses the values in the boxes as they are, so you can try a change before saving. It works the other way round too: paste an in-character line that keeps getting re-rolled and it names the rule catching it, which tells you what to put in "Never treat these as a refusal".
- **Copy and Clear on the live log.** The on-screen log is there because the browser console is out of reach on a phone, which is also where selecting text by hand is worst. Copy puts the whole log on your clipboard in one tap. Clear empties it so a long session does not bury what you are watching for.
- **Session totals in the debug report.** Alongside the activity timeline it now counts how many replies came back fine, how many retries fired, how many messages it gave up on, and a breakdown of retries by reason. It turns "it retries too much" into "ninety retries, all of them for cut off".
- **Keyboard access to the Advanced sections.** Those headers could not be reached without a pointer, so refusal tuning, find and replace, buttons, debug info and import/export could not be opened from a keyboard at all. They are proper buttons now: tab to one and press Enter or Space.

### Changed

- **Descriptions no longer shove the list around.** Tapping a **?** used to open the description inside the row and push everything below it down the screen, and opening a second one moved everything again. It now floats just below that setting, so nothing moves and the setting you asked about stays visible. Only one shows at a time. Tap the description, tap the **?** again, tap elsewhere, scroll, or press Esc to dismiss it.
- **Find and replace says what a preset carries.** That section is split under two headings. **Saved in a preset** holds your rules and the two options that decide how they match. **Yours, whatever preset you load** holds everything a preset leaves alone: whether swapping is on at all, which buttons appear in your Extras menu, whether a reply can be swapped twice, and whether it confirms before editing. Loading a preset cannot change any of those.
- **Word swap presets follow your account.** They used to live only in the browser you made them in, so your settings would move to a new device and your presets would not. They now sync the same way your settings do, with a copy kept in the browser so the list is on screen instantly.
- **More of the list on screen.** The settings panel was capped short enough that a tall phone had most of its screen sitting empty underneath it. It now uses the height it is given, so roughly 40% more options are visible before scrolling.
- **The panel looks more like the rest of Lumiverse.** Buttons like Reset to defaults were see-through with just an outline, so they read as text rather than as buttons. They use the theme's own button colour now, and hovering one switches to the theme's hover colour instead of just brightening it. The retry pop-up, the on-screen log and the full-size text editor were also rendering with the wrong corner rounding, asking for the small radius when they meant the larger one.
- **Word swaps update the chat faster.** Applying swaps to what is already on screen walked the whole page once per rule, so a long rule list meant dozens of passes over every message. It walks once now.

### Fixed

- **Button labels stay readable on any theme.** On a theme whose accent colour sits close to white, the label on a filled button like Save washed out and was hard to make out. The panel now checks as it draws whether each label stands out from what is behind it, and repaints only the ones that do not. Everything else is left exactly as the theme set it.
- **The retry pop-up was nearly see-through.** It was painted with a colour meant to tint a surface rather than be one, so it came out as a faint smudge over the chat instead of a solid pill.
- **Backups were leaving out four settings.** Pause when everything is failing, Failed runs before pausing, How long to pause, and Extra dialog buttons it may press were all missing from import and export, so a backup came back without them. They are included now, and any setting added in future is carried automatically.
- **The debug report was missing the same four.** It now reads the list straight from the options themselves, so every setting is always in it and there is no second list to fall out of date.
- **Duplicate swap-whole-chat button.** That Extras entry was not being cleaned up when the extension reloaded, so it stacked up another copy each time.

**Updating:** the on/off button setting was renamed internally, so if you had it switched on you will need to switch it back on. Lumiverse also asks you to approve the new `ui_panels` permission. That one only lets the floating button take up screen space; everything except the floating button works without it.

## 3.2.1

_2026-07-29_

### Changed

- **Clearer settings descriptions.** Trimmed the wordier ones again so they say what the setting does without the bloat.
- **The docs are split into pages.** The README was getting long, so it is now a short intro plus a `docs/` folder: when it retries, word swaps, all settings, buttons it clicks, import and export, reporting a bug.

## 3.2.0

_2026-07-28_

### Added

- **Extra dialog buttons it may press.** New box in the buttons settings, for the rare case where that dialog's button says something other than Skip. Type the wording exactly as it appears, one per line. Most people will not need it.

### Changed

- **README.** New sections on how Regeneration Feedback and Auto Retry work together, and on writing selectors by hand.

### Fixed

- **Works with Regeneration Feedback.** Reported by a Discord user. With Lumiverse's Regeneration Feedback turned on, pressing regenerate opens a box asking for guidance, and that box is what actually starts the reply. Auto Retry was clicking regenerate and stopping there, so nothing happened. It now presses Skip and carries on. Manual regenerates still open the box normally, so neither setting needs changing.

## 3.1.2

_2026-07-24_

### Fixed

- **Catches more refusals.** Some slipped through, especially ones phrased around roleplay itself, like "I cannot participate in romantic or sexual roleplay scenarios, even in a fictional context." Those are caught now. In-character lines are still safe, so a character saying "I cannot participate in this duel" will not trigger a retry.

## 3.1.1

_2026-07-24_

### Fixed

- **Word swaps show up right away.** Swapped replies were saved correctly but the chat did not render them, so the change only appeared after leaving and reopening the window. The swap now updates the reply on screen as soon as it is made.

## 3.1.0

_2026-07-24_

### Changed

- **Word swap presets stick to your rules.** A preset used to carry the whole word-swap section, so loading one could switch swapping on, remove the confirm-before-editing prompt, allow double swaps, or move buttons around in your Extras menu. It now saves your rules plus **Pick a swap at random** and **Match case exactly**, and leaves the rest alone. Exports are unchanged, and presets already saved keep working.

### Fixed

- **Fewer good replies re-rolled as refusals.** A character saying something like "I can't assist you with the horses" no longer reads as the model refusing, and a self-identifying AI character in dialogue is treated as a character rather than the model breaking scene. Real refusals are still caught.
- **The short-reply check measures the visible reply.** It was counting reasoning blocks toward the length, so on a thinking model a two-word reply could pass as long enough. Only affects you if that option is on, since it is off by default.

## 3.0.2

_2026-07-24_

### Fixed

- **Tidier buttons section.** The Test and Reset messages sat in different places depending on their length, since a short one fit beside the button and a long one wrapped below it. On narrow screens the Test result also got squeezed by the two buttons sharing its line. Both messages now sit on their own line under the buttons, with the space reserved so nothing shifts when text appears.

## 3.0.1

_2026-07-23_

### Added

- **Reset button selectors.** New button at the bottom of Advanced: buttons it clicks. Puts all three selectors back to the defaults without touching anything else. It fills the boxes, so press Save to keep it or close the panel to undo.

### Changed

- **Shorter description on that section.** It had grown into a wall of text. The panel now covers what you need on first read, and the README has the fallback list and selector syntax.
- **README: word swaps.** Clearer on how the longest-match rule and the random option differ. Longest match decides which rule fires when two compete for the same spot; random decides which replacement one rule uses when several are given. Also notes that identical left sides fall back to list order.

## 3.0.0

_2026-07-23_

**Reinstall required.** The repository history was rewritten, so this version does not arrive as a normal update. Remove Auto Retry and install it again from the repo.

**Check your word swap rules.** Rules now run in a single pass, so no rule acts on what another rule just wrote. `cat => dog` alongside `dog => wolf` turns cats into dogs and dogs into wolves, and never turns a cat into a wolf. If you relied on rules chaining, your output will change. The upside is that two rules can now swap past each other: `hot => cold` with `cold => hot` exchanges the two words instead of making everything one of them. Where two rules could match the same spot, the one with the longer left side wins.

### Added

- **Pick it for me.** New button by each button setting. Press it, then click the real button in Lumiverse. It builds selectors from labels and data attributes rather than class names, which Lumiverse regenerates every release.
- **Pause when everything is failing.** On by default. Several failed runs in a row pauses auto-retry instead of retrying on every message. Two boxes set how many and how long. A good reply ends it early.

### Changed

- **Empty and cut-off checks no longer depend on the refusal option.** Turning off Ignore the thinking / reasoning was also switching off those two checks, so thinking-only and mid-thought replies slipped through. That option covers refusal matching only now.
- **Selectors can contain commas.** `:is(a, b)` and `[aria-label="Next, swipe"]` each count as one entry.
- **Debug info says why retries stopped.** It now opens with whether auto-retry is active, off, or paused.

### Fixed

- **Retry by adding a new reroll works on everything.** It used to fall back to regenerate on empty replies and errors, which is exactly when regenerate clears the rerolls the option exists to keep.
- **It will not click a dead button.** Hidden and disabled buttons accept a click and do nothing, and that counted as a retry. They are skipped now, and if a click starts no reply it tries the other button once.
- **Fewer false cut-offs on reasoning models.** Punctuation inside a closed thinking block was skewing the check.
- **Stalled replies retry every time.** Only the first stall per chat worked; after that the extension's own stop click was read as the user pressing Stop.

## 2.8.3

_2026-07-22_

### Fixed

- **Retries failed on truncated or refused replies.** Chat UIs attach a swipe or regenerate button to every message. The extension was clicking the button for the first message in the chat instead of the newest one, so the retry did nothing. It now targets the last matching button on the page.

## 2.8.2

_2026-07-22_

### Added

- **New refusal pattern.** Covers models breaking character to refuse by naming specific prohibited content policies.

### Changed

- **Selectors check in the order given.** Comma-separated selectors now evaluate left to right. Put specific selectors first, broad ones last.

### Fixed

- **Retries stopped on empty replies.** With "Retry by adding a new reroll", an empty or error reply creates no message bubble on screen. The extension looked for the swipe button anyway, and because the real one was missing, the broad default selector (`aria-label*="next"`) matched an unrelated button such as "Next chat" and clicked it. The extension believed the retry had fired and waited indefinitely. It now checks whether the reply has visible content, and if not it skips the swipe button and goes straight to regenerate.
- **Stuck retry budgets.** Added a 6-second timer. If a retry click fails to start a generation, the extension resets its counter so the next message works normally.

**Action required unless you rely on your own selectors.** Saved settings override the new code defaults. Open settings, press Reset to defaults, and Save. Or paste these by hand:

- Regenerate: `[title="Regenerate"], [data-action="regenerate"], [data-testid="regenerate"], button[aria-label*="regenerate" i], button[title*="regenerate" i]`
- Next / swipe: `[aria-label="Next swipe"], [data-action="swipe-right"], [data-testid="swipe-right"], button[aria-label*="next swipe" i], button[aria-label*="swipe right" i], button[aria-label*="reroll" i], button[title*="swipe" i]`
- Stop: `[aria-label="Stop generation"], [data-action="stop"], [data-testid="stop"], button[aria-label*="stop" i], button[title*="stop" i], [class*="_sendBtnStop_"]`

## 2.8.1

_2026-07-21_

### Fixed

- **A refusal buried in the thinking with no reply after it now retries.** Some models put their whole refusal inside the reasoning and then write nothing. The empty-reply check now sees through inline think blocks, so an output that is all thinking and no reply gets retried instead of slipping past. A refusal in the thinking followed by a fine reply is still left alone, and a reply that itself refuses was already caught.

## 2.8.0

_2026-07-21_

### Changed

- **Whole-chat swapping is its own button.** The "Button swaps the whole chat" toggle is gone. A new option adds a second Extras button, **Swap words in every reply**, that applies your rules once to every generated reply in the chat, which is handy after adding a rule mid-chat or loading a different preset. Off by default, and the original swap button now always does just the latest reply.

## 2.7.0

_2026-07-21_

### Added

- **Presets travel with your exports.** Advanced: import / export has a Word swap presets option. Tick it to include your presets in the export file. Importing merges them: same-named presets are replaced, new ones are added, the rest are left alone. Since presets lived on one browser at the time, this was how to move them between devices or share them. Imported presets save right away, with no Save press needed.

### Changed

- **Shorter store description.** Trimmed to one line so it reads clean in the extension list.

### Fixed

- **Importing no longer jumps the panel.** Importing general settings used to snap the panel back to the top. It now fills the fields where you are and stays put.

## 2.6.0

_2026-07-19_

### Added

- **Word swap presets.** Save your word-swap setups as named presets and switch between them without copying rules by hand. They live at the bottom of Advanced: find and replace. Pick one and press Load, or use Save as new, Rename selected, Update selected and Delete. Loading takes effect right away. Kept on your browser, so they do not sync across devices.
- **Expand button on long text boxes.** Any multiline field, such as word-swap rules or refusal phrase lists, has an Expand button that opens a full-size editor. It opens without popping the keyboard, so you can read first and tap in when you want to type.

### Changed

- **Descriptions moved into tooltips.** Each setting's explanation sits behind a small **?** next to its name. Hover on a computer, tap on a phone. Keeps the panel much more compact.
- **Follows your theme's fonts.** Panels and the settings UI use the Lumiverse global font, headers use the bold version, and the code areas (debug preview and live log) use the mono font.
- **Advanced sections stay put.** Opening the find and replace Advanced section and then loading a preset no longer scrolls the panel up.
- **Leaner wording.** Trimmed the rambling hints and descriptions in the settings and README down to what helps.

### Removed

- **The input focus glow.** Text boxes get a clean border tint on focus instead of the glow ring.

## 2.5.0

_2026-07-18_

### Added

- **Keep your rerolls on retry.** Suggested by a Discord user. New toggle, "Retry by adding a new reroll", under a "How it redoes a reply" section. Off (the default), a retry redoes the reply in place with your regenerate button, which on some builds clears the other rerolls on that message. On, a retry clicks your next / swipe button instead, adding a fresh reroll and leaving existing ones in place. It falls back to regenerate if the swipe button is not found, so set that selector in the buttons section if retries stop after turning it on.
- **Heads up on repeated failures.** In the new mode each retry adds a reroll rather than replacing one, so a reply that fails a few times before it lands can leave a couple of empty or partial rerolls stacked next to the good one. Regenerate mode still replaces in place with no pile-up, so both behaviours are there to pick from.

### Changed

- **Easier to debug.** Copy debug info lists which retry mode is active, and the "couldn't find your button" messages no longer assume regenerate, so they read right in either mode.

## 2.4.0

_2026-07-17_

### Added

- **Optional "swap words now" button.** A toggle adds a button to the input's Extras menu that applies your word swaps on demand, so you can swap by hand instead of leaving automatic swapping on. Off by default.
- **More swap control.** Two new options: swap the whole chat at once instead of just the latest reply, and allow re-swapping a reply already swapped, useful after changing your rules. By default a reply is only swapped once, so swaps never stack.
- **Ask before editing.** New toggle that makes every swap, automatic or manual, ask for confirmation before changing a reply. Off by default.

### Changed

- **The greeting is never touched.** Word swaps only apply to generated replies. The opening message is always left alone, in both automatic and manual modes.
- **Clearer input focus.** Text and swap-rule boxes show a soft accent glow when focused instead of a barely visible outline.
- **Follows your theme.** The settings panel, buttons, live log and preview box use your Lumiverse theme's corner rounding instead of fixed values.
- **Tidier settings.** Find and replace moved under Advanced so it does not clutter the main list.

### Fixed

- **Sturdier under bad input.** A single malformed swap rule is skipped on its own instead of turning off all your swaps with no message, plus general error-handling hardening.

## 2.3.0

_2026-07-16_

### Added

- **Custom thinking tags.** Add any wrapper your model uses under "Extra thinking tag names", one per line. New "Ignore the thinking / reasoning" toggle, on by default.

### Changed

- **Refusals inside the model's thinking are ignored.** Only the final reply is checked. Reasoning blocks (`<think>`, `<thinking>`, `<reasoning>`, `<reflection>`, `<scratchpad>` and similar, plus `[tag]` forms) are stripped before matching.
- **Settings sync to your Lumiverse account.** They follow you across browsers and devices instead of living in one browser. Existing settings migrate up automatically.
- **Broader refusal detection.** Added apology-style refusals like "I'm sorry, but I can't create...", "that's not something I can help with" and "I'm not going to generate that". Written so in-character lines stay untouched.

### Fixed

- **Content blocks were dropped instead of retried.** Errors like "403 Forbidden: content blocked" now route to the accidental-refusal retry. Real auth, not-found and balance errors are still skipped.
- **Cleanup.** Removed an unverified `metadata` field from message edits, fixed a broken word-swap hint, and fixed checkbox theming.

## 2.2.0

_2026-07-15_

### Added

- **More retryable error codes.** Added 408, 500, 502, 503, 504 and Cloudflare 520 to 524. These temporary server failures now trigger exponential backoff retries. Also added `timeout`, `temporary` and `network` message patterns to catch network transients.
- **Expanded hard error detection.** Added `permission`, `forbidden` and `not allowed` to the existing auth checks, plus HTTP 411 and 415, to prevent retries on malformed-request errors where the problem is the request rather than the server.

## 2.1.1

_2026-07-15_

### Added

- **HTTP 402 (Payment Required) counts as a hard failure.** If a provider cuts you off for running out of credits or an unpaid balance, the extension recognises it immediately and stops retrying.

## 2.1.0

_2026-07-15_

### Added

- **A "Skip hard failures" toggle.** Permanent errors such as missing models, invalid API keys or dead-end HTTP codes now stop the extension immediately instead of leaving it in a useless retry loop. On by default, and can be turned off.

## 2.0.0

_2026-07-14_

### Changed

- **Rules and phrases are one per line.** Word swaps, custom refusal phrases, the whitelist and reword rules are no longer separated by commas.
- **Word swap and refusal settings use multi-line text areas.** Easier to manage lists, and Enter starts a new rule directly in the settings.
- **Commas can be used inside your swaps.** Because rules split on line breaks, you can swap phrases and full sentences containing commas, for example `Well, she left. => He stayed quietly.` **If you have multiple comma-separated entries on one line, move them onto separate lines.**

### Fixed

- **Word deletions clean up trailing spaces.** Leaving the right side empty to delete a word now swallows one trailing space, so removing a word mid-sentence leaves single spacing instead of a double space.

## 1.5.0

_2026-07-13_

### Added

- **That cap can be set to 0 to turn it off entirely**, so refusals are caught at any length. It stays safe because length alone never triggers a retry: a reply still has to match the refusal patterns, so a long scene will not be re-rolled just for being long.

### Changed

- **The "longest reply to treat as a refusal" cap defaults to 2000.** (was 1200). Some models write long, padded refusals (apology, "as an AI", a paragraph of reasoning, then offered alternatives) that ran past the old limit and slipped through. 2000 catches those while leaving genuinely long replies alone.

## 1.4.1

_2026-07-13_

### Changed

- **Renamed:** the "On / off" settings group to "Basics".
- **Renamed:** the "Notifications" import/export category to "On-screen", since it covers the retry pop-up and the live log.
- **Moved:** the retry pop-up toggle out of the old feedback group and up into Basics, since it is not an advanced option.
- **Regrouped:** the live log and the debug info section now sit next to each other as one debugging area, instead of being split apart by import/export, which moved to the bottom.
- **Reworded:** the import/export description, so it is clearer about what importing does.

### Fixed

- **Resizing the live log on mobile.** Dragging the panel's corner to resize now works on Android and other touch screens. It was mouse-only before, so there was nothing to grab on a phone.

### Removed

- **The "write technical details to the console" toggle.** Redundant now that the live log shows the same activity on screen and the debug report already captures it.

## 1.4.0

_2026-07-13_

### Added

- **Import and export your settings.** Open Advanced: import / export, tick which parts to include (retry behaviour, refusal detection, word swaps, button selectors, notifications), then Export to file to save them or Import from file to load one. An import puts the values into the settings without saving, so you can review them first, then press Save to keep them or close to discard.
- **Choosable debug info**, in a new Advanced: debug info section. Pick which parts to include (your settings, button match status, browser and screen, recent activity), build a preview, edit out anything you would rather not share, then copy. The old Copy debug info button in the footer is gone, replaced by this section.
- **A live log you can watch on screen**, under Advanced: feedback. A small panel shows recent activity as it happens: generations, retries and why, finishes. Handy on mobile especially, where the browser console is out of reach.

## 1.3.1

_2026-07-12_

### Fixed

- **Cleaned up a quote in one of the settings descriptions** so it displays properly.

## 1.3.0

_2026-07-12_

### Added

- **A word can have more than one swap.** List it more than once and it collects all the options. By default it uses the first one.
- **New random-pick toggle.** Turn it on and each time that word appears it picks one of its options at random, so "the sky... the sky" can become "the blue... the aqua".
- **New match-case-exactly toggle.** Off by default, swapping any case and keeping the original capitalisation. On, it only swaps when the case matches your rule, which also lets `sky` and `Sky` have different swaps.

## 1.2.0

_2026-07-12_

### Added

- **Accidental-refusal retry (beta).** When the model breaks character to refuse something harmless, it re-sends the exact same request and tries again, capped by your retry limit. It never changes your prompt or swaps any words, it just gives the reply another roll. A refusal the model really means will repeat and stop on its own.
- **Refusal detection you can tune**, under a new Advanced section: add your own refusal phrases, whitelist lines that should never count, reword the built-in ones, or switch the built-in list off entirely. The whole feature is one toggle to turn off.
- **Find and replace in replies (beta).** Swap words you do not like for ones you prefer, saved right into the reply. Whole words only by default, so "cat" will not touch "category". It keeps capitalisation, and an empty right side deletes a word. Off by default.
- **A new chat-editing permission.** (`chat_mutation`) alongside the existing generation one. Only used by find and replace, so it can save its edits to a reply. Depending on your setup it may need admin approval when you update, and if you never use find and replace nothing is touched.

### Changed

- **Advanced settings now collapse**, tucked behind a tap-to-open header so the basic switches stay front and centre.
- **README cleaned up and updated**, reorganised, and now covering the new settings and permissions.

## 1.1.6

_2026-06-27_

### Fixed

- **Contradictory wording in the frozen-reply settings.** The settings said to make the timeout numbers bigger for a slow connection or local model, but the defaults are already long for exactly that case. It now matches the README: the defaults lean long on purpose, and you lower them if your provider is fast and you want quicker retries.

## 1.1.5

_2026-06-27_

### Changed

- **The selector Test no longer overstates what it knows.** A failed match says "no match right now" instead of "not on screen right now".

## 1.1.4

_2026-06-27_

### Changed

- **Closing settings with the X discards unsaved changes.** Before, edits took effect the moment you made them, so closing without saving still left them applied to the current session, which was confusing. Now only **Save** keeps changes, and closing with X or tapping outside throws away anything unsaved. **Reset** counts as a change you made, so it sticks.
- **Testing an empty selector box gives a clearer message.** It says "type a selector first" instead of reporting no match right now.
- **The "buttons it clicks" settings are explained better.** It spells out that each box takes a CSS selector, that the three boxes are three different buttons for three different jobs, and that you can list a few separated by commas as fallbacks. The README matches.

### Fixed

- **Number settings cannot be left blank or broken.** Empty, non-numeric, negative or absurd values snap back to a sensible default or the nearest allowed limit, whether you blur the box, hit Save mid-edit, or load saved settings. Before, clearing a box could save a zero without saying so.
- **A rare timing bug where a retry could cancel itself.** If a stalled reply was aborted and its "stopped" signal arrived late, the extension could mistake it for you pressing Stop and cancel the retry it had just started. It now remembers aborted replies properly, so their late signals are ignored even after the next reply has begun.

## 1.1.3

_2026-06-26_

### Changed

- **Testing a button selector is clearer.** A button only exists in the page while it is on screen, so a correct selector will not match if that button is not showing. The result now reads "not on screen right now" instead of "no match", and the settings and README explain it: the **Stop** button only appears while a reply is generating, so test that selector mid-generation rather than from an idle screen.
- **Tightened up the wording** in the settings descriptions and README.

## 1.1.2

_2026-06-25_

### Fixed

- **Buttons respond to taps on mobile.** Before, they only reacted to mouse hover, which phones do not have, so pressing Save, Test or Cancel gave no feedback. The retry pop-up's Cancel button is also sized to match the others for an easier tap.
- **A settings hint showed the wrong example time**, so the "wait before the first retry" description matches the actual default.

## 1.1.1

_2026-06-17_

### Changed

- **Copy debug info includes a recent-activity log**, so a bug report shows what actually happened (generations starting, retries firing and why, a clean finish) rather than just a snapshot of your settings.

## 1.1.0

_2026-06-17_

### Added

- **A Cancel button on the retry pop-up**, so you can pull the plug even while it is counting down to a retry.
- **A Copy debug info button**, so reporting a bug is one tap with no dev tools needed.

### Changed

- **Settings reworded in plain language**, with a short description on each group.
- **Tuned the defaults** so a slow reply or a slow local model is not mistaken for a frozen one and retried into a pile-up.
- **Better on mobile**, with bigger tap targets and a layout that fits narrow screens.
- **README updated** with the new settings, defaults, and a how-to-report-a-bug section.

### Fixed

- **A couple of menu colours** were not matching the theme.

## 1.0.0

_2026-06-08_

First release.

---

[Back to the README](README.md)
