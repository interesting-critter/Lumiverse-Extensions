# Customizing Vellum themes — fonts & text size

Vellum themes are designed so you can change the **text size** and the **fonts**
without editing the theme or rebuilding anything. Everything below is done from
inside Lumiverse after you import a `.lumitheme` pack.

There are two independent things you can change:

1. **Text size** — handled by Lumiverse's own font-size control. No CSS needed.
2. **Fonts** — handled by a one-line snippet you paste into the Custom CSS editor.

---

## 1. Changing the text size

Vellum no longer locks the font size, so Lumiverse's built-in control works on
every theme.

1. Open **Settings → Appearance** (the theme/appearance panel).
2. Find the **Font size** (font scale) control.
3. Drag it up or down. All message text — prose, names, everything — scales
   with it, live.

That's the whole thing. If you just want bigger or smaller text, stop here.

> Under the hood the theme sets `font-size: calc(16px * var(--lumiverse-font-scale, 1))`,
> so the app's font-size setting multiplies the base size instead of being
> ignored.

---

## 2. Changing the fonts

Each Vellum theme drives its fonts through a few CSS variables:

| Variable        | What it styles                          | Themes that use it |
|-----------------|-----------------------------------------|--------------------|
| `--vm-body`     | The message prose (the main reading text) | all |
| `--vm-serif`    | Character names + display headings       | all |
| `--vm-deco`     | Deco display font for names              | Gatsby |
| `--vm-ethereal` | Rounded label font for the "in scene" chip | Ember, Faewild |

To change a font, you override one of these variables. You do that in the
**Custom CSS editor**, not by editing the theme file.

### Step by step

1. Open **Settings → Appearance → Custom CSS** (the CSS editor / "Add custom CSS"
   panel).
2. Paste a snippet that sets the variable(s) you want (examples below).
3. Save / apply. The change is live and is kept alongside the theme.

### Example — change the reading font

```css
:root {
  --vm-body: 'Palatino', Georgia, serif;
}
```

### Example — change reading font AND names

```css
:root {
  --vm-body: 'Iowan Old Style', Georgia, serif;  /* prose */
  --vm-serif: 'Baskerville', Georgia, serif;      /* names / headings */
}
```

### Example — a clean sans-serif everywhere

```css
:root {
  --vm-body: 'Inter', 'Segoe UI', sans-serif;
  --vm-serif: 'Inter', 'Segoe UI', sans-serif;
}
```

Always keep a fallback after your font (e.g. `, Georgia, serif`) so text still
renders if the font isn't found.

---

## Important: which fonts you can use

Lumiverse strips external font links from custom CSS for safety. That changes
what "a font" can be:

**You CAN use any font already installed on your device.**
Just name it — `'Baskerville'`, `'Palatino'`, `'Georgia'`, `'Iowan Old Style'`,
`'Times New Roman'`, `'Inter'` if you have it, etc. This works instantly and
adds nothing to load. This is the recommended way.

**You CANNOT paste a Google Fonts (or any web) link.**
Lines like `@import url('https://fonts.googleapis.com/...')` or
`src: url(https://...)` are removed by the app's CSS sanitizer, so the font
silently won't load and you'll get the fallback. Don't rely on web URLs.

**You CAN add your own font file — but it has to be an asset, not a link.**
If you want a font that isn't installed on your device, you add the font file
itself to the theme, then point the CSS variable at it:

1. Open the **Theme Assets** panel (in the same Custom CSS / theme area).
2. **Upload** your font file (`.woff2`, `.woff`, `.ttf`, or `.otf`).
3. The panel gives you a **relative path** for the uploaded file, e.g.
   `assets/MyFont.woff2`. Use that path — not a web URL — in an `@font-face`,
   then point a Vellum variable at the new family:

   ```css
   @font-face {
     font-family: 'My Font';
     src: url(assets/MyFont.woff2) format('woff2');
     font-display: swap;
   }

   :root {
     --vm-body: 'My Font', Georgia, serif;
   }
   ```

   Lumiverse rewrites that relative `assets/...` path to the correct internal
   URL at runtime, and — because the font is a real bundled asset — it keeps
   working even when you export and share the theme.

> This is exactly how the built-in Vellum fonts are shipped: the font files
> travel inside the `.lumitheme` pack as assets, and the CSS references them by
> relative path. Uploading your own asset is the supported way to add a brand
> new font.

---

## Quick reference

| I want to…                    | Do this |
|-------------------------------|---------|
| Make all text bigger/smaller  | Settings → Appearance → Font size slider |
| Use a font I already have     | Custom CSS: `:root{ --vm-body: 'FontName', serif; }` |
| Change the name/heading font  | Custom CSS: `:root{ --vm-serif: 'FontName', serif; }` |
| Add a brand-new font file     | Theme Assets → upload → `@font-face { src: url(assets/…) }` then set `--vm-body` |
| Use a Google Fonts link       | Not supported — install the font, or upload it as an asset instead |
