<!-- Mirror of upstream documentation. Everything below the metadata table is the original file, unmodified. -->

| Field | Value |
| --- | --- |
| Extension | Greeting Image Generator |
| Source repository | https://github.com/ajrc0re/Lumiverse-GreetingImageGenerator |
| Original link (from Lumiverse-Extensions README) | https://github.com/ajrc0re/Lumiverse-GreetingImageGenerator.git |
| Upstream path | `README.md` |
| Retrieved | 2026-09-29 @ `main` (`899bd66`) |
| Note | The upstream repository also vendors 168 files of Lumiverse **platform** documentation under `docs/user-docs/` and `docs/developer-docs/` (backend/frontend API references, chat, presets, world books, etc.). That content documents Lumiverse itself rather than this extension, and its README never links to it, so it has been intentionally omitted here. See the source repository if you need it. |

---

# Lumiverse Greeting Image Generator

Illustrate existing character greetings in a native Lumiverse drawer. The extension adds or removes image markup; it never writes greeting prose or changes existing chat messages.

## Install

The repository includes ready-to-load `dist/frontend.js` and `dist/backend.js` bundles and a `spindle.json` manifest. Install the published repository through Lumiverse’s Extensions panel, then grant **Characters**, **Chats**, **Images**, and **Image generation** permissions. Requires Lumiverse 1.2.0 with the public active-character selector and native image prompt-preview endpoint (the API surface used by `lumiverse-spindle-types` 0.6.31).

For local development:

```sh
bun install
bun run typecheck
bun test tests
bun run build
```

The isolated UI preview runs with `bun run preview` at `http://127.0.0.1:4318`. It uses in-memory sample characters, simulated generations, and lightweight stand-ins for Lumiverse’s mounted form components. It does not contact image providers or Catbox. Real host components are used in the installed extension.

Local-network HTTP access is supported. Frontend request IDs use `crypto.getRandomValues()`, which works without HTTPS. **Copy image link** falls back to the browser's legacy copy action or a selectable link when the Clipboard API is unavailable. Server-side image job IDs and Catbox credential hashing remain in the Bun backend.

## Use

1. Open a character’s chat, then open **Greeting Images** from the drawer or command palette.
2. In **Settings**, choose an existing image connection profile. Its model and provider parameters are inherited unless overridden here.
3. Configure Lumiverse’s native **Chat-aware custom** prompt parser beforehand. The extension’s Settings page shows the inherited parser and lets you select a native prompt preset and add image instructions.
4. Click a greeting’s **+**, or enable **Multiselect**, select greetings, and choose **Generate selected**. One image is generated for each selected greeting, sequentially. A batch accepts up to 50 greetings.

Images are stored locally and appended to greetings by default. Settings can put them first, require image review, or require prompt review. **View/edit image prompt** prepares a prompt without generating an image; saving it lets the next generation reuse the edited text without another parser call.

Search explicitly by **Text** or **Title**, and filter by image presence. **Select all** selects the filtered greetings; previously selected hidden greetings remain selected and are counted. Image previews, confirmations, prompt editing, and recovery actions stay within the drawer.

**Attach image** uploads an existing file locally or embeds a supplied HTTP(S) URL as-is. It does not publish an existing file to Catbox. Uploaded/reference files are limited to 20 MB, using PNG, JPEG, WebP, or GIF.

## How it works

The browser calls Lumiverse’s authenticated `/api/v1/image-gen/preview-prompt` with `promptMode: "parsed_custom"`, the native preset instructions, the existing greeting, and optional style guidance. The backend then calls `spindle.imageGen.generate()` with the independently selected image connection. SwarmUI/ComfyUI profiles advertising preview streaming use `generateStream()`.

**Native parser limitations:** the existing endpoint requires a real chat and inherits native text-parser settings and live chat context. Greeting-focused instructions cannot completely prevent influence from later chat events. The extension does not create temporary chats, swap global settings, or implement another text-generation pipeline. Image generation uses the selected image profile’s defaults and overrides; it does not reproduce the full native pipeline’s separate global LoRA assembly. Configure the needed workflow/parameters in that connection.

### Consistent sets

Save shared appearance/style instructions per character. Choose the avatar, an existing greeting image, or an uploaded file as a reference. Remote greeting images are copied locally when the host allows browser downloads; if blocked by CORS, download the file and use **Upload reference**. A saved generated result can also become the set reference. OpenAI, Google Gemini, and OpenRouter receive references as `{ data, mimeType }` objects through their existing adapters. Individual models may not support image inputs. Other providers use the shared text instructions only. This is guidance, not a guarantee of identical appearance, and introduces no extra AI planning call.

### Storage and sharing

- **Local:** uses the existing Lumiverse image store. Relative image links work when the recipient can access that server; they are not embedded portable card assets. Server access/authentication configuration still controls accessibility.
- **Catbox anonymous:** publishes the image and keeps the local original. Catbox currently removes anonymous files after two years without access. Anonymous uploads cannot be deleted by this extension.
- **Catbox authenticated:** uses your account’s userhash, stored in Lumiverse’s encrypted enclave. The frontend only receives a “saved” indicator. Upload records retain an account fingerprint, never the credential. Catbox describes account uploads as permanent, subject to its policies and service availability.

Catbox links are public. Some networks block Catbox; it is not an availability guarantee or a backup service. Its commercial hotlinking restrictions also apply. See [Catbox’s API](https://catbox.moe/tools.php) and [FAQ](https://catbox.moe/faq.php).

An upload failure keeps the local image. **Retry upload** resumes uploading; **Use local** inserts the saved local result without generating again. Recent history stores metadata only, retaining 50 finished records per character plus pending jobs. Pruning records does not delete images.

### Removal and undo

The image’s × removes exactly that occurrence after confirmation. **Undo last image edit** restores the previous image edit for that character only if the greeting has not subsequently changed.

Permanent deletion is a separate opt-in for recorded extension-owned assets. Local deletion uses Lumiverse’s `unused=true` guard and retains files still referenced elsewhere. A currently selected set reference is not eligible. Catbox deletion requires the matching authenticated account and breaks all shared links to that file. Existing unowned links are never deleted. Removing a Catbox link or its public file does not delete the local backup. Deleting a local file does not delete its Catbox copy. Successful permanent deletion clears undo.

### Reliability and boundaries

Each job captures its original character, greeting text/index, image settings, prompt, and style/reference. Before writing, the extension re-fetches the character and checks the greeting. Edits or reordering cause a retained result with an **Apply to greeting…** action. Writes affect only the main greeting or a freshly read alternate-greeting array; the host API does not offer an atomic compare-and-swap, so simultaneous external saves cannot be made fully transactional by an extension.

Closing the drawer leaves the queue running. **Stop** prevents new jobs and further insertion; supported streams are aborted. A provider that cannot be cancelled may finish and charge for its request; the returned image is retained locally. A save already accepted by the host cannot be recalled. Browser reload or extension restart never automatically resubmits pending paid requests. Review retained results and explicitly retry uncertain generations; repeating them can incur another charge.

Recognized image markup includes inline/reference Markdown images, HTML `<img src>`, Risu `<img="…">`, `gallery://` and `embeded://` references resolved through the card’s asset map. Fenced/inline/indented code is ignored. Unresolved references show placeholders. CSS background images, images generated by executable scripts, and arbitrary custom message widgets are outside this parser. Greeting HTML is never executed in the drawer.

## Validation

Active-character tracking uses the public state selector when available, with a public `getActiveChat()` fallback for older builds and builds reporting `spindle_authority_map_unwired`. The fallback checks for selection changes every 750 ms, refreshes on chat changes, and stops on extension unload. This prevents an unavailable selector from aborting drawer registration; it does not change permission grants.

Tests cover source-preserving parsing and edits, prompt reuse, references, ownership checks, local persistence, mocked Catbox requests, queue interruption/recovery, and drawer interactions. Browser QA uses the isolated preview in light/dark themes and narrow/wide layouts. Automated checks do not perform paid generations or real Catbox uploads; those require an explicit live smoke test with a configured account.
