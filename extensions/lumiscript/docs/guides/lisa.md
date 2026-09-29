# Working with Lisa

Lisa is the in-app assistant for LumiScript scripting questions — ask her what the right primitive is, paste a script and ask why it isn't firing, or have her draft one, all without leaving the editor.

Lisa lives inside the LumiScript panel and answers questions about LumiScript's `api.*` surface and the host Spindle platform. She's a Q&A and code-drafting helper, not an autonomous agent: she doesn't run your scripts, and she can't see your live chat, your characters, or your variables. She knows what you ask her, plus any scripts you attach to the conversation (see [Attaching your scripts](#attaching-your-scripts)) and a small set of durable notes she keeps about you (see [Memory](#memory)).

## Opening a chat with Lisa

Click the **Lisa** button in the editor's top bar — the speech-bubble icon, alongside **Run** and **Reload** — or the speech-bubble button in the **script-manager toolbar** (next to Import / Export / the bundle button). Either opens the chat as an overlay.

A fresh chat shows a few example prompts to get you going. The composer sends on **Enter** and inserts a newline on **Shift+Enter**; while she's answering, the send button turns into a **Stop** button that aborts the turn. You'll see "Lisa is thinking…" before the first token, then her reply streams in.

## What Lisa knows — and when to trust her

Lisa's knowledge comes from the same corpus the in-app **Reference** tab is built from: every `api.*` method, its signature, permissions, and examples — plus the extras the Reference renders as prose: per-namespace concept paragraphs, the trigger- and permission-model intros, and a set of redirects that steer her off common wrong guesses (like reaching for an `api.on(...)` that doesn't exist).

When a one-line summary isn't enough, she looks the method up in full mid-answer — you'll sometimes see a brief lookup step in the transcript. That's her drilling into the reference rather than guessing at a signature.

**The Reference tab is the source of truth, not Lisa.** She's good, but she's a language model working from a snapshot, and she can still get a signature or an enum value wrong. For anything load-bearing, confirm it against the Reference tab — or just ask her to look it up. If the *Reference itself* is wrong or unclear, that's a corpus bug: flag it and we fix it at the source.

What she can't do, by design: run or test your scripts, read your active chat or character data, or touch your variables. She reasons about code; she doesn't execute it.

## Attaching your scripts

To ask about your own code, type **`@`** in the composer and pick a script from the menu. It attaches as a chip above the input and rides along as context for the turn. Attach as many as you need — the menu lists all your scripts and narrows as you type.

Attachments are **per-conversation and persistent**: reopen the thread later and they're still attached. Each script's code is included up to ~24 KB. If you edit an attached script between turns, Lisa is told it changed, so she reasons about the current version rather than the one she first saw.

Applying her code back into a script is a separate action. Every code block in her reply has an **Apply** control on hover:

- With no script attached, it creates a **new** script from that block.
- With scripts attached, it offers **Create new script** or **Update «that script»**. *Update* overwrites the script's code in place — after a confirmation, since it replaces what's there.

Either way, a "Created / Updated «script» from this conversation" marker is added to the transcript, so you have a record of what was applied where.

## Attaching files

Sometimes the thing you want Lisa to read isn't a script — it's a design note, a spec, or a snippet of reference material. Click the **paperclip** beside the composer to open the file picker. It lists text files from a dedicated **"userfiles"** folder in your LumiScript storage; click one to attach it as a chip, the same way scripts attach.

You don't need a script to put files there — the picker has an **Add a file** form (give it a name like `notes.md` and paste the text). You can delete files from the picker too, so it doubles as a small reference library you build up over time. Allowed types are text formats (`.md`, `.txt`, `.json`, `.csv`, and a few more), up to 256 KB each.

File attachments behave like script attachments: **per-conversation and persistent** (they restore when you reopen the thread), re-read fresh each turn, and flagged if they change between turns. Lisa treats their contents as **reference material, not instructions** — so a doc that happens to contain "ignore your previous instructions" is read as data, not obeyed.

Two things to know about scope: only that one reserved folder is ever visible to the picker (your scripts, threads, and other LumiScript data are never exposed as attachable), and this is **local files only** — pulling a file straight from a URL is planned for a later release.

## Memory

Lisa keeps a small set of **durable notes about you** — stated preferences, recurring project facts, and corrections you've made — and carries them across conversations. This is separate from attached scripts: attachments are the *live* code for one thread; memory is the *durable* context that follows you into every chat. (It's also unrelated to [Memory Cortex](memories.md), which is chat-memory your *scripts* manage — different feature, similar name.)

Open the memory panel with the **notebook icon** in the chat header ("Memory — what Lisa remembers about you"). Everything she's saved is listed there, tagged by who wrote it (**Lisa** or **you**), and you can add, edit, or delete any note — you're always in control of what she keeps. The **Consolidate** button (it needs a handful of notes — six or more — before it'll run) asks the model to merge duplicates and tidy the wording. It's safe by construction: if the pass fails for any reason, your notes are left exactly as they were.

Memory never overrides the Reference. If a note ever conflicts with the API surface, the API wins — notes are for *your* facts and preferences, not for API details.

## Choosing Lisa's model

Lisa runs on one of your Lumiverse LLM connections, independent of your main chat model. That's useful when a model that's unremarkable for roleplay turns out to be an excellent coding brain.

- **Per session:** the **Connection** picker at the top of the chat — switch any time, even mid-conversation.
- **As the default:** Settings → Assistant → **Connection**. Pick one here and every new chat starts on it; leave it on *Lumiverse default* to follow your app-wide default. The per-session picker always overrides this when you want a one-off.

## Threads & tuning

Past conversations live in the left sidebar. **New chat** starts a fresh thread; each thread row reveals **rename**, **export** (downloads the conversation as Markdown), and **delete** on hover. To clear everything at once, use **Clear all threads** in Settings → Assistant.

That same Assistant settings section has a few knobs for how she generates:

- **Tool iterations** — how many lookups she may chain in a single turn (2–20, default 8). Lower it if a model thrashes on hard questions; raise it if she's hitting the ceiling on genuinely complex ones.
- **Temperature / Top-P / Max tokens** — leave blank to use the connection's own preset, or set them to override.
- **Parallel tool calls** — on by default; uncheck only for providers that choke on parallel tool use.

## Tips for good answers

- **Be specific about the goal.** "I want to debounce a handler that fires on every keystroke — what's the right primitive?" beats "how do timers work?"
- **Attach the script you're asking about.** She reasons far better about concrete code than about a paraphrase of it.
- **Verify the surface details.** Treat signatures and enum values as something to glance at the Reference for, especially for anything you're shipping.
- **Correct her, and let her remember.** If she gets one of your project's conventions wrong, tell her — she can save the correction so the next chat starts from it.

For the authoritative, always-current API surface, the in-app **Reference** tab is the place. These notes are about working *with Lisa* — they're not a substitute for it.
