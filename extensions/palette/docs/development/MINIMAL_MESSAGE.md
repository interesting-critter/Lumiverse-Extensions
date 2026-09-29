# MinimalMessage anatomy audit

Palette treats `MinimalMessage` as an alternate message composition, not a
smaller `BubbleMessage`. This audit reflects the Lumiverse staging source used
for the Palette 1.0 parity pass.

## Current native mount

`MinimalMessageDefault` mounts:

- root: `[data-component="MinimalMessage"]`
- assistant marker: the root does not contain the CSS-module `_user_` class
- user marker: the root contains `_user_` and exposes `data-part="user"`
- avatar wrapper: a direct root child with `_avatar_`
- avatar image: an `img` below that wrapper
- content column: `_bubble_`
- header: `_header_`
- assistant/user names: `_nameChar_` / `_nameUser_`
- metadata: `_metaPill_`, `_metaSegment_`, and `_metaDot_`
- prose: `[data-component="MessageContent"]`
- reasoning: a `ReasoningBlock` container whose direct button has
  `data-reasoning-toggle="true"`; its expanded prose is below `_bodyInner_`
- native image attachments: `_attachments_`, `_inlineImageBtn_`,
  `_inlineImageWrap_`, and `_inlineImage_`
- swipes: `[data-component="SwipeControls"]` inside the content column
- greetings: the alternate-greetings button inside the content column
- long-message control: `_longMessageTogglePill_` below `MessageContent`
- actions: a root-child `_actionsWrap_`
- native accent rail: `MinimalMessage::before`

The avatar and content column are siblings under the full-height message root.
That makes an assistant portrait a viable sticky flex item for long messages;
no React-owned DOM needs to be moved. The message root itself remains the
sticky containing block, so the portrait persists only for its own message.

## Semantic roles

Shared roles remain available when both sides should receive one treatment:

```text
minimal.frame
minimal.avatar.frame
minimal.avatar.image
minimal.content
minimal.actions
minimal.actions.controls
```

Side-aware roles express real composition differences without contaminating
the user portrait with the assistant treatment:

```text
minimal.assistant.frame
minimal.user.frame
minimal.bubble.assistant
minimal.bubble.user
minimal.avatar.assistant.frame
minimal.avatar.assistant.image
minimal.avatar.user.frame
minimal.avatar.user.image
minimal.content.assistant
minimal.content.user
minimal.actions.assistant
minimal.actions.assistant.controls
minimal.actions.user
minimal.actions.user.controls
```

Every native-aware selector remains rooted in
`[data-component="MinimalMessage"]`. Assistant selectors exclude `_user_`;
user selectors require it. No naked persistent `_avatar_` selector is legal.

Renderer-specific Minimal furniture roles exist where geometry differs from
Bubble rather than where the native concept differs. Visual Novel uses these
for its route-log inner voice and HUD; Manga, Editorial, and Journal continue to
use shared reasoning/greetings/pager recipes where the same visual treatment is
intentional.

## Media and reasoning

Raw Markdown/XML images remain owned by `MessageContent` and are naturally
renderer-independent. Native attachment roles now provide both Bubble-rooted
and Minimal-rooted candidates, covering the container, inline button, frame,
and image instead of styling `img` globally.

The current ReasoningBlock marker still matches
`data-reasoning-toggle="true"`. Shared Manga, Editorial, and Journal thinking
recipes route through that marker. Visual Novel Minimal uses its own rooted
geometry roles so applying Bubble and Minimal VN families together cannot
collide.

## Pack compositions

- Manga Minimal is a printed reading page with mirrored sticky assistant/user
  speaker rails, mirrored corner punctuation, a text-only action strip, and
  explicit non-sticky mobile resets on both portraits.
- Editorial Minimal separates the author column from a contributor stamp placed
  beside the user identity.
- Visual Novel Minimal is an ADV route log with separate assistant/player
  entries, an inner-voice ribbon, compact route HUD, and shared CG media.
- Journal Minimal preserves the paper card while giving assistant and user
  portraits intentional sides and docking actions outside header flow.

All four packs deliberately suppress the native `::before` strip through the
existing `minimal-native-strip-off` recipe. Persistence remains schema v39;
these are static roles and recipes, so no migration is required.

### Side-local Design edits
When a side-specific MinimalMessage scope is selected, editing a broader matching pack packet now materializes a sparse local Assistant/User override instead of mutating the shared selector. This keeps the inspector's compiled selector and generated CSS ownership aligned.
