# Generating images

`api.imageGen.*` fires image generations against the user's configured connection profiles, returns a base64 data URL plus a canonical `imageId` for persistence, and exposes provider / connection / model metadata so your script can build dynamic parameter UIs without hard-coding any of it. img2img and inpainting are supported by passing previous `imageId`s into provider-specific parameters.

Five methods, one permission:

- **`generate(input)`** — fire a generation, get back image bytes + persistence handle.
- **`getProviders()`** — enumerate providers + their capability schemas.
- **`listConnections()`** / **`getConnection(id)`** — the user's connection profiles (no API keys exposed).
- **`getModels(connectionId)`** — models available on a specific connection (may incur a network round-trip).

All five require the `image_gen` permission. They throw `Error: PERMISSION_DENIED:image_gen — grant this permission to use this API` when the permission isn't granted.

## The big idea

LumiScript doesn't ship its own image-generation backend. Lumiverse already has one — a provider registry (Stable Diffusion, ComfyUI, DALL-E, Imagen, Midjourney-via-proxy, etc.), per-user connection profiles (API URL + key + default model + default parameters), and a unified `generate(request)` entrypoint that dispatches to whichever provider the chosen connection points at. `api.imageGen.*` is a thin wrapper over that — your script can drive a generation without knowing which provider the user has configured.

The script-author payoff: `await api.imageGen.generate({ prompt: '...' })` works regardless of whether the user is using ComfyUI locally, Stable Diffusion through a cloud endpoint, or an OpenAI-image account. The user's default connection is consulted automatically; you only need to be explicit (pass `connectionId`) when you have a reason to override it.

The other four methods are the discovery surface — they let scripts that *want* to be explicit do so cleanly: build a connection picker, surface a model dropdown, validate `parameters` against the provider's declared schema before firing the generation.

## Quick start

The smallest useful generation script: prompt the LLM for a scene description tied to a chat event, fire an image, drop the result into the chat.

```js
// @triggers MESSAGE_SENT
// @description Visualize the user's last message — generate an image and inject it as an assistant note.

if (!data.message.is_user) return;

// Use the user's default image-gen connection — no connectionId needed.
const result = await api.imageGen.generate({
  prompt: `Cinematic illustration of: ${data.message.content}. Detailed, dramatic lighting.`,
  parameters: { width: 1024, height: 1024, steps: 30 },
});

// `imageDataUrl` is base64; drop it straight into an <img>. The persisted
// `imageId` survives chat reloads — pass it to api.images.get later if you
// need to refetch.
await api.chat.sendMessage(
  `<img src="${result.imageDataUrl}" alt="${data.message.content}" data-image-id="${result.imageId ?? ''}">`,
  { role: 'assistant' },
);
```

That's the whole loop. The user's default connection picks the provider + model + default parameter set; your script supplies the prompt and any overrides. `result.imageId` is the canonical persistence handle (see [Integration with the rest of the API](#integration-with-the-rest-of-the-api) below) — it survives chat reloads and is the input to every other `api.*` surface that takes an image handle.

## Surface

```ts
interface ImageGenAPI {
  generate(input: ImageGenInput):              Promise<ImageGenResult>;
  getProviders():                              Promise<ImageGenProviderInfo[]>;
  listConnections():                           Promise<ImageGenConnectionInfo[]>;
  getConnection(connectionId: string):         Promise<ImageGenConnectionInfo | null>;
  getModels(connectionId: string):             Promise<Array<{ id: string; label: string }>>;
}
```

The input + result shapes:

```ts
interface ImageGenInput {
  prompt:             string;          // Required.
  connectionId?:      string;          // Default: user's default connection.
  negativePrompt?:    string;          // Provider-dependent.
  model?:             string;          // Override the connection's default model.
  parameters?:        Record<string, unknown>;
                                       // Provider-specific (width/height/steps/cfg_scale/input_images/...).
                                       // Merged with the connection's `defaultParameters` host-side.
  ownerCharacterId?:  string;          // Tag the persisted result with character ownership.
  ownerChatId?:       string;          // Tag the persisted result with chat ownership.
}

interface ImageGenResult {
  imageDataUrl:  string;               // base64 data URL — assignable directly to <img src>.
  model:         string;               // Model that actually ran (may differ from input.model).
  provider:      string;               // Provider id that handled the generation.
  imageId?:      string;               // Canonical persistence handle (present when persistence succeeded).
  imageUrl?:     string;               // Auth-free public URL for the persisted image.
}
```

Two result fields cover two different rendering paths:

- **`imageDataUrl`** is the inline bytes. Use it for `<img src="...">`, `<canvas>` blitting, anywhere the renderer is in the same process as your script's output. No network round-trip; no auth.
- **`imageUrl`** is a persisted public URL. Use it when you need an auth-free URL the *client* can fetch — push notifications, external embeds, anywhere outside the chat UI's own session. The browser would otherwise have no way to authenticate against Lumiverse's regular image-store URLs; `imageUrl` skips that.
- **`imageId`** is the persistence handle. It outlives the data URL (which is just bytes in your script's memory) and the URL (which is an addressing detail). Save the `imageId` if you'll need the image later.

## Picking a connection and model explicitly

When you need control beyond the user's defaults — e.g. a tool script that always uses a specific high-fidelity connection regardless of what the user's chat is configured with — list the connections and pick by name:

```js
const connections = await api.imageGen.listConnections();

// Find a connection by name. (You could also filter on `provider`, on
// `hasApiKey: true`, etc.)
const target = connections.find(c => c.name === 'ComfyUI – Local');
if (!target) {
  api.ui.toast('No "ComfyUI – Local" connection configured', 'warning');
  return;
}

// Optionally pick a model that the connection supports.
const models = await api.imageGen.getModels(target.id);
const preferredModel = models.find(m => m.id.includes('sdxl'))?.id ?? target.model;

const result = await api.imageGen.generate({
  connectionId: target.id,
  model:        preferredModel,
  prompt:       'A cyberpunk skyline at dusk',
});
```

`listConnections` never exposes API keys — connections come back with `hasApiKey: boolean` only. The key never leaves the host. If your script needs to know whether a connection is *usable* (key is present), check `hasApiKey` and fall back gracefully.

`getModels` behaviour depends on the provider:

- **`modelListStyle: 'static'`** providers (typically self-hosted): the list comes from the provider's declared `capabilities.staticModels`. Cheap, no network.
- **`modelListStyle: 'dynamic'`** providers (most cloud APIs): the list is fetched live from the upstream API on each call. Cache the result if you call it more than once in a script body.
- **`modelListStyle: 'google'`**: dynamic with provider-specific quirks; treat as dynamic for caching purposes.

## Provider capability schemas

`getProviders()` is the contract for building parameter UIs without hard-coding any provider's quirks:

```js
const providers = await api.imageGen.getProviders();
const sd = providers.find(p => p.id === 'stable-diffusion');
if (!sd) return;

// Each parameter declares its type, range, default, and description.
for (const [key, schema] of Object.entries(sd.capabilities.parameters)) {
  console.log(`${key}: ${schema.type}, default=${schema.default}, ${schema.description}`);
}
// width: integer, default=512, Image width in pixels.
// height: integer, default=512, Image height in pixels.
// steps: integer, default=30, Sampling steps. Higher = slower but more detail.
// cfg_scale: number, default=7.5, Classifier-free guidance scale.
// …
```

Each entry follows `ImageGenParameterSchema`:

```ts
interface ImageGenParameterSchema {
  type:         'number' | 'integer' | 'boolean' | 'string' | 'select' | 'image_array';
  default?:     unknown;
  min?:         number;            // For 'number' / 'integer'.
  max?:         number;
  step?:        number;
  description:  string;             // Always populated — surface this in your UI.
  required?:    boolean;
  options?:     Array<{ id: string; label: string }>;   // For 'select'.
  group?:       string;             // Optional UI-grouping hint.
}
```

The six `type` values cover the realistic parameter space: numeric ranges (sliders), booleans (toggles), free strings (text inputs), discrete choices (selects), and **image arrays** (the img2img / inpainting / reference-image path — see below). The `group?` field is a hint for grouping related parameters under a heading in your UI ("Sampler", "Output dimensions", "ControlNet", etc.) — providers that surface it return parameters grouped logically; providers that don't omit it and your UI can render them flat.

## img2img and inpainting

Providers that support image inputs declare an `image_array`-typed parameter. The convention: pass an array of `imageId` strings as the parameter's value. The host resolves each `imageId` against the persisted image store before dispatching to the provider.

```js
// Generate a base image first.
const base = await api.imageGen.generate({
  prompt: 'A photo of a cat in a meadow',
});

// Feed it back in as an img2img input.
const variant = await api.imageGen.generate({
  prompt:     'The same cat, but wearing a wizard hat',
  parameters: {
    input_images: [base.imageId],   // Provider-specific parameter name.
    denoising_strength: 0.55,
  },
});

await api.chat.sendMessage(
  `<img src="${variant.imageDataUrl}">`,
  { role: 'assistant' },
);
```

The parameter name (`input_images` in this example) is provider-specific. `getProviders()` is the source of truth for which provider exposes which parameter name — look up the `image_array`-typed entries in `capabilities.parameters`. Common names: `input_images`, `init_images`, `reference_image`, `mask_image`.

Image inputs can come from any `imageId` source — a previous `api.imageGen.generate` result, an `api.images.upload` call (uploading bytes from `api.utils.http`), or an existing image already in the store (e.g. a character avatar). The host doesn't distinguish — `imageId` is `imageId`.

## Integration with the rest of the API

The `imageId` is the integration seam. Once persistence succeeds, the same handle works across four namespaces:

```js
const result = await api.imageGen.generate({ prompt: '...' });
if (!result.imageId) return;   // Persistence failed — only inline rendering possible.

// 1. Re-fetch the bytes later (e.g. from a different chat session).
const info = await api.images.get(result.imageId);

// 2. Derive a theme palette from the generated image.
const palette = await api.theme.extractColors(result.imageId);
await api.theme.applyPalette({ accent: palette.dominantHsl });

// 3. Set a character's avatar to the generated image. setAvatar takes raw
//    bytes, not an imageId — decode the data URL the generation already returned
//    (dataUrlToBytes yields the { data, mimeType } shape setAvatar wants).
const chat   = await api.chats.getActive();
const avatar = api.utils.image.dataUrlToBytes(result.imageDataUrl);
if (chat?.characterId && avatar) {
  await api.characters.setAvatar(chat.characterId, avatar);
}

// 4. Send it via push notification (uses the auth-free URL, not the imageId).
//    pushNotification is positional: (title, body, options). The `image`
//    field on options accepts the auth-free relative URL. Requires the
//    `push_notification` permission (separate from image_gen).
await api.ui.pushNotification(
  'Your generation is ready',
  'Tap to view the result.',
  { image: result.imageUrl },
);
```

Each consumer takes the `imageId` directly. There's no conversion step, no separate upload — the persisted row produced by `generate` IS the row those other APIs operate on.

If you also want to manipulate raw image bytes (cropping, resizing, format conversion) before consuming them downstream, that's `api.images.*` territory (the `images` permission, separate from `image_gen`). Note: `api.images.get(imageId)` returns an `ImageInfo` whose `url` field is a *relative authenticated URL* — NOT raw bytes. To get the bytes, fetch the URL via `api.utils.http` with `responseType: 'arraybuffer'`:

```js
const gen = await api.imageGen.generate({ prompt: '...' });
if (!gen.imageId) return;

// Read the bytes back, transform, re-upload as a new image.
const info = await api.images.get(gen.imageId);
if (!info) return;

// `info.url` is a relative authenticated URL. Fetch its bytes via the
// HTTP helper (requires `cors_proxy` permission + `allowDangerous`).
const response = await api.utils.http.get(info.url, { responseType: 'arraybuffer' });
const bytes = response.body;   // Uint8Array

// ... transform `bytes` ...

const transformed = await api.images.upload({
  data:     modifiedBytes,
  mimeType: 'image/png',
});
// `transformed.id` is now a fresh imageId for the modified bytes.
```

## Persistence ownership

`generate` accepts two ownership tags that flow through to the persisted image's metadata:

- **`ownerCharacterId`** — associates the result with a character. The image appears in that character's image gallery in the Lumiverse UI.
- **`ownerChatId`** — associates the result with a specific chat. Useful for "generations made in this conversation" filters.

These are advisory — the image is fully persisted in the host's image store regardless of whether you set them. They affect categorization, not lifetime. Don't supply them for one-off generations the user shouldn't see attributed; do supply them when the generation is part of an ongoing scene (e.g. a character-portrait script tagging avatars with `ownerCharacterId`).

## Error handling

`generate` throws on any of:

- **Permission denied** — `Error: PERMISSION_DENIED:image_gen — grant this permission to use this API`. The user hasn't granted `image_gen` to the LumiScript extension.
- **Unknown connection** — when `connectionId` doesn't resolve. List with `listConnections()` first if you accept user input.
- **Missing API key** — for cloud providers whose connection has `hasApiKey: false`. Surface a clear "go set the API key in Lumiverse settings" message to the user.
- **Provider rejection** — the upstream API returned an error (rate limit, content filter, malformed parameters). The error message includes the upstream provider's text where available.
- **Network error** — for cloud providers. Treat as recoverable; offer a retry.

A robust pattern:

```js
try {
  const result = await api.imageGen.generate({ prompt });
  // ... use result ...
} catch (err) {
  const msg = err instanceof Error ? err.message : String(err);
  if (msg.includes('PERMISSION_DENIED')) {
    api.ui.toast(
      'Image generation needs the image_gen permission — enable it in LumiScript settings.',
      'warning',
      { duration: 8000 },
    );
    return;
  }
  api.ui.toast(`Generation failed: ${msg}`, 'error');
  console.error('[my-script] imageGen failed:', err);
}
```

## When to use this vs `api.images.*`

Two related namespaces, two different jobs:

| Need | Use | Permission |
|---|---|---|
| Produce a NEW image from a prompt | `api.imageGen.generate` | `image_gen` |
| Upload bytes you already have (HTTP fetch, base64 from elsewhere, etc.) | `api.images.upload` / `uploadFromDataUrl` | `images` |
| Read the bytes of an image already in the store | `api.images.get` | `images` |
| Delete a persisted image | `api.images.delete` | `images` |
| Discover what providers / connections / models are available | `api.imageGen.getProviders` / `listConnections` / `getModels` | `image_gen` |

Scripts that both generate AND post-process typically need both permissions. Scripts that only consume already-stored images (e.g. theme derivation from a character's existing avatar) need only `images`.

## Version notes

Implemented in **v1.0.0-rc.5+**. All five methods have been stable since rc.5; the underlying provider registry (and therefore which providers are available) is host-side and changes independently of LumiScript versions. `getProviders()` reflects whatever the host knows about at call time.
