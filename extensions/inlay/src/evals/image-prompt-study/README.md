# Image-grounded prompt study

This harness evaluates the prompts that the sidecar conformance suite already produced by sending them through the real ComfyUI workflow with paired seeds. It does not call a sidecar. The latest passing saved prompt for each scenario, paragraph, and selected model is reused directly.

The default command is a dry run and makes no images:

```text
bun run eval:images --workflow=fixed_detailer_scheduler_workflow_EN_API_named_fields_renamed.json
```

Generate the default pilot of at most three comparable cases, three sidecar prompt sources, and two paired seeds:

```text
bun run eval:images --workflow=fixed_detailer_scheduler_workflow_EN_API_named_fields_renamed.json --generate
```

Useful controls:

- `--scenario=rhea_platform_conflict` filters the canonical scenario ID.
- `--exclude-scenario=nsfw_` excludes matching scenario IDs from a broader source root.
- `--models=deepseek,gemini,luna`, `--models=kimi`, or `--models=all` selects saved prompt sources.
- `--strategies=compact-production,production,legacy-production` compares the frozen pre-hybrid Dynamic projection, the current rendering, and the pre-shotPlan rich layout from the same accepted sidecar payload. `saved-production` preserves the exact source-run text. `merged-character-blocks`, `merged-tags-first`, `anchored-natural-language`, the fixture-controlled `role-bound-actions`, and `focused-dynamic` are also available.
- `--seeds=1103,2909` sets paired seeds.
- `--max-cases=3` and `--max-images=24` are safety caps.
- `--steps=30` overrides the workflow's sampler step count; omitting it preserves the workflow value.
- `--positive-prefix=...` and `--negative-prefix=...` override the fixed evaluation quality prefixes; pass an empty value to disable either.
- `--source-root=eval-results/raw` selects the saved sidecar result tree; narrow it to `eval-results/raw/general` or `eval-results/raw/nsfw` when desired.
- `--baseline-root=eval-results/raw/.../<frozen-run>` compares exact saved prompts from a frozen run against `--source-root` prompts. Use `--strategies=saved-production` so later renderer changes cannot contaminate either side.
- `--include-failed-baseline` is an explicit evaluation-only escape hatch for comparing a rejected historical prompt against an accepted tuned run. It never changes production acceptance.
- `--comfy-url=http://127.0.0.1:8188` selects ComfyUI.

Jobs are deliberately queued sequentially. The harness patches a cloned workflow in memory and never edits the supplied workflow. ComfyUI outputs use `InlayEval/<run>/...`, keeping them separate from ordinary `RisuAI` images. Downloaded images, a progressively written manifest, and a blinded pairwise review page are written beneath the gitignored `eval-results/image-study/raw/<run>/` directory.

The default evaluation affixes are held constant across every candidate:

```text
positive: (masterpiece, best quality:1.6), score_8, score_7, highres, detailed, cinematic lighting, thin line, (@starshadowmagician:1.2), (@nishikujic:0.6), (@kuzuvine:1.2), (@ezu \(e104mjd\):1.8)
negative: (worst quality, low quality:1.4), score_1, score_2, score_3, artist name, blurry, jpeg artifacts, lowres, censor, bad anatomy, (bad hands:1.3), extra digits, (@bb \(baalbuddy\):1.6), @nel-zel formula, @konoshige \(ryuun\), @haganef, @saigalisk, @iesupa, @firolian, @foxicube, @cinnabus, @yukimaru ai, @mutsutake, shiny hair, eyelashes, long chin, (long jawline:1.4)
```

The review page hides model and prompt identity until explicitly revealed. It records separate votes for identity/attire, action ownership, environment/camera, emotional tone, aesthetics, and the overall winner. Export the resulting judgment JSON when finished. Automatic vision judging should be used only for triage: it can reduce manual review, but it cannot replace paired seeds or final human inspection.

Run blinded vision triage with Google AI Studio after image generation:

```text
$env:INLAY_VISION_API_KEY="<your Google AI Studio key>"
bun run eval:images:vision --manifest=eval-results/image-study/raw/<run>/manifest.json
```

The evaluator uses `gemini-flash-lite-latest` by default through the official `@google/genai` SDK. It sends only the source rubric and a blinded same-seed image pair, never candidate/model labels or API keys. Raw responses, compatible judgment JSON, and a compact Markdown summary remain under the gitignored run directory. Use `--model=...`, `--max-pairs=...`, or `--output=...` to override the defaults. Calls that are blocked, ambiguous, or below 0.7 confidence are marked for human review.

For a prompt-format-only test, hold one saved sidecar output constant and vary only the rendering strategy:

```text
bun run eval:images --workflow=fixed_detailer_scheduler_workflow_EN_API_named_fields_renamed.json --scenario=rhea_platform_conflict --models=luna --strategies=production,merged-character-blocks,merged-tags-first --seeds=1103,2909 --generate
```

Convert an exported review into a ranked Markdown report with:

```text
bun run eval:images:report --manifest=eval-results/image-study/raw/<run>/manifest.json --judgments=<downloaded-judgments.json>
```
