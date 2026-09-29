# Personas

`api.personas.*` is CRUD over Lumiverse's **personas** — the user's own identity profiles (name, description, pronouns, avatar, an optional attached world book) — plus active-persona switching and a read hook into the world book a persona carries. It's the LumiScript surface for everything from "let the user swap personas from a script UI" through "auto-attach a lorebook when a persona activates" to "sync persona metadata from an external profile."

A persona is the *user's* side of a chat, the counterpart to a character. There's exactly one **active** persona at a time (or none), plus a **default** persona (`isDefault: true`) that seeds new chats.

## Surface

Two layers plus a nested add-ons namespace:

- **Persona CRUD + active switching** — `list` / `get` / `getDefault` / `getActive` / `create` / `update` / `delete` / `switchActive`.
- **Attached world book** — `getWorldBook`, a convenience read that resolves the persona's `attachedWorldBookId` to a full `WorldInfo`.
- **Global add-ons** (nested under `addons`, **new in v2.0**) — `list` / `get` / `update` over the injectable content blocks that pair with a generation's persona add-on states.

```ts
interface PersonasAPI {
  list(options?):        Promise<{ data: Persona[]; total: number }>;
  get(personaId):        Promise<Persona | null>;
  getDefault():          Promise<Persona | null>;
  getActive():           Promise<Persona | null>;
  create(input):         Promise<Persona>;
  update(personaId, input): Promise<Persona>;
  delete(personaId):     Promise<boolean>;
  switchActive(personaId): Promise<void>;
  getWorldBook(personaId): Promise<WorldInfo | null>;

  addons: {
    list(options?):      Promise<{ data: PersonaAddonInfo[]; total: number }>;
    get(addonId):        Promise<PersonaAddonInfo | null>;
    update(addonId, input): Promise<PersonaAddonInfo>;
  };
}
```

## Permissions

The entire namespace — persona CRUD, `switchActive`, `getWorldBook`, **and** the nested `addons.*` — is gated on the single **`personas`** permission. Missing it throws `Error: PERMISSION_DENIED:personas — grant this permission to use this API`.

Note the one asymmetry worth knowing: `getWorldBook` returns a `WorldInfo` but needs **only `personas`**, not `world_books`. It's a scoped read of the persona's own attached book, so it rides the persona gate.

## Refs: UUID only

Unlike `api.worldInfo.*`, every persona method takes the persona's **UUID string** — there is no name-based resolution and no `getByName`. If you only have a name, `list()` and filter:

```js
const { data } = await api.personas.list();
const persona = data.find(p => p.name === 'Aria');
if (persona) await api.personas.switchActive(persona.id);
```

`list()` is paginated: `list({ limit, offset })` returns `{ data, total }`. Called with no argument it returns the first page (host default page size) plus the full `total` count, so you can page if needed.

## The `Persona` shape

`get` / `getDefault` / `getActive` / `create` / `update` all return a `Persona` (or `null` where noted):

| Field | Type | Meaning |
|---|---|---|
| `id` | `string` | Persona UUID. |
| `name` | `string` | Persona name. |
| `title` | `string` | Short tagline shown in the persona picker. |
| `description` | `string` | Persona description. |
| `imageId` | `string \| null` | Avatar image ID. Fetch via `api.images.*`; `null` if no avatar. |
| `attachedWorldBookId` | `string \| null` | World book attached to this persona, or `null`. See `getWorldBook`. |
| `folder` | `string` | Organisational folder label. |
| `isDefault` | `boolean` | Whether this is the user's default persona. |
| `subjectivePronoun?` | `string` | e.g. `"he"`, `"she"`, `"they"`. Optional. |
| `objectivePronoun?` | `string` | e.g. `"him"`, `"her"`, `"them"`. Optional. |
| `possessivePronoun?` | `string` | e.g. `"his"`, `"her"`, `"their"`. Optional. |
| `metadata` | `Record<string, unknown>` | Arbitrary metadata. |
| `createdAt` / `updatedAt` | `number` | Unix-ms timestamps. |

Fields arrive in **camelCase** — LumiScript maps the host's snake_case DTO (`attached_world_book_id`, `is_default`, `image_id`, the pronoun fields) to camelCase for you, so scripts never see the raw DTO shape.

## Quick start

Create a persona, make it default, then activate it:

```js
// @triggers ls:startup

const persona = await api.personas.create({
  name:              'Aria',
  title:             'The wandering scholar',
  description:       'A soft-spoken researcher with a knack for forgotten lore.',
  subjectivePronoun: 'she',
  objectivePronoun:  'her',
  possessivePronoun: 'her',
  isDefault:         true,   // clears the previous default
});

// Switch the live session over to it.
await api.personas.switchActive(persona.id);

const active = await api.personas.getActive();
console.log(active?.name); // → "Aria"
```

`create` takes only `name` as required; every other field is optional. `update(personaId, input)` is a **partial** update — only the fields you pass change; omitted fields are left untouched. Setting `isDefault: true` on either call clears whatever persona was previously default.

## Switching the active persona

```js
// Activate a persona:
await api.personas.switchActive(persona.id);

// Deactivate — no active persona:
await api.personas.switchActive(null);
```

`switchActive` returns `Promise<void>` and emits a **`SETTINGS_UPDATED`** event, so the frontend persona picker updates immediately without a manual refresh. Pass a persona UUID to activate, or `null` to leave no persona active.

`getActive()` reads the currently active persona (`null` if none); `getDefault()` reads the persona flagged `isDefault` (`null` if the user hasn't set one). They're distinct: the default seeds new chats, the active one drives the current session.

## Reading the attached world book — `getWorldBook`

A persona can carry a world book (`attachedWorldBookId`). `getWorldBook(personaId)` resolves it to a full `WorldInfo` in one call, saving you a separate `api.worldInfo.get`:

```js
const active = await api.personas.getActive();
if (active) {
  const book = await api.personas.getWorldBook(active.id);
  if (book) {
    console.log(`Persona "${active.name}" carries lorebook "${book.name}"`);
    // Enumerate its entries via the world-info surface:
    const { data: entries } = await api.worldInfo.entries.list(book.id);
    console.log(`${entries.length} lore entries`);
  }
}
```

Returns `null` when the persona has no book attached. Remember this needs only `personas` — you don't have to also request `world_books` just to *read* which book is attached. (To CRUD the book's entries, as above, you do need `world_books` for `api.worldInfo.*`.)

## Global add-ons

> **New in v2.0.** `api.personas.addons.*` exposes Lumiverse's **global add-ons** — named, sortable injectable content blocks that live alongside personas. Authoring and deletion stay in the host UI; scripts can **list, read, and update** existing add-ons.

An add-on is a reusable snippet of prompt content (a writing-style directive, a boilerplate instruction, a content warning) with a `label`, a `content` body, and a `sortOrder`. At generation time, each add-on is toggled on or off per request through the generation's `personaAddonStates` map (`ChatGenerationOptions.personaAddonStates`, a `Record<string, boolean>` keyed by add-on ID). Those states carry only add-on **IDs** — `api.personas.addons.*` is how you resolve an ID to its human-readable `label` and its `content`.

### `PersonaAddonInfo`

Returned by `addons.get` / `addons.update`, and as each element of `addons.list().data`:

| Field | Type | Meaning |
|---|---|---|
| `id` | `string` | Add-on UUID. This is the ID that appears in a generation's `personaAddonStates`. |
| `label` | `string` | Human-readable name shown in the host UI. |
| `content` | `string` | The injectable prompt content. |
| `sortOrder` | `number` | Ordering position in the host UI (mapped from the host's `sort_order`). |
| `metadata` | `Record<string, unknown>` | Arbitrary metadata. |
| `createdAt` / `updatedAt` | `number` | Unix-ms timestamps. |

### Methods

- **`addons.list(options?)`** — `list({ limit, offset })` → `{ data: PersonaAddonInfo[]; total: number }`. Paginated, same shape as `personas.list`.
- **`addons.get(addonId)`** — resolve one add-on by UUID. Returns `null` if not found.
- **`addons.update(addonId, input)`** — partial update. `input` is a `PersonaAddonUpdateInput`: any of `label` / `content` / `sortOrder` / `metadata`, all optional; only the fields you pass change. Returns the updated `PersonaAddonInfo`.

There is deliberately **no `create` or `delete`** here — add-ons are host-UI-owned. Scripts adjust and read the ones the user has already authored.

### Example: resolve the add-ons a request has enabled

```js
// List all add-ons, then map an on/off state map to labels.
const { data: addons } = await api.personas.addons.list();
const byId = new Map(addons.map(a => [a.id, a]));

// `states` here is a ChatGenerationOptions.personaAddonStates-shaped map
// (addonId → enabled boolean) that your script obtained elsewhere.
const enabled = Object.entries(states)
  .filter(([, on]) => on)
  .map(([id]) => byId.get(id)?.label ?? `(unknown add-on ${id})`);

console.log('Active add-ons:', enabled.join(', '));
```

### Example: patch an add-on's content

```js
const addon = await api.personas.addons.get(addonId);
if (addon) {
  await api.personas.addons.update(addon.id, {
    content: `${addon.content}\n\nAlways write in second person.`,
  });
}
```

## Error handling

The methods throw on:

- **Permission denied** — `Error: PERMISSION_DENIED:personas — grant this permission to use this API`, for anything in the namespace including `addons.*`.
- **Validation** — host-side validation of `create` / `update` payloads (e.g. missing `name`). The message includes the offending field.

Lookups don't throw on "not found": `get`, `getDefault`, `getActive`, `getWorldBook`, and `addons.get` return `null` instead. `delete` returns `false` if nothing was deleted.

```js
try {
  await api.personas.switchActive(personaId);
} catch (err) {
  const msg = err instanceof Error ? err.message : String(err);
  if (msg.includes('PERMISSION_DENIED')) {
    api.ui.toast('Persona switching requires the personas permission.', 'warning');
    return;
  }
  throw err;
}
```

## Version notes

`api.personas.*` (persona CRUD + `switchActive` + `getWorldBook`) has been implemented since v1.0.0-rc.1+. The nested **`api.personas.addons.*`** global-add-ons surface is **new in v2.0**, wrapping the host's `spindle.global_addons.*`. Persona and add-on fields track Lumiverse's own schema in camelCase — the host's snake_case DTOs (`sort_order`, `attached_world_book_id`, `is_default`, `image_id`) are mapped for you.
