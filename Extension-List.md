# Lumiverse Extensions — Local Documentation Mirror

This repository is a **searchable, offline mirror of the documentation for every
Lumiverse extension** linked from the community extension list.

It exists so that AI agents (and humans) can answer questions like *"what
extensions exist?", *"what does the LumiBooks extension do?", *"how do I
configure Auto Retry?"* by grepping a local checkout, without hitting the
network and without cloning 50 separate repositories.

---

## Start here

| File | What it is |
| --- | --- |
| [`index.md`](./index.md) | **The master table.** Every collected extension, its source repository, its original link, its local directory, and how many Markdown files were mirrored. Start here. |
| [`extensions-list.md`](./extensions-list.md) | The original community extension list — the *source document* this mirror was built from, preserved verbatim. |
| [`failures.md`](./failures.md) | Every source entry that could **not** be collected, with the reason. Nothing is silently dropped. |
| [`extensions/`](./extensions/) | The mirrored documentation, one directory per extension. |

---

## How an AI agent should use this

1. **Read `index.md` first.** It is the routing table. Scan the *Extension*
   column to find candidates, then go straight to that extension's directory.

2. **Search with `grep`/`glob` across `extensions/`.** Every document is plain
   Markdown, so ordinary text search works. This is usually faster and more
   precise than reading whole READMEs:
   - *"which extensions do image generation?"* →
     `grep -ril "image gen" extensions/`
   - *"which extensions have a settings reference?"* →
     `glob extensions/*/docs/settings.md`
   - *"what does LumiStage do?"* → read `extensions/lumistage/README.md`

3. **Always read `extensions/<slug>/README.md` before answering** about an
   extension. Every extension README opens with a metadata table:

   ```markdown
   | Field | Value |
   | --- | --- |
   | Extension | LumiStage |
   | Source repository | https://github.com/Archkr/Lumiverse-LumiStage |
   | Original link (from Lumiverse-Extensions README) | https://github.com/Archkr/Lumiverse-LumiStage |
   | Upstream path | `README.md` |
   | Retrieved | 2026-09-28 @ `main` (`abc1234`) |
   ```

   This tells you which project the text came from, whether the path was
   relocated, and the exact commit it was taken from. Everything below the
   metadata table is the **upstream file, unmodified**.

4. **Follow the directory structure, don't flatten it.** Deeper documentation
   lives under its original path, so prefer the most specific file:
   - `extensions/lumiscript/docs/cookbook/*.md` — worked examples
   - `extensions/canvas/docs/custom-css.md` — a specific subsystem
   - `extensions/auto-refine/docs/settings.md` — a settings reference
   - `extensions/simtracker/docs/template-guide-*.md` — template authoring

5. **Cite the repository, not just this mirror.** These are third-party
   projects; when you tell a user how to do something, point them at the source
   repository from the metadata table so they get the current version.

6. **Respect the caveats.** If a directory contains only a generated stub
   README, the upstream project ships no Markdown documentation — say so instead
   of inventing behaviour. See the "No upstream documentation" note in
   `index.md`.

7. **Don't trust the docs to be current.** This is a point-in-time snapshot.
   Check the metadata table's `Retrieved` field and the upstream repository if
   the answer is version-sensitive.

---

## What is mirrored

For each extension, the upstream repository was cloned at its default branch and
its **Markdown documentation** was copied with the **original relative directory
structure intact**:

```
extensions/
  <extension-slug>/
    README.md          <- upstream README, with a metadata header prepended
    docs/              <- upstream docs/, structure preserved
      cookbook/ ...
    references/ ...
```

Only documentation is mirrored. No source code, JSON, lockfiles, or build
artefacts are copied.

### Inclusion and exclusion policy

**Included:** `README.md`, and everything under `docs/`, `documentation/`,
`examples/`, `guides/`, `cookbook/`, `concepts/`, `references/`, `tutorials/`,
plus other clearly-usage-related Markdown (`ARCHITECTURE.md`, `SPEECH.md`,
design documents, `*.README.md`, `.github/readme-*.md` translations).
`CHANGELOG.md` files **are** included: for these small projects the changelog is
a user-facing feature history and answers "when did X appear / what changed?".

**Excluded:**

| Excluded | Why |
| --- | --- |
| `LICENSE*`, `LICENCE*`, `COPYING*`, `NOTICE*`, `THIRD_PARTY_NOTICES*` | Legal text, not documentation. (Note: extension licences are frequently non-standard — don't assume.) |
| `CODE_OF_CONDUCT.md`, `SECURITY.md` | Policy documents, not documentation. |
| `CONTRIBUTING.md`, `CONTRIBUTORS.md` | Maintainer-facing contributor guides. |
| `AGENTS.md`, `CLAUDE.md`, `.github/copilot-instructions.md` | AI/coding-agent instructions for working *on* the repo, not docs for using the extension. |
| `docs/plans/`, `PLAN.md`, `*implementation-plan.md` | Dated internal development plans and audit trails about past refactors. |
| `eval-results/` | Generated model benchmark dumps, not documentation. |
| Vendored **platform** docs (`docs/user-docs/`, `docs/developer-docs/`) | Lumiverse's own product documentation accidentally committed inside an extension repo. It describes the host application, not the extension, and is hundreds of files deep — it would swamp this mirror and mislead searches. See below. |

Where a judgement call was genuinely ambiguous, the file was **included** and
the decision is noted in [`index.md`](./index.md). Where a whole subtree was
clearly not extension documentation, it was omitted **and the omission is
recorded** — see below.

### Omitted subtrees

`Greeting Image Generator` had 168 files of Lumiverse platform documentation
(`docs/user-docs/`, `docs/developer-docs/`) committed into it. Those describe
the host application — backend/frontend API references, chatting, presets, world
books, the Weaver studio — and none of it is about the extension, whose README
never links to it. The whole `docs/` tree was dropped; only the extension's own
`README.md` is mirrored, and its metadata header says so. After this removal
that extension contributes 1 file instead of 169, and the mirror holds 175
Markdown files in total.

### Path relocations

Two extensions (`LumiAgent`, `LumiBooks`) keep their README at
`.github/README.md` instead of the repository root. It is mirrored to
`extensions/<slug>/README.md` so the documented layout always holds; the
original path is recorded in the file's metadata header.

---

## Status

- 50 of 52 source-list rows have documentation mirrored.
- 2 rows have no URL at all (a "coming soon" announcement and an empty
  placeholder row) — see [`failures.md`](./failures.md).
- 1 source link pointed at a repository that has since been **renamed**; the
  canonical successor was used and the source list link updated.
- 3 extensions ship **no** Markdown documentation upstream; their directories
  hold a generated metadata stub.
- 1 extension had Lumiverse's own platform docs mixed into its repo; that
  subtree was omitted and the omission is recorded. 175 Markdown files mirrored.

---

## Provenance and caveats

- Snapshots were taken from each repository's default branch. Commit SHAs and
  the retrieval date are recorded in each extension README's metadata header.
- Document contents are reproduced **as-is**. They have not been rewritten,
  summarised, corrected, or normalised — so upstream typos, broken relative
  links (which will not resolve within this mirror) and stale information are
  preserved deliberately.
- Relative links *between* mirrored documents generally still work, because the
  directory structure is preserved. Links pointing outside the mirror (to
  upstream files, images, or URLs) will not resolve locally.
- This mirror is **read-only with respect to upstream**. Nothing in this
  repository modifies any extension project.
