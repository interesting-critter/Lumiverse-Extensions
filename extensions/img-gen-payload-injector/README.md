<!-- Mirror of upstream documentation. Everything below the metadata table is the original file, unmodified. -->

| Field | Value |
| --- | --- |
| Extension | Img Gen Payload Injector |
| Source repository | https://github.com/WanderingMonk8/img-gen-payload-injector |
| Original link (from Lumiverse-Extensions README) | https://github.com/WanderingMonk8/img-gen-payload-injector |
| Upstream path | `README.md` |
| Retrieved | 2026-09-29 @ `main` (`897bfab`) |

---

# Image Payload Injector

A small [Lumiverse](https://lumiverse.chat/) extension for adding custom JSON fields to NanoGPT image generation requests. It also adds one-click image generation shortcuts to each message toolbar and right-click menu.

## Install

In Lumiverse, open **Extensions → Add Extension → Install from Source** and enter:

```text
https://github.com/WanderingMonk8/img-gen-payload-injector
```

Requires Lumiverse 1.2.0 or newer. Enable the extension after installation.

## Use

Open the **Image Payload Injector** drawer tab, select a NanoGPT image connection, enter a JSON object, and click **Save override**. For example:

```json
{
  "showExplicitContent": true,
  "resolution": "1k",
  "aspect_ratio": "1:1"
}
```

The editor also accepts a captured `{ "body": { ... } }` object or a full NanoGPT capture with `request_example.body`. It saves every body field you enter without filtering. These fields are merged into future requests made through the selected connection. If you include `prompt` or `model`, those values may override Lumiverse's current prompt or selected model. **Clear override** removes the injected JSON while retaining the connection's other settings.

## Message shortcuts

Hover a chat message and click the image icon after the speaker icon, or right-click a message and choose **Generate image** immediately before the TTS action. Both shortcuts use the active image connection, prompt mode, prompt preset, custom prompt fields, timeout settings, and connection defaults where the JSON override is stored.

These shortcuts follow the **Output** selection in Lumiverse's Image Gen pane:

- **Set as background** displays the result behind the chat using the configured opacity and transition.
- **Insert into chat** creates Lumiverse's normal image generation message.
- **Attach to last message** resolves and updates the actual last chat message.
- **Preview only** opens the result in a Lumiverse preview modal.

Errors from Lumiverse or the image provider appear in a toast instead of leaving the button spinning.

To install from a local copy, place the project under `data/extensions/image_payload_injector/` on the Lumiverse server and choose **Import Local**.
