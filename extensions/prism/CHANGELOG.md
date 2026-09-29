## 1.0.2.9

- Adds an opt-in **Bake manual corrections** preference for Local and Hybrid. Manual speaker corrections can now be written back into the active stored swipe as portable `<font color>` markup instead of living only in DOM reconstruction.
- Recolors an exact existing `<font>` segment when the model used the wrong canonical color; otherwise wraps only the corrected visible segment in a new portable font tag.
- Uses quote occurrence plus nearby visible context to avoid rewriting the wrong duplicate line, and leaves the normal manual override intact as attribution evidence.
- Stores a transcript recovery backup before each successful manual bake.
- Fixes the master-switch subtitle layout so its description sits beneath the title with actual breathing room.

## 1.0.2.8

- Adds a user-wide **Prism is on / Prism is off** master switch above the scene workspace.
- Keeps the switch state across characters and chats while preserving all saved colors, bindings, and configuration.
- Dims and locks Prism controls while disabled, while keeping navigation, window controls, and the master switch reachable.
- Disables generation-time prompt injection/macros, automatic persona coloring, local DOM attribution/paint, Hybrid discovery, and post-generation hydration while Prism is off.
- Shows **Prism off** in the toolbar status instead of pretending the disabled extension is merely saved.

## 1.0.2.7

- Remember the persona-color enabled/disabled choice as the default for new chats.
- Save chat persona options before slower global preference writes so newly sent messages immediately observe the toggle.
- Add a chat-local **Persona is part of the cast** mode for impersonation / Speak for User workflows.
- Cast mode exposes the active persona to assistant-side attribution, manual recoloring, and the model speaker registry without coupling it to automatic user-message coloring.

# Prism changelog

## 1.0.2.6 — safe mobile workspace and prompt controls

- Routes phone and iOS PWA layouts through a Prism-owned, safe-area-aware body portal with an always-reachable Close control and live `visualViewport` sizing.
- Keeps Settings available as a top-level tab in every roster layout, preventing Vertical Sidebar from locking users out of presentation controls.
- Centers Horizontal Carousel selections by changing only the roster's own scroll position; selecting a character no longer shifts the desktop modal or page.
- Reorganizes Settings into aligned **Look & feel**, **Prompt**, and **Cortex & attribution** sections.
- Adds persistent interface scale and modal corner-style controls.
- Extends interface scale down to 80% and 70%, and gives the unresolved review pill its own phone/PWA row in Vertical Sidebar as well as Horizontal Carousel.
- Adds an explicit disk-style save control beside character removal; it flushes queued autosave work and persists the current visible paint without rerendering the editor.
- Adds Automatic and Preset macro prompt delivery, a custom prompt editor, and live `{{prismPrompt}}` / `{{prismHexes}}` macros.
- Preserves the saved/sync indicator preference in a user-wide cache across chat switches and transient state reloads.
- Extends release regression coverage for the safe-area portal, carousel isolation, grouped settings, prompt placeholders, and macro-only delivery.

## 1.0.2.2 — I admit defeat because nobody but me is using Lumiverse mobile at 67% browser zoom

- Replaces the cramped side roster in narrow landscape Split mode with a contained horizontal cast carousel above the editor.
- Uses compact glowing color-orb chips, selected-character emphasis, edge fades, scroll snapping, a visible slim scrollbar, and previous/next controls instead of floating initial tiles.
- Keeps Add character, Assign missing, and Regenerate in a balanced three-action row under the carousel.
- Removes the redundant large character header in this layout and automatically centers the selected character in the rail.
- Leaves normal wide Split mode and the high-scale fullscreen accessibility workspace unchanged.

## 1.0.1.9f

- Replaces the unreliable high-scale vertical editor scroll with a native horizontal two-card workspace: Paint and Identity.
- Supports both tab clicks and swipe/side-scroll navigation with scroll snapping.
- Keeps aliases, registry details, save state, and removal on the Identity card instead of below the fold.
- Opens Add Character and high-scale confirmations in Prism-owned portal dialogs above the fullscreen workspace.

## 1.0.1.9e

- High-scale/tabbed Prism no longer opens or hides a Lumiverse native modal behind its body-level workspace.
- Removes the host modal's global vertical touch/scroll lock from the accessibility layout while preserving the normal native modal on desktop split layout.
- Removes the body overflow mutation; the fixed Prism overlay and its own overscroll containment now own the fullscreen interaction.
- Adds Escape-key dismissal directly to the standalone workspace.

## 1.0.1.9d

- Outranks the older tab-layout `height: 100%` / `overflow: hidden` rule on the fullscreen `.ldc-main` container.
- Lets the editor content contribute its real height to the single `.ldc-main-wrap` scroll surface.
- Removes the scrollbar-shaped placebo where content painted past a clipped box but could not be reached.

## 1.0.1.9c

- Replaces the unreliable nested editor scroller with one native vertical scroll surface on the fullscreen main workspace.
- Keeps the horizontal roster as its own horizontal scroller while the character editor flows normally beneath it.
- Removes the competing JavaScript touch-drag fallback so Android/PWA no longer has two systems fighting over the same vertical gesture.
- Preserves the fullscreen workspace scroll position across ordinary rerenders.

## 1.0.1.9b

- Adds a guarded single-touch vertical drag and momentum fallback for Android/PWA WebViews that refuse the editor's native nested scroll gesture. Horizontal roster swipes and two-finger pinch zoom remain native.
- Sizes the fullscreen accessibility portal from the live visual viewport and updates it as browser chrome, orientation, or the on-screen keyboard changes.
- Keeps the editor pane as the only vertical scroll surface and preserves its position across ordinary rerenders.
- Aligns manifest, package, diagnostics, and distribution version metadata after the accessibility branch drifted between 1.0.9 and 1.0.17.

## 1.0.1.9 — The blind fucks update

- Removes the fullscreen portal's ancestor-level `touch-action: none`, which blocked every descendant scroll gesture even though the editor displayed a scrollbar.
- Gives the editor pane explicit vertical pan ownership and the roster strip explicit horizontal pan ownership.
- Keeps pinch zoom available while preventing the two scroll surfaces from stealing each other's gestures.
- Mounts the high-scale accessibility workspace in a body-level fullscreen portal, outside Lumi's transformed UI-scale tree.
- Fixes 1.30× UI scale expanding Prism beyond the visible viewport in both PWA and browser installs.
- Makes the editor pane the single vertical scroll surface with mobile momentum scrolling.
- Clamps every high-scale editor row and control to the visible width to prevent sideways clipping.
- Restores body scrolling and removes the portal cleanly when Prism closes or switches layouts.
- Makes the horizontal/high-scale layout a true viewport-filling Prism workspace so PWA and browser installs no longer disagree about Lumi's outer modal shell.
- Removes the redundant character profile header and bottom chat-status footer in horizontal mode; the selected roster chip remains the active-character indicator.
- Replaces the oversized **Remove from scene** action with an accessible trash icon in horizontal mode while retaining its confirmation step.
- Adds a sticky accessibility utility rail containing Local/Hybrid/LLM, expand, Settings, and Close controls so hostile zoom cannot strand the modal.
- Removes the redundant Settings button from the horizontal roster action grid.
- Makes the saved/sync preference authoritative in the editor footer and updates it optimistically before persistence.
- Replaces gradient rails with compact circular stop buttons in the horizontal accessibility layout.
- Keeps all two- and three-stop hex values visible at high UI scale with tighter, centered inputs.
- Moves **Reverse direction** onto its own full-width row beneath Direction and Stops so it cannot escape the viewport.
- Adds a first-class **Settings** tab whenever the horizontal accessibility layout is active.
- Adds a second **Settings** action beside the roster controls, so configuration remains reachable even when hostile mobile scaling pushes the header off-screen.
- Pins the accessibility header while its workspace scrolls and changes mobile roster actions to a clean two-by-two grid.
- Adds an Auto/Split/Horizontal roster preference. Auto switches to the horizontal roster above roughly 1.15× effective UI scale or on cramped viewports.
- Reflows the character roster above the editor, removes chunky roster initials, and prevents high-scale/mobile controls from colliding.
- Adds a global toggle to hide Prism saved/sync indicators from both the chat toolbar and editor footer.
- Keeps the accessibility layout available as a manual preference at normal UI scale.
- Gives the host modal body one explicit viewport-safe height and makes Prism fill it exactly.
- Shortens Compact and Auto layouts so they stop drifting toward the bottom edge with dead editor space.
- Budgets for Lumi's title bar and modal chrome instead of sizing the inner shell against the whole viewport.
- Keeps scrolling inside the roster, editor, settings, and review panes; the modal shell itself stays fixed.

## 1.0.1.8 — Quiet no-chat state

- Treats startup and chat-exit state probes without an active chat as an ordinary empty Prism state instead of throwing a backend error.
- Removes the redundant **Open a chat first** host toast while preserving the inline no-chat message if Prism is opened manually.

## 1.0.1.7 — Fast UI preferences and compact fit

- Moves modal size and expanded-state persistence onto a global-only backend route that bypasses chat mutation queues and scene rebuilds.
- Serializes global preference writes separately to prevent lost preference updates without blocking behind hydration.
- Tightens Compact spacing, keeps scrolling inside Prism, hides compact pane scrollbars, and prevents the host modal from growing its own scrollbar.

## 1.0.1.6 — Responsive desktop modal

- Prism now grows automatically on large and ultrawide displays instead of remaining fixed at 780px.
- Added remembered **Auto**, **Compact**, and **Large** interface sizes plus an in-modal expand/restore button.
- Expanded mode uses up to 96% of the viewport with a sensible 1800px cap; mobile remains near full-screen regardless of the desktop preference.
- Large layouts widen the scene roster and scale key controls, avatars, typography, and spacing without using transform scaling.
- Modal dimensions are recalculated when the preference changes, and diagnostics now report the active interface mode.

## 1.0.1.5 — Persona DOM identity fix

- Persona dialogue candidates now use the binding's stable `speakerUid`, matching the override keys persisted by the backend.
- Persona candidates explicitly report themselves as paintable when a canonical registry color exists, removing the false “color not assigned” underline and tooltip.
- Manual reassignment of a user quote to the active persona no longer loses the speaker after state reload or strips its paint.

## 1.0.1.3

- Simplified Hybrid review to genuinely new tagged characters only; alias proposals, merge guesses, color drift, known-speaker conflicts, and anonymous colors no longer appear in the user-facing inbox.
- Replaced merge/alias adjudication with three explicit outcomes: **Add to registry**, **Keep temporary**, and **Ignore**.
- **Keep temporary** now resolves the yellow warning while preserving the speaker's name/color as a chat-local provisional continuity hint outside the permanent roster.
- Known characters and temporary cameos are resolved silently during hydration, preventing repeat review cards.
- Temporary speakers expire with provisional evidence and are cleared by **Clear temporary speakers & evidence**.

## 1.0.1.2

- Resolve inferred Hybrid names against the existing scene roster before creating alias suggestions.
- Treat explicit mismatched known speakers as conflicts instead of proposing that they become aliases of the color owner.
- Suppress one-off weak structural guesses and fragment names such as “in the” until independent evidence exists.
- Preselect known unbound characters when Hybrid finds a color for them.

## 1.0.1.1

- Serializes per-user, per-chat backend state mutations so editor autosaves, Hybrid hydration, roster changes, and review approval cannot overwrite one another with stale config.
- Makes manual character creation atomic, including an optional detected color, and verifies the new roster entry before reporting success.
- Prevents explicit manual characters from being swallowed by an incidental alias on an automatically discovered speaker.
- Lets Hybrid Review merge tentative evidence into unbound scene characters, including manually added roster entries.
- Selects and scrolls to a newly added character immediately after creation.

## 1.0.0

- Correlates Hybrid hydration with user-scoped generation registry snapshots and never substitutes a previous assistant message.
- Retries delayed message storage, prunes stale observations and overrides, bounds registry state, and expires tentative evidence without counting Prism-induced echoes as independent proof.
- Caps and sanitizes prompt registry data; conflicting canonical colors are quarantined from model injection.
- Separates safe font-tag normalization from optional historical persona markup, with dry runs, recovery backups, rollback, and restore.
- Adds retry-current hydration, temporary-evidence reset, privacy-safe diagnostics, collision-safe registry export/import, and visible collision health.
- Uses the first dialogue stop as the sole canonical registry color.
- Renders gradient dialogue as its first solid stop while a message streams, restoring the full gradient only after completion or cancellation.
- Generates missing colors against the currently rendered Lumi message surface, including light themes, instead of assuming a dark background.
- Ships a reproducible TypeScript build, runnable regression harness, fixtures, and CI verification.

## 0.8.0 – 0.8.3

- Added closed-loop Hybrid discovery, tentative speaker review, provisional continuity, registry revisions, and stable editor identity/order.

## 0.7.0 – 0.7.5

- Added Hybrid, reversible gradient/thought channels, existing-tag attribution, manual roster editing, canonical color synchronization, and theme-paint filtering.

## 0.6.x

- Added Local attribution, one-click scene setup, Cortex reconciliation, manual quote correction, and the integrated Prism modal.
