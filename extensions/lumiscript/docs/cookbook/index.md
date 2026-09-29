# Cookbook

Goal-oriented recipes. Each one is a complete, copy-paste-runnable script for a real task, plus the reasoning behind the load-bearing parts. Reach for these once you're past "hello world" and want a working start for a *specific thing* — they pull several `api.*` primitives into one script the way a real build does, rather than documenting one API at a time (that's what the [guides](../index.md) are for).

Every recipe follows the same shape: **what you'll use** (APIs / events / permissions, linked to the guides) → **the script** (one runnable block) → **how it works** → **make it yours** → **gotchas**.

## LLM & tools

- **[Build an agentic tool-call loop](agentic-tool-loop.md)** — let the model call local tools, feed the results back, and loop until it has an answer.
- **[Register a tool the Council can use](council-tool.md)** — publish a tool (a dice roller) that both your scripts and the Council can invoke.

## Prompt context

- **[Inject dynamic prompt context per turn](dynamic-prompt-context.md)** — compute something fresh each turn and slip it into the prompt the model sees, without touching the visible chat.

## Interactive UI

- **[Build a draggable info panel](draggable-panel.md)** — a script-owned, draggable DOM panel that survives navigation.
- **[Theme the app from a character's image](theme-from-image.md)** — recolor Lumiverse to match the active character's avatar.

## Data & coordination

- **[Share state between two scripts](share-state-broadcast.md)** — one script produces state, another reacts in real time, neither knowing the other's triggers.
- **[Persist state the right way](persist-state.md)** — match each kind of state to the storage tier that actually fits it.

## Integration

- **[Call an OAuth-protected API (PKCE)](oauth-pkce.md)** — connect a third-party account and call its API on the user's behalf, with no client secret to leak.

---

More recipes will land here over time. For the full per-API reference, see the [guides](../index.md) and the in-app **Reference** tab.
