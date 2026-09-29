# Register a tool the Council can use

Publish a `roll_dice` tool that a Lumiverse **Council** member can fire mid-generation — *and* that any script can call directly. Along the way you'll meet the one non-obvious thing about Council tools: the Council doesn't fill your parameter schema for you.

## What you'll use

- [`api.tools.register`](../guides/tools.md) — publish the tool; needs the `tools` permission.
- [`api.llm.generateStructured`](../guides/llm.md#generatestructuredmessages-schema-options--validated-json) — used on the Council path to decide *what* to roll; needs the `generation` permission.
- `api.variables.local.set` — persist the last roll for other scripts (free tier). ([Storage model](../concepts/storage-model.md))
- The `z` global (Zod) for the structured-decision schema. ([Trigger model](../concepts/trigger-model.md))

## The script

```js
// @triggers ls:startup
// A dice tool for the Council (and for any script via api.tools.invoke).
// Permissions: tools, generation.

// Roll "NdS+M" notation. Self-contained — no external dice library.
function roll(notation) {
  const m = /^\s*(\d*)d(\d+)\s*([+-]\s*\d+)?\s*$/i.exec(notation);
  if (!m) throw new Error(`unparseable notation "${notation}" (try "1d20+3")`);
  const count = Number(m[1] || '1');
  const sides = Number(m[2]);
  const mod   = m[3] ? Number(m[3].replace(/\s+/g, '')) : 0;
  if (count < 1 || count > 100 || sides < 2) throw new Error(`out-of-range notation "${notation}"`);
  const dice  = Array.from({ length: count }, () => 1 + Math.floor(Math.random() * sides));
  return { notation, dice, mod, total: dice.reduce((a, b) => a + b, 0) + mod };
}

api.tools.register(
  'roll_dice',
  {
    display_name: 'Roll dice',
    description:  'Resolve an uncertain action with a dice roll. Use when the scene calls for a random outcome.',
    council_eligible: true,
    // The schema is the contract for the DIRECT paths only (api.tools.invoke +
    // api.llm.generateWithTools). The Council path does NOT fill it — see below.
    parameters: {
      type: 'object',
      properties: { notation: { type: 'string', description: 'Dice notation, e.g. "1d20+3" or "2d6".' } },
      required: ['notation'],
    },
  },
  async (args, api, ctx) => {
    // 1. Work out a notation. Two paths converge here.
    let notation = typeof args.notation === 'string' ? args.notation : '';

    // Council path: the host hands us { context, __deadlineMs } — NOT the
    // schema's `notation` (extension tools get no sidecar LLM to fill it). So
    // when there's no notation but there IS scene context, decide one ourselves.
    if (!notation && typeof args.context === 'string') {
      const decision = await api.llm.generateStructured(
        [
          { role: 'system', content: 'You are a tabletop GM. Choose dice notation to resolve the most uncertain action in the scene, or return "" if no roll is warranted.' },
          { role: 'user',   content: args.context },
        ],
        z.object({ notation: z.string().describe('Dice notation like "1d20+3", or "" for no roll.') }),
        { connectionName: 'fast' },
      );
      notation = decision.notation.trim();
    }

    if (!notation) return 'No roll needed.';

    // 2. Roll. A bad notation becomes the tool result (not a throw) so the
    //    Council deliberation shows a useful message instead of a red error.
    let r;
    try { r = roll(notation); }
    catch (err) { return `Couldn't roll that: ${err.message}`; }

    // 3. Persist the latest roll so panels / other scripts can read it.
    await api.variables.local.set('lastDiceRoll', { ...r, at: Date.now() });

    // 4. Council path: narrate in the invoking member's voice. Direct: plain.
    const who = ctx?.councilMember ? `${ctx.councilMember.name} rolls` : 'Rolled';
    const breakdown = (r.dice.length > 1 || r.mod)
      ? ` [${r.dice.join(', ')}${r.mod ? `, ${r.mod > 0 ? '+' : '−'}${Math.abs(r.mod)}` : ''}]`
      : '';
    return `${who} ${r.notation} → **${r.total}**${breakdown}`;
  },
);

console.log('roll_dice registered. Assign it to a Council member, or: api.tools.invoke("roll_dice", { notation: "2d6+1" })');
```

Enable the script. Then either assign **Roll dice** to a Council member in Lumiverse's Council UI (it fires when their chance roll succeeds during generation, with the result inlined into the reply), or test it directly from any script:

```js
console.log(await api.tools.invoke('roll_dice', { notation: '1d20+5' }));
// → "Rolled 1d20+5 → 17 [12, +5]"
```

## How it works

**One registration, two invocation surfaces.** [`api.tools.register(name, def, handler)`](../guides/tools.md#registername-def-handler-reference) publishes the tool to both Lumiverse's Council (because `council_eligible: true`) and the direct-invocation surface (`api.tools.invoke`, and `api.llm.generateWithTools` tool loops). The same handler runs for all of them.

**The handler signature is `(args, api, ctx)`.** Use the `api` *parameter*, not the closure — the engine injects the current live API surface at call time, so a tool registered minutes ago still works (see [why `api` is a parameter](../guides/tools.md#the-handler-signature)). `ctx?.councilMember` is populated **only** on the Council path; it's `undefined` for `invoke` / `generateWithTools`. Always optional-chain it.

**The load-bearing subtlety: the Council does not fill your schema.** This trips everyone up. On the direct paths, the *model* produces structured args matching your `parameters` schema, so `args.notation` is there. On the **Council** path, Lumiverse invokes your extension tool with `{ context, __deadlineMs }` — the scene text and a deadline, but **not** `notation`. There's no sidecar LLM filling the schema for extension tools. So a naïve `args.notation`-only handler silently produces nothing useful when a Council member fires it. The fix is the dual-mode branch above: trust `args.notation` when present, otherwise read `args.context` and run a quick `generateStructured` to *decide* the roll. Point that inner call at the user's `'fast'` profile — it's auxiliary work.

**Errors vs. results.** Returning a string is the success path; that string is what the Council shows / what `invoke` resolves to. *Throwing* marks your script errored (red status dot + toast) and Council records a failure. So for "expected" bad input (an unparseable notation), return a message rather than throw — reserve throws for genuine faults.

## Make it yours

- **Hot-reload during dev.** Add `// @ls:reload-on-edit` near the top so saving the script re-runs the body and re-registers the handler (same-script re-registration replaces silently). Otherwise the old handler lives until you disable/enable.
- **Richer GM logic.** A production dice tool layers on difficulty classes, success/failure margins, an "impossible DC" self-correction loop, and per-character roll history in `api.db`. Graft those on once the basics work.
- **Council member voice.** For prose-heavy tools, replace the simple narration with the `ls:council-prompt` built-in library, which builds a system prompt byte-identical to Lumiverse's own Council tools — `const { buildCouncilMessages } = await script.require('ls:council-prompt')`. See [Registering tools → the `ls:council-prompt` library](../guides/tools.md#the-lscouncil-prompt-built-in-library).
- **Use it inside an agent.** Because it's registered, an [agentic loop](agentic-tool-loop.md) can dispatch model tool-calls to it with `await api.tools.invoke(tc.name, tc.args)` instead of a local function map.
- **Observe every fire.** Subscribe to the `ls:tool:invoked` broadcast (`api.broadcast.on('ls:tool:invoked', …)`) — it carries `{ name, args, result, callMs, councilMember? }` on both paths, handy for a tool-activity panel.

## Gotchas

- **Council ≠ structured args.** Covered above — the single biggest surprise. Build the `args.context` fallback or your tool no-ops for Council members.
- **Handlers must return a string.** Return an object and it stringifies inelegantly. JSON-stringify structured data yourself and shape the `description` to tell the model what to do with it.
- **Tool names are a single global registry.** Two scripts can't both register `roll_dice` — the second throws. Prefix names (`mygame_roll_dice`) on scripts you share with others.
- **`council_eligible` defaults to `false`.** If the tool doesn't appear in the Council picker, you forgot to set it `true`.
- **Auto-cleanup is handled.** On disable/delete LumiScript unregisters your tools for you; you only call `api.tools.unregister` for mid-life removals.

## See also

- [Registering tools](../guides/tools.md) — the full `api.tools` surface, Council context fields, and the `ls:council-prompt` helper.
- [Build an agentic tool-call loop](agentic-tool-loop.md) — the consumer side: have the model call tools like this one.
- [Calling the LLM](../guides/llm.md) — `generateStructured` (used here) and the rest of `api.llm`.
