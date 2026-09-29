<!-- Mirror of upstream documentation. Everything below the metadata table is the original file, unmodified. -->

| Field | Value |
| --- | --- |
| Extension | Bionic |
| Source repository | https://github.com/patatasbravasch-arch/bionic |
| Original link (from Lumiverse-Extensions README) | https://github.com/patatasbravasch-arch/bionic |
| Upstream path | `README.md` |
| Retrieved | 2026-09-29 @ `main` (`d824f16`) |

---

# Lumiverse Bionic-style Reading

An unofficial Spindle extension for Lumiverse that adds a Bionic-style fixation effect to rendered chat prose.

Example:

- `Lumiverse makes reading easier.`
- becomes visually similar to **Lumiv**erse **mak**es **read**ing **eas**ier.

The extension only changes the rendered version of a message. Stored chat content, generation prompts, memories, embeddings, and exports remain unchanged.

> This project is not affiliated with or endorsed by Bionic Reading®.

## Why this implementation

Lumiverse staging exposes a `registerMessageContentProcessor` hook with `origin === "render"`. That is a better fit than rewriting mounted DOM nodes because it follows Lumiverse's own message render lifecycle and survives virtualized message mounting/unmounting automatically.

The transform is deliberately conservative. It rewrites normal prose while preserving:

- fenced code blocks
- inline code
- Markdown image syntax
- Markdown link destinations and raw URLs
- raw HTML elements / Lumiverse HTML islands
- HTML entities
- Markdown escapes

Markdown link *labels* are transformed because they are visible prose.

## Files

```text
spindle.json
src/
  backend.ts
  frontend.ts
  transform.ts
dist/
  backend.js
  frontend.js
  transform.js
package.json
tsconfig.json
```

## Before publishing

Edit `spindle.json` and replace:

- `YOUR_NAME`
- `YOUR_USERNAME`

with your actual author name and GitHub username/repository URL.

## Build

Lumiverse can auto-build from `src/`, but for a normal local build:

```bash
bun install
bun run build
```

The official Spindle TypeScript package is included as a dev dependency for editor/type support.

## Install in Lumiverse

1. Push the extension to a GitHub repository.
2. In Lumiverse staging, open the Extensions panel.
3. Install the repository URL.
4. Approve the `chat_mutation` permission.
5. Enable the extension and open/re-open a chat.

## Tuning the fixation amount

The current rule bolds roughly the first 50% of each word (single-letter words are left alone).

To change it, edit `fixationLength()` in `src/transform.ts`:

```ts
return Math.max(1, Math.ceil(length * 0.5))
```

For a lighter effect, try `0.4`; for a stronger effect, try `0.6`.

The visual weight is controlled in `src/frontend.ts`:

```css
.lumibionic-fix {
  font-weight: 700;
}
```

Try `600` for a subtler contrast.

## Notes

- The backend keeps a small render cache because Lumiverse currently invokes render processing twice per visible message.
- The cache is cleared on chat/message edit, swipe, and delete lifecycle events.
- The extension does not make network requests and does not need any permissions besides the one required by Lumiverse's message-content-processor API.
