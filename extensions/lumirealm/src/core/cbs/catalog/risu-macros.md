# RisuAI CBS Macro Catalog

## Overview

Complete inventory of RisuAI's Curly-Brace Script (CBS) system macros, extracted from source code for implementation in a Lumiverse runtime shim.

**Total Macros: 173** (from `src/ts/cbs.ts` registerFunction calls)

## Macro Count by Category

| Category         | Count | Notes                                                                                       |
| ---------------- | ----- | ------------------------------------------------------------------------------------------- |
| other            | 67    | Miscellaneous: br, newline, codeblock, ruby, tex, reverse, risu, file, etc.                 |
| logic            | 18    | Boolean/comparison: and, or, not, equal, greater, less, contains, etc.                      |
| chat_context     | 12    | Chat state: previouscharchat, previoususerchat, userhistory, charhistory, lastmessage, etc. |
| escape_markup    | 10    | Escape/encoding: unicode, escape, hex, crypt, xor, display bracket variants, etc.           |
| arrays           | 10    | Array operations: arraypush, arraypop, arrayshift, arrayassert, arraysplice, etc.           |
| math             | 10    | Math: abs, ceil, floor, round, pow, min, max, sum, average, fixnum, etc.                    |
| character_fields | 5     | Character data: char, personality, description, scenario, exampledialogue                   |
| strings          | 5     | String ops: upper, lower, capitalize, trim, replace, length (alt implementations)           |
| control_flow     | 7     | Block macros (from cbs.ts): #if, #if_pure, #when, #pure, #puredisplay, #escape, #each       |
| display          | 8     | Asset/media: image, asset, emotion, audio, video, button, bg, bgm, etc.                     |
| time             | 5     | Time/date: time, date, unixtime, isotime, isodate, datetimeformat, messagedate, etc.        |
| random           | 4     | Randomization: random, roll, pick, dice, randint, rollp, hash                               |
| variables        | 7     | Variable ops: getvar, setvar, addvar, settempvar, tempvar, getglobalvar, etc.               |
| flow_control     | 1     | return, declare (explicit flow control)                                                     |
| identity         | 4     | Identity: user, char, trigger_id, role, model, axmodel                                      |

## Lumiverse Collision Analysis

**Total Collisions: 33 macros** with potential Lumiverse equivalents.

### Compatible Collisions (High Confidence Reuse)

Macros with matching names AND compatible argument shapes:

- `char`, `user` — identity macros (Lumiverse equivalents exist)
- `getvar`, `setvar` — variable access (Lumiverse has matching API)
- `random` — randomization (Lumiverse RNG compatible)
- `lower`, `upper`, `capitalize`, `trim`, `replace`, `split`, `join` — string ops (standard implementations)
- `abs`, `ceil`, `floor`, `round`, `min`, `max` — math (standard)

### Collision-Rename Required (Semantic Mismatch)

Macros with name collisions but differing argument shapes or behavior:

- `date` vs `datetimeformat` — Risu uses format string arg; Lumiverse may differ
- `time` — Risu time() returns local time; timestamp handling may differ
- `comment` — Risu comment ignored; Lumiverse may display
- `and`, `or`, `not` — Logic operators; verify truthiness semantics

### No Collision (Risu-Only)

~140 macros with no Lumiverse equivalent—require new implementation:

- All block constructs: `#if`, `#when`, `#pure`, `#each`, `#func`, `:else`
- All display asset macros: `bg`, `emotion`, `audio`, `video`, `button`, `asset`
- All escape markup macros: Unicode/hex/bracket variants, `xor`, `crypt`
- Array operations: `arraypush`, `arrayshift`, `arraysplice`, etc.
- Special: `previouscharchat`, `previoususerchat`, `charhistory`, `userhistory`

## Hard Macros (Non-Trivial Implementation)

These require careful, spec-accurate implementation to avoid silent bugs:

### Control Flow (7)

- `#if`, `#if_pure` — Conditional blocks with whitespace handling modes
- `#when` — Advanced conditional with stack-based operator evaluation (and/or/not/is/isnot/var/toggle/vis/visnot/tis/tisnot/>/</>=/<=)
- `#pure`, `#puredisplay` — Disable all CBS parsing in block
- `#code` — Normalize whitespace, handle Unicode escape sequences (\uXXXX, \n, \t, etc.)
- `#escape` — HTML escape content (with optional ::keep mode)
- `#each` — Loop construct with optional ::keep mode for whitespace
- `#func` — Function definition
- `:else` — Else clause for #when blocks

### RNG-Dependent (4)

- `random` — True random (0-1) or from array/comma-delimited list
- `roll` — Dice notation (XdY); must generate new random per call
- `pick` — Like random but **hash-deterministic** based on chatID+charID
- `rollp` — Like roll but hash-deterministic

**Action:** Verify Lumiverse RNG hook; replicate Risu's hash-based determinism for pick/rollp.

### Message/History Access (3 - reads chatState)

- `userhistory`, `charhistory` — Returns JSON array of message objects
- `previouscharchat`, `previoususerchat` — Searches backwards for last char/user message
- Must reconstruct from RisuAI message store; Lumiverse may have different indexing

### Deprecated Macros (to mark for removal)

- `#if` (use `#when`)
- `#if_pure` (use `#pure` + `#when`)
- `#when` (deprecated but still works; syntax complex)
- `#pure`, `#puredisplay` — Legacy; may consolidate

### Legacy/Risky Edge Cases

- `calc` — Evaluates math expressions with parentheses; no explicit security check visible
- `metadata::language` — Returns system language; fragile to environment
- `crypt` (Caesar cipher) — Symmetric with default shift (32768 = identity cipher for obfuscation, not security)
- `xor`, `xordecrypt` — XOR + base64 encoding; reversible but simple

## Sanity Checks Done

1. **Macro count:** 173 extracted from `registerFunction({` calls in cbs.ts (lines 146-2497)
2. **Block constructs:** 9 block types identified in `parser.svelte.ts` (lines 1137-1409), mostly not in matcher registry
3. **Aliasing:** ~45 macros have aliases; e.g., `char` has alias `bot`
4. **Deprecated flag:** 3 macros marked deprecated in source
5. **Collision check:** Lumiverse set of ~185 macros; 33 name collisions found (18% overlap)

## Next Steps

1. **Verify argument shapes** for each collision candidate (compare Risu `Usage::` string with Lumiverse signature)
2. **Implement hard macros** first: control flow (#if, #when, #pure, etc.), then RNG (pick/rollp), then message access
3. **Stub remaining macros** as "NOT YET IMPLEMENTED" to prevent silent failures
4. **Test determinism** for pick/rollp against RisuAI runtime behavior
5. **Document unsafe operations** (calc expression eval, crypt for obfuscation-only, metadata env access)

## File Structure

```
LumiRealm/packages/core/src/cbs/catalog/
├── risu-macros.json          # Full catalog with metadata (76 KB)
└── risu-macros.md            # This file
```

## Schema (risu-macros.json)

Each entry:

```json
{
  "name": "getvar",
  "aliases": [],
  "category": "variables",
  "argShape": "UNCERTAIN",
  "minArgs": 1,
  "maxArgs": 1,
  "pure": false,
  "readsState": ["localVars"],
  "writesState": [],
  "lumiverseCollision": {
    "name": "getvar",
    "compatible": true,
    "notes": "Needs verification"
  },
  "risuFile": "src/ts/cbs.ts",
  "risuLine": 792,
  "summary": "Gets the value of a persistent chat variable...",
  "notes": ""
}
```

---

**Source Repo:** RisuAI | **Target:** Lumiverse Runtime Shim
