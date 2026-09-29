# Worker pool

LumiScript doesn't run your scripts in the main Lumiverse process — it runs them across a pool of **worker subprocesses**. Spreading scripts over several workers means one misbehaving script (an infinite loop, a crash) can't take the others down with it. The pool tunes itself, but three knobs and a button under **Settings → Workers** let you shape it.

## The knobs

**Worker count** (default 4, range 1–16). How many worker subprocesses to run. More workers means better fault isolation — scripts are spread thinner, so one bad script affects fewer peers — at the cost of memory (roughly 50–100 MB per idle worker under the AsyncFunction engine). Scripts are assigned to workers lazily and stick to their worker; the host caps you at 16 processes per extension.

**Idle timeout** (default 30 min). A worker with nothing to do — no script firing, no handler invocation — is torn down to reclaim memory after this long. The next event for any script assigned to it respawns the worker (~150–300 ms cold-start hitch). Set it very high to effectively disable idle eviction.

**Memory ceiling** (default 512 MB). A ceiling on the *combined* resident memory of all workers. When the total crosses it, the pool LRU-evicts an *eligible* worker — one with no active run, and never the last warm worker — until the total drops back under. It's a soft, graceful mechanism, not a hard kill.

**Rebalance pool** (button). Releases every script→worker assignment; each script's next fire reassigns it to the least-loaded worker. Assignments are sticky by default, so this is what you press after *raising* the worker count — to spread existing scripts onto the new workers instead of leaving them where they were.

## Eviction is graceful, not forced

Both idle and memory eviction only ever remove a worker that's **idle and unpinned**, and never the last warm worker. A worker is *pinned* — exempt from eviction — while it holds any live registration: a `broadcast.on` subscription, a registered tool or macro, a drawer tab, an RPC endpoint, an OAuth callback. That's what keeps background subscriptions responsive (see [Handler lifetime](handler-lifetime.md)). The flip side: a pinned worker that's grown large won't be force-evicted, so the memory ceiling is a target the pool works toward, not a guarantee it can always hit.

## QuickJS weighs more

The memory ceiling bounds each worker's whole-process RSS, and a worker running the [QuickJS engine](engine.md) weighs substantially more than an AsyncFunction worker: it carries the WASM runtime (~112 MB baseline) plus per-script context heap — up to ~512 MB for a fully-loaded per-script-isolation pool. So the *same* ceiling evicts and respawns far more aggressively under QuickJS. If you run QuickJS across several workers — especially with per-script isolation — expect to **raise the memory ceiling** (or lower the worker count) to avoid churn. That interaction is the main reason these settings are worth thinking about together once you're on QuickJS.

## See also

- [Execution engine](engine.md) — AsyncFunction vs QuickJS, and why QuickJS workers weigh more.
- [Handler lifetime](handler-lifetime.md) — what pins a worker and keeps it from being evicted.
- [Diagnostics](../guides/diagnostics.md) — the Script-runner and Engine panels show live worker + memory state.
