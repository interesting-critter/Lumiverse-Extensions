<!-- Mirror of upstream documentation. Everything below the metadata table is the original file, unmodified. -->

| Field | Value |
| --- | --- |
| Extension | Auto Img |
| Source repository | https://github.com/sca255/lumiverse-autoimg |
| Original link (from Lumiverse-Extensions README) | https://github.com/sca255/lumiverse-autoimg |
| Upstream path | `README.md` |
| Retrieved | 2026-09-29 @ `main` (`2587831`) |

---

# lumiverse-autoimg

Lumiverse/Spindle backend extension that generates images **only when the assistant intentionally emits a tag**, then replaces that tag in the assistant message with the generated image markdown.

## Behavior

1. Adds a prompt interceptor instruction telling the model to use an image tag sparingly, only when it improves immersion.
2. Watches assistant messages for this tag format:

```
[[AUTOIMG: your image prompt here]]
```

3. Generates the image from the extracted prompt.
   - The extension always appends this LoRA suffix at the end of the generation prompt:
     `<lora:Anima Turbo LoRA v0.2:1>`
   - Existing character LoRA tokens and tag prefixes in the prompt are preserved.
4. Replaces the tag in that same assistant message with:

```
![Generated scene image: ...](...)
```

No tag = no image generation.

## Files

- `spindle.json` — extension manifest
- `src/backend.js` — runtime logic (interceptor + tag detection + replacement)

## Install

1. Put this extension folder where your Lumiverse server can read it.
2. Install it from the Lumiverse Extensions panel (or your existing extension install flow).
3. Grant required permissions:
   - `interceptor`
   - `image_gen`
   - `chat_mutation`
