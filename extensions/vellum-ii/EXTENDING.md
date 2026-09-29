# Extending VELLUM Engine

VELLUM II is built to grow. Adding a feature touches **three small, well-defined seams** — never the core, the reducer's plumbing, or the lifecycle orchestration.

## The architecture in one breath

```
raw turn ──parse──▶ ParsedState ──extract(Feature)──▶ VellumEvent[] ──append──▶ log
                                                                                  │
UI ◀──broadcast── ChronicleState ◀───────────────reduce(events)──────────────────┘
```

- **`ParsedState`** (`src/parse/parsed.ts`) is the format-agnostic seam. JSON block, regex fallback, imports — all produce it.
- **`VellumEvent`** (`src/core/events.ts`) is the immutable source of truth.
- **`reduce()`** (`src/core/reduce.ts`) is the *only* place state is derived. Pure, exhaustive, tested.
- **`Feature`** (`src/bus/registry.ts`) is how you plug in.

## To add a feature (e.g. "inventory")

**1. Add an event kind** — `src/core/events.ts`:
```ts
export const EvItem = z.object({ ...base, kind: z.literal('item.op'), op: z.enum(['gain','lose']), who: z.string(), item: z.string() });
// add EvItem to the VellumEvent discriminated union
```

**2. Handle it in the reducer** — `src/core/reduce.ts`: add one `case 'item.op':` to the switch. The exhaustiveness guard (`const _never: never = e`) will fail the build until you do — so you *can't* forget.

**3. Extend derived state** — `src/domain/types.ts`: add `inventory: Record<string, string[]>` to `ChronicleState` + `freshState()`.

**4. Register a Feature** — a new `src/domain/inventory-feature.ts`:
```ts
export const inventoryFeature: Feature = {
  id: 'inventory',
  extract(parsed, ctx) {
    const items = (parsed.ext?.items ?? []) as any[];
    return items.map((it) => ({ seq: ctx.seq(), turn: ctx.turn, day: ctx.day, src: 'model', kind: 'item.op', op: it.op, who: canonId(it.who), item: it.item }));
  },
  inject(ctx) { /* optional: contribute a prompt block (Phase 2+) */ return null; },
};
```
Then `registerFeature(inventoryFeature)` in `src/backend.ts`. That's it for the engine — the parser already carries arbitrary `ext` data, the lifecycle already runs every feature's extractor, and the store/broadcast already ship the new state.

**5. (Optional) UI** — add an isolated component (`src/ui/component.ts`) and a tab. Because every panel has an error boundary, a new panel can never break existing ones.

**6. Test it** — drop a `test/inventory.test.ts` asserting `reduce(foldTurn(content).events)` yields the inventory you expect. Golden-fixture style, like `test/parse-fold.test.ts`.

## Guarantees that keep growth safe

- **Exhaustive reducer switch** — a new event kind won't compile until handled.
- **zod-validated log** — a malformed/old log is caught at load and migrated (`src/core/migrate.ts`) or degrades to fresh, never crashes a reducer.
- **Feature isolation** — `runExtractors` wraps each feature in try/catch; one bad feature can't break the fold for the rest.
- **UI error boundaries** — one panel's render throw degrades to a placeholder.
- **Pure core** — `parse`, `reduce`, `extract`, `fold` are pure and unit-tested; I/O lives only in `host/`, `store/`, and `backend.ts`.
- **One identity layer** — never resolve `userId` ad-hoc; always `host/user.ts`.
- **Continuity guardrail** — structured facts are injected verbatim, never similarity-retrieved. A retrieval feature must respect this.

## What NOT to do

- Don't mutate `ChronicleState` outside the reducer.
- Don't read/write storage outside `store/`.
- Don't `JSON.parse` model output outside `parse/` (use/extend the parser).
- Don't add a `catch(e){}` that swallows silently — return a `Result` or log with context.
