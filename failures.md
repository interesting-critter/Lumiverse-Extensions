# Failures — Source Entries That Could Not Be Collected

This file records **every** entry in the source list
([`extensions-list.md`](./extensions-list.md)) for which extension documentation
could **not** be mirrored into `extensions/`.

No entry is silently skipped. The verification pass re-read the source list and
confirmed that all 52 of its rows are accounted for: 50 collected into
`extensions/` and 2 listed below.

---

## 1. Agentic Preset Composer

| Field | Value |
| --- | --- |
| Name (from source list) | **Agentic Preset Composer** |
| Section in source list | The Liquor Priest |
| Original URL | _none — the source list row has no link target_ |
| What was attempted | Read the source list row verbatim: `\| **Agentic Preset Composer** \| *(Coming Soon (?))* \|`. Extracted every `http(s)://` URL from the document (50 in total) and searched the list for a link belonging to this entry; none is present. No repository name, owner, or URL hint is given anywhere in the document. |
| Reason it could not be processed | The source document provides **no URL at all** for this entry, and the entry is explicitly marked *(Coming Soon (?))* — i.e. it is announced but unreleased, so no repository is expected to exist yet. There is nothing to resolve, clone, or document, and inventing a repository would be wrong. |

**Action required from a human:** supply a repository URL once the extension is
published, then add it to `extensions/`.

---

## 2. Nada

| Field | Value |
| --- | --- |
| Name (from source list) | **Nada** |
| Section in source list | "Table for me to copy and paste lol" (a copy/paste template table) |
| Original URL | `https://` is absent — the link target is empty: `**[Nada]()**` |
| What was attempted | Parsed the link target. It is the empty string, so it resolves to nothing; it is not a valid URL and not a repository reference. Confirmed the surrounding table is a blank template row ("Table for me to copy and paste lol") rather than a real entry. |
| Reason it could not be processed | **Empty link target.** The row is an unfilled placeholder in the source document's copy/paste template, not an extension. There is no repository to locate and no documentation to mirror. |

**Action required from a human:** remove the placeholder row, or fill it in with a
real extension URL.

---

## Verification statement

- Total rows in the source list: **52**
- Rows containing a real URL: **50** — all 50 collected successfully into
  `extensions/`, none of them listed as a failure.
- Rows without a usable URL: **2** — both listed above.
- Repositories that could be located but contained no Markdown documentation:
  `Character Nudges`, `Shitposting Chat Room`, `Spotify Controls`. These are
  **not** failures (the repositories were obtained); they are present in
  `extensions/` with a generated metadata stub, and are flagged in
  [`index.md`](./index.md).
- One source link (`japolino/cue-visual-novel`) resolved to a renamed
  repository (`japolino/cue-living-novel`). It was collected from the canonical
  repository rather than recorded as a failure; see the notes in
  [`index.md`](./index.md).
