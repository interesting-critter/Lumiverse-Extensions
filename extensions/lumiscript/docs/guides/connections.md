# Connection profiles

`api.connections.*` is a **read-only** window onto the user's LLM connection profiles — the same profiles `api.llm` generates against and the same ones the Lumiverse UI manages. Use it to let the user pick a connection, introspect what's configured, or route a specific generation to a named profile. Free tier — no permission required.

```js
const conns = await api.connections.list();
for (const c of conns) {
  console.log(c.name, '→', c.provider, c.model, c.is_default ? '(default)' : '');
}
```

## Four reads

- `list()` → `Connection[]` — every profile the user has.
- `get(connectionId)` → `Connection | null` — one by id, or `null` if absent.
- `getDefault()` → `Connection | null` — the `is_default` profile, falling back to the first available, or `null` if there are none.
- `findByName(name)` → `Connection | null` — case-insensitive name match.

## The `Connection` shape is safe

A `Connection` mirrors the host profile in **snake_case**:

```js
{
  id, name, provider, api_url, model,
  preset_id,            // bound generation preset id, or null
  is_default,           // boolean
  has_api_key,          // boolean — whether a key is stored
  metadata,             // provider-specific bag
  reasoning_bindings,   // ConnectionReasoningBindings | null — { settings, promptBias? }
  created_at, updated_at,
}
```

The one field you might reach for and **won't find is the API key**. A connection carries `has_api_key: boolean` so you can tell whether a key is configured — never the key itself. This is deliberate and matches every credential-bearing surface in LumiScript.

`reasoning_bindings` is the connection's saved reasoning configuration — a `settings` snapshot (effort, thinking display, plus delimited-parsing rules) and an optional `promptBias` "Start Reply With" prefill, or `null` when the connection has none bound. It's read-only here; to change reasoning for a *single* generation without touching the profile, pass the `reasoning` option to `api.llm` instead (see the **Reasoning** section of [the LLM guide](llm.md)).

## Why read-only

There is no `create` / `update` / `delete` here. Connections hold provider credentials, so they're owned by the user and edited only in Lumiverse settings — a script can read which connections exist and route to them, but never mint or mutate one. (If you want to *send the user* to the connection settings, that's a navigation call — see [UI navigation](ui-navigation.md): `api.ui.openSettings('connections')`.)

## Routing a generation to a chosen connection

A connection's `id` and `name` map directly onto `api.llm` options `connectionId` / `connectionName`. So "pick a connection, then generate with it" is a two-liner:

```js
const conn = await api.connections.findByName('Claude Opus');
if (conn) {
  const reply = await api.llm.generate(
    [{ role: 'user', content: 'Write a haiku about cron jobs.' }],
    { connectionId: conn.id },     // or { connectionName: conn.name }
  );
}
```

`connectionId` takes precedence over `connectionName`. See [Calling the LLM](llm.md) for the full options surface (the LLM calls themselves require the `generation` permission).

## Building a connection picker

Pair `list()` with the host `mountSelect` component for a themed dropdown (see [Mounting shared host components](components.md)):

```js
const conns = await api.connections.list();
const slot  = api.ui.dom.inject('body', '<div></div>');   // needs app_manipulation

api.ui.components.mountSelect(slot, {
  options: conns.map((c) => ({ value: c.id, label: `${c.name} · ${c.provider}` })),
  onChange: async (id) => {
    await api.variables.global.set('preferredConnection', id);
  },
});
```

For a model-aware picker, `mountModelCombobox` binds to a connection and lets the host manage the model list for you.

## Notes

- **Free tier**, but the data is per-user — the call is scoped to the active user automatically; you never pass a user id.
- `getDefault()` mirrors how `api.llm` resolves a connection when you don't name one: `is_default` first, else the first profile.
- Not persisted/cached for you — call `list()` when you need a fresh view (e.g. the user may have added a profile since your last read).

See the in-app **Reference tab** for the full `Connection` field list.
