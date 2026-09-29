# Picking files from disk

`api.ui.pickFile()` opens the browser's native file picker and hands your script the selected file(s) as bytes. Use it to import JSON configs, character cards, images, or any other user data into a script. Free tier — the native dialog is the user-action gate (the user has to actively choose a file), so no permission is required.

```js
const [file] = await api.ui.pickFile({ accept: ['.json'], maxSizeBytes: 1_000_000 });
if (file) {
  const config = JSON.parse(new TextDecoder().decode(file.bytes));
  // …use config
}
```

## Signature

```ts
pickFile(options?: {
  accept?: string[];        // extensions and/or MIME types, e.g. ['.json', 'application/json']
  multiple?: boolean;       // allow multiple selection (default false)
  maxSizeBytes?: number;    // per-file size limit
}): Promise<PickedFile[]>
```

It **always returns an array** (even with `multiple: false` — you get a one-element array, or an empty one). Each `PickedFile` is:

```ts
{ name: string; mimeType: string; sizeBytes: number; bytes: Uint8Array }
```

`bytes` is the raw file content as a `Uint8Array`. (Note: `mimeType` falls back to `"application/octet-stream"` for types the browser can't classify — e.g. `.md` files — so don't rely on it for format detection; sniff `bytes` or trust the extension.)

## Two outcomes to handle

- **Cancel** → resolves with `[]`. The user closed the picker without choosing.
- **Oversize** → **rejects**. If a selected file exceeds `maxSizeBytes`, the host throws and the promise rejects (mirroring `ctx.uploads.pickFile`). Wrap in try/catch if you set a limit:

```js
try {
  const files = await api.ui.pickFile({ multiple: true, maxSizeBytes: 50_000 });
  console.log(`Picked ${files.length}: ${files.map((f) => f.name).join(', ')}`);
} catch (err) {
  api.ui.toast(`Too big: ${err.message}`, 'error');
}
```

## What to do with the bytes

`bytes` is a `Uint8Array`, so it drops straight into the rest of the API:

```js
const [file] = await api.ui.pickFile({ accept: ['image/*'] });
if (file) {
  // → Lumiverse image store (needs the `images` permission)
  const info = await api.images.upload({ data: file.bytes, mimeType: file.mimeType, filename: file.name });

  // → ephemeral storage to process across fires (needs allowDangerous + ephemeral_storage)
  await api.files.tempWriteBinary(`import/${file.name}`, file.bytes, { ttlMs: 600_000 });

  // → text, for JSON / CSV / config
  const text = new TextDecoder().decode(file.bytes);
}
```

See [Storage model](../concepts/storage-model.md) for the file-storage tiers, and the in-app **Reference tab** for `api.images.*`.

## How it works (and why it's free-tier)

`pickFile` is a frontend round-trip: the call asks the frontend to open `ctx.uploads.pickFile`, the user picks, and the bytes come back to your script (base64 over the message bus, decoded to a `Uint8Array` before the promise resolves — you never see the encoding). Because the native OS dialog requires an explicit user choice, there's no silent-exfiltration risk, so no permission gates it — the same rationale as `api.ui.prompt` / `confirm` / `showContextMenu`.

## Notes

- **Call it in response to a user action** (a button/command), not on a timer — a picker that pops unprompted is hostile UX, and some browsers only honor `pickFile` inside a user-gesture handler.
- The promise stays pending until the user picks or cancels — there's no timeout.
- See the in-app **Reference tab** for the `PickFileOptions` / `PickedFile` field lists.
