<!-- Mirror of upstream documentation. Everything below the metadata table is the original file, unmodified. -->

| Field | Value |
| --- | --- |
| Extension | Auto Refine |
| Source repository | https://github.com/starlitcode/Lumiverse-Auto-Refine |
| Original link (from Lumiverse-Extensions README) | https://github.com/starlitcode/Lumiverse-Auto-Refine |
| Upstream path | `README.md` |
| Retrieved | 2026-09-29 @ `stable` (`e490633`) |

---

![Auto Refine: a woman writes at a desk on a balcony at night, with messy pages beside her and clean pages floating away into the sky.](docs/auto-refine-banner.png)

# Auto Refine

A Lumiverse extension. It sends a finished reply to a model with the rules you wrote, and saves the rewrite over the original.

- **It never touches the greeting.** A person wrote that message, and no setting changes this.
- **Nothing is written that you cannot undo.** **Put it back** restores the wording a refine replaced.

It works alongside [Auto Retry](https://github.com/starlitcode/Lumiverse-Auto-Retry). Auto Retry decides whether a reply is worth keeping. Auto Refine improves the ones that are.

## What it does

- **Refines replies.** Press a button to refine the latest reply, or go through every reply in a chat. Automatic refining of each new reply is off until you turn it on.
- **Refines part of a reply.** Select some text and refine only that, or take it out.
- **Refines messages you have already sent.** Press the refine button on one of your own messages in the chat. This is never done automatically. Your messages get their own prompt, which fixes mistakes and leaves your style alone.
- **Refines what you are still typing.** An optional button rewrites the draft in the input box, before you send it.
- **Shows what changed.** After a refine, the tab shows the before and after, either mixed in one text or side by side.

It lives in a tab in the sidebar drawer, so you can keep it open while you write.

## Install

1. In Lumiverse, open **Extensions**.
2. Install from this address:

   ```
   https://github.com/starlitcode/Lumiverse-Auto-Refine
   ```

3. Open the sidebar drawer and pick the **Auto Refine** tab.

A prompt comes with it, so switching it on is all the setup you need. Everything else is optional. For the settings that work well for most people, see [What to use](docs/recommended.md).

## You are in control

- **The switch at the top** stops everything. No refine runs and no model is called.
- **Turn off here**, on the Setup tab, leaves one chat alone.
- **Ask before saving a refine** shows you each rewrite before it is written.
- **Put it back** undoes a refine.

## Not for sexual content involving minors

Auto Refine is not intended for sexual content involving minors, and I do not condone or support anybody using it for that. It is a writing tool for fiction between adults, and for stories with younger characters in them where nothing sexual happens to those characters.

So nobody has to guess what that means for their own writing, here is exactly how it works:

- **The code does not judge your story.** It does not read your writing for this, keeps no list of words about it, and does not block, change or delete anything on its own account.
- **The four built-in prompts carry one paragraph about it.** It tells the model that sex scenes are between adults, 18 and older. A sexual scene involving anyone younger, or anyone written to seem younger, is not edited: the model hands that passage back exactly as it came and says why. It sits in the first block of each, **The Judge** or **Line Judge**, where you can read it, and word for word on [How the prompt is built](docs/prompt.md#the-one-thing-they-do-not-edit).
- **When a model follows it, your reply is left exactly as it was.** Nothing is saved, and the Log says the model handed it back and why.
- **Nothing else is excepted.** A younger character in a scene with nothing sexual in it is edited like anyone else, and the tone of a story between adults, from gentle to dark, violent or sexually explicit, is yours to decide.
- **A prompt you write yourself carries only what you put in it.**

I cannot control what somebody does with an extension once they have it. What I can do is say plainly where I stand, and make sure nothing in this one is built to help.

## What it checks before saving

A rewrite is dropped, and your reply is left as it was, when:

- the model talks about the edit instead of making it, like "Here is the rewritten message"
- the model declines, sends nothing, or changes nothing
- the rewrite is much longer or shorter than the reply (you set both limits)
- the rewrite lost your formatting, such as tags, code or image links

Also:

- The model's own thinking is never sent, so a rewrite cannot change it.
- The original is kept so you can put it back, for as long as the page is open.
- Every check says why it fired, in the Log tab.

More in [What it refuses to save](docs/guardrails.md).

## The tabs

| Tab | What is on it |
| --- | --- |
| **Prompt** | The request, block by block, the macros, and your presets |
| **Context** | How much of the chat and lorebook goes in, and a preview of what is sent |
| **Model** | Which connection refines, thinking, the timeout, samplers, and two models (beta) |
| **Limits** | What it protects, what it refuses to save, and what it does before it writes |
| **Log** | What is happening now, what it did, what the last refine cost, and a bug report button |
| **Setup** | This chat, alerts, the optional buttons, import and export, and resetting |

The switch, the refine buttons and the refines you can put back sit above the tabs.

## What it costs

A refine is a second model call on every reply it refines. To spend less:

- **Refine using** (Model tab): point it at a cheaper model than the one you chat with.
- **Let it think first** (Model tab): off by default. Rewriting does not need a reasoning model.
- **How much it is told** (Context tab): less chat history means a cheaper call.
- **Two models** (Model tab, beta): a small second model, Jev or Span, checks each reply first, so only the ones that need it are refined. It can be wrong, so a reply that needs a refine is sometimes left alone. See [It can be wrong](docs/two-models.md#it-can-be-wrong).

Put your provider's prices on the Model tab to see costs in money instead of tokens. On a free tier or a local model, **Wait out a provider that will not take the call** (Limits tab) waits and tries again when the provider is busy.

## Your prompt

The whole request is a list of blocks you can edit, reorder, switch off or add to. Macros like `{{message}}`, `{{history}}` and `{{description}}` are filled in when the refine runs.

Four prompts come with it:

- **A judge**, for replies. It improves how a reply reads and leaves what happens alone.
- **A line judge**, for your own messages. It fixes mistakes and leaves your style alone.
- Each has a smaller version **that thinks**, for a model that reasons.

All four change only a line the model could quote as breaking a rule. The two for a model that thinks score each area and write the scores down. The two plain ones give the same limit as a plain rule, and never ask the model to check its work, because a model that does not reason writes its answer once. All four work with one character or several, and in group chats. See [The scorecard](docs/prompt.md#the-scorecard).

**Show me the request**, on the Context tab, shows the request a refine of that reply would send, without calling a model or costing anything. Two things only a real refine can fill in are named rather than shown: what the checks found, and the passes after the first.

## Documentation

- [What to use](docs/recommended.md): the prompt, model, thinking and sampler settings to start with
- [How the prompt is built](docs/prompt.md): blocks, macros, roles, the built-in prompts, and the preview
- [Making your own prompt](docs/rules.md): a step-by-step guide to a prompt that fits your story and your model
- [What it refuses to save](docs/guardrails.md): the checks on what comes back
- [Settings](docs/settings.md): every tab and setting
- [Ways to reach it](docs/settings.md#setup): the floating button, the chat buttons, and refining a selection
- [Presets](docs/prompt.md#presets) and [Import and export](docs/prompt.md#import-and-export)
- [Two models](docs/two-models.md): a beta where a second model, Jev or Span, picks which replies to refine
- [Privacy](docs/privacy.md): what it can reach, what it sends, and what it keeps
- [Security policy](SECURITY.md): how to report a security problem
- [Changelog](CHANGELOG.md): what changed in every version

## How it works

- The refining runs in the extension's backend, because editing a saved message is a backend job.
- Every refine goes through Lumiverse to the provider you set up. With two models on, the call to the second model goes through Lumiverse's own proxy to the host you picked.
- The extension has no networking of its own. Search the two source files for `fetch(`, `XMLHttpRequest`, `WebSocket`, `sendBeacon` or `EventSource` and you will find nothing.
- **Nothing is ever added to your chat.** It only edits messages that already exist.
- **Refine what I am typing** reads and writes the input box on the page, because Lumiverse has no API for it. If an update moves the box, **Where the input box is** on the Setup tab points it at the new one.
- The files Lumiverse loads are `dist/frontend.js` and `dist/backend.js`. They are plain, readable JavaScript. Nothing is minified.

It asks for seven permissions:

| Permission | What it is for |
| --- | --- |
| `generation` | running the refine |
| `chat_mutation` | saving the rewrite |
| `chats` | knowing which chat you are in, and what Lumiverse remembers of it |
| `characters` | reading the character card |
| `world_books` | reading the lore the chat has active |
| `ui_panels` | the floating button |
| `cors_proxy` | reaching the second model, when two models are on |

[Privacy](docs/privacy.md) says what still works without each one.

## Credits

- **starlitcode**: built and maintains the extension
- **[Claude](https://claude.ai)** (Anthropic): wrote the code, directed and tested by starlitcode
- **[Hone](https://github.com/AMousePad/Hone)**: the extension that showed refinement was worth having. Auto Refine is written from scratch and shares no code with it
- Everyone who has reported a bug or suggested something that turned into a fix

Licensed under the GNU General Public License v3.0. See [LICENSE](LICENSE).
