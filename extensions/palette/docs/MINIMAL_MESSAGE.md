# MinimalMessage anatomy audit

Theme Studio v18 stops pretending `MinimalMessage` is a smaller `BubbleMessage`.
The shipped native catalog reports a different component-local structure.

## Shared concepts we can adapt

- Frame: `[data-component="MinimalMessage"]`
- Bubble: `[class*="_bubble_"]`
- Header: `[class*="_header_"]`
- Avatar wrapper: `[class*="_avatar_"]`
- Portrait image: expected `... [class*="_avatar_"] img` (confirm on a mounted message)
- Character name: `_nameChar_` / `_name_`
- Metadata: `_metaPill_` plus `_metaSegment_` / `_metaDot_`
- Actions: `_actionsWrap_`
- Message content: expected shared `[data-component="MessageContent"]` nesting (confirm mounted DOM)

## Important differences from BubbleMessage

MinimalMessage does **not** advertise BubbleMessage's `headerLeft`, `metaWrap`,
`avatarBgImg`, or `actionsPill` classes. Its actions use `actionsWrap`, and its
metadata appears flatter.

That means composition recipes should describe intent (portrait, identity,
metadata, prose, actions), then use a BubbleMessage or MinimalMessage adapter.
We should not run the BubbleMessage hero choreography against MinimalMessage and
patch the wreckage afterward.

V18 exposes this anatomy to the Style Library but intentionally marks the
existing message recipes as BubbleMessage-only. The first MinimalMessage visual
adapters belong in the next content pack, after confirming the medium-confidence
portrait/content candidates against a mounted MinimalMessage.

## V19: first real adapter

Manga is the first pack to consume the MinimalMessage map instead of merely documenting it.

Its `Margin speaker` composition targets MinimalMessage's own frame, header, avatar, name, meta, content, and `actionsWrap`. Shared pack pieces such as Chapter headings and Printed code target `MessageContent` independently of which message renderer owns it.

This is the adapter rule going forward: a visual family may share intent and palette, but BubbleMessage and MinimalMessage receive different structural choreography whenever their anatomy differs.
