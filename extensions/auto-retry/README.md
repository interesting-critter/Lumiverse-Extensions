<!-- Mirror of upstream documentation. Everything below the metadata table is the original file, unmodified. -->

| Field | Value |
| --- | --- |
| Extension | Auto Retry |
| Source repository | https://github.com/starlitcode/Lumiverse-Auto-Retry |
| Original link (from Lumiverse-Extensions README) | https://github.com/starlitcode/Lumiverse-Auto-Retry |
| Upstream path | `README.md` |
| Retrieved | 2026-09-29 @ `stable` (`9a92192`) |

---

![Auto Retry: on a rooftop at night, a woman smiles up at a new glowing chat bubble above her hand while an old cracked one turns to dust.](docs/auto-retry-banner.png)

# Auto Retry

A Lumiverse extension. It re-runs an AI reply when it fails, comes back empty, stalls partway, gets cut off mid-sentence, or refuses by mistake, so you do not have to catch it and press regenerate yourself.

It works alongside [Auto Refine](https://github.com/starlitcode/Lumiverse-Auto-Refine). Auto Retry decides whether a reply is worth keeping. Auto Refine improves the ones that are.

## What it does

It watches each reply and tries again when:

- the provider sends back an error (by default it skips errors that will not go away, like a wrong API key)
- the reply comes back empty, including one that "thinks" but never writes anything
- the thinking or the reply is one character over and over, like `!!!!!!!!` (see [One character over and over](docs/detection.md#one-character-over-and-over))
- the reply is cut off mid-sentence (see [Cut-off detection](docs/detection.md#cut-off-detection))
- the reply is an accidental out-of-character refusal, or the model breaks off mid-scene
  (see [Accidental-refusal detection](docs/detection.md#accidental-refusal-detection))
- the reply stops arriving partway and nothing more comes for a while
- a reply never starts or never finishes
- (optional, off by default) the reply is very short
- (optional, off by default) the model leaves the scene to offer real-world support
  (see [Stopping to offer support](docs/detection.md#stopping-to-offer-support), and [Safety](docs/safety.md) before switching it on)

Also:

- Every retry waits a little longer than the last, and longer again when the server says it is busy.
- All the reasons share one retry limit, so no reply is retried more than you allow, and it can never loop forever.
- A reply written by **Impersonate** into your input box is never retried.

## Install

1. In Lumiverse, open **Extensions**.
2. Install from this address:

   ```
   https://github.com/starlitcode/Lumiverse-Auto-Retry
   ```

3. To see its settings, open the chat input bar, tap **Extras**, and choose **Auto Retry settings**.

It works with no setup. Everything else is optional.

## You are in control

- **Stop**, or **Cancel** on the retry pop-up, stops it right away. Any waiting retry is dropped, the count resets, and a stopped reply cannot start again by itself.
- **Cancel** on the pop-up belongs to the extension, so it always works.
- To switch it off for one chat, or everywhere, see [Turning it off](docs/settings.md#turning-it-off-in-one-chat).

## Not for sexual content involving minors

Auto Retry is not intended for sexual content involving minors, and I do not condone or support anybody using it for that.

So nobody has to guess what that means for their own chats, here is exactly what it does:

- **It never reads your story to judge it, and never changes a word of it.** It watches how a reply ended and, when that looks wrong, presses Lumiverse's own retry button. It has no rules about what a story may contain.
- **A retry asks your model again for the same reply.** Whether anything is written is still up to your model and your provider. Retries stop at **Most tries per message**, 4 by default.
- **It recognises a refusal about a character's age for one reason:** models sometimes misread an adult character, written as an adult, as a minor, and that refusal should not have happened. It cannot tell a mistaken refusal from a correct one, so with **It looks like an accidental refusal** on, which it is by default, any refusal it recognises is retried the same way, up to that limit. It is not there to get sexual content involving a minor past a model.
- **A provider that blocks a request before anything is written is treated as a refusal too**, for the same reason, and with the same limit.
- **The retry notes are off by default.** Switched on, one is sent with a refusal retry. None of them mentions age or any subject, and every word of each is on the panel before you pick it.

The words it recognises refusals by, including the ones about age, are listed on [When it retries](docs/detection.md). None of them reaches a prompt.

I cannot control what somebody does with an extension once they have it. What I can do is say plainly where I stand.

## Documentation

- [When it retries](docs/detection.md): cut-off detection and accidental-refusal detection
- [All settings](docs/settings.md): every option with its default, the panel, and turning it off
- [Buttons it clicks](docs/buttons.md): fixing the regenerate button, Regeneration Feedback, writing selectors
- [The on-screen panel](docs/settings.md#the-on-screen-panel): the log, the prompt viewer and the stats
- [Import and export](docs/import-export.md): moving your setup between devices
- [Reporting a bug](docs/troubleshooting.md)
- [Safety](docs/safety.md): who this is built for, the one setting that asks before it turns on, and what the retry loop can turn into
- [Privacy](docs/privacy.md): what the extension can and cannot reach, what it keeps, and how to check any of it
- [Security policy](SECURITY.md): how to report a security problem
- [Changelog](CHANGELOG.md): what changed in every version

## How it works

- It listens to Lumiverse's own generation events. When a reply goes wrong, it presses your swipe or regenerate button to try again.
- Pressing that button is the only part that depends on the page layout. If a Lumiverse update moves the buttons, [Buttons it clicks](docs/buttons.md) shows how to point it at the new ones.
- A small backend keeps your settings with your account, not in one browser, and holds the retry note for the one reply it is meant for.
- It makes no network calls of its own. [Privacy](docs/privacy.md) covers the five permissions it asks for and what still works without each one.
- The files Lumiverse loads are `dist/frontend.js` and `dist/backend.js`. They are plain, readable JavaScript. Nothing is minified. [Privacy](docs/privacy.md#checking-any-of-this-yourself) goes through the rest of the repo file by file.

## Credits

- **starlitcode**: built and maintains the extension
- **[Claude](https://claude.ai)** (Anthropic): wrote the code, directed and tested by starlitcode
- **[Hikarushmz](https://github.com/Hikarushmz/fetch-retry)**: their SillyTavern fetch-retry gave me the idea. Auto Retry is written from scratch and shares no code with it
- Everyone who has reported a bug or suggested something that turned into a fix

Licensed under the GNU General Public License v3.0. See [LICENSE](LICENSE).
