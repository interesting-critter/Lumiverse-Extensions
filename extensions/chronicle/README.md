<!-- Mirror of upstream documentation. Everything below the metadata table is the original file, unmodified. -->

| Field | Value |
| --- | --- |
| Extension | Chronicle |
| Source repository | https://github.com/j-dandelion/Lumiverse-Chronicle |
| Original link (from Lumiverse-Extensions README) | https://github.com/j-dandelion/Lumiverse-Chronicle |
| Upstream path | `README.md` |
| Retrieved | 2026-09-29 @ `main` (`929e022`) |

---

# 📖 Chronicle

Similar to the MemoryBooks extension for SillyTavern, this extension provides a quick and easy way to turn selected messages into a customizable summary or memory. The results can then be saved as a lorebook entry.

Since you can use lorebooks lots of different ways, Chronicle is useful for all sorts of memory and context management techniques. 

## What sets it apart

> :gear: It's simple and lightweight. It doesn't automate a lot of stuff across different systems. It's just one small handy tool you can fit into your own workflow.

> :closed_book: Lumiverse's prompt presets and lorebook system already give you all the UI and tools you need to choose *when*, *where*, and *how* your summaries/memories are shown to the LLM. Chronicle's only concerned with helping you generate and save them from within chat, structured the way you like.

> :brain: You can use Chronicle standalone or alongside the default Lumiverse Summarize, Memory Cortex, and extensions like Lore Recall. They all do different things, and can harmonize pretty well with the right setup.

## How to use

There's a **Select Messages** button built into Lumiverse at the top-right of chat. It opens a message selection bar. 

On PC, you can use shift + click to select a set of messages. On mobile, that behavior is active by default: just tap two messages and it selects everything between.

Chronicle adds a **Summarize** button to the message selection bar and works with selected messages. It opens a pop-up menu with settings/presets and a button to start generating. 

Everything else is inside that menu!

## Canvas integration

If you have the [Canvas extension](https://github.com/j-dandelion/Lumiverse-Canvas) installed and enabled alongside Chronicle, you can also type **`/summarize`** in the chat input and press Enter — same behavior as clicking the Summarize button.

- Operates on the currently selected messages.
- Toasts an error if nothing is selected ("No messages selected. Enter select mode and pick messages first.") instead of silently no-op'ing.
- Toasts an error if a summary is already generating.

No setup required — the registration happens automatically when both extensions are active. If you enable Canvas mid-session (after Chronicle is already running), toggle Chronicle off and on to re-register the command.

## Screenshots

<div align="center">                                                                                                                      
  <img width="780" alt="Summarize modal with message selection"                                                                           
src="https://github.com/user-attachments/assets/b6478e1b-7c1f-4f17-9501-d7fe20ef0bfc" />       
</div>

<div align="center" style="display: flex; justify-content: center; gap: 10px;">                                                       
  <img width="260" alt="Prompt preset editor" src="https://github.com/user-attachments/assets/60ca1457-8a0f-4ed9-b7ec-e0b001f74afe" />
  <img width="260" alt="Lorebook and settings management"                                                                             
src="https://github.com/user-attachments/assets/5db61426-496a-4962-8705-cf03c99e1c87" />                                              
  <img width="260" alt="Generation Preview" src="https://github.com/user-attachments/assets/466f63d0-5527-481c-af7a-0c144e920b57" />                       
</div>                                                                                                                                

<br/>

## Features included so far

- **Auto-hide**: After summary is finalized, automatically hide summarized messages and/or all previous messages from context. (Applies to database, no need to scroll up and load messages)
- **Preserve recent messages**: Choose a number of recent messages to keep visible when auto-hide is used. (Helps with coherence)
- **Generation preview**: See and edit results before saving to lorebook.
- **Generate in background**: Use Lumiverse freely while summary is generating.
- **Recent memories in context**: Choose to include previous summaries/memories from the lorebook in context when generating.
- **Theme consistent**: All UI is integrated with Lumiverse's built-in theming and visual settings.
- **Connection profiles**: Choose a different provider/model to use for summaries.
- **Parameter controls**: Control temperature, top p, top k, and max tokens.
- **Prompt presets**: Save different summarization prompts and switch between them.
- **Lorebook settings presets**: Save sets of lorebook settings and switch between them.

##  Planned features

- **Heads up**: Choose a number, and when that many messages are visible in context/unsummarized, you get a small notification reminder to make a memory.
- **Quick scroll**: Button on the message select bar that brings you right to the first unsummarized message.
- **Scenes into arcs**: An easy way to combine scenes (multiple entries) into an arc (one entry).
- **In-depth guide**: A guide (served on the side) that explains a few of my favorite memory + context management setups.

## FYI

> ⚠️ Vibecoded with high-end models, a custom harness, and a careful workflow. But still, this is amateur hour. You know how it goes.

> 🛠️ Any reported bugs should be fixed within a day or two. Requests, suggestions, and criticism are very much welcome.
