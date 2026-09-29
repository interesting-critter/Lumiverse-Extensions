# What's new in v2.0

LumiScript 2.0 is a major release built around a new, opt-in execution engine. It is **backwards-compatible**: the public `api.*` surface is unchanged, so existing scripts keep working. The major-version bump signals the new engine, not breaking changes (see [API stability](api-stability.md)).

> **One behavior change to know about — private-network egress.** As of v2.0, outbound requests to **localhost / private-IP / IP-literal** hosts are blocked by default. If you have a script that reaches a local model server (Ollama, ComfyUI, LM Studio) or a LAN device, it will now be rejected until you allowlist that host under **Settings → Network → Allowed private hosts**. Full details + the fix: [Network egress](guides/network.md).

## The headline: the QuickJS-WASM isolate engine

Scripts still run under the **AsyncFunction** engine by default — it's unchanged and now the long-term-support runtime. v2.0 adds an **opt-in QuickJS-WASM isolate** engine you can switch to in the LumiScript settings:

- User code runs in a WebAssembly VM where host globals like `Bun` and `process` don't exist — real structural isolation, not just a `with`-scope shadow.
- **Per-script memory and CPU limits**, a bounded context pool, and graceful teardown of a misbehaving script.
- **Optional per-script context isolation** — one script can't see or poison another's `globalThis` or shared libraries (`z`, Handlebars, …).

The two engines are behaviorally faithful across the observable surface (proven by a dual-engine parity harness); a handful of deliberate divergences are catalogued in [API stability → Engine divergence](api-stability.md). Pick and tune the engine in [Execution engine](concepts/engine.md); tune isolation and the worker pool in [Worker pool](concepts/workers.md).

## New API surface

Additive, non-breaking — reach for these when you need them:

- **`api.permissions`** — proactive permission introspection. `await api.permissions.has('images')` and `api.permissions.getGranted()` let a script check what's granted *before* calling a gated method, instead of only catching the denial. Free tier, no permission of its own. See [Permissions](concepts/permissions.md).
- **`api.worldInfo` global activation** — `getGlobal` / `setGlobal` / `activateGlobal` / `deactivateGlobal` manage the user's *global* world books, which apply to every chat regardless of character/chat binding. See [World info](guides/world-info.md).
- **`api.personas.addons`** — list / get / update the user's global persona add-ons. See [Personas](guides/personas.md).
- **`api.chat.setStyleMode`** — choose how strictly the host scopes your extension's injected styles (`bounded` vs `extension-relaxed`) — useful for full-bleed or heavy-theming scripts. See [Theming the host UI](guides/theme.md).

## For script authors: sharper IntelliSense

The editor's type hints (Monaco IntelliSense) are now **generated directly from the frozen `api.*` contract** rather than hand-maintained, so the autocomplete and hover docs can no longer drift out of date — what you see in the editor is exactly what your script receives.

## Also in 2.0

- Engine + worker-pool **diagnostics** panels, and a **Download Report** button for support snapshots.
- Tighter outbound-network security (the private-host allowlist above).
- **Export-pack** is now a script-selection modal; sidebar folders gained an explicit remove button; plus a stack of smaller fixes.

---

*Upgrading and something looks off? The in-app **Reference** tab is always current with your installed version, and **Lisa** (the in-app assistant) knows the whole API — ask her.*
