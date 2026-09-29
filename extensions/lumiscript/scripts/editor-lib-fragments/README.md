# editor-lib overlay fragments

Hand-authored, editor-only `.d.ts` fragments that `scripts/gen-editor-lib.ts`
concatenates around the **generated** api/script surface to produce
`src/types/editor-lib.ts` (the Monaco IntelliSense mirror).

The file numbers mirror the assembled layout of the emitted mirror:

| # | File | Content |
|---|------|---------|
| 01 | `01-zod-shim.d.ts` | Minimal `z` (Zod) ambient shim — a Monaco-friendly subset; the real `zod` types are too large/complex for the editor. |
| **02** | *(none — generated)* | **The api/script type surface**, rolled up from `src/types/script.ts` by dts-bundle-generator at build time. Not a file. |
| 03 | `03-script-and-ls.d.ts` | `ScriptNamespace` (the `script` global) + the `ls:*` `require()` overloads, which are editor-only. |
| 04 | `04-globals.d.ts` | The ambient `declare const api / script / z / data` block. |

**There is intentionally no `02-*.d.ts`** — slot 02 is where the generated bulk
is spliced in.

These are plain `.d.ts` files (not template literals), so backticks are fine;
the generator escapes everything when it emits the final template-literal string.
Edit a fragment only for genuinely editor-only declarations; a new `api.*`
method/type reachable from `LumiScriptAPI` appears automatically via the
generated bulk (add a standalone type to `src/types/editor-lib-entry.ts`).
