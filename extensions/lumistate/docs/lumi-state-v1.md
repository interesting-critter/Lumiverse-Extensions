# LumiState Protocol v1

LumiState v1 is a small wire protocol for sharing the latest active-chat scene state through Lumiverse's Shared RPC Pool.

## Endpoint model

Publishers register a stable synchronous endpoint:

```text
<extension-id>.state.current
```

The active chat ID belongs in the payload, never in the endpoint name. A publisher also exposes `<extension-id>.contract.v1` with its protocol version, capabilities, and endpoint metadata.

The aggregate endpoint is:

```text
lumi_state.scene.current
```

## Required envelope

Every snapshot contains:

- `protocol: "lumi_state.v1"`
- `schemaVersion: 1`
- Source extension, version, and endpoint
- `chatId`
- Monotonic source-local `revision`
- `freshness`
- Publication and data-update timestamps
- Visibility
- Six claim collections: locations, times, cast, objects, conditions, and threads

Revisions are comparable only within the same endpoint and chat. They are not global timestamps and must never be compared across publishers.

## Claim scope

Locations, time, and conditions identify their subject:

- `{ "kind": "scene" }` describes the visible scene.
- `{ "kind": "actor", "actor": ... }` describes a particular actor.

This prevents a visible scene location from being confused with a private or offscreen actor location.

## Identity

Entity references use a namespace plus an ID. Examples:

```text
host.character
host.persona
lumi_mind.actor
```

Cast claims may include `links` to connect a timeline-local actor with a stable host character or persona ID.

## Provenance

Every claim records:

- Source extension ID
- Production method, such as `story`, `manual`, `simulation`, or `derived`
- Observation timestamp
- Optional confidence
- Optional upstream endpoint revisions in `derivedFrom`

An aggregator preserves original provenance. It does not replace it with its own identity.

## Freshness

- `fresh`: latest valid state known to the publisher
- `stale`: valid fallback state while newer work is pending or failed
- `unavailable`: the publisher has no state for the active chat

Age alone does not make a state stale. Consumers may apply their own age limits using `updatedAt` and provenance timestamps.

## Privacy

The standard `.state.current` endpoint is public extension interoperability and uses `{ requires: [] }`. Publishers must map to a deliberately safe shape rather than exposing their stored internal state directly.

Private state is outside the core v1 aggregate. A future private endpoint must be explicitly enabled by the user, gated by an appropriate permission, and documented separately.

## Consumer rules

A consumer must:

1. Catch missing endpoint reads and continue normally.
2. Require `protocol === "lumi_state.v1"` and `schemaVersion === 1`.
3. Require the expected source extension and endpoint.
4. Reject a mismatched `chatId`.
5. Track revisions per endpoint and chat.
6. Preserve provenance when combining claims.
7. Report conflicts instead of silently rewriting source state.

## Ownership in the initial ecosystem

- LumiWeather owns visible location, calendar time, and weather.
- LumiWorld owns its simulation clock. Its private schedule and mental state are not public claims.
- LumiMind owns cast identity, aliases, presence, and public stance.
- LumiState validates and aggregates. It owns none of the source facts.
