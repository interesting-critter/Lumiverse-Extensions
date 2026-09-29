# bench/ — LumiScript performance + stability harness

Measurement foundation for the deep perf/stability analysis (see the project
plan). Built on the product's existing `diagnostic-stats` IPC + the Diagnostics
registry counters — running these needs no production code changes.

## Running

```bash
bun run bench/baseline.bench.ts        # Phase 0 plumbing sanity check
# (Phase 2+)
bun run bench/engine/*.bench.ts        # in-process engine/storage timing
bun run bench/ipc/*.bench.ts           # real-child IPC round-trip timing
```

`*.bench.ts` files are plain Bun scripts (NOT `bun test`), so they stay out of
the test suite and may run long. The CI bundle-size budget guard is a real
`bun test` file at `tests/bundle-size.test.ts` — it lives under `tests/` (not
here) because `bunfig.toml` scopes `bun test` discovery to `root = "./tests"`.

## Layout

```
_harness/
  timer.ts           Histogram (p50/p95/p99) + Stopwatch + timeAsync/timeBatch
  report.ts          aligned console tables + JSON/CSV emit
  registry-probe.ts  registry-cardinality snapshot/diff (leak detection)
  mem-sampler.ts     diagnostic-stats IPC memory time-series
  e2e-bootstrap.ts   bench-local parent+child bring-up (no bun:test dependency)
  scenario.ts        generic setup -> warmup -> timed-iters -> teardown runner
baseline.bench.ts    Phase 0 plumbing validation
results/             raw run artifacts (gitignored)
```

## Fidelity caveats (read before trusting numbers)

- **R1 — in-memory IPC != real subprocess pipe.** The child runs in-process
  over a mock channel, so IPC latencies exclude kernel context-switch +
  structured-clone-over-pipe, and memory is the COMBINED process footprint, not
  an isolated child. Good for relative/algorithmic regressions + leak TREND;
  validate absolutes against the running app.
- **R2 — observer effect.** Per-call stamping perturbs sub-millisecond ops; use
  `timeBatch` for the tightest loops.
- **R3 — in-memory storage != disk.** Storage benches measure CPU/serialization,
  a lower bound on production `spindle.userStorage` latency.
- **R4 — RSS is coarse + GC-timing-dependent.** Prefer registry-cardinality for
  leak ATTRIBUTION; use RSS for trend only.
- **R6 — frontend render perf is NOT measurable here** (headless Bun + happy-dom,
  no layout/paint/reflow).

Real-app idle reference: ~112 MB mean RSS over 6.3 h, no leak
(`notes/rss-discord-writeup.md`).
