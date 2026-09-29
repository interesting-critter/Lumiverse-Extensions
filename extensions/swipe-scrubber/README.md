<!-- Mirror of upstream documentation. Everything below the metadata table is the original file, unmodified. -->

| Field | Value |
| --- | --- |
| Extension | Swipe Scrubber |
| Source repository | https://codeberg.org/targren/Lumiverse-SwipeScrubber |
| Original link (from Lumiverse-Extensions README) | https://codeberg.org/targren/Lumiverse-SwipeScrubber |
| Upstream path | `README.md` |
| Retrieved | 2026-09-29 @ `master` (`8cf9758`) |
| Note | Hosted on Codeberg rather than GitHub; cloned directly from `codeberg.org`. |

---

# Swipe Scrubber - Chat Housekeeping
Cleans up unneeded swipes - a Lumiverse extention inspired by Avilnetro's indispensable ['All But This Swipe' SillyTavern Extension](https://github.com/Avilnetro/all-but-this-swipe). 


<div style="text-align: center; border: 1px solid #f39c12; padding: 12px; background: #fff3cd; color: black;">
<strong>⚠️ Warning:</strong> 
<p>
This is <strong>BETA</strong>, in-testing software. For the love of all that's good and right, backup your chats before testing it. (The fastest way in Lumiverse is to "Fork" your chat from the last message).
</p>
</div>

Using the extension is straightforward. It adds a button right into the character messages' Action Bars. Push the button and any swipes other than the current message will be removed. 

<img src="https://i.imgur.com/n7ul9nz.png">



## New Settings

_Settings are located in the "Settings"->"Extensions" sidebar_

- "Enable 'Scrub All'" - Control whether the "Scrub Unused Swipes (all msg)" option appears in the "Extras" menu
- "Enable 'Scrub Reasoning'" - Control whether the "Remove Think Blocks (all msg)" option appears in the "Extras" menu
- "Enable 'Auto-Scrubbing' - (Requires "Generation" permission) Automatically scrub past messages at an interval
   * "Auto-Scrub Delay" - How often to auto-scrub (every _N_ messages)
   * "Auto-Scrub Buffer" - When autoscrubbing, *DON'T* scrub the past _N_ character messages. 
   * "Quiet Mode" - Suppress success messages when autoscrubbing - only show warnings/errors
   * "Include Reason Scrubber" - When autoscrubbing, remove Think blocks as well as unused swipes
