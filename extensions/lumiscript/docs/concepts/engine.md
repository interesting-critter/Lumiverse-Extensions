# Execution engine & isolation

Your scripts run inside a sandbox. As of v2.0 that sandbox comes in two flavours — and, under one of them, with a choice of how strictly scripts are isolated from each other. Both are global settings under **Settings → Script Execution**. The defaults are good; this page is for when you want to understand or change them.

## The two engines

**AsyncFunction** (default) runs each script body as a fresh `new AsyncFunction(...)` inside the LumiScript worker subprocess. It's fast, battle-tested, and what every script has run on since day one. All scripts in a worker share that subprocess's single JavaScript realm.

**QuickJS (experimental isolate)** runs each body inside a QuickJS-WASM virtual machine embedded in the same worker. It's a stronger sandbox: user code executes in a separate WASM realm that never had a reference to the host runtime in the first place, rather than one locked down after the fact. It's behaviourally faithful to AsyncFunction across the observable surface — the same `api.*`, the same `data`, the same handler model — proven by a dual-engine parity test suite. A handful of narrow divergences are catalogued in [API stability → Engine divergence](../api-stability.md#engine-divergence-quickjs-isolate-opt-in); read that before you switch a non-trivial script.

Why choose QuickJS? Stronger isolation of user code from the host, a per-context memory cap, and — with per-script isolation (below) — the guarantee that one script can't observe or corrupt another's sandbox. It's marked experimental because it's newer; AsyncFunction stays the default while QuickJS accumulates a field record.

### Switching engines

The **Engine** dropdown flips it. Because a script's live state — registered handlers, panels, timers — belongs to one engine and can't be moved across, switching **reloads every active script**:

- **Startup scripts re-run** immediately (spaced out). Any panels, toasts, or LLM calls they make on startup happen again.
- **Event-driven scripts** have their live state cleared and re-arm on their next trigger — no automatic re-run.

A confirm dialog spells out exactly which scripts are affected before you commit. The setting is **global** (every script, not per-script) and persisted. In-flight runs finish on the current engine first.

If the QuickJS WASM module can't instantiate on your platform, an individual run **degrades to AsyncFunction for that run** rather than failing — and without changing your setting, so it self-heals if a later run can load the VM. (The [Diagnostics](../guides/diagnostics.md) Engine panel tells you whether this is happening.)

## Isolation (QuickJS only)

Under QuickJS, the **Isolation** dropdown chooses how scripts share the VM. It has no effect under AsyncFunction — there the control is disabled and reads *Shared context*, since AsyncFunction already runs every script in one shared realm.

- **Shared context** (default) — one QuickJS context backs every script. Lower memory. LumiScript's own scaffolding is frozen, so the only residual is that a script mutating a third-party library's internals (e.g. `z.object = evil`) would see that bleed across its *own* later runs; it still can't reach another script or the host.
- **Per-script isolation** — each script gets its own context: its own `globalThis`, its own `zod` / `Handlebars` / `api` bundle, and its own memory cap (~64 MB per context, from a bounded LRU pool). One script cannot observe or poison another's sandbox.

Switching Isolation respawns the QuickJS worker(s) so every context rebuilds under the new model — the same reload impact as an engine switch. If you set per-script and later switch back to AsyncFunction, your choice is **preserved**: the dropdown greys out and reads *Shared context*, but the moment you return to QuickJS your per-script preference is active again.

Per-script isolation costs memory — each context reserves up to ~64 MB, and QuickJS workers weigh more than AsyncFunction workers generally, so a busy pool can pressure the worker memory ceiling. See [Worker pool](workers.md) for tuning.

> **`globalThis` scope:** under per-script isolation each script has its *own* `globalThis`, so the cross-script key-collision warning in the [storage model](storage-model.md) doesn't apply there. Under AsyncFunction and under QuickJS *shared* isolation, `globalThis` is shared across scripts in the worker — namespace your keys.

## Stream buffer (QuickJS only)

A QuickJS `api.llm.generateStream` can outlive the run that opened it, so its chunk queue is bounded. The **Stream buffer** setting (default 512) caps how many undrained chunks may queue before the stream is cancelled with an error — a backstop against a slow or abandoned consumer growing memory without limit. No effect under AsyncFunction. You'll only touch this if you stream large volumes and deliberately consume slowly.

## Behavioural divergences

QuickJS is behaviourally faithful, but not byte-identical. The full, current list — self-`api.tools.invoke`, the bare-`fetch` response shape, `AbortSignal` handling, the fired-handler environment — lives in **[API stability → Engine divergence](../api-stability.md#engine-divergence-quickjs-isolate-opt-in)**. If you're moving a non-trivial script to QuickJS, skim it first.

## See also

- [API stability → Engine divergence](../api-stability.md#engine-divergence-quickjs-isolate-opt-in) — the authoritative divergence catalogue.
- [Worker pool](workers.md) — worker count, idle timeout, and the memory ceiling (which QuickJS pressures harder).
- [Network egress](../guides/network.md) — reaching local model servers past the SSRF block.
- [Diagnostics](../guides/diagnostics.md) — the Engine panel: is QuickJS actually running here, and did anything fall back?
