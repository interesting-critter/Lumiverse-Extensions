# Image generation pipeline audit

September 4, 2026. Audited repository commit `57d825a`.

The pipeline has useful reliability protections, but its stages do not consistently agree on which character, outfit, environment, or job an image represents. The highest-priority work is correcting those disagreements. Prompt wording improvements alone will not fix them.

Three parallel audit agents examined the compiler, planner/context, and generation lifecycle. The primary agent reviewed their evidence, tested expression selection, and ran the combined reproductions. No production code was changed and no live planner or image-provider requests were made.

## Evidence and limits

- Existing suite: 372 tests passed on rerun. The first run had 371 passes and one file-enumeration test timeout. That test then passed independently and in the full rerun.
- Both `bun run typecheck` and `bun run typecheck:build` passed.
- New audit probes: 34 tests passed across four files, with 89 assertions. These are characterization tests: they assert the current problematic behavior, so passing confirms a reproduction rather than certifying correctness.
- Probes call production functions with synthetic inputs and mocked provider/storage responses. They establish missing, contradictory, malformed, or incorrectly routed prompt data. They do not measure generated-image quality or how often a live planner produces each input.
- The audit distinguishes accidental behavior from deliberate restrictions. Git history shows multi-character/attire support and reference anchoring were added later. The repeated-job problem was partially anticipated in a planner comment, but the existing mitigation only handles duplicate paragraph indexes.

## Current implementation

| Stage | What runs today | Relevant limitation |
|---|---|---|
| Context | Recent messages, enabled card/persona/lore context, previous scene, known appearance memory | Character counts and message counts are bounded in places; there is no unified token budget for the complete request. |
| Planner | A sidecar returns scenes, character descriptions, cue expressions, attire and speaker attribution. Normalization and JSON repair precede validation; failures get one retry before fallback. | Structural validity does not establish usable visual coverage. |
| Visual state | A durable protagonist appearance and scene-level identity are combined with cue metadata | Clothing does not persist as evolving state, and cue character switches do not resolve a new textual identity. |
| Prompt compiler | A narrow adapter into the vendored Inlay compiler emits configured prefix, solo subject, identity, catalogue pose, environment fragments, fixed camera, and suffix | Always uses Anima-style tag assembly and ComfyUI section formatting. Rich dynamic shot capabilities in the vendored code are not active here. |
| Image generation | Per-batch scheduler, prompt deduplication, optional reference portraits, persisted image IDs and frontend delivery | Deduplication loses later cue ownership. Initial reference capture is not a dependency of subsequent renders. |

The active camera tags are `upper body, eye level, straight-on`. The runtime does not compile `basePrompt`, environment `description`, composition lock, dialogue-safe region, or scene aspect ratio into provider requirements. A configured image workflow may supply dimensions, but this extension does not enforce the scene's declared 16:9 ratio.

The README and architecture document still describe a single frozen protagonist, a small pose catalogue, and a wider camera. The actual catalogue has 92 entries, the planner supports active-character changes, and the compiler uses upper-body framing. Those documents should be updated before they guide further implementation.

Existing strengths worth retaining include strict public contracts, serialized storage writes, planner retry/fallback handling, persona-attribution guards, deterministic prompt compilation, stale-turn guards, and image decode before display. The findings below occur despite those protections.

## Confirmed findings

Priority means implementation order. P1 affects ordinary requested behavior or recovery. P2 affects specific inputs, continuity, or provider compatibility. It is not a security severity scale.

### 1. P1: switching the cue character preserves the previous character's prompt

Hypothesis: prompt identity and reference identity can disagree after a character switch.

Reproduction: a scene starts with Mira, described as silver-haired with green eyes and a red coat. The roster also defines Rin with black hair, blue eyes and a blue kimono. The next cue explicitly selects Rin. Both `cue.character` and the portrait lookup name become Rin, but the compiled prompt still contains Mira's traits and none of Rin's.

The compiler reads `scene.identityPrompt`; portrait lookup prefers `cue.character`. This can combine Rin's portrait with Mira's text. If no Rin portrait exists, the wrong textual identity could also seed Rin's canonical portrait. That last consequence is inferred from the capture path, not a live image observation.

Resolve one character-specific visual state per cue and use it for prompt text, reference lookup, capture and fingerprinting. Test alternating speakers in the same location, including a character with no stored appearance.

Evidence: [cue assignment](src/backend/runtime/planner.ts:1093), [scene-only identity](src/backend/runtime/images.ts:37), [portrait identity](src/backend/runtime/images.ts:160). Probe: planner H12.

### 2. P1: repeated expressions leave later image jobs queued indefinitely

Hypothesis: prompt reuse does not preserve every cue's job identity.

Reproduction: two smile cues at paragraphs 0 and 1 compile identically. Generation makes one provider call and returns statuses `[generated, queued]`. The second job has no image ID even after the batch completes.

The scheduler returns the first job's promise for a matching prompt. The caller ignores the reuse metadata and only updates the emitted job ID. Paragraph-index deduplication does not address identical prompts at different paragraphs.

Keep per-cue jobs while sharing the provider result. Propagate success, failure and cancellation to every dependent cue. An acceptance check should require every finished batch to have no unexplained queued jobs.

Evidence: [scheduler reuse](src/backend/core/asset-scheduler.ts:88), [job updates](src/backend/runtime/images.ts:255), [partial planner mitigation](src/backend/runtime/planner.ts:1071). Probe: lifecycle H1.

### 3. P1: explicit retry replays failed stored assets

Hypothesis: content-based idempotency overrides the user's retry request.

Reproduction: invoke the actual registered `vn_retry_turn` handler with a stored failed image and a changed prompt prefix. It sends the old failed turn back to the frontend and makes zero image-provider calls.

Retry uses normal message processing, whose matching-content branch returns the cached record regardless of job status or changed settings.

Give retry a separate recovery path for failed, cancelled and interrupted jobs. Snapshot the settings used by each attempt. A separate regenerate operation can make replacement of successful images explicit.

Evidence: [retry handler](src/backend/runtime/controller.ts:639), [cache early return](src/backend/runtime/controller.ts:343). Probe: lifecycle H6.

### 4. P1: an outfit override can delete physical identity and change inferred gender

Hypothesis: removing clothing from comma-separated prose also removes permanent traits.

Reproduction:

```text
Identity: a tall man with silver hair wearing a red coat, green eyes
Override: white blouse
Result:   green eyes, white blouse
Subject:  1girl, solo
```

The replacement deletes every comma fragment containing a clothing word, then gender classification runs on the damaged result. Related probes show `bow-shaped lips` is deleted, while `sundress`, `sweatshirt`, `dress_shirt` and `breastplate` survive replacement. Underscore tags such as `knee_high_boots` also evade clothing and crop matching. Empty identity plus known `yellow raincoat` loses the attire entirely because override application requires a nonempty identity.

Represent permanent appearance and attire separately. Append known attire even when physical identity is unresolved. Normalize tag spelling, but avoid treating a larger regex vocabulary as a complete solution to mixed prose.

Evidence: [override gate](src/backend/runtime/images.ts:37), [clothing deletion](src/backend/runtime/images.ts:349). Probes: compiler attire, empty-identity and underscore cases.

### 5. P1: changed clothing reverts on the next cue and turn

Hypothesis: the planner is instructed to emit clothing changes as events, while the consumer interprets them as one-cue values.

Reproduction: three cues produce attire `[undefined, "blue pajamas", undefined]`. The next turn also has no attire override, while the durable baseline still contains a red coat.

The planner says to specify attire when it changes and otherwise return null. The consumer resolves only `cue.attire || scene.attire`. No reducer carries the new outfit forward, and the turn emits no continuity deltas.

Persist mutable visual state per character. A missing attire value should mean unchanged; an explicit reset should mean return to the baseline outfit. Test multiple changes, character switches, and the next assistant turn.

Evidence: [planner attire instruction](src/backend/runtime/planner.ts:164), [scene attire reset](src/backend/runtime/planner.ts:1058), [cue resolution](src/backend/runtime/planner.ts:1110), [empty deltas](src/backend/runtime/planner.ts:1196). Probe: planner H1.

### 6. P1: empty visual plans can be marked successfully planned

Hypothesis: successful JSON parsing bypasses the retry/fallback behavior even when no illustration can be generated.

Reproduction: `cues: []`, only paragraph index 99 for a three-paragraph response, and `{}` each produce zero visual cues, `planningStatus: "planned"`, `usedFallback: false`, and only one planner attempt.

Schema defaults accept empty collections. Out-of-range filtering can remove all cues without a later semantic check.

After normalization, require usable visual coverage when image generation is expected, including an opening cue. Repair or retry missing essential visual information, and expose a degraded status if it remains absent.

Evidence: [output defaults](src/backend/runtime/planner.ts:75), [cue filtering](src/backend/runtime/planner.ts:1076), [status assignment](src/backend/runtime/planner.ts:1198). Probe: planner H3.

### 7. P2: substring and positional expression fallback invent emotions

Hypothesis: fallback selection is deterministic but semantically unreliable.

| Input | Selected expression |
|---|---|
| `She puts on her gloves.` | `lovestruck` |
| `She opens the illustrated atlas.` | `lustful` |
| `She did not smile.` | `smile` |
| Preferred expression `not angry` | `angry` |
| Neutral `The door is blue.` at paragraph 0 | `idle` |
| The same neutral sentence at paragraph 19 | `aroused` |
| Portuguese `Ela está furiosa.` at paragraph 0 | `idle` |

Whole-paragraph scanning also attributes another person's emotion to the visible character. A valid explicit catalogue expression overrides these fallbacks correctly, so this risk concentrates in missing, unfamiliar or malformed expression output.

Use exact expression IDs plus a small explicit alias map. Prefer neutral/listening fallback over cycling through the entire catalogue. If keyword fallback remains, handle word boundaries, negation and subject attribution. Intimate expressions should require positive evidence.

Evidence: [expression matching](src/shared/character.ts:437), [keyword matching](src/shared/character.ts:447), [index cycling](src/shared/character.ts:453). Probes: pose-selection suite and planner H7.

### 8. P2: identity classification and visibility filtering are too coarse

Hypothesis: incidental words or mixed anatomical tags change the subject or remove visible features.

Confirmed classifications include `man, maid uniform, black hair` becoming `1girl`; `male android, sister's scarf` becoming `1other`; and an unspecified nonbinary person or `golden retriever, four legs, fur` becoming `1girl`. The classifier scans the entire identity and defaults to female.

Separately, `silver hair and knee-high boots, green jacket with a waist belt` loses both silver hair and green jacket under upper-body framing. The first matching body region determines whether the entire comma fragment survives.

Store explicit subject type separately from clothing and relational prose. Classify atomic appearance facts by visibility. Preserve unknown traits until there is a reason to exclude them.

Evidence: [subject classification](src/backend/runtime/images.ts:43), [body-region rules](src/backend/inlay-prompt/shot-resolution.ts:82), [visibility filtering](src/backend/inlay-prompt/prompt.ts:474). Probes: compiler classification and compound visibility.

### 9. P2: filtering can break weighted syntax; weights are not serialized by provider

Hypothesis: comma-based transformations lose the structure of weighted groups.

Confirmed: replacing attire in `man, (red coat, green eyes:1.2)` produces `man, green eyes:1.2), black kimono`. Cropping `woman, (silver hair, black boots:1.2)` leaves an unmatched opening parenthesis before the remaining pose and environment. The malformed string is proven; provider recovery or weight spillover is unmeasured.

There is a separate compatibility issue. Of 92 pose entries, 76 contain curly/square weighting syntax, which the runtime sends unchanged through its ComfyUI-formatted prompt. Official [ComfyUI documentation](https://docs.comfy.org/tutorials/basic/text-to-image) describes parentheses and numeric weights, while [NovelAI documentation](https://docs.novelai.net/en/image/strengthening-weakening/) describes braces and brackets. The selected workflow may have a custom parser, so actual impact requires testing against that workflow.

Use structured weighted terms and serialize for the selected provider/workflow. Validate balance after transformations. The hypothesis that the catalogue itself has broadly unbalanced braces was not supported: all 92 entries were balanced before and after ordinary baseline compilation.

Evidence: [comma filtering](src/backend/runtime/images.ts:354), [prompt serialization](src/backend/inlay-prompt/prompt.ts:119), [runtime syntax selection](src/backend/runtime/images.ts:28). Probes: compiler weighted-group and catalogue inventory.

### 10. P2: background changes are lost through both reuse and truncation

Hypothesis: scene stability is preventing legitimate local visual updates.

Reproduction: a continuing bedroom changes from a closed window to an open window. The resulting scene retains the closed-window environment and base prompt. The reducer only accepts major boundaries; the planner then reuses the entire old environment.

A separate compiler probe supplies location `Library, flooded with ankle-deep water`, time `night, heavy rain`, and weather `lightning`. The output retains only `Library, night, lamplight`. The assembler splits each semantic string at commas and caps location/time-weather to one fragment. The unused environment description cannot recover the missing facts.

Allow local environment patches without requiring a new scene. Preserve location, time and weather as separate facts, and apply limits to semantic items rather than comma fragments. Test doors, windows, weather changes and moved props.

Evidence: [environment reuse](src/backend/runtime/planner.ts:1055), [snippet splitting](src/backend/inlay-prompt/prompt.ts:494), [environment caps](src/backend/inlay-prompt/prompt.ts:806). Probes: planner H2 and compiler environment truncation.

### 11. P2: first-reference capture races with subsequent renders and lacks recovery

Hypothesis: default parallel generation does not deliver the sequential identity anchoring covered by existing tests.

Reproduction: three different poses of one character at default concurrency 2 produced capture flags `[true, false, false]`; all three requests lacked references. The first image was still being generated or persisted when later work started. The exact number of unanchored requests is timing-dependent.

With concurrency 1, a first response missing `imageId` made that job fail, but the capture flag remained held. Later successful images never requested capture, and no portrait was stored.

Use one capture promise per character and wait for it before dependent renders. Release ownership in `finally`, covering validation and persistence failures. Allow unrelated characters to render concurrently.

Two further probes confirm missing recovery policy. Correcting the textual identity still sends the original first-wins portrait; structurally valid storage also accepts non-base64 text with `text/plain` MIME and forwards it as a reference. First-wins is intentional, but a replace/reset mechanism, provenance and data validation are missing.

Evidence: [capture selection](src/backend/runtime/images.ts:277), [post-call validation](src/backend/runtime/images.ts:311), [portrait publication](src/backend/runtime/images.ts:323), [stored portrait validation](src/backend/runtime/storage.ts:469), [first-wins persistence](src/backend/runtime/storage.ts:517). Probes: lifecycle H2 through H5.

### 12. P2: context and repair edge cases degrade otherwise usable planning

- `includeRecentMessages: 0` still includes all supplied messages because `slice(-0)` is `slice(0)`. A captured real planner request retained an obsolete-setting sentinel. The controller repeats this slicing pattern. Fix zero explicitly and apply a complete-request token budget. [Planner slicing](src/backend/runtime/planner.ts:172), [controller slicing](src/backend/runtime/controller.ts:370). Probe: planner H5.
- An otherwise valid scene, identity and cues plus `choices: [null]` cause both attempts to fail and fall back to a generic scene with empty identity. Validate optional choices separately so they cannot discard usable visual output. [Choice normalization](src/backend/runtime/planner.ts:306), [whole-document validation](src/backend/runtime/planner.ts:699). Probe: planner H11.
- JSON repair changes Python-style literals inside quoted strings. A repaired `False ceiling, None logo, True blue wallpaper` becomes `false ceiling, null logo, true blue wallpaper`. Use string-aware repair. The test establishes text mutation; this particular basePrompt is not used by the image adapter, so direct visual impact remains unproven. [Repair substitutions](src/backend/runtime/planner.ts:429). Probe: planner H6.
- Long cards can exhaust the 5,200-character planner-context block before late appearance, scenario and tags. Full structured description remains available and may salvage some appearance through deterministic extraction. Reserve separate budgets for essential fields. [Context construction](src/backend/runtime/context.ts:82). Probe: planner H9.

## Deliberate limits and additional hypotheses

Closed pose generation deliberately drops free-form actions and prompt deltas. A brass key in a raised hand does not survive into the cue. Frozen identity deliberately rejects even a story-directed permanent change in hair, eyes or anatomy. These are product limitations rather than newly discovered implementation mistakes. A bounded action/prop schema and explicit transformation state would improve narrative fidelity while retaining control over prompts.

Other observed limits include selecting the first N cues when the planner overproduces, which can omit a later setting change, and discarding later same-setting scene entries, which can lose character/outfit metadata placed there. Prioritize semantic beats and salvage useful state changes from redundant scene proposals.

The following require live evaluation and were not claimed as measured image defects:

- How strongly contradictory text and reference portraits affect identity, outfit and background.
- Whether an arbitrary first scene render is a good identity reference across poses and lighting.
- Whether the active ComfyUI workflow actually uses the mapped reference image and understands the emitted weights.
- Whether the default style and negative tags suit each selected model.
- Whether workflow dimensions and framing reliably keep faces clear of dialogue UI.

## Improvement order and evaluation plan

1. Correct per-cue character resolution, shared-result job completion and retry recovery. These fix incorrect output routing and blocked workflows without changing artistic policy.
2. Introduce structured permanent identity, explicit subject classification, mutable attire and environment state. Resolve this state once per cue and share it across all downstream consumers.
3. Add semantic planner validation and partial recovery. Require visual coverage, retain valid visual sections when optional sections fail, and replace positional expression cycling with a neutral fallback.
4. Make prompt transformations weight-aware and provider-aware. Preserve mandatory facts, record any deliberately omitted facts, and test syntax after serialization.
5. Coordinate reference capture, validate portrait data and add replacement/versioning. Preserve the ability to opt out of anchoring.
6. Persist an attempt record containing the exact positive and negative prompts, resolved visual state, provider/model/workflow, relevant parameters and seed, portrait ID/version, compiler version, and planner fallback/repair diagnostics. The current prompt fingerprint cannot reconstruct the request and omits negative prompt and provider/reference settings.

For regression coverage, turn each reproduced defect into an acceptance test with corrected expectations. Add invariants: requested visible traits survive compilation; clothing changes do not mutate anatomy; text and portrait identify the same character; completed batches resolve every cue; and successful visual plans contain usable opening coverage.

For image evaluation, build a fixed set of short conversations covering speaker switches, changed outfits, negated emotion, non-English text, non-human subjects, weighted prompts and evolving backgrounds. Compare baseline and candidate with the same provider/workflow, dimensions and seeds where supported. Use several samples per case and blind review for character identity, outfit continuity, expression relevance, scene facts and framing. Report success rates separately for each dimension and provider. No claim of improvement should rest only on longer prompts or cleaner JSON.

## Reproducing this audit

The four supporting probe files are retained locally under `.cache/image-pipeline-audit/`, which is git-ignored. They are audit evidence, not additions to the normal regression suite.

```powershell
bun test --isolate ./.cache/image-pipeline-audit/compiler-audit.test.ts ./.cache/image-pipeline-audit/lifecycle.test.ts ./.cache/image-pipeline-audit/planner-context.audit.test.ts ./.cache/image-pipeline-audit/pose-selection.test.ts
```

Recorded result: 34 passed, 0 failed, 89 assertions. Production source and the pre-existing untracked audio directory were left unchanged.
