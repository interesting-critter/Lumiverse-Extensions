# Sidecar simulation

This harness sends the production Inlay parser contract and production-shaped continuity context to the configured proxy models, then evaluates each response through the same local recovery, continuity, selection, and rendering transformations used by the extension.

Set these environment variables in the process that launches Bun:

- `INLAY_SIDECAR_ENDPOINT`: OpenAI-compatible `/v1/chat/completions` URL
- `INLAY_SIDECAR_API_KEY`: bearer token

Run the complete suite with `bun run eval:sidecars`. Optional filters use `--model=`, `--scenario=`, `--rounds=`, and `--max-requests=`. `--scenario=` accepts one substring or a comma-separated list. For example:

```text
bun run eval:sidecars --model=deepseek --scenario=adaptive_urgent_action --rounds=2
```

Run the isolated adult-only NSFW conformance suite with `bun run eval:sidecars --suite=nsfw`. Its tracked summary omits explicit prompt previews, while full raw responses remain gitignored. In this suite, an empty Gemini response is explicitly reported as censorship rather than a generic parser failure.

Run the cross-genre expansion suite with `bun run eval:sidecars --suite=expanded`. It combines new adult-only NSFW, medieval, magical, and combat fixtures and omits explicit prompt previews from its tracked summary. To benchmark only the added Claude sidecar:

```text
bun run eval:sidecars --suite=expanded --model=sonnet --report-label=claude-sonnet-5
```

The command resolves exact IDs from `/v1/models`, runs models sequentially, writes a sanitized summary to `eval-results/latest-sidecar-summary.md`, and stores detailed raw artifacts in the gitignored `eval-results/raw/` directory. Never put credentials in command arguments, source files, or result artifacts.

To benchmark one additional proxy model without changing the default target set, pass its exact discovered ID through `--model-id=`, an optional display name through `--model-label=`, and a safe summary suffix through `--report-label=`.

To evaluate the already-rendered prompts as actual Anima images without making new sidecar calls, use the paired ComfyUI harness documented in `../image-prompt-study/README.md` or run `bun run eval:images` for a dry-run plan.

## Offline replay

Re-run saved raw sidecar responses through the current local normalization,
canonical planning, and prompt compilation path without making model or image
requests:

```text
bun run src/evals/sidecar-sim/replay-runner.ts --source-root=eval-results/raw/expanded/<run-id>
```

Use `--scenario=`, `--model=`, and `--max-files=` to narrow a replay.
Use `--local-camera-repair=off --allow-differences` for a local-camera-repair
ablation. The summary reports strict-JSON rate, terminal-state omissions,
missing Dynamic `primaryAction` values, repair hits, and camera collisions
before/after transformation. Differences or pipeline errors return a non-zero
exit code; pass `--allow-differences` only for exploratory audits. New live-run artifacts embed
the effective config, source paragraphs, and previous visual state so future
replays do not silently inherit changed fixtures. Older artifacts fall back to
the current scenario registry.

Replay covers the deterministic local half of the harness. It deliberately does
not claim coverage of production remote repair, remote camera repair, avatar
vision, lorebook retry, or host persistence; those stages require separate
instrumented or integration evaluation.
