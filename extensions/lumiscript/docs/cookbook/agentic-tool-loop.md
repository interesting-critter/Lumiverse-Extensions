# Build an agentic tool-call loop

Let the model decide which tools to call, run them, feed the results back, and repeat until it has an answer — the core loop behind every "the assistant looked something up before replying" interaction.

## What you'll use

- [`api.llm.generateWithTools`](../guides/llm.md#generatewithtoolsmessages-tools-options-schema--tool-using-generation) — one round-trip per call; **you** write the loop around it.
- The `generation` permission. ([Permissions](../concepts/permissions.md))
- `ls:startup` as the demo trigger so the script runs once the moment you enable it. ([Trigger model](../concepts/trigger-model.md))

The one thing to internalise up front: **`generateWithTools` is not itself a loop.** Each call returns *either* `tool_calls` (the model wants to run tools) *or* a final `content` string. Turning that into a multi-step agent is your job — which is good, because it means you control the cap, the dispatch, and what happens between rounds.

## The script

```js
// @triggers ls:startup
// An agentic loop: the model answers a question by calling local tools.
// Permission: generation.

// 1. The tools, as plain functions. Each takes the model's args, returns a string.
const TOOLS = {
  async get_distance({ from, to }) {
    const table = { 'paris|tokyo': 9714, 'new york|paris': 5837, 'new york|tokyo': 10848 };
    const key = [from, to].map((s) => String(s).toLowerCase()).sort().join('|');
    return table[key] ? `${table[key]} km` : `No distance on file for ${from} ↔ ${to}.`;
  },
  async convert_currency({ amount, from, to }) {
    const usd = { USD: 1, EUR: 1.08, JPY: 0.0064, GBP: 1.27 };
    if (!usd[from] || !usd[to]) return `Unknown currency (have: ${Object.keys(usd).join(', ')}).`;
    return `${amount} ${from} = ${((amount * usd[from]) / usd[to]).toFixed(2)} ${to}`;
  },
};

// 2. The schemas the model reads to decide when + how to call each tool.
//    Flat list — no { type: 'function', function: {...} } wrapper; the engine adds that.
const toolDefs = [
  {
    name: 'get_distance',
    description: 'Distance in km between two major cities. Use for any "how far" question.',
    parameters: {
      type: 'object',
      properties: { from: { type: 'string' }, to: { type: 'string' } },
      required: ['from', 'to'],
    },
  },
  {
    name: 'convert_currency',
    description: 'Convert an amount between USD, EUR, JPY, GBP. Use instead of guessing rates.',
    parameters: {
      type: 'object',
      properties: { amount: { type: 'number' }, from: { type: 'string' }, to: { type: 'string' } },
      required: ['amount', 'from', 'to'],
    },
  },
];

const question = 'How far is Tokyo from Paris, and what is 250 EUR in JPY?';

const messages = [
  {
    role: 'system',
    content: 'You answer travel questions. Call the tools for any fact you are unsure of — never guess a distance or an exchange rate.',
  },
  { role: 'user', content: question },
];

let answer = '(the model never produced a final answer)';

for (let step = 0; step < 6; step++) {              // hard cap — your runaway guard
  const result = await api.llm.generateWithTools(messages, toolDefs, { connectionName: 'fast' });

  // No tool calls → the model is done. result.content is the final answer.
  if (!result.tool_calls?.length) {
    answer = result.content;
    break;
  }

  // (a) Record the model's tool-use turn as an assistant message of tool_use parts.
  messages.push({
    role: 'assistant',
    content: result.tool_calls.map((tc) => ({
      type: 'tool_use',
      id: tc.call_id,
      name: tc.name,
      input: tc.args,
    })),
  });

  // (b) Run each requested tool; feed every result back as a tool_result part.
  for (const tc of result.tool_calls) {
    const fn = TOOLS[tc.name];
    const output = fn ? await fn(tc.args) : `No such tool: ${tc.name}`;
    messages.push({
      role: 'user',
      content: [{ type: 'tool_result', tool_use_id: tc.call_id, content: String(output) }],
    });
  }
}

console.log(answer);
```

Enable the script and watch the editor console: the model calls `get_distance` and `convert_currency`, then writes a sentence using both results — usually in two rounds.

## How it works

**One call, one decision.** `generateWithTools(messages, toolDefs, options)` does a single round-trip and hands back an [`LLMRawResult`](../guides/llm.md#the-result-shape). The `tool_calls` array (each entry `{ name, args, call_id }`) is populated when the model wants tools; otherwise `content` holds the answer. The loop just keeps calling until `tool_calls` comes back empty.

**The message protocol is strict.** After a tool-call round you must append *two* things before the next call:

1. an `assistant` message whose `content` is an array of `tool_use` parts — one per call, carrying the model's own `call_id` as `id`, plus `name` and `input`;
2. a `user` message of `tool_result` parts — each referencing the matching call via `tool_use_id`, with the tool's output as a **string** in `content`.

The `call_id` is the thread that ties a result back to its request — copy it across verbatim. Note there is no `role: 'tool'`; tool traffic always rides inside `content` parts of `user` / `assistant` messages. (See [the message shape](../guides/llm.md#message-shape).)

**The cap is yours.** The host has no idea this is a loop, so there's no runaway guard — the `step < 6` bound is the only thing standing between you and an infinite tool-calling spiral if a tool keeps returning something the model can't resolve. Most tasks converge in one to three rounds; pick a cap a few above that.

**Why `connectionName: 'fast'`.** Tool loops can fire several model calls per task. Point them at the user's cheap/low-latency profile rather than burning their flagship on orchestration — see [the connection-profiles note](../guides/llm.md#connection-profiles--pick-fast-for-sub-agent-work).

## Make it yours

- **Real tools.** The stubs above are pure functions so the recipe runs with nothing set up. Swap them for real capability: `api.databanks.*` lookups, an `api.utils.http.get` fetch (needs `allowDangerous`), `api.worldInfo.*` queries, `api.chat.getMessages` for scene context. The loop body doesn't change — only the `TOOLS` map and `toolDefs`.
- **Publish the tools instead of inlining them.** If you register the tools with [`api.tools.register`](../guides/tools.md), dispatch each call with `await api.tools.invoke(tc.name, tc.args)` instead of a local map — now the Council can use the same tools too. See [Register a tool the Council can use](council-tool.md).
- **Drive it from chat.** Replace the `ls:startup` trigger + hard-coded `question` with `// @triggers MESSAGE_SENT`, then `if (!data.message.is_user) return;` (MESSAGE_SENT also fires for the assistant placeholder) and `const question = data.message.content`, then `await api.chat.sendMessage(answer, { role: 'assistant' })` to reply in-thread (needs `chat_mutation`).
- **Stop early.** Inspect tool outputs mid-loop and `break` when you have enough, or short-circuit a round by injecting your own `user` message between calls.

## Gotchas

- **`generateWithTools` ≠ a loop.** The single most common surprise. One call = one round-trip. (The 4-arg form takes a final-content schema, but it still doesn't loop — and it *silently* falls back to unvalidated content on a schema mismatch, unlike `generateStructured` which throws. See [the 4-arg semantics](../guides/llm.md#generatewithtoolsmessages-tools-options-schema--tool-using-generation).)
- **`content` of a `tool_result` must be a string.** Stringify structured data yourself (`JSON.stringify(obj)`) and tell the model in the system prompt what shape to expect.
- **Forgetting to append the `tool_use` turn** before the `tool_result` turn confuses the model and some providers reject the mismatched history. Always push the assistant `tool_use` message first, then the results.
- **No `await` inside the dispatch loop?** Each tool runs sequentially here. If your tools are independent and slow, `Promise.all` them — but keep the result-append order matched to the `call_id`s.

## See also

- [Calling the LLM](../guides/llm.md) — the full `api.llm` surface, including `generateStructured` and streaming.
- [Register a tool the Council can use](council-tool.md) — the publisher side; pairs with this loop.
- [Handler lifetime](../concepts/handler-lifetime.md) — relevant once you move tool registration into a long-lived script.
