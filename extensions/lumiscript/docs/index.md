# LumiScript

> A JavaScript scripting platform for the Lumiverse AI chat application. Write scripts that react to chat events, automate behaviour, inject prompt context, register custom tools and macros, and build interactive UI.

LumiScript is a Lumiverse extension. Your scripts run server-side inside a sandboxed Bun subprocess; they get an `api.*` object that lets them touch chat messages, world books, presets, the LLM, persistent storage, custom DOM in the host app shell, and 15+ other capability namespaces. Scripts are written, edited, and tested entirely inside Lumiverse — there's no external toolchain, no separate IDE, no deploy step. Save the script and it's live. Bodies run under a selectable engine — the default AsyncFunction runtime, or an opt-in QuickJS-WASM isolate with stronger sandboxing (see [Execution engine](concepts/engine.md)).

## Who this is for

These docs assume:

- You've used Lumiverse as a regular user and you're now ready to extend it.
- You know JavaScript well enough to read async/await code without flinching. We don't teach JavaScript here — only the LumiScript-specific shape on top.
- You're authoring scripts that you'll either keep to yourself or share as `.lumiscript.zip` packs with a small circle. The current LumiScript story is "small testing/dev community, no public marketplace" — the docs reflect that posture.

If you're contributing to the LumiScript extension itself (not writing scripts that run inside it), see the gitignored `notes/` directory in the repo for design journals and post-mortems.

## Start here

| If you want to … | Read |
|---|---|
| See what's new in this release (and one upgrade heads-up) | [What's new in v2.0](whats-new-v2.md) |
| Get a script running in the next 10 minutes | [Getting started](getting-started.md) |
| Understand how scripts fire, what `data` is, what `api` is | [Trigger model](concepts/trigger-model.md) |
| Understand permissions, `allowDangerous`, and graceful denial | [Permissions](concepts/permissions.md) |
| Pick the right place to put your data (variables / db / scriptStorage / enclave / files) | [Storage model](concepts/storage-model.md) |
| Understand how registered handlers (`broadcast.on`, `commands.onInvoked`, `macros.register`, etc.) survive past the body that registered them | [Handler lifetime](concepts/handler-lifetime.md) |
| Choose an execution engine (AsyncFunction vs the QuickJS isolate), or tune isolation and the worker pool | [Execution engine](concepts/engine.md) · [Worker pool](concepts/workers.md) |
| Reach a local model server or LAN device past the network block | [Network egress](guides/network.md) |
| Diagnose a problem or export a support report | [Diagnostics](guides/diagnostics.md) |
| Know which `api.*` namespace covers which use case | The in-app **Reference** tab (auto-generated, comprehensive — open the LumiScript panel and switch to the Reference tab) |
| Look up a specific method's signature, args, permissions | Same place — the in-app Reference, or the exported cheat-sheet (`src/assistant/corpus/cheat-sheet.md`) |
| Ask LumiScript questions while you write — and have the assistant draft or fix scripts | [Working with Lisa](guides/lisa.md) |
| Build a specific feature (DOM injection, LLM, macros, tools, broadcast bus, OAuth, theming, image generation, world info, databanks, persistent events, cross-extension RPC, shared components, web search, connection profiles, UI navigation, host/user context, etc.) | [DOM injection](guides/dom-injection.md) · [Calling the LLM](guides/llm.md) · [Custom macros](guides/macros.md) · [Registering tools](guides/tools.md) · [Broadcast bus](guides/broadcast.md) · [OAuth callbacks](guides/oauth.md) · [Theming the host UI](guides/theme.md) · [Generating images](guides/image-gen.md) · [World info](guides/world-info.md) · [Personas](guides/personas.md) · [Databanks](guides/databanks.md) · [Persistent events](guides/persistent-events.md) · [Cross-extension RPC](guides/rpc.md) · [Shared components](guides/components.md) · [Web search](guides/web-search.md) · [Connection profiles](guides/connections.md) · [UI navigation](guides/ui-navigation.md) · [Host & user context](guides/host-context.md) · [Picking files](guides/file-picker.md) · [Reactive UI state](guides/ui-events.md) · [Memory Cortex & chat memory](guides/memories.md) · [Mounting a full-bleed surface](guides/app-mount.md) · [Bundling scripts into character cards](guides/card-scripts.md) — more guides coming, by topic |
| Find a recipe for building a specific thing (agentic loop, Council tool, dynamic prompt context, draggable panel, theming from an image, sharing state, choosing a storage tier, OAuth/PKCE) | **[Cookbook](cookbook/index.md)** — [agentic tool loop](cookbook/agentic-tool-loop.md) · [Council tool](cookbook/council-tool.md) · [dynamic prompt context](cookbook/dynamic-prompt-context.md) · [draggable panel](cookbook/draggable-panel.md) · [theme from image](cookbook/theme-from-image.md) · [share state](cookbook/share-state-broadcast.md) · [persist state](cookbook/persist-state.md) · [OAuth (PKCE)](cookbook/oauth-pkce.md) |

## Where things live

- **In-app Reference tab.** Authoritative auto-generated reference for every `api.*` method, key type, event, permission, broadcast event, and runtime directive. Always current with whatever LumiScript version you're running — these prose docs may lag, the Reference doesn't. **Treat it as the source of truth for surface details.**
- **In-app Lisa (the assistant).** Built-in chat assistant for LumiScript scripting questions. Has the same corpus the Reference renders, plus extra context (REDIRECTS for common hallucinations, namespace concept paragraphs, the trigger / permission model intros). Good for "I want to do X, what's the right primitive?" queries. See [Working with Lisa](guides/lisa.md) for the full walkthrough.
- **Editor console.** Inside the script editor, every `console.log` / `warn` / `error` your script emits lands here, plus any rejected security pattern surfaces as a `[security]` entry. The editor console is what you tail while you iterate.
- **These docs.** Narrative explanations of how the pieces fit together, the "why this is shaped this way" you can't get from a method-list, and recipes for common patterns.

## Conventions in these docs

- **Code blocks are runnable.** If a snippet looks self-contained — copy-paste into a script and it works, modulo any permission your script needs to have granted.
- **Permissions are flagged explicitly.** When a method requires a Spindle permission and/or the per-script `allowDangerous` toggle, we say so inline.
- **Versions** are noted where behaviour differs across LumiScript versions. The current target is **v2.0**.
- **Cross-references** to topics that haven't been written yet are bracketed *(coming next)* or *(coming, by topic)* — they'll resolve as the docs fill in.

## A note on what's NOT here

These docs cover **writing LumiScript scripts**. They do not cover:

- **Using Lumiverse as a chat app.** That's the Lumiverse user docs.
- **Building Lumiverse extensions in general.** That's the Spindle developer docs (shipped with Lumiverse).
- **Forking and modifying the LumiScript extension itself.** LumiScript is MIT-licensed — forks are permitted, but unofficial and unsupported. If you fork, [`FORKING.md`](../FORKING.md) at the repo root has a minimum starting point (build commands, repo layout, what's missing from the public tree).
