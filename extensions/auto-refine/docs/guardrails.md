# What it refuses to save

This extension sends your writing to a model and saves what comes back. Most of it is about not saving the wrong thing.

Every check below leaves the reply exactly as it was, and writes a line in the panel saying which check it was. Nothing fails without telling you.

## The greeting, always

The opening message is written by a person, so it is never refined. The automatic pass skips it, the buttons skip it, and no setting changes that. Asking for it by name is refused too.

This is the only rule in the extension with no switch.

## The model answered the wrong question

A rewrite is dropped when it is:

- **A preamble**, such as "Here is the rewritten message", "Sure!" or "I've polished this for you". The model wrote about the edit instead of making it. Saving it would put that line in your chat as if the character said it.
- **A refusal.** The model declined the job. Saving it would replace your scene with an apology.
- **Empty**, or the same text back. Neither is a rewrite.

## The rewrite softened the reply

A softened rewrite is not a refusal, is the right length, and keeps every protected token. It only looks wrong next to the original, so the other checks cannot see it.

**Refuse a rewrite that sanitised the reply** counts the strong words in each version: the explicit and violent ones. If the rewrite dropped most of them, it is refused and your reply is left as it was.

It is built to avoid false alarms:

- The built-in list only holds words that are hard to use innocently. Everyday words like hit, skin or pain are left out, because they are normal description.
- The reply needs at least three strong words before the check can fire.
- **How much of it may go** sets how many may be dropped, 60% by default, so losing one or two counts as an ordinary edit.

Because the list is narrow, it misses some real softening. That is on purpose. A missed one leaves your reply as it was. A false one would throw away a good rewrite. **Words of your own to watch** is for the words that matter in your own writing.

### The word that arrives in its place

When a model softens a scene, it often swaps a blunt word for a softer one: killed becomes unalived, blood becomes crimson liquid, sex becomes intimacy. The same check looks for these swaps, from a built-in list of pairs.

A pair only counts when **both halves happen**:

1. The soft word is in the rewrite and was not in your reply.
2. The blunt word it replaces was in your reply and is gone from the rewrite.

Either half alone proves nothing. A model can write "eliminate" about a possibility, and a refine can drop "kill" while tightening a line. Needing both is what lets ordinary words like eliminate, defeat or intimacy be on the list.

One confirmed swap is enough. A reply whose only "killed" came back as "unalived" has been softened, even though one word is below what the counting check needs.

**Swaps of your own to watch** takes one pair a line, written `soft => blunt`. An exported find-and-replace list can be pasted in as it is. A line with nothing on the right is skipped.

What it cannot catch:

- A rewrite that keeps every word but softens the scene another way, such as fading out at the key moment or going vague.
- It can be wrong on an idiom. If your reply said "killed the engine" and the rewrite says "eliminated the noise", the rewrite is refused. That costs you the rewrite, never your reply.

## Asking again

**Ask again when a check fails** is 0 by default. A failed check is often a one-off, and the same request can come back fine.

- Only failures that a second try could fix are retried.
- A rewrite refused for its length is not retried, because the model meant it and would give the same answer.
- A call that errored, or one you stopped, is never repeated.
- Every retry is another call on your bill, which is why it is off by default.

## The limit on one reply

The automatic pass refines a reply again when its words are new. That is what lets it work alongside [Auto Retry](https://github.com/starlitcode/Lumiverse-Auto-Retry): every reply Auto Retry re-rolls is new, and each gets its refine.

But the same test cannot tell new writing from a loop, such as:

- another extension rewriting what this one wrote
- Lumiverse announcing the same reply under new ids
- a chat being swiped through very fast

So the automatic pass refines one reply at most twelve times, for as long as Lumiverse is running. It is a safety stop. Nobody reaches it in normal use.

Pressing the refine button on a reply always works, whatever the count. The Log says when the automatic pass has stopped on a reply, and why.

## Waiting out a provider that will not take the call

**Wait out a provider that will not take the call** is on by default, and tries twice. It is different from **Ask again when a check fails**:

- A failed check was paid for. The model read the prompt and wrote something.
- A refused call was never read, so nothing was spent. Waiting and asking again gets the refine you already asked for.

It waits on:

- "too many requests"
- a key at its limit
- a server saying it is overloaded
- a local server answering 503 while it loads a model

It never waits on a wrong key, a model that does not exist, or a prompt too long for the model. Waiting cannot fix those.

How long it waits:

- If the provider says how long, it waits exactly that. Waiting less would only get the same answer.
- The most it waits is an hour. A provider asking for longer is naming a daily limit.
- If the provider does not say, it waits fifteen seconds, then doubles each time.

The status line counts down while it waits. **Stop** ends the wait and the call.

## The rewrite is the wrong size

- **Too long.** A rewrite that grows the reply by more than your limit has written new scene. The default limit is 60%.
- **Too short.** A rewrite that shrinks it by more than your limit has thrown writing away. The default limit is 40%.

Set either to 0 to allow any length. Before raising the growth limit, read [Making your own prompt](rules.md#rules-to-keep-in-every-prompt): a rule that asks for more writing is asking for a new reply.

## Wrapping, which is fixed instead of dropped

Two kinds of answer are unwrapped instead of refused, because the rewrite inside is fine:

- **The whole answer in quotation marks.** The quotes come off, but only when there are no other quotes inside, so a reply that starts and ends with dialogue keeps its own.
- **The whole answer in a code fence.** The fence comes off.

## The reply changed while it was being rewritten

A refine reads the reply, sends it to a model, and writes the answer back. That takes seconds, and the reply can change in the meantime.

- If the reply changed before the answer arrived, **the refine is dropped and nothing is written**. The refine can be run again on the new text. An edit written over cannot be recovered.
- This covers you editing the reply while you wait, and another extension writing to it at the same time.
- **Thinking moved out of the reply does not count as a change.** If only the thinking at the start or end of the reply was moved, for example into the Reasoning box, the refine is saved. It is saved without the thinking that was moved out.
- If the reply cannot be read again at all, the refine goes ahead. A failed read does not mean anything changed.

## Putting one back

**Keep what a refine replaced** is on by default. It keeps the text from before each refine, so you can put it back from the panel.

It is kept in memory while the page is open, and never written anywhere. It is there to undo a refine you did not like. It keeps no record of your chat.

---

[Back to the README](../README.md)
