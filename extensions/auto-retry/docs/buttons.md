# Buttons it clicks

Lumiverse has no way for an extension to regenerate a reply directly. So a retry presses your own swipe or regenerate button on the screen. The built-in list finds these on current versions of Lumiverse, but a future update could change them.

## The three buttons

There are three button settings, in the order a retry uses them:

- **next / swipe** adds a new reroll. A retry tries this first.
- **regenerate** redoes the reply in place. This is the backup.
- **stop** stops a reply that has frozen.

How a retry finds the button:

- Your own selectors are tried first, then the built-in list. So a list you saved in an older version can never hide a button the built-in list would find.
- The built-in list starts with the mark Lumiverse puts on its own Regenerate button. It keeps working if the button's title or language changes.
- It presses the button on the newest message.
- A button that is hidden or greyed out is skipped, because pressing it would do nothing and waste a retry.
- If no button is on screen yet, it keeps looking for a couple of seconds. Lumiverse sometimes shows its buttons a moment after a reply ends.

## Swipe or regenerate

By default, a retry presses the **next / swipe** button. This adds a new reroll and keeps the old ones, so if a retry should not have happened, you can swipe back to the reply it replaced. This is **Retry by adding a new reroll**, at the end of "How it retries" in the settings.

Turn it off, and a retry presses **regenerate** instead. This redoes the reply in place. On some versions it also clears the other rerolls on that message, so a reply it replaces is gone. It is a little faster, and the one to use if your version has no swipe button.

- Whichever button the switch prefers, the other is the backup.
- This applies to every reason for a retry, including empty replies and errors.
- If a retry has to use the backup, the log says so.
- If a press lands but no reply starts, it presses the other button once before giving up on that try. This happens when a swipe button moves between existing rerolls instead of making a new one.

## Setting a button without writing a selector

Each button setting has a **Pick it for me** button next to **Test**.

1. Press **Pick it for me**. The settings panel hides.
2. Press and hold the real button in Lumiverse. The selector is filled in for you.

- The press is blocked, so picking your stop or regenerate button does not also press it.
- A short press works as normal. Use this to reach a button that only appears while a reply is running: send a message, then hold the stop button.
- Text selection is turned off while picking, because a long press also starts a selection.
- Press **Cancel**, or Esc on a keyboard, to stop picking.

It builds the selector from things that survive Lumiverse updates: `aria-label`, `title` and `data-` attributes. It never uses class names, because Lumiverse changes them every release. If the element has nothing it can use, it says so instead of saving something that will break. Holding the button itself, not the icon inside it, usually fixes that.

## Regeneration Feedback

Lumiverse has a **Regeneration Feedback** option. With it on, pressing regenerate opens a box asking for guidance for the next attempt, and the reply only starts once a button in the box is pressed.

When a retry opens the box, Auto Retry presses **Skip**, which regenerates without guidance. The box is hidden while it does this, so you should not see it.

- **A box you opened yourself is left alone.** Auto Retry only acts right after its own press, so your own regenerate still opens the box and waits for you.
- If you tap anything, or press stop, just before it skips, it stops and leaves the box alone.
- It only ever presses **Skip**, never **Cancel**, so a draft you typed in the box is never sent.
- If it cannot close the box, it shows it again straight away. The hidden box never blocks your taps.

You do not need to change any setting for this.

## Extra dialog buttons it may press

This is behind a switch, **My dialog's button says something else**, which is off by default. Almost nobody needs it.

Turn it on only if the Regeneration Feedback box stays on screen after a retry opens it. That means Auto Retry did not recognise its button. It already knows `Skip`, `Regenerate`, `Confirm`, `Proceed`, `Submit` and `OK`. If your button says something else, for example in another language, add that word here.

- Type the button's text exactly as shown, one per line. Capitals do not matter. **Expand** opens a bigger editor.

  ```
  Omitir
  Regenerar
  ```

- Your words are tried before the built-in list.
- While the switch is off, the box is not read. Your words are kept, so you can turn it back on later.
- It only presses buttons in a box that opened right after a retry. Adding `Continue` here will not press a Continue button on your toolbar.

## Writing selectors by hand

Each button setting takes CSS selectors, the kind you would pass to `document.querySelector`.

- Separate several with commas. They are tried left to right, and the first that finds a button you can press is used.
- Put the most exact ones first, like `data-action` or `data-testid`, and broader ones last, like `aria-label` or `title`.
- A comma inside brackets, parentheses or quotes is part of that selector. So `:is(a, b)` and `[aria-label="Next, swipe"]` each count as one.
- Do not use class names. Lumiverse changes them with every release.

If a retry happens (the pop-up shows) but nothing regenerates, fix the selector:

1. Open your browser's developer tools (F12) with an AI message on screen.
2. Right-click the regenerate button and choose **Inspect**.
3. Find a stable attribute on it: a `data-` attribute, `aria-label` or `title`. Write a selector that matches it.
4. Paste it into the regenerate setting, and press **Test** with an AI message on screen. Save when it says it matches.

"No match" does not always mean the selector is wrong. A button only exists while it is showing. The **stop** button, for example, only appears while a reply is being written, so test it then.

To put the built-in selectors back, open **Reset…** at the bottom of the panel, tick only **Button selectors**, and press **Save**. Every other setting is left alone. See [Resetting](settings.md#resetting).

---

[Back to the README](../README.md)
