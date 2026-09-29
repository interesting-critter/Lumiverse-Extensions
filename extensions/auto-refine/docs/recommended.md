# What to use

Settings that work well for most people, and when to change them. Everything here is a starting point. Your model and your story may want something different.

## Start here

| Setting | Where | Use |
| --- | --- | --- |
| **Prompt** | Prompt tab | **A judge** for replies. **A line judge** for your own messages. |
| **Refine using** | Model tab | A fast, cheaper model that follows instructions well. It does not need to be the model you chat with. |
| **Let it think first** | Model tab | **No, keep it quick.** |
| **Temperature** | Model tab | Blank, or 0.5 to 0.7 if your model takes it. |
| **Every other sampler** | Model tab | Blank. |
| **Messages of run-up to send** | Context tab | 4, the default. |
| **Give up waiting after** | Model tab | 240, the default. |
| **Two models** | Model tab | Off, until you want to spend less. |

## Thinking on or off

**Leave thinking off for most refines.** A refine is an edit of a passage that already exists. The built-in prompts give the model clear rules, and a model can follow those rules as it writes.

Thinking costs time and tokens on every refine. A reasoning model reads the whole prompt and then thinks before it writes a word. On a slow or busy provider, that can take minutes.

**Turn thinking on when:**

- the model you use is a reasoning model and cannot turn thinking off
- your own rules need judgement, such as "keep each character's voice", more than a list does
- a refine without thinking keeps missing things your rules name

**Match the prompt to the setting.**

- Thinking off: use **A judge** or **A line judge**. They give rules the model can follow in one pass.
- Thinking on: use **A judge that thinks** or **A line judge that thinks**. They ask the model to score the passage first, and the scores are kept on the Log tab.

With thinking on, **Low** is enough for a refine. A higher level takes longer and rarely changes the result.

## Samplers

**Leave them all blank to start.** Blank means your connection's own preset decides.

**Temperature is the one worth changing.** It sets how loose the wording is.

- Lower, such as 0.5, keeps the rewrite close to the original.
- Higher, such as 1.0, lets the model reword more, and makes it more likely to add things nobody wrote.
- A rewrite usually wants a lower temperature than the one you roleplay with.

**Leave the rest blank unless you have a reason:**

- **Top P, Top K and Min P** narrow the model's choices. Your connection's values are fine for a refine.
- **Frequency, presence and repetition penalty** push the model away from words it has used. A refine repeats the passage's own words on purpose, so a penalty can make it change words that were fine.
- **Longest answer** set too low cuts the rewrite off, and the rewrite is then dropped as too short. Leave it blank, or set it well above your longest reply.
- **Context size** is almost always right blank.

**Some models refuse samplers.** Many reasoning models ignore them or return an error. If a refine fails after you set one, clear it.

## When a refine is slow

Most of the time is the model, not the extension. A refine is one model call. It only becomes more than one when:

- an answer fails a check and **Ask again when a check fails** asks again
- **Two models** is on, and the second model reads the reply first
- the provider is busy, and **Wait out a provider that will not take the call** waits

To make it faster:

- Turn **Let it think first** off.
- Send less on the Context tab. Less run-up and less lorebook means less for the model to read.
- Pick a faster model in **Refine using**.

The Log tab shows how long each refine took, so you can compare settings.

## When the rewrite changes too much

- Use **A judge** or **A line judge** as they come, before you add your own rules.
- Lower the temperature.
- Turn **Ask before saving a refine** on (Limits tab), to see each rewrite before it is saved.
- Look at [Making your own prompt](rules.md) for how to write rules a model follows.

## When the rewrite changes nothing

- A rewrite with nothing to fix comes back unchanged, and that is a correct result.
- If a passage has clear faults and still comes back unchanged, try a larger model in **Refine using**.
- Check that the prompt blocks you need are switched on, on the Prompt tab.

---

[Back to the README](../README.md)
