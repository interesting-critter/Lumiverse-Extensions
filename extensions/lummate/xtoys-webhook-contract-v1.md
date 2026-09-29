# XToys Webhook Contract V1

This document defines the minimal XToys-side webhook contract that matches Lummate's current Phase 11 backend delivery behavior.

It is intentionally narrow:

- Lumiverse sends **one beat or control event at a time**
- XToys receives webhook payloads and reacts through trigger/action matching
- true multi-action device-layer blending is **not** assumed in v1

Lummate now uses one delivery mode for XToys private webhooks:

- `action_trigger_compat`

## XToys Endpoint

Lummate assumes the XToys private webhook pattern described in the XToys guide:

- base URL: `https://webhook.xtoys.app`
- final endpoint: `https://webhook.xtoys.app/<privateWebhookId>`
- method: `POST`
- body: JSON

The XToys guide indicates that webhook payloads should include an `action` field, which XToys can react to.

## Recommended Action Names

The default Lummate action mappings now assume XToys is organized around these semantic base names:

- `tease`
- `stroke`
- `thrust`
- `suction`
- `grind`
- `pulse`
- `lick`
- `squeeze`
- `pause`

The default control action names are:

- `stop`
- `hold`
- `resume`
- `panicstop`

Users can still override these names in Lummate settings if their XToys setup uses different identifiers.

For beat events, Lummate automatically appends one of these suffixes:

- `-low`
- `-medium`
- `-high`

So if the base action mapping is `thrust`, the webhook action sent to XToys will be:

- `thrust-low`
- `thrust-medium`
- `thrust-high`

## Delivery Model

This mode is designed to match the XToys Connect style used by existing community integrations while keeping Lumiverse out of direct slider control:

- a trigger field such as `action`
- trigger values that match XToys trigger/action blocks
- explicit intensity and ramp-time fields
- optional metadata fields for debugging or script branching

Recommended default field names:

- trigger field: `action`
- intensity field: `intensity`
- ramp seconds field: `seconds`

In this mode, beat payloads look more like:

```json
{
  "action": "thrust-high",
  "intensity": 79,
  "seconds": 0.24,
  "kind": "beat",
  "message_id": "msg_123",
  "beat_index": 0,
  "order_index": 0,
  "semantic_action_type": "thrust",
  "intensity_level": "high",
  "contact_zone": "genitals",
  "transition_style": "steady",
  "execution_profile": "pattern_funscript",
  "duration_ms": 12572,
  "tempo": 72
}
```

Recommended v1 meaning:

- `action` = XToys trigger/action name with quantized level suffix
- `intensity` = target toy intensity `0-100`
- `seconds` = ramp time in seconds to reach the target intensity

Control events use the same trigger field, for example:

```json
{
  "action": "stop",
  "kind": "control",
  "control": "stop",
  "message_id": "msg_123"
}
```

This mode exists because working XToys community setups often use named trigger blocks over the private webhook path rather than a direct JSON action-dispatch script.

## Recommended XToys Build Style

The simplest v1 XToys setup is:

1. Create one Private Webhook block.
2. Treat the incoming `action` value as the primary selector.
3. Create one XToys trigger/action for each Lummate action string you want to support.
4. Attach each trigger/action to a prebuilt pattern or toy behavior.

In practice, the contract is:

`Lummate action string -> XToys trigger/action -> XToys pattern or toy behavior`

For v1, this is more reliable than requiring XToys-side logic to derive levels from a base semantic action.

## Recommended Trigger Table

Suggested beat trigger/action names:

- `tease-low`
- `tease-medium`
- `tease-high`
- `stroke-low`
- `stroke-medium`
- `stroke-high`
- `thrust-low`
- `thrust-medium`
- `thrust-high`
- `suction-low`
- `suction-medium`
- `suction-high`
- `grind-low`
- `grind-medium`
- `grind-high`
- `pulse-low`
- `pulse-medium`
- `pulse-high`
- `lick-low`
- `lick-medium`
- `lick-high`
- `squeeze-low`
- `squeeze-medium`
- `squeeze-high`
- `pause-low`
- `pause-medium`
- `pause-high`

Suggested control trigger/action names:

- `stop`
- `hold`
- `resume`
- `panicstop`

## Beat Payload

When a beat starts, Lummate sends a payload like:

```json
{
  "action": "thrust-high",
  "kind": "beat",
  "semantic_action_type": "thrust",
  "intensity": 79,
  "intensity_level": "high",
  "tempo": 72,
  "duration_ms": 12572,
  "seconds": 0.24,
  "transition_style": "steady",
  "execution_profile": "pattern_funscript",
  "message_id": "msg_123",
  "beat_index": 0,
  "order_index": 0,
  "contact_zone": "genitals"
}
```

### Field Meanings

- `action`
  The XToys trigger/action name to react to, including the `low`, `medium`, or `high` suffix.
- `kind`
  Always `beat` for beat-start events.
- `semantic_action_type`
  The original semantic action label before any script-side interpretation.
- `intensity`
  Final resolved strength, clamped `0-100`.
- `intensity_level`
  Quantized intensity band chosen by Lummate: `low`, `medium`, or `high`.
- `tempo`
  Final resolved tempo, clamped `0-100`.
- `duration_ms`
  Beat duration in milliseconds.
- `seconds`
  Suggested XToys ramp time in seconds.
- `transition_style`
  Narrative transition hint such as `steady`, `ramp`, `snap`, or `fade`.
- `execution_profile`
  Suggested XToys-side mechanical style such as:
  - `intensity_direct`
  - `pattern_scripted`
  - `pattern_funscript`
  - `composite_blend`
- `message_id`
  The Lumiverse message this beat came from.
- `beat_index`
  Zero-based index in the current ordered beat sequence.
- `order_index`
  Original beat order.
- `contact_zone`
  Tracked contact zone such as `genitals`, `anus`, `mouth`, or `custom`.

## Control Payloads

Lummate also emits control events.

### Stop

```json
{
  "action": "stop",
  "kind": "control",
  "control": "stop",
  "reason": "manual_stop",
  "message_id": "msg_123"
}
```

### Hold

```json
{
  "action": "hold",
  "kind": "control",
  "control": "hold",
  "message_id": "msg_123",
  "contact_zone": "genitals",
  "held_action_type": "suction",
  "amplitude": 72,
  "tempo": 69
}
```

### Resume

```json
{
  "action": "resume",
  "kind": "control",
  "control": "resume",
  "message_id": "msg_123",
  "contact_zone": "genitals",
  "resumed_action_type": "stroke",
  "amplitude": 55,
  "tempo": 48
}
```

### Panic Stop

```json
{
  "action": "panicstop",
  "kind": "control",
  "control": "panic_stop",
  "reason": "panic_stop",
  "message_id": "msg_123"
}
```

## XToys Runtime Responsibilities

For v1, the XToys side should:

1. Receive webhook payloads.
2. Switch on the `action` field.
3. Treat beat actions as the primary execution path.
4. Treat the full action string such as `thrust-high` as the main selector.
5. Optionally use `intensity` and `seconds` for local expressions.
6. Optionally use metadata such as `tempo`, `duration_ms`, or `execution_profile` for debugging or advanced branching.
7. Handle `stop`, `hold`, `resume`, and `panicstop` as explicit control paths.

## Minimal V1 Behavior

The simplest viable XToys behavior is:

### Beat actions

- `tease-low`, `tease-medium`, `tease-high`
- `stroke-low`, `stroke-medium`, `stroke-high`
- `thrust-low`, `thrust-medium`, `thrust-high`
- `suction-low`, `suction-medium`, `suction-high`
- `grind-low`, `grind-medium`, `grind-high`
- `pulse-low`, `pulse-medium`, `pulse-high`
- `lick-low`, `lick-medium`, `lick-high`
- `squeeze-low`, `squeeze-medium`, `squeeze-high`

Each one should directly trigger the corresponding XToys-side pattern or toy behavior.

### Control actions

- `stop`
  - stop current local behavior and ramp down safely
- `hold`
  - keep the current toy behavior active without advancing
- `resume`
  - restore the locally remembered prior held behavior if the script tracks one
- `panicstop`
  - immediately stop all active toy behavior

## Suggested Execution-Profile Interpretation

These are recommendations only:

- `intensity_direct`
  - best for vibrators and generic intensity toys
- `pattern_scripted`
  - best for XToys scripted pattern control
- `pattern_funscript`
  - best for strokers or thrust-position devices
- `composite_blend`
  - treat as a single resultant pattern, not true parallel playback

## What V1 Does Not Promise

This contract does **not** assume:

- true simultaneous semantic action layering on one toy
- per-character multi-toy routing
- full sequence upload to XToys
- native XToys understanding of semantic action names without a script

Those remain later-phase or deferred features.
