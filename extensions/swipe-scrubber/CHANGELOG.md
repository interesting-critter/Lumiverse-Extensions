# Changelog
All notable changes to this project will be documented in this file.


## 1.0
- Logic Rewrites and Cleanup
  * Fix results count checks for autoscrub - Should (hopefully) stop from bypassing quiet mode
  * Added a slight wait (~2s) for settings - Should (hopefully) fix inconsistent race on docker 
  * Metadata patch fixes - don't crash when incrementing msg count on a new chat (no existing metadata), don't clobber chat settings (AN, persona addons, etc..) stored in metadata 
  * Added in-flight task cap to scrubAll tasks (50), to prevent running on long existing chats from slamming backend
  * Fixed potential prototype-pollution vulns - Not sure that's still a thing on modern browsers, but WTH.
  * Fixed B2F error propagation not tied to userid
- Version 1.0, baby!


## 0.9.9-beta1 

- "Thinking" Scrubbing is finally available (req LV 1.0.6 or staging after 2026-06-28)
  * Enable "On Demand" scrubbing in the "Extras" menu, or perform as part of auto-scrubbing
- Make the Auto-scrubber less chatty
    * It shouldn't spam you with useless messages when there's nothing for it to do anymore
    * Added "Quiet" mode option to shut it up even more - don't receive any success toasts, only warnings/errors
- Tighten up the per-message processes for auto-scrubbing and Scrub all 
- Rework most Front/Back roundtrip logic and slimmed down frontend.setup() significantly. This should hopefully solve any more weird behavior in docker installs
- Readiness Enforcement: 1.0.6-ready 
- Shouldn't have any more userId-passing weirdness

## [0.0.7-alpha-debug2]
- Prepare for LV 1.0.6 readiness enforcement
- Refactor Extension Startup Process
- WIP - Refactor command/event handling

## [0.0.6-alpha-debug2]
- Remove stray pendingRequest deletion from response handler

## [0.0.6-alpha-debug1]
### Trying to hunt down a weird permission check failure that only seems to hit my live docker instance itermittently
- Clean up setup() 
- Set debug and loghandoff settings to "on"
- Reduce reliance on fire-and-forget handoffs
- Reduce number of userid-exempt responses


## [0.0.6-alpha] - 2026-05-24

- Fix autoscrub messageList filtering logic (slice off buffer BEFORE filtering for multi-swipe messages)
- Make Debug breadcrumbing + Handoff Logging user-settings
- Adjust Settings Panel CSS and Layout to play more nicely in "Modal Width: Compact"

## [0.0.5-alpha] - 2026-05-13

- More behind the scenes shuffling and debugging
- Implemented settings panel 
- Finally implemented AutoScrub (requires optional "Generation" permission). Set the options in the settings

## [0.0.1-alpha] - 2026-04-30
### Added
- Initial posting

