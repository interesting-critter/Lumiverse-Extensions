<!-- Mirror of upstream documentation. Everything below the metadata table is the original file, unmodified. -->

| Field | Value |
| --- | --- |
| Extension | LumiState |
| Source repository | https://github.com/Archkr/Lumiverse-LumiState |
| Original link (from Lumiverse-Extensions README) | https://github.com/Archkr/Lumiverse-LumiState |
| Upstream path | `README.md` |
| Retrieved | 2026-09-29 @ `main` (`3dc4ebf`) |

---

<div align="center">

# LumiState

**Shared scene state for Lumiverse extensions.**

[![Version](https://img.shields.io/badge/version-0.1.0-65dac8)](./spindle.json)
[![Lumiverse](https://img.shields.io/badge/Lumiverse-%E2%89%A5%201.0.6-d4a35a)](https://github.com/prolix-oc/Lumiverse)
[![Status](https://img.shields.io/badge/status-beta-e6a45a)](https://github.com/Archkr/Lumiverse-LumiState)
[![License](https://img.shields.io/badge/license-Lumiverse%20Community%202.0-6f9f78)](./LICENSE.md)

*One scene contract. Many independent extensions.*

</div>

LumiState is the scene-state interoperability layer for Lumiverse.

It reads the public, spoiler-safe state published by LumiWeather, LumiWorld, and LumiMind; validates that every snapshot belongs to the active chat; combines their claims without erasing source ownership; and republishes one stable endpoint for other extensions:

```text
lumi_state.scene.current
```

A downstream extension can integrate with LumiState instead of separately coupling itself to every scene producer.

> **The important distinction:** LumiState coordinates scene state; it does not own it. LumiWeather still owns visible weather and location, LumiWorld owns its simulation clock, and LumiMind owns public cast presence. LumiState preserves those boundaries.

---

## Table of contents

1. [At a glance](#at-a-glance)
2. [Why LumiState](#why-lumistate)
3. [How it works](#how-it-works)
4. [Source ownership](#source-ownership)
5. [Compatibility](#compatibility)
6. [Installation](#installation)
7. [Quick start](#quick-start)
8. [Scene Inspector](#scene-inspector)
9. [Public interface](#public-interface)
10. [Consuming LumiState](#consuming-lumistate)
11. [Source health and conflicts](#source-health-and-conflicts)
12. [Privacy](#privacy)
13. [Reliability model](#reliability-model)
14. [Troubleshooting](#troubleshooting)
15. [Development](#development)
16. [License](#license)

---

## At a glance

| | |
|---|---|
| **Unified scene read** | Downstream extensions read one aggregate instead of integrating with every publisher. |
| **Active-chat safety** | Snapshots for another chat are rejected rather than leaking across conversations. |
| **Source ownership** | Original provenance stays attached to every claim. LumiState never pretends a source fact is its own. |
| **Partial availability** | Missing or disabled publishers are normal. Available scene state continues flowing. |
| **Conflict visibility** | Disagreements are reported and preserved instead of silently resolved. |
| **Revision discipline** | Source revisions are tracked per endpoint and aggregate revisions are stable per chat. |
| **Spoiler-safe contract** | Only deliberately public state enters the aggregate. Private minds and world simulation details stay private. |
| **No model calls** | Reads, validation, aggregation, diagnostics, and publication are deterministic and in-memory. |
| **Scene Inspector** | A drawer surface shows the scene, publisher health, provenance, freshness, conflicts, and sanitized diagnostics. |

---

## Why LumiState

Scene-aware extensions need the same basic context: where the visible scene is, what time it is, what conditions are active, and who is present.

Without a shared layer, every consumer must understand every producer:

```text
Consumer A ── reads LumiWeather, LumiWorld, and LumiMind
Consumer B ── reads LumiWeather, LumiWorld, and LumiMind
Consumer C ── reads LumiWeather, LumiWorld, and LumiMind
```

That creates duplicated validation, incompatible assumptions, direct extension-to-extension dependencies, and more opportunities to use state from the wrong chat.

LumiState turns that integration web into a hub:

```text
LumiWeather ─┐
LumiWorld ───┼──▶ LumiState ──▶ lumi_state.scene.current ──▶ downstream extensions
LumiMind ────┘          │
                        └──▶ Scene Inspector
```

Publishers only describe the public scene facts they own. Consumers only need to understand LumiState v1. The extensions remain independently useful and can be installed, updated, disabled, or missing without turning the rest of the ecosystem into a hard dependency chain.

### What LumiState is not

LumiState is not:

- a scene editor;
- an AI controller;
- a replacement for LumiWeather, LumiWorld, or LumiMind;
- a database of private world or character state;
- an authority that chooses which publisher is “correct.”

Its product is a validated shared contract.

---

## How it works

```text
Active chat changes or a source updates
                  │
                  ▼
Read public source endpoints through the Shared RPC Pool
                  │
                  ▼
Validate protocol, schema, source identity, visibility,
chat ownership, freshness, and source-local revision
                  │
                  ▼
Merge six claim families while preserving provenance
                  │
                  ├── locations
                  ├── times
                  ├── cast
                  ├── objects
                  ├── conditions
                  └── threads
                  │
                  ▼
Detect overlapping claims that disagree
                  │
                  ▼
Publish lumi_state.scene.current
```

1. **LumiState follows the active chat.** The `chats` permission provides chat routing; the frontend also sends the current chat during activation and refresh.
2. **Each known publisher is read independently.** A missing endpoint becomes a source-health result, not a fatal error.
3. **Every payload is validated.** Unsupported schemas, unexpected endpoint identities, private snapshots, malformed claims, and wrong-chat data are excluded.
4. **Source revisions remain source-local.** LumiState never compares LumiWeather revision `12` with LumiMind revision `12`; those numbers belong to different streams.
5. **Accepted claims are merged without rewriting them.** Original source, method, observation time, confidence, and upstream dependencies remain attached.
6. **Conflicts are explicit.** When multiple publishers disagree about the same subject and claim kind, every claim remains visible and the aggregate gains conflict metadata.
7. **The aggregate is republished synchronously.** Other extensions can read the latest accepted scene without triggering generation or waiting for an analysis job.

LumiState refreshes on relevant chat and extension lifecycle events, after generation, and through a lightweight five-second safety poll for asynchronous publisher changes.

---

## Source ownership

The initial ecosystem has three publishers:

| Publisher | Minimum compatible version | Public contribution | Deliberately excluded |
|---|---:|---|---|
| **LumiWeather** | `1.3.1` | Visible location, calendar date and time, weather conditions, manual/story-sync provenance | Prompt content and unrelated extension settings |
| **LumiWorld** | `0.3.2` | Simulation day, hour, and running state | Schedule, private location, mood, activity, thought, goal, and history |
| **LumiMind** | `0.1.1` | Actor identities, aliases, presence, confirmation state, and public stance | Beliefs, secrets, goals, evidence, relationships, and full private minds |

The v1 schema also supports object and narrative-thread claims. The current aggregate keeps those collections available for compatible publishers without inventing data when no source supplies them.

Each source remains useful without LumiState. Publishing is a public interoperability capability, not a dependency on the aggregator.

---

## Compatibility

| Requirement | Value |
|---|---|
| Lumiverse | `1.0.6` or newer |
| LumiState | `0.1.0` beta |
| Permission | `chats` |
| Controller connection | Not used |
| Generation calls | None |
| Required source extensions | None; all publishers are optional |
| Public protocol | `lumi_state.v1` with `schemaVersion: 1` |
| Build output | Committed `dist/backend.js` and `dist/frontend.js` |

LumiState degrades by source. Installing only one compatible publisher still produces a valid partial scene. Installing none produces an available aggregate with empty claim collections and source-health metadata.

---

## Installation

### Install from GitHub

```text
1. Copy:    https://github.com/Archkr/Lumiverse-LumiState
2. Open:    Lumiverse → Extensions → Install
3. Paste:   the URL into the repository field
4. Press:   Install
5. Enable:  LumiState and grant the chats permission
6. Verify:  Scene Inspector appears in the drawer and command palette
```

No local build is required for a normal installation because release-ready `dist/` bundles are committed.

### Install scene publishers

LumiState can only aggregate data that another extension publishes. Install or update whichever sources you want:

- LumiWeather for visible location, calendar time, and weather;
- LumiWorld for the simulation clock;
- LumiMind for public cast identity and presence.

Missing sources are shown as missing and do not prevent other publishers from working.

---

## Quick start

| Step | Action | Result |
|---|---|---|
| 1 | Install at least one compatible scene publisher | LumiState has a public source to read. |
| 2 | Install and enable LumiState | The aggregate RPC endpoints are registered. |
| 3 | Grant `chats` | LumiState can validate state against the active conversation. |
| 4 | Open a chat with source state | LumiState assembles that chat’s shared scene. |
| 5 | Open **Scene Inspector** | You can review publisher health, claims, provenance, and conflicts. |
| 6 | Use or build a compatible downstream extension | It reads `lumi_state.scene.current`. |

There is no prompt macro, controller connection, activation toggle, or per-character setup. LumiState begins coordinating public state when it loads.

---

## Scene Inspector

Scene Inspector is the observability surface for the shared contract. It is useful for users, extension authors, and bug reports, but downstream extensions do not need the drawer to be open.

It shows:

- the combined active-chat scene;
- connected, stale, unavailable, missing, invalid, or mismatched publishers;
- source-local revisions and update age;
- a source label on every claim;
- cast presence and public stance;
- overlapping claims and their participating publishers;
- the aggregate revision and freshness;
- a privacy-safe diagnostics copy action.

On the Lumiverse home screen, Scene Inspector intentionally waits. It does not substitute state from the last conversation that happened to be open.

---

## Public interface

LumiState registers two public synchronous endpoints through Lumiverse’s Shared RPC Pool:

| Endpoint | Purpose |
|---|---|
| `lumi_state.contract.v1` | Protocol version, capabilities, source endpoints, and aggregate endpoint metadata. |
| `lumi_state.scene.current` | Latest validated aggregate for the active chat. |

The aggregate uses this envelope:

```json
{
  "protocol": "lumi_state.v1",
  "schemaVersion": 1,
  "source": {
    "extensionId": "lumi_state",
    "extensionVersion": "0.1.0",
    "endpoint": "lumi_state.scene.current"
  },
  "chatId": "active-chat-id",
  "revision": 42,
  "freshness": "fresh",
  "generatedAt": 1784246400000,
  "updatedAt": 1784246395000,
  "visibility": "public",
  "sources": [],
  "state": {
    "locations": [],
    "times": [],
    "cast": [],
    "objects": [],
    "conditions": [],
    "threads": []
  },
  "conflicts": []
}
```

The complete normative contract lives in [`docs/lumi-state-v1.md`](./docs/lumi-state-v1.md). A machine-readable JSON Schema is available at [`schema/lumi-state-v1.schema.json`](./schema/lumi-state-v1.schema.json), and the TypeScript definitions and parser live in [`src/protocol.ts`](./src/protocol.ts).

### Provenance

Every claim carries its original provenance:

| Field | Meaning |
|---|---|
| `extensionId` | Publisher that owns the claim. |
| `method` | How the source produced it, such as `story`, `manual`, `simulation`, or `derived`. |
| `observedAt` | When the source observed or produced the fact. |
| `confidence` | Optional normalized confidence from `0` to `1`. |
| `derivedFrom` | Optional upstream endpoint/chat/revision dependencies. |

Aggregation never replaces source provenance with `lumi_state`.

---

## Consuming LumiState

A downstream Spindle extension reads the aggregate directly:

```ts
async function readSharedScene(activeChatId: string) {
  try {
    const snapshot = await spindle.rpcPool.read("lumi_state.scene.current") as any;

    if (snapshot?.protocol !== "lumi_state.v1") return null;
    if (snapshot?.schemaVersion !== 1) return null;
    if (snapshot?.source?.extensionId !== "lumi_state") return null;
    if (snapshot?.source?.endpoint !== "lumi_state.scene.current") return null;
    if (snapshot?.chatId !== activeChatId) return null;

    return snapshot;
  } catch {
    // LumiState is optional. Continue without shared scene context.
    return null;
  }
}
```

Production consumers should use the JSON Schema or an equivalent defensive parser rather than trusting `any`.

### Consumer rules

A correct consumer must:

1. catch missing endpoint reads and continue normally;
2. require `protocol === "lumi_state.v1"` and `schemaVersion === 1`;
3. require the expected source extension and endpoint;
4. reject a snapshot whose `chatId` does not match its active chat;
5. track revisions per endpoint and chat;
6. ignore revisions older than the newest one already accepted for that stream;
7. preserve claim provenance;
8. handle partial scenes and empty claim collections;
9. inspect conflict metadata instead of assuming the first claim is authoritative.

Source revisions, aggregate revisions, and revisions from different chats are separate streams. Never compare them as if they were global timestamps.

---

## Source health and conflicts

### Source status

Each configured publisher receives one status:

| Status | Meaning |
|---|---|
| `connected` | A valid current snapshot for the active chat was accepted. |
| `stale` | Valid fallback state is available, or an older/unchanged revision was retained. |
| `unavailable` | The publisher is installed but has no public state for this chat. |
| `missing` | The endpoint is not registered or could not be read. |
| `invalid` | The endpoint returned an unsupported, malformed, private, or incorrectly identified snapshot. |
| `chat_mismatch` | The snapshot belongs to another active chat and was excluded. |

`stale` is a publisher/validation state, not merely an age calculation. Consumers may apply their own age policy using `updatedAt` and claim provenance timestamps.

### Conflict behavior

LumiState detects disagreements in overlapping location, time, condition, and cast claims. A conflict records:

- the claim kind and subject;
- participating source extensions;
- affected claim IDs;
- a stable conflict ID and human-readable message.

LumiState does not choose a winner. The consumer decides whether to show every claim, prefer a domain-specific authority, pause an action, or ask the user to reconcile the sources.

---

## Privacy

LumiState reads only public, spoiler-safe snapshots registered with `requires: []`.

It does **not** read or republish:

- LumiMind beliefs, secrets, goals, evidence, relationships, or full private minds;
- LumiWorld schedules, private locations, moods, activities, thoughts, goals, or history;
- chat messages or assembled prompts;
- credentials, API keys, controller inputs, or controller responses.

LumiState keeps accepted snapshots in memory for validation and stores only the per-chat aggregate revision/signature metadata needed to keep aggregate revisions stable across reloads. That metadata contains chat and source stream identifiers, statuses, freshness, and revisions—not shared claim values or private source state.

The copied diagnostics report excludes claim values, actor names, aliases, public stance text, full chat IDs, and full entity IDs.

---

## Reliability model

- Endpoint reads are in-memory and never trigger generation.
- A malformed source cannot invalidate claims from healthy sources.
- Wrong-chat snapshots are excluded from the aggregate.
- Unknown protocol or schema versions are rejected.
- Source identity and endpoint identity must match the configured publisher.
- During a running session, regressed source revisions retain the newest accepted snapshot.
- A source that changes claims without increasing its revision is treated as stale during that session.
- Aggregate revisions change when the accepted source stream signature changes.
- Publisher unloads and reloads trigger refreshes.
- Relevant chat mutations and generation completion schedule refreshes.
- A five-second poll catches asynchronous LumiMind analysis and LumiWorld clock changes.
- Closing or switching chats never reuses the last chat’s aggregate as a substitute.

The protocol assumes every publisher increments its own revision whenever its public state changes. That is the contract that lets consumers distinguish a new snapshot from a repeat read.

---

## Troubleshooting

<details>
<summary><b>Every publisher says Missing</b></summary>

Confirm that at least one compatible source extension is installed, enabled, and up to date. LumiState currently reads these exact endpoints:

```text
lumi_weather.state.current
agent_world.state.current
lumi_mind.state.current
```

Reload the source extension and LumiState after updating an older install.

</details>

<details>
<summary><b>A publisher is connected, but the scene has no claims</b></summary>

The publisher may not have produced public state for this chat yet.

- LumiWeather needs a story weather tag or manual scene lock.
- LumiWorld needs an initialized World Agent clock.
- LumiMind needs an activated timeline with resolved public cast state.

Open the source extension’s own interface to confirm that it has current state.

</details>

<details>
<summary><b>A source says Chat mismatch</b></summary>

LumiState received a valid snapshot for a different conversation and excluded it. Switch back to the intended chat, refresh the source extension, then use **Refresh** in Scene Inspector.

If the mismatch persists, copy diagnostics and report which source remains mismatched.

</details>

<details>
<summary><b>The aggregate or a source says Stale</b></summary>

Open the source extension first. It may be intentionally serving a valid fallback while newer work is pending or failed.

If Scene Inspector reports a revision regression or a change without a revision increase, update or reload that publisher. LumiState will not silently accept a broken revision stream during the current session.

</details>

<details>
<summary><b>A downstream extension sees the wrong conversation</b></summary>

The consumer must compare `snapshot.chatId` with its own active chat on every read. It must also key cached revisions by both endpoint and chat. Never carry the latest accepted scene across a chat switch.

</details>

<details>
<summary><b>Scene Inspector says the chats permission is unavailable</b></summary>

Open **Lumiverse → Settings → Extensions**, select LumiState, and grant `chats`. Without it, LumiState cannot reliably route the aggregate to the active conversation.

</details>

<details>
<summary><b>I need to report a bug</b></summary>

Open **Scene Inspector**, choose **Diagnostics**, and copy the privacy-safe report. Include the LumiState version, affected source status, and the steps that produced the problem.

The report intentionally omits scene claim values and private extension state.

</details>

---

## Development

### Source layout

```text
src/
  backend.ts        active-chat routing, source reads, refresh queues, RPC publication
  frontend.ts       Scene Inspector and privacy-safe diagnostics
  messages.ts       frontend/backend message contracts
  protocol.ts       v1 types, parser, revision guards, merge, and conflict detection
  protocol.test.ts  validation, aggregation, conflict, and revision tests
  styles.ts         Scene Inspector visual system

docs/
  lumi-state-v1.md  normative protocol documentation

schema/
  lumi-state-v1.schema.json  machine-readable snapshot schema

scripts/
  check-dist.ts     verifies committed bundles match the source build

dist/
  backend.js        bundled backend entrypoint
  frontend.js       bundled frontend entrypoint
```

### Commands

```bash
bun install            # install development dependencies
bun run typecheck      # strict TypeScript validation
bun test               # run the protocol and aggregation suite
bun run build          # rebuild backend and frontend bundles
bun run build:backend  # backend bundle only
bun run build:frontend # frontend bundle only
bun run check:dist     # confirm committed dist matches source
```

Built `dist/` files are committed so Lumiverse can install and run LumiState directly from the repository.

When changing the wire format, update the TypeScript definitions, parser, tests, normative protocol document, and JSON Schema together. A breaking contract change requires a new schema version rather than silently changing v1.

---

## License

LumiState is provided under the **Lumiverse Community License 2.0**. See [`LICENSE.md`](./LICENSE.md) for the complete terms.

LumiState is an independent community extension compatible with Lumiverse. It is not the official Lumiverse provider.
