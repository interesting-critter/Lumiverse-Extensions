<!-- Mirror of upstream documentation. Everything below the metadata table is the original file, unmodified. -->

| Field | Value |
| --- | --- |
| Extension | Shutter |
| Source repository | https://github.com/cfigure/Lumiverse-Shutter |
| Original link (from Lumiverse-Extensions README) | https://github.com/cfigure/Lumiverse-Shutter |
| Upstream path | `README.md` |
| Retrieved | 2026-09-29 @ `main` (`f76ab24`) |

---

# Shutter

A [Lumiverse](https://github.com/prolix-oc/Lumiverse) extension for quick-access and automated native ImageGen workflows, inline image controls, generation history, prompt tools, and optional image layout controls.

## Features

- Generate images from a floating widget or input bar action.
- Append, replace, insert, or remove Shutter images from the newest message.
- Automatically trigger ImageGen after AI responses.
- Preview and edit resolved prompts before generation.
- Save successful Shutter-managed generations in Generation History.
- View, copy, reuse, insert, or replace images from saved history.
- View saved or embedded prompt metadata from Lumiverse's native image viewer.
- Navigate history with swipes on mobile or arrow keys on desktop.
- Optionally control the width and alignment of inline Shutter images.

Shutter uses your existing native ImageGen provider, model, prompt, scene-change, preview, and placement settings.

## Image placement

| Native ImageGen mode | Shutter behaviour |
|---|---|
| **Preview only** or **Set as background** | Shutter can ask whether to insert the result or insert it automatically. It can append a new image or replace the latest Shutter image in the target response. Shutter does not set an active background, so **Preview only** is recommended for Shutter workflows. |
| **Insert into chat** or **Attach to last message** | Native ImageGen handles placement. Shutter does not insert the result, and automatic generation is skipped in these modes. |

When Shutter shows the **Image Generated** modal, you can insert or replace the result, regenerate it from the resolved prompts, or rebuild the prompt from the original chat context.

Long-press or right-click the floating widget to open actions for the newest message:

- **Append**
- **Replace**
- **Force Generate**
- **Insert**
- **View Prompt**
- **Remove**
- **Remove All**

**Force Generate** appears only when native scene-change detection could otherwise skip the request. **Insert** appears only while Generation History is enabled.

## Generation History

Optional **Generation History** saves successful Shutter-managed generations for each message response and text swipe.

Saved generations can be:

- Previewed and navigated.
- Inserted into the message.
- Used to replace an existing Shutter image.
- Opened in **Image Prompt** to view, copy, or reuse their prompts.

Turning Generation History off stops new records but does not delete existing history. Use **Clear Generation History** to remove saved history metadata without deleting generated images or changing messages.

If a saved image has been deleted from Lumiverse's gallery, Shutter keeps its prompt information but prevents the missing image from being inserted or used as a replacement.

## Lightbox prompt tools

When **Show Prompt and History in Lightbox** is enabled, Shutter adds a compact toolbar to Shutter images opened in Lumiverse's native image viewer:

- **Copy**
- **Prompt / Collapse**
- **History**
- **Hide**

**History** appears when saved records are available. **Hide** restores the image's full viewing area, and **Show Details** brings the toolbar back.

The expanded prompt view can switch between metadata saved by Shutter and metadata embedded in the image when both are available.

## Auto Generate

Shutter can trigger ImageGen after AI responses in three modes:

- **Every message**
- **Every X messages**
- **Random interval**

Automatic generation has its own post-generation behaviour and can optionally use the prompt preview flow. Automatic-generation errors are suppressed so they do not interrupt the conversation.

## Main settings

### General

| Setting | Description |
|---|---|
| **Floating Widget** | Show the quick-access widget. |
| **Widget Size** | Small, Medium, Large, or XL. |
| **Widget Style** | Colour or Monochrome. |
| **Icon** | Aperture, Cherry Blossom, or Kitty Lotus. |
| **Toast on Insert and Replace** | Show a confirmation when Shutter inserts or replaces an image. |
| **Remove Image Tags from Context** | Remove Shutter image markdown from prompts sent to the model. Enabled by default. |
| **Remove Confirmation** | Never, Bulk Only, or Always ask before removing images. |

### Image handling

These settings apply only when Shutter handles image placement.

| Setting | Description |
|---|---|
| **After Generation** | Ask to insert or insert automatically after a manual generation. |
| **Default Widget Action** | Append a new image or replace the latest Shutter image. |
| **Generation History** | Save successful Shutter-managed generations and their submitted prompts. |
| **Swipe & Keyboard Navigation** | Navigate history with swipes on mobile or arrow keys on desktop. |
| **Clear Generation History** | Delete saved Shutter history metadata without deleting images or changing messages. |
| **Show Prompt and History in Lightbox** | Add prompt and history controls to Shutter images in Lumiverse's native viewer. |
| **Shutter Image Layout** | Enable scoped custom layout controls for inline Shutter images. |
| **Image Width** | Set inline Shutter image width from 1% to 100%. |
| **Image Alignment** | Align reduced-width images left, centre, or right. |

### Auto Generate

| Setting | Description |
|---|---|
| **Mode** | Off, Every message, Every X messages, or Random interval. |
| **Interval** | Number of AI messages between generations in Every X mode. |
| **Random Range** | Minimum and maximum interval for Random mode. |
| **After Auto Generate** | Insert automatically or ask to insert. |
| **Preview Prompt on Auto** | Show prompt preview for automatic generations when native prompt preview is enabled. |

## Notes

- Shutter follows Lumiverse's native ImageGen provider, model, prompt-preview, scene-change, and placement settings.
- Shutter images are not sent to the model as multimodal context.
- Custom image layout affects only inline Shutter images.
- Generation History is available across devices using the same Lumiverse deployment and account.

## Compatibility

Shutter 1.1.0 requires Lumiverse 1.1.0 or newer.

When upgrading:

- **Shutter Image Layout**, **Generation History**, and **Swipe & Keyboard Navigation** default to Off.
- The old Shutter **Force Generation** setting was removed in 1.0.6. Native ImageGen settings now control scene-change behaviour.
- The legacy [regex script](Shutter-regex-scripts.json) is redundant on Shutter 1.0.5 or newer and is retained only for archive purposes.

## Permissions

| Permission | Why |
|---|---|
| `chat_mutation` | Append, replace, or remove Shutter images in chat messages. |
| `ui_panels` | Render Shutter's settings and quick-access controls. |
| `interceptor` | Remove Shutter image tags from model context when enabled. |
| `app_manipulation` | Add prompt controls to Lumiverse's native image viewer. Optional; Shutter otherwise runs normally. |

## Installation

Install from URL in **Settings → Extensions**:

```text
https://github.com/cfigure/Lumiverse-Shutter
```

Permissions are requested automatically on first load.

## License

Shutter is licensed under the [MIT License](LICENSE).

## Credits

Shutter icon "[shutter](https://loading.io/icon/vg8pdg)" provided by [loading.io](https://loading.io) under the [Loading.io BY Licence](https://loading.io/license/#by-license).
