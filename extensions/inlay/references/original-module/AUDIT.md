# Original module behavior audit

These files are retained only as historical implementation references. Runtime code does not read or bundle this directory.

| Concern | Original module | Behavior before this change | Adopted resolution |
| --- | --- | --- | --- |
| Illustration count | Defaults to 3–5 shots. | Defaults to 1–3 shots. | Restore 3–5 for new or missing configuration while retaining explicit saved values. |
| Preprocessing | Selects 3–5 paragraphs with the most significant visual changes or actions and asks for a camera angle. | Requires every paragraph marker, so it cannot act as a visual editor. | Select a valid min/max-sized unique subset of the strongest beats and require a camera/composition note; fall back to raw numbered paragraphs on invalid output. |
| Parser message roles | Stable core and tagging prompts are system messages; the current source is a user message; user overrides follow it. | The schema, rules, context, and current source are combined into one large user message. | Put stable schema/tagging guidance in a system message, continuity references in a separate system message, current numbered source in a concise user request, and overrides last. |
| Same-paragraph shots | Later results replace earlier results when mapped by paragraph. | The first shot for a paragraph wins, discarding later alternatives. | Keep distinct same-paragraph shots in model order and collapse only exact visual duplicates. |
| Batch choice | Preprocessing focuses on significant visual changes/actions. | First-per-paragraph deduplication followed by paragraph sorting tends to favor early paragraphs. | Cap model-prioritized distinct candidates first, then sort the chosen set into paragraph order for rendering. |
| Continuity | Repeats stable appearance and ongoing scene facts; continuous POV is tied to an established viewpoint. | Continuity wording can encourage camera repetition and POV persistence. | Preserve appearance, attire, location, and persistent actions while explicitly varying cinematography; preserve POV only when the narrative establishes it. |

## Deliberately excluded runtime behavior

Artist presets, response-encoding or placeholder-code bypasses, uncensor/prefill content, and provider-specific sampling behavior in the archived card are not adopted. Prompt presets, parser parameters, image providers, cleanup, asset mode, and generation scheduling remain controlled by the current extension implementation.

## Reference artifacts

- `card.json`: original character-card/lorebook configuration and prompt material.
- `original_script.txt`: original Lua module implementation.
