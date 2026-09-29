<!-- Mirror of upstream documentation. Everything below the metadata table is the original file, unmodified. -->

| Field | Value |
| --- | --- |
| Extension | Multi Image Gen |
| Source repository | https://github.com/EmpeQ17/Lumiverse-Auto-Image-Gen |
| Original link (from Lumiverse-Extensions README) | https://github.com/EmpeQ17/Lumiverse-Auto-Image-Gen |
| Upstream path | `README.md` |
| Retrieved | 2026-09-29 @ `main` (`c402a39`) |

---

# Multi-Image-Gen

A Lumiverse  extension that watches AI responses for `[img: description]` tags, generates images via ComfyUI using your Dream Weaver workflow, and inserts the results directly into the chat message.

Created it mostly just for ability to create multiple images from a single LLM call.

## Summary

When the AI writes something like `[img: masterpiece, best quality, 1girl, standing in a field]` anywhere in its response, Auto Image Gen:

1. Strips the `[img: ]` wrapper, leaving the prompt text in the message
2. Sends the prompt to your default ComfyUI image gen connection
3. Patches your Dream Weaver workflow with the prompt, negative prompt, seed, resolution, and checkpoint model
4. Appends the generated image to the end of the same message as it completes

Multiple `[img: ]` tags in one message are all processed. Images appear live as each one finishes generating.

## Settings

Open the **ImgGen** tab in the sidebar drawer to configure:

- **Enabled** — toggle the whole extension on or off
- **Model** — dropdown populated from your ComfyUI connection's available models (mapped to the checkpoint node in your workflow)
- **Negative Prompt** — textarea for your negative prompt (injected into the workflow's negative prompt node)
- **Width / Height** — resolution in pixels (injected into the workflow's latent image node)
- **Insert into chat** — whether generated images are appended to the message

Settings persist across restarts.

## Requirements

- A ComfyUI image gen connection configured in Lumiverse with a Dream Weaver workflow imported
- At minimum, your workflow must have the **positive prompt** field mapped

## Permissions

| Permission | Why |
|---|---|
| `image_gen` | Call `spindle.imageGen.generate()` with your ComfyUI workflow |
| `chat_mutation` | Edit messages to strip `[img: ]` wrappers and append generated images |
| `images` | Read generated image metadata (URLs and ownership) |

## Installation
Lumiverse --> extend --> install from source

```
https://github.com/EmpeQ17/Lumiverse-Auto-Image-Gen
```
Allow the permissions before turning on the extension.

## Usage

1. Install and enable the extension
2. Open the **ImgGen** settings tab in the sidebar drawer
3. Configure your negative prompt, resolution, and model
4. Instruct your character card to output `[img: prompt goes here]` when it should generate an image
5. The extension handles everything from there

## How It Works

1. The extension listens for `MESSAGE_EDITED` events (fires when AI responses finish streaming)
2. Each message is scanned for `[img: ...]` tags using a regex that handles multi-line prompts inside brackets
3. Tags are removed from the message content, replaced with their prompt text only
4. The Dream Weaver ComfyUI workflow is cloned from your connection.
5. Each `field_mappings` entry routes values to the correct node/input — `positive_prompt` to your text encode node, `negative_prompt` to the negative encode node, `seed/steps/cfg/sampler/scheduler` to the KSampler, `width/height` to the latent image, `checkpoint` to the LoadCheckpoint
6. The patched workflow is sent to ComfyUI with `workflowFormat: "api_prompt"`
7. Generated images are persisted with `owner_character_id` and `owner_chat_id`
8. Image markdown (`![prompt](gallery-url)`) is appended to the original message, updating live as each image completes

Multiple `[img: ]` tags in one message are processed sequentially so each image appears as soon as it's ready.

## License

MIT

## Credits

Basically whole extension was made with usage of https://github.com/AMousePad/LumiAgent 
MousePad the GOAT
