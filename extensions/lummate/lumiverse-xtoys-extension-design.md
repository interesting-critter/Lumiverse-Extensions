# Lumiverse XToys Extension Working Design

## Purpose

This document captures the current end-to-end user experience and system behavior for a Lumiverse extension that connects XToys to AI roleplay scenes.

The extension's goal is to convert erotic scene narration into tactile toy behavior in a way that feels:

- narratively coherent
- physically reflective of progression inside the scene
- safe and user-controlled
- explicit and opt-in at playback time

## Product Summary

The extension is a full-stack Lumiverse Spindle extension that:

1. waits for a final assistant message to finish generating
2. lets the user read the scene first
3. creates a message-specific tactile playback plan only when the user presses the Play button on that message
4. starts playback only after that plan is created
5. sends the resulting control sequence to XToys

## Core Experience

The intended user experience is:

1. Lumiverse generates a final assistant message.
2. The user reads the message at their own pace.
3. The user presses the Play button on that exact message.
4. The extension analyzes that message on demand.
5. The extension creates a tactile plan tied to that specific message.
6. XToys performs the sequence.
7. After the sequence ends, the extension either:
   - stops
   - loops
   - holds the final action
   - resumes a previous held state

The extension should never require automatic playback by default.

## Message-Level Playback Model

Each assistant message gets its own tactile plan.

This removes ambiguity because the user is always choosing to play the plan attached to a specific message, not a global pending queue.

Each message plan is generated only when the user presses Play on that message.

The extension should not keep a long-term library of generated message plans.

Instead, it only needs two runtime plan slots:

- the current message plan
- the immediately previous message plan

When a new message is played:

- the previous slot becomes the former current plan
- the current slot becomes the newly generated plan

Older message plans do not need to be preserved.

After creation, the plan may be cached for replay until it is invalidated or regenerated.

## Narrative Parsing Scope

The parser should only care about configured intimate contact involving the user.

By default this means the user's genital contact zone, but the user should be able to override that default zone for a given roleplay context.

Examples of alternate default tracked contact zones may include:

- anus
- mouth
- other user-defined intimate zones

It does not matter whether the user is:

- acting on another character
- being acted on by another character

If the action results in contact with the user's currently configured primary contact zone, it is relevant.

Examples:

- "you thrust into them" -> relevant
- "they thrust into you" -> relevant
- "they kiss your neck" -> not relevant

## Combined Actor-Actee Resolution

The final tactile result of a beat should be resolved from all actor-side participants together with the actee-side participant.

Every action is a combined physical result, even when one side is clearly leading.

The actor side may contain:

- one actor
- multiple coordinated actors

All actor-side and actee-side participants should be processed through the same contribution pipeline.

That means every participant uses the same general structure for:

- profile traits
- current state
- internal mechanical axes
- per-participant contribution calculation

The difference between them should come from scene interpretation and weighting, not from using different calculation machinery.

Examples:

- "She quickens her suction." -> her profile and state lead the result, but the user's profile and state may still influence the final feel
- "He deepens his thrusts." -> his profile and state lead the result, but the user's profile and state may still influence the final feel

This can take several forms:

- a single actor dictates most of the action
- multiple actors jointly shape the action
- the actee meets the actors' action
- the actee backs away from the actors' action

Example:

- "She arches her hips meeting his thrusts."

## User-Genital Contact Filtering

Participant inclusion should not start from every participant mentioned in the scene.

Instead, each beat should first be filtered through the question:

`which participants are in direct contact with the user's genital involvement in this beat?`

Recommended rule:

1. identify whether the user's genital is involved in the beat at all
2. if not, ignore the beat
3. if yes, trace the participants whose action is directly in contact with that user-genital interaction
4. include only those participants in the final tactile calculation
5. normalize weights only across that included subset

This prevents unrelated parallel actions from influencing the tactile result.

Examples:

- if the user uses their genital to act on two other characters, both contacted characters are included
- if one actor acts on multiple actees and the user is only one of those actees, only the actor-user contact is included
- if multiple actors are simultaneously acting on the user, all such actors are included together with the user

So the planner should begin by enumerating all participants and then, starting from the user's genital contact, determine which participants are actually connected to that contact chain for the current beat.

## Response Modes

Each beat should resolve a response mode between the participants.

Recommended response modes:

- `lead`
- `meet`
- `withdraw`
- `mutual`

These determine how much each side contributes to the final tactile result.

## Contribution Weighting

Every beat should consider both sides, but one side may have a greater influence depending on the narration.

Recommended internal fields:

- `actor_weights`
- `actee_weight`
- `response_mode`

The narrated action still remains primary.
The weighting model only determines how the participants' profiles and states shape the final mechanics.

The simplest implementation model is a normalized weighted combination.

Example:

`final_result = sum(actor_i_weight * actor_i_contribution) + (actee_weight * actee_contribution)`

With the constraint that:

`sum(actor_i_weight) + actee_weight = 1.0`

This allows straightforward cases such as:

- `0.5 * actor + 0.5 * actee`
- `0.7 * actor + 0.3 * actee`
- `0.4 * actor_a + 0.2 * actor_b + 0.4 * actee`
- `0.3 * actor_a + 0.3 * actor_b + 0.4 * actee`

The parser should infer these weights from the scene language and response mode.

Examples:

- an actor-led action may bias toward the actor
- a mutual action may approach `0.5 / 0.5`
- a withdrawing actee may reduce the actee's contribution or dampen the combined result

## Shared Contribution Pipeline

The extension should use one reusable participant-contribution function for every participant, whether that participant is an actor or the actee.

Conceptually:

`participant_contribution = f(profile, traits, current_state, narrative_role_context)`

Then:

- compute each `actor_i_contribution` with that shared pipeline
- compute `actee_contribution` with that same pipeline
- combine them through normalized weights and response effects

This keeps the implementation simpler and ensures that user profiles and character profiles behave consistently.

If multiple actors are involved in the same beat, each one should have:

- its own character profile
- its own current state
- its own contribution weight

Those actor-side contributions are then summed together with the actee-side contribution.

## Additive and Subtractive Response Effects

Contribution weight alone is not enough to represent how the actee responds.

The extension should separately model whether the actee:

- amplifies the action
- remains mostly neutral
- dampens the action

Recommended implementation model:

`final_result = sum(actor_i_weight * actor_i_contribution) + (actee_weight * actee_modifier * actee_contribution)`

Where `actee_modifier` is determined by the parsed response mode.

Example tendencies:

- `lead` -> near-neutral or mildly positive modifier
- `meet` -> positive additive modifier
- `mutual` -> stronger positive additive modifier
- `withdraw` -> reduced or negative modifier

This allows:

- meeting the action to amplify the final result
- mutual action to reinforce the final result even more strongly
- withdrawal or shyness to dampen the final result

## Per-Axis Response Effects

In practice, response effects may not affect every mechanical axis equally.

For example:

- `withdraw` may reduce frequency more than strength
- `meet` may boost rhythm and persistence
- `mutual` may boost both strength and frequency

Because of this, response effects may later need to be applied separately across:

- strength
- frequency
- persistence
- transition aggression

For v1, however, a simpler shared actee modifier is acceptable as long as the final values are clamped safely.

## Progression-First Design

The tactile output should not be static.

A single final message may describe progression such as:

- teasing contact
- stronger contact
- change in contact style
- climax or peak
- easing off

The extension should preserve that progression by converting the message into an ordered sequence of tactile beats.

In other words:

message -> scene timeline -> tactile timeline

## Beats

A beat is one short phase of relevant tactile action inferred from the message.

A beat should contain:

- `action_type`
- `strength`
- `frequency`
- `duration_class`
- `duration_ms`
- `transition_style`
- `count_hint`
- `persistence`
- `explicit_change`
- `explicit_stop`
- `fallback_behavior`

### Beat Field Intent

- `action_type`: the kind of contact, such as stroke, thrust, suction, grind, tease, lick, squeeze, or pause
- `strength`: how forceful the action should feel
- `frequency`: how fast or rhythmic the action should feel
- `duration_class`: narrative duration such as instant, brief, sustained, or ongoing
- `duration_ms`: actual playback duration used by the controller
- `transition_style`: how the beat should enter or exit, such as ramp, snap, pulse, or fade
- `count_hint`: whether the action is one-shot, a few repetitions, repeated, or continuous
- `persistence`: whether the action is momentary, sustained, ongoing, or stopping
- `explicit_change`: whether the message clearly changes from an earlier action
- `explicit_stop`: whether the message clearly ends or withdraws contact
- `fallback_behavior`: what to do when the beat ends, usually `resume_previous` or `idle`

`action_type` is a semantic label derived from the scene, not a guarantee that XToys or a given toy has a built-in direct equivalent for that sensation.

## Character Modulation Layer

The parser should not be the only source of mechanical values.

The final tactile output should be resolved from multiple layers:

`narrative beat` + `character profile` + `current character state` + `user overrides` = final beat values

This allows the same narrated action to feel different depending on who is involved and what state each participant is in.

Example:

- a strong, bold, highly aroused character may resolve to higher baseline strength, faster tempo, and sharper ramps
- a shy, tired, restrained character may resolve to softer onset, lower tempo, and gentler sustain

The narrative still defines what is happening.
The participant profiles and states define how that action tends to feel.

## Character Inputs

Character modulation should combine three input groups for each participant involved in the beat.

### 1. Character Disposition

These are relatively stable behavioral traits, such as:

- feral
- meek
- sexual
- shy
- bold
- dominant
- gentle
- restrained

These should bias style and default intensity tendencies.

### 2. Physical Traits

These are longer-term embodied traits, such as:

- strong
- athletic
- thin
- weak
- heavy
- agile

These should bias strength ceiling, tempo ceiling, motion weight, and endurance.

### 3. Current Physical or Emotional State

These are more dynamic conditions, such as:

- tired
- aroused
- sedated
- well-rested
- overstimulated
- desperate
- shaky
- focused

These should bias the current output rather than permanently redefining the character.

The current physical and emotional state of both actor and actee should be parsed dynamically from semantic context when possible.

If no current-state signal is found in the scene, the extension should fall back to baseline values for that participant.

## Mechanical Axes

User-facing traits should resolve internally into a smaller set of mechanical dimensions.

Recommended axes:

- `baseline_strength_bias`
- `baseline_tempo_bias`
- `ramp_aggression`
- `motion_weight`
- `endurance`
- `smoothness`
- `dominance_pressure`
- `teasing_tendency`

This keeps the system expressive for users while remaining stable and tunable for implementation.

## Resolution Order for Final Beat Values

The final values sent to XToys should be resolved in this order:

1. parse the narrative beat from the message
2. apply character disposition modifiers
3. apply physical trait modifiers
4. apply current state modifiers
5. apply user-defined per-character overrides
6. clamp the result through global safety limits and user caps

This ensures the narrative remains primary while still allowing character-specific tactile identity.

## Character Profiles and User Overrides

Users should be able to adjust base tactile behavior per character.

Examples of likely per-character controls:

- baseline strength
- baseline tempo
- aggressiveness
- smoothness
- endurance
- escalation tendency
- hold tendency

These controls should act as modifiers, not replacements for scene parsing.

The goal is:

- the parser decides the relative action
- the character profile colors how that action feels
- the user can tune the final expression for each character

The user should also have a tactile profile stored in the same general way as other characters so scenes can consistently resolve the user's side of combined actions.

## Character Profile Initialization and Caching

Character tactile profiles should be treated as mostly static derived data.

They should not be regenerated on every message.

Instead, the extension should separate:

- `character source data`
- `derived tactile profile`
- `user overrides`

The same caching approach should apply to the user's own tactile profile where relevant.

Character cards should not always be assumed to map one-to-one with a single participant.

Some cards may actually describe:

- multiple named characters
- a pair or group presented in one card
- a merged persona where the prose later treats those identities separately

Because of this, the extension should distinguish between:

- `character card source`
- `participant identities extracted from that source`
- `derived participant tactile profiles`

Recommended behavior:

1. when a character card is first encountered, the extension should determine whether it represents one participant or multiple participant identities
2. if multiple participant identities are present, each one should receive its own cached participant profile
3. each derived participant profile should still remember which character card source it came from
4. later parsing and current-state tracking should operate on the participant identities, not only on the top-level card id
5. regeneration should refresh all participant profiles derived from that card source

Recommended behavior:

1. when a character is first encountered, or when the extension is first enabled for that character, Lumiverse derives the tactile profile in the background
2. the derived profile is stored for reuse across later messages
3. later message parsing uses the stored profile rather than re-reading full character text each time
4. if the character profile changes, the extension should detect that change and offer regeneration
5. the user should also be able to manually trigger regeneration at any time

This keeps token usage low while also improving consistency of tactile behavior.

## Character Profile Invalidation

The extension should regenerate a character's derived tactile profile only when needed.

Recommended invalidation triggers:

- the underlying character description changes
- the source personality or trait fields change
- the user presses a `Regenerate` action
- the user explicitly requests a reset of derived values

If possible, the extension should track a lightweight fingerprint or hash of the relevant character source fields so it can cheaply determine whether regeneration is necessary.

If one character card yields multiple participant identities, the invalidation should apply to that whole source card and all participant profiles derived from it.

## Design Principle for Character Modulation

Character traits should bias ranges and tendencies, not hard-code absolute output.

Example:

- a strong character should not always produce high strength
- instead, strength-related beats should resolve higher within their allowed range than they would for a weak or tired character

This preserves narrative meaning while making the tactile output feel more personalized and embodied.

## Composite Character Cards

The system should treat composite character cards as source containers, not as proof that only one participant exists.

That means:

- one card may yield one participant profile
- one card may yield several participant profiles
- current-state parsing may need to track several participant states that all originate from the same underlying card

This matters for:

- group scenes
- parent/daughter or sibling pair cards
- duo or trio cards written as a single entity
- any card where the message later assigns separate actions, states, or speech to different named individuals

In those cases, participant resolution should happen before profile derivation and before current-state tracking, so that:

- each named participant can have its own profile
- each named participant can have its own parsed current state
- beat weighting can operate on actual participants rather than on one merged card identity

## Beat Timing

Natural sexual actions often last seconds or minutes.

The system should therefore distinguish between:

- narrative duration
- playback duration

Narrative duration describes what the text implies.
Playback duration is how long the toy actually performs that beat.

Playback duration may be compressed for practicality, but the shape of progression should be preserved.

In practice, LLM-generated erotic scenes often do not state exact times.
They usually imply duration through wording such as:

- "briefly"
- "for a moment"
- "slow, lingering strokes"
- "long strokes"
- "for a while"
- "keeps going"

Because of this, the parser should not try to infer exact seconds directly from prose.
Instead, it should map narrative duration language into a small set of duration buckets that the user can tune in settings.

## Duration Quantization

The parser should convert vague narrative duration into normalized duration classes first, then resolve those classes into concrete playback seconds using user settings.

Recommended duration classes:

- `instant`
- `very_short`
- `short`
- `medium`
- `long`
- `extended`
- `ongoing`

This makes the parser stable even when the prose is imprecise.

## Narrative-to-Duration Mapping

The parser should default ongoing erotic contact to persistence unless the prose gives a reason to bound it.

In practice, the parser should look first for:

- discrete acts
- explicit counts
- explicit seconds or other direct time language
- explicit pauses

If those are not present, maintained erotic contact should default to `ongoing` rather than `short`.

Examples of likely mappings:

- "one deep thrust" -> `instant`
- "briefly" -> `very_short`
- "for a moment" -> `very_short`
- "a few strokes" -> `short`
- "slow strokes" -> `medium`
- "long strokes" -> `long`
- "for a while" -> `long`
- "keeps going" -> `extended` or `ongoing`
- "continues" -> `ongoing`
- "pauses" -> bounded interruption or `pause`
- "holds still for a beat" -> brief pause, then resume if the surrounding action is still active

These are parser defaults, not final playback times.

## User-Adjustable Duration Table

Each duration class should resolve to a user-configurable number of seconds.

Example default table:

- `instant` = `1s`
- `very_short` = `2s`
- `short` = `4s`
- `medium` = `7s`
- `long` = `12s`
- `extended` = `20s`

`ongoing` should not resolve to one fixed duration by default.
Instead, it should be governed by playback mode, hold behavior, loop behavior, or a maximum hold limit.

This gives the system a concrete output model while still respecting vague scene language.

## Duration Resolution Rules

The controller should resolve beat duration in this order:

1. explicit numeric time in the message, if present
2. explicit pause or interruption language
3. one-shot or other discrete semantics such as "once" or "one deep thrust"
4. explicit count semantics such as "twenty strokes"
5. narrative duration words mapped to duration classes
6. default ongoing persistence when erotic contact is maintained but not temporally bounded
7. user-configured duration table for final second values

This allows the parser to stay semantically grounded while still producing exact timing values for XToys playback.

## Action-Type Default Durations

Action-type defaults should be used cautiously.

When a message implies contact but gives no duration words at all, the parser should not automatically collapse the action into a short beat.

Instead:

- discrete acts may still use short action-type defaults
- explicit counted actions may use bounded repeated defaults
- maintained erotic contact should default to `ongoing`
- action-type defaults should mainly help choose relative shape when a bounded action is clearly present but incompletely specified

Examples:

- thrust -> usually shorter by default
- stroke -> usually medium by default
- suction -> often medium to long by default
- tease -> often short to medium by default
- grinding -> often long by default

These defaults should also be user-adjustable, either directly or through a profile system later.

## Duration and Persistence Are Different

Duration and persistence should be treated as separate concepts.

Examples:

- "one deep thrust" may have `duration_class = instant` and `persistence = instant`
- "slow strokes continue" may have `duration_class = medium` for the immediate beat, but `persistence = ongoing`
- "firm suction lingers" may have `duration_class = long` and `persistence = sustained`
- "she pauses, then resumes sucking" may have a brief pause beat, while suction remains the surrounding ongoing action before and after the pause

This distinction matters because a beat may play for a short concrete time while still implying a held final state afterward.

## User Experience Goal for Duration

The user should be able to shape how long tactile actions last without changing the parser's understanding of the prose.

In practice, that means:

- the parser decides the relative category
- the user decides how many seconds that category means for their body and toy

This keeps the system both semantically consistent and personally adaptable.

## Playback Styles

The user can choose how a sequence behaves once played.

### 1. Run Once

The full sequence plays one time, then ramps down to idle.

### 2. Loop Sequence

The full sequence repeats until stopped, replaced, or timed out.

### 3. Run Once, Hold Final

The full sequence plays once, then the final tactile state is sustained.

## Narrative Overrides Playback Preference

Playback style is a default preference, not an absolute rule.

Narrative meaning should override it unless the user explicitly chooses to force another behavior.

Example:

- "one deep thrust" should play once
- it should not automatically become a permanently held action just because the global mode is `Run Once, Hold Final`

The system should prioritize:

1. explicit stop or withdrawal
2. explicit one-shot action
3. explicit sustained or ongoing action
4. user playback preference when the text is ambiguous

## Continuity Across Messages

The extension should behave like a scene-state tracker, not a simple per-message trigger.

Absence of mention is not the same as stoppage.

If a new message does not explicitly change or stop the ongoing genital contact, the previous held tactile state may remain valid.

Examples:

- if one message ends in sustained suction and the next message is mostly dialogue, the suction may continue
- if the next message explicitly slows the pace, the active state should transition down
- if the next message says the characters pull away, the active state should stop

## Continuity Verdict

Each parsed message should produce a continuity verdict relative to the currently held scene state.

Possible values:

- `continue`
- `progress`
- `modify`
- `replace`
- `stop`

This verdict determines how the new message plan should interact with an existing held state.

The continuity verdict informs transition planning, but the final playback decision should still be resolved through the transition modes defined later in this document.

`modify` means the message keeps the same underlying contact mode but changes one or more of its parameters.

Examples:

- "She quickens her suction."
- "He deepens his thrusts."
- "The strokes grow slower but firmer."

These are not the same as `replace`.

- `modify` means the existing action is still fundamentally the same, but its strength, frequency, depth, rhythm, or intensity profile changes
- `replace` means the contact mode itself changes, such as going from stroking to suction, thrusting to grinding, or contact to withdrawal

## Active, Held, and Message Plan State

The system should keep three distinct concepts:

### Active Tactile State

What XToys is currently doing right now.

### Held State

The last narratively persistent tactile state that remains in effect after a prior playback.

The held state must remain in memory so the extension can compare it against newly played messages and decide the best transition strategy.

Held state is runtime-only.

If the user leaves a chat and later reopens it, the held state should not be restored automatically.

Reopening a chat should start from a clean runtime state with no active held action, no carried-over current plan, and no carried-over previous plan.

Recommended held-state fields:

- `action_family`
- `resolved_action_preset`
- `resolved_execution_profile`
- `contact_zone`
- `strength`
- `frequency`
- `persistence`
- `actor_contribution`
- `actee_contribution`
- `source_message_id`
- `started_at`
- `last_updated_at`

### Message Plan

The parsed tactile plan attached to a specific assistant message.

Message plans are created on demand when the user presses Play on that message, not automatically at generation time.

### Parser Session State

Whether erotic semantic parsing is currently armed or dormant.

This state exists to reduce token cost without requiring a visible master switch in the UI.

## Per-Message Play Behavior

The Play button is tied to each individual message in the Lumiverse UI.

That means:

- playback is explicit
- the user always knows which message will be enacted
- there is no ambiguity about which sequence is starting

When the user presses Play on a message:

1. if needed, the extension parses the message and creates its tactile plan on demand
2. the extension loads the tactile plan for that message
3. the controller compares it with any currently held state
4. XToys executes the message progression
5. the controller decides the resulting end state

Pressing Play should also arm the parsing session if it is currently dormant.

The plan generated for that message is only needed for immediate runtime use.

The extension does not need to retain a permanent archive of prior message plans.

## Per-Message Control Surface

The main UI should expose a single primary `Play / Stop` toggle button for each message.

To keep the interface compact, secondary controls should appear in a breakout bubble when the user:

- hovers on desktop
- long-presses on mobile

The breakout bubble may contain:

- `Regenerate`
- playback mode toggle: `Play Once`, `Loop`, `Play Once and Hold`
- tracked participant selector, defaulting to the user persona but allowing any current chat participant to become the focal tracked participant

This keeps the default interaction simple while still exposing advanced control per message when needed.

## Hidden Parsing Session Arming

The extension should use a hidden arm/disarm model for erotic semantic parsing.

Behavior:

- parsing is dormant by default
- the first time the user presses Play on a message, parsing becomes armed
- while armed, later final assistant messages are eligible for erotic relevance and continuity evaluation
- if enough consecutive messages are judged non-relevant, parsing is disarmed again
- pressing Play later re-arms parsing

This avoids exposing a separate erotic parsing toggle while still controlling token usage.

Being armed does not require every new message to receive a full tactile plan in the background.
In v1, full message plans are still created only when the user presses Play on a specific message.

## Parsing Session Deactivation Threshold

The extension should support an adjustable threshold for how many consecutive non-relevant assistant messages are allowed before parsing disarms itself.

Example behavior:

- threshold = `3`
- after 3 consecutive non-relevant assistant messages:
  - parsing disarms
  - any held tactile state is ramped down and cleared
  - temporary continuity context is cleared
  - the extension returns to a dormant state until Play is pressed again

This threshold should be user-adjustable because some scenes may include more dialogue or non-contact narration between erotic beats than others.

## Parsing Session Design Principle

The hidden parsing session is a token-efficiency mechanism, not a statement of consent or scene meaning.

Its purpose is:

- to avoid parsing every message forever
- to keep erotic parsing active during likely erotic stretches
- to naturally return the extension to idle after extended non-erotic dialogue or narration

If parsing disarms too early, the user can always re-arm it by pressing Play on a later message.

## Transition Rules Between Messages

When a new message is played:

1. if the message explicitly changes the action, transition from the prior held state into the new sequence
2. if the message explicitly stops contact, ramp down and clear the held state
3. if the message is only dialogue or non-contradictory narration, the prior held state may remain valid
4. if the message adds contact without clearly replacing the old one, treat it as progression rather than reset

## Transition Modes

When a new message plan is played while a held state exists, the extension should compare the new plan against the held state and choose one of three transition modes:

- `replace`
- `modulate`
- `blend`

The basic planner model is:

`held_state + new_message_plan -> transition_mode -> execution_plan`

### Replace

Use `replace` when:

- the contact family changes
- the old action is explicitly ended
- the new action is not mechanically or narratively compatible with the old one

Examples:

- stroking becomes suction
- thrusting becomes grinding
- contact ends and a new contact form begins later

### Modulate

Use `modulate` when:

- the core action family remains the same
- the new message changes urgency, force, depth, rhythm, or similar parameters

Examples:

- thrusts become more urgent
- suction slows but deepens
- strokes become firmer

### Blend

Use `blend` when:

- the new message adds a compatible concurrent action on top of the held action
- both actions can be represented as one combined tactile result

Example:

- licking begins while suction continues

For v1, `blend` should usually compile into one composite resultant beat rather than assuming true parallel device-layer playback.

## Composite vs Parallel Blend

`blend` should not be treated as a blanket promise that XToys can literally stack two semantic actions independently on every toy.

The default interpretation should be:

- `composite_blend`: Lumiverse combines the held action and the new action into one resultant mechanical profile

This means the planner may adjust:

- amplitude
- tempo
- timing
- texture or pattern preference
- transition behavior

to approximate the combined sensation as one unified output.

An optional advanced interpretation may exist for certain calibrated setups:

- `parallel_blend`: multiple channels, outputs, or compatible toy capabilities are used to express concurrent layers more directly

This should only be assumed when the user's calibrated action presets and toy setup explicitly support it.

## Blend Realism and Testing Constraint

The realism of blending remains device-dependent and should not be over-promised in the design.

At the design level, the safe assumption is:

- `replace` is broadly reliable
- `modulate` is broadly reliable
- `blend` is reliable as planner-side compositing
- `parallel_blend` is experimental until verified on actual XToys setups and physical devices

Real-world testing is required before confirming how convincingly specific toys can express concurrent blended actions in practice.

## Default Transition Policy for V1

The controller should default to:

- `modulate` when the action family is the same
- `replace` when the action family changes
- `blend` only when the actions are narratively concurrent and mechanically composable

For v1, that should usually mean `composite_blend`, not assumed `parallel_blend`.

This keeps the first implementation predictable while still supporting richer transitions.

## End-of-Playback Resolution

After a message plan finishes playing, the controller should resolve to one of the following:

- `idle`
- `hold_new_final`
- `resume_previous_held`
- `loop_current_plan`

Resolution depends on:

- the parsed narrative persistence of the final beat
- the selected playback style
- whether a prior held state still makes narrative sense
- whether the message explicitly stopped or replaced prior contact

## Example Resolution Cases

### Case 1: Full progression with sustained ending

Message:

"Gentle strokes build into firm suction."

Result:

- play sequence in order
- final suction may become the new held state

### Case 2: Dialogue while prior act continues

Previous held state:

- suction

New message:

"She smiles and whispers in your ear while staying close."

Result:

- no new tactile progression required
- prior held state may continue

### Case 3: One-shot action

Message:

"One deep thrust."

Result:

- play one strong pulse
- then return to previous held state if still valid
- otherwise go idle

### Case 4: Explicit stop

Message:

"They pull away and the contact ends."

Result:

- ramp down
- clear held state

## XToys Integration Model

Lumiverse should send structured semantic events to XToys rather than trying to encode raw toy behavior directly in the parser.

Lumiverse is responsible for:

- message parsing
- beat sequencing
- character-aware modulation
- continuity handling
- playback decisions
- user-facing controls

XToys is responsible for:

- toy-specific behavior
- script-side execution
- jobs, triggers, and variables
- intensity/speed mapping
- transition smoothing
- loop and hold execution

## XToys Control Approach

The extension should use XToys webhook-based communication to send high-level semantic commands from Lumiverse into an XToys script.

Research into the XToys guide suggests that XToys is best understood as:

- a `Webhook` tool that receives external events
- a `Script` that reacts to those events
- `Actions`, `Triggers`, `Variables`, and `Jobs` that control behavior
- optional `Patterns` that can be selected or dynamically adjusted

This means XToys should be treated as a programmable tactile runtime, not as a raw toy-control API.

In XToys terms, the primary execution path is:

`Webhook -> Script -> Variables/Jobs/Actions -> Toy Output or Pattern Control`

XToys itself exposes mechanical building blocks such as intensity, timing, pattern shape, loops, positional patterns, and script logic.
It does not provide a universal built-in semantic definition of sensations like `suction`, `stroking`, or `grinding`.

Because of that, semantic action types should be interpreted through user calibration rather than assumed to map identically across all toys.

## V1 Design Decision

For version 1, Lumiverse should remain the primary sequencer.

That means:

- Lumiverse parses the final message into beats
- Lumiverse resolves continuity and playback timing
- Lumiverse decides when each beat should start
- XToys executes the currently active beat and any resulting hold, loop, or stop behavior

This decision is based on the XToys docs: webhook payloads clearly support `action` plus extra key/value pairs, and XToys scripts are clearly strong at reacting to scalar values, running jobs, changing variables, and controlling toy state over time. The docs are less explicit about ergonomically consuming and iterating over a large structured beat array entirely inside the no-code script layer.

Because of that, v1 should favor reliability and use Lumiverse as the planner and XToys as the executor.

## V1 XToys Responsibilities

In v1, XToys should:

- receive one beat or control event at a time
- map the current beat onto toy-appropriate behavior
- apply strength and frequency
- manage short ramps and local smoothing
- sustain final states when requested
- stop safely when requested
- optionally run local jobs for beat timing, ramps, or hold behavior

## Semantic Action Mapping

Lumiverse should parse scene actions into semantic `action_type` labels such as:

- `tease`
- `stroke`
- `thrust`
- `suction`
- `grind`
- `pulse`

These are semantic scene categories, not guaranteed one-to-one XToys or toy-native sensations.

The role of XToys is to express those categories through its available mechanical controls.

## User-Calibrated Action Presets

Because different toys vary significantly in what they can express, the extension should allow end users to calibrate how each semantic action type feels on their own XToys setup.

That means the runtime model should be:

`semantic action type -> user-calibrated tactile preset -> XToys mechanical parameters`

Each user-calibrated action preset may define:

- base amplitude
- base tempo
- preferred pattern or texture profile
- transition style
- repeat style
- hold tendency
- supported or unsupported status

This avoids trying to impose a universal built-in meaning for action types that may feel very different across toys.

## Calibration and Fallback Behavior

If a user has calibrated a semantic action type, the planner should start from that preset and then apply:

- character and state modulation
- actor and actee weighting
- additive or subtractive response effects
- safety clamps

If a semantic action type is not calibrated, the extension may:

- use a generic default preset
- map to the nearest supported preset
- or treat that action type as unsupported

If a user marks an action type as unsupported for their setup, the planner should fall back to the nearest acceptable calibrated action or skip that beat if no reasonable fallback exists.

## XToys Execution Modes

XToys appears to support three useful execution styles:

### 1. Single Mechanical Action Triggers

XToys can trigger one preconfigured action on one mechanical axis, such as vibration, inflation, or position, usually with a target intensity and ramp speed.

This is useful for simple testing and direct device sanity checks, but it gives Lumiverse very little room to shape tactile variation beyond selecting the action and passing scalar values.

Because the mechanical behavior is largely predefined on the XToys side, this should be treated as a minimal compatibility path rather than the preferred semantic runtime.

### 2. Action Trigger Plus Scalar Control

This should be the preferred v1 delivery model.

In this model, Lumiverse resolves the semantic beat and emits:

- one base XToys action trigger such as `thrust` or `suction`
- a scalar `intensity` value
- a scalar `seconds` value used for XToys-side ramp timing

Example:

- `action=thrust`
- `intensity=72`
- `seconds=0.24`

The XToys side then reacts to the base action trigger and uses the scalar values inside the same webhook event to shape the local output behavior.

This preserves semantic control while staying compatible with the webhook request shape that has been verified against real XToys setups.

### 3. Direct Scripted Axis Modulation

This is the long-term highest-control model, but it should be deferred beyond v1.

In this model, Lumiverse continuously drives the XToys runtime through repeated webhook updates, treating XToys more like a live execution engine than a pattern launcher.

That would let Lumiverse:

- directly vary intensity over time
- directly vary ramp behavior over time
- modulate multiple mechanical axes continuously
- own fades, holds, releases, and ongoing transitions explicitly

This would provide the richest semantic fidelity, but it also means Lumiverse must own much more runtime responsibility, including active persistence and ramp-down behavior.

## Mechanical Execution Profiles

After a semantic action type is resolved through user calibration, XToys can express it differently depending on device capability.

Examples:

- vibrators may emphasize intensity, ramp, and pulse timing
- strokers may emphasize positional patterns or funscript movement
- suction toys may emphasize pulse timing, amplitude, and hold behavior
- other supported devices may use their own best-fit mechanical expression

## Payload Direction for V1

Version 1 should prefer sending one beat at a time rather than one full sequence payload.

Reasoning:

- XToys documentation clearly supports webhook actions with scalar parameters
- XToys jobs and variables are good at local execution
- XToys documentation is less explicit about whole-sequence parsing and iteration from webhook payloads
- one-beat-at-a-time control keeps sequencing logic inside Lumiverse, where the narrative parser already lives

So the preferred v1 flow is:

- Lumiverse computes the message timeline
- Lumiverse starts playback on user command
- Lumiverse sends beat 1 to XToys
- XToys executes beat 1
- Lumiverse sends the next beat when appropriate
- XToys holds, loops, modifies, or stops according to the current control event

## Future Direction

A later version may explore sending fuller sequence payloads to XToys if:

- the XToys script layer proves comfortable for sequence parsing
- XToys JavaScript provides a clean way to ingest and iterate over structured payloads
- offloading more sequencing to XToys becomes desirable for latency or portability reasons

For now, the preferred architecture is:

- Lumiverse = narrative parser and sequence planner
- XToys = tactile execution runtime

For v1 specifically, the preferred XToys execution model should be a base action trigger plus scalar `intensity` and `seconds` values delivered in the same webhook request.

## Safety and Comfort

The extension should support:

- explicit user-triggered playback
- stop button
- panic stop
- cooldown or rate limiting
- max intensity cap
- max ramp-time cap
- max hold time
- transition smoothing
- safe ramp down behavior

The system should prefer no action over uncertain action.

If genital contact is not clear, the parser should do nothing.

## Optional User Controls

Likely user settings include:

- playback style
- default tracked contact zone for the current roleplay
- per-chat tracked participant override, persisted when the user leaves and later reopens that chat
- respect narrative duration
- allow one-shot override
- per-character tactile profile tuning
- per-action tactile calibration
- regenerate cached character tactile profile
- transition smoothing
- max intensity
- max ramp time
- max hold time
- parsing session deactivation threshold
- auto-play new scenes as an advanced opt-in option

Auto-play, if supported, should be off by default.

## Proposed High-Level Flow

1. Lumiverse finishes generating an assistant message.
2. The user reads the scene.
3. If parsing is armed, the extension may evaluate later assistant messages for erotic relevance and continuity state.
4. The user presses Play on a specific message.
5. If needed, Play arms the parsing session for later messages.
6. The extension parses that message and extracts ordered beats and a continuity verdict.
7. The extension parses or infers the current physical and emotional state of the relevant participants, using baseline values where no state is found.
8. The extension loads the calibrated tactile preset for each beat's semantic action type, or uses fallback behavior if no direct calibrated preset exists.
9. The extension modulates beat values using cached profiles, parsed current state, calibrated action presets, and user overrides.
10. The extension stores the resulting message-specific tactile plan only in short-lived runtime memory.
11. The playback controller compares the message plan with the current held state.
12. The extension sends the resulting sequence to XToys.
13. XToys executes the sequence.
14. The controller resolves the final state as idle, held, resumed, or looped.

At most, the runtime only needs:

- the current message plan
- the immediately previous message plan

If the chat is exited and later reopened, those runtime plan memories should be cleared.

## Open Design Questions

The following items still need further design:

- exact UI for the per-message action card
- exact parser schema and confidence thresholds
- exact XToys payload format
- exact mapping from action types to XToys pattern families
- exact user override rules for narrative persistence

## Deferred Features

The following features are intentionally deferred beyond v1:

### Participant Profile Override Editing

The cached participant profile model already distinguishes between:

- source data
- derived tactile profile values
- user override values

However, v1 does not need a full editing surface for those overrides.

So for v1:

- participant profiles may be derived, cached, regenerated, and reused
- the user does not need a dedicated UI for manually editing mechanical-axis override values
- regeneration should continue to refresh only the derived layer

This feature can be added later as a settings-side editing workflow where users can:

- inspect derived profile axes
- adjust override deltas per participant
- reset overrides independently of regeneration
- view effective values after derived and override layers are combined

### Multi-Toy Character Routing

XToys appears to support connecting one script to multiple toy outputs, but v1 does not need to model per-character toy routing yet.

So for v1:

- tactile planning may assume a single effective toy output path
- participant weighting and contact filtering may still resolve to one combined tactile result
- the extension does not need to assign specific characters to specific toys

This feature can be added later as a multi-toy routing layer where:

- different characters in the same scene may be mapped to different toys
- users may define additional erogenous zones beyond the default genital assumptions, such as anus, mouth, or other custom zones
- those user-defined zones may be parsed from scene context and mapped to specific toy outputs
- one beat may resolve into multiple toy-targeted output streams
- participant inclusion may be evaluated separately for each relevant user zone
- participant inclusion, weighting, and contact filtering may be resolved separately per toy path
- Lumiverse may choose whether a scene should produce one combined tactile result or several parallel toy-specific results

### True Action Blending

V1 should not promise faithful simultaneous semantic blending of multiple tactile actions on a single toy path.

Although the planner may still use composite blending internally, true action blending such as:

- stroking while sucking
- thrusting while licking
- grinding while pulsing

should be treated as a deferred feature until XToys-side implementation and real-device testing confirm how realistic it is.

For now:

- `replace` is a reliable core transition mode
- `modulate` is a reliable core transition mode
- `blend` should default to planner-side compositing into one resultant tactile profile

Later versions may support richer parallel blending when:

- the user has calibrated multi-toy or multi-channel output
- the XToys script explicitly supports concurrent routed actions
- real testing confirms the felt result is believable rather than muddy or contradictory

### Direct Scripted Axis Modulation

XToys appears capable of live modulation through repeated webhook-driven updates, but v1 does not need to depend on that model yet.

Although this is likely the most powerful long-term integration path, it should be treated as deferred until:

- the quantized v1 path is stable on real devices
- Lumiverse-side persistence and ramp scheduling are proven reliable over longer sessions
- multi-axis execution behavior is validated in actual XToys setups

For now:

- Lumiverse should quantize beats into `low`, `medium`, and `high` XToys action variants
- XToys should own the local prebuilt pattern or action behavior behind each quantized trigger
- fully continuous axis modulation should remain a later feature

## Potential Issues

The following items are acknowledged but intentionally deferred for now:

### Runtime Plan Memory

The extension does not need to keep a durable history of generated message plans.

Instead, it should only retain:

- the current message plan
- the immediately previous message plan

These runtime-only plan slots are sufficient for:

- comparing the newly played message against the immediately preceding one
- supporting replace, modulate, and blend decisions
- preserving short-range narrative continuity during an active session

When a new message is played:

- the previous slot becomes the old current plan
- the current slot becomes the newly generated plan

When the user exits a chat or later reopens it:

- current plan memory is cleared
- previous plan memory is cleared
- held state is cleared

Because long-term message-plan storage is not required in this design, stale-plan tracking is also no longer required for old message plans.

### Per-Message Advanced Controls

The current UI direction is:

- one visible `Play / Stop` toggle in the main message UI
- additional controls revealed through hover on desktop or long-press on mobile

The breakout controls should include at least:

- `Regenerate`
- playback mode toggle: `Play Once`, `Loop`, `Play Once and Hold`

This keeps the main interface minimal while still supporting plan refresh and per-message playback overrides.

### XToys Delivery and Mechanical Reliability

Because Lumiverse will send one beat at a time and XToys will execute those beats through webhook-driven scripts, real-world behavior may reveal timing drift, delivery issues, or toy-specific quirks that are hard to predict on paper.

This remains intentionally unresolved until the extension is tested against actual XToys setups and physical devices.

## Current Design Principles

The current design is guided by these principles:

- read first, play second
- per-message explicit control
- preserve narrative progression
- do not assume silence means stoppage
- let narrative meaning override mechanical defaults
- keep the user in control at every stage
