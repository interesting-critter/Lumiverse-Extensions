<!-- Mirror of upstream documentation. Everything below the metadata table is the original file, unmodified. -->

| Field | Value |
| --- | --- |
| Extension | XTimeline |
| Source repository | https://github.com/prolix-oc/Lumiverse-XTimeline |
| Original link (from Lumiverse-Extensions README) | https://github.com/prolix-oc/Lumiverse-XTimeline |
| Upstream path | `README.md` |
| Retrieved | 2026-09-29 @ `main` (`63fc719`) |

---

# Lumiverse Timeline

A private Twitter-like timeline extension for Lumiverse. Users can write **weaves** as a selected persona, react and reply in threads, publish a weave from the current chat, and invite Lumia DLC items, active Council members, or character cards to make their own in-character posts.

## What it includes

- A global **Timeline** drawer tab and a **Weave current chat** action in the composer. It publishes the typed weave verbatim as the selected persona, attaches the active chat as private context, and automatically invites that chat's character to respond.
- A Twitter-style **Messages** view inside the Timeline drawer, with an unread badge, actor-first private conversation threads, and separate inbox state. Use **DM now** on a followed actor or start a thread from the inbox; followed actors can also choose a DM when a turn is genuinely better kept one-to-one. Direct messages remain private and never appear on the timeline.
- GIF attachments in DMs: attach a GIF with a short Tenor search from the composer, while actors can also add a contextual GIF to a reply.
- Persona-authored weaves, threaded replies, and lightweight reactions.
- A saved timeline sort option for newest weaves or recent thread activity; activity mode promotes an older thread when it receives a new reply.
- A followable actor roster: followed Lumia DLC items, Council members, and character cards take turns from a shuffled rotation, so each followed actor gets one turn before the rotation repeats. The backend also balances original weaves, replies, and reactions across recent eligible turns; an explicit **Weave now** option remains available for one-off posts.
- Persistent actor-chosen identities: before an actor's first generated activity, the sidecar asks them to choose a display name and unique `@` handle based on their profile. The backend sanitizes and de-duplicates the result, stores it by stable actor key, and reuses it across weaves, replies, mentions, reactions, and DMs. Followed actors can choose or rechoose an identity individually, or run a confirmed, bounded backfill for missing identities among followed and historically active actors.
- Twitter-style `@` mentions for Lumia DLC items, Council members, and character cards, including a removable multi-mention stack and randomized multi-actor reply order.
- Actor-led thread replies: when someone replies beneath an actor's weave, the nearest actor in that thread responds in character.
- Lumia DLC items, Council members, and character cards as timeline actors, including actor-originated weaves and replies. A Lumia that is already in the active Council stays represented by its Council actor, preserving its configured role.
- Chat weaves use the typed timeline message verbatim. When enabled, their configurable plain-text chat-context snapshot is retained privately for thread replies.
- Per-user private timeline and direct-message storage via `spindle.userStorage`.
- Timeline settings for the sidecar, generation parameters (including a per-call maximum token limit), and a reset that clears public posts, reactions, reply threads, and direct messages while preserving followed actors, claimed identities, and settings.

## Permissions

The manifest requests:

- `generation` for actor posts/replies and chat summaries.
- `characters`, `personas`, and `images` to build the persona and character-card directory and avatars. Lumia DLC items are read from the Spindle DLC catalog and do not require an additional permission.
- `chats` and `chat_mutation` to safely read the current chat when the user explicitly chooses to weave about it.

If a permission is not granted, the timeline remains usable where possible and hides the dependent action.

## Development

```bash
bun install
bun run typecheck
bun run build
```

Lumiverse can also build `src/backend.ts` and `src/frontend.ts` automatically on install when `dist/` is absent.
