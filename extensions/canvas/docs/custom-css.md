# Custom CSS & Theming

How to make your own CSS look good with **Canvas drawers** (the main drawer in taskbar mode, and the second drawer) without fighting two different UIs.

---

## The short version

| What you want | What to do |
|---------------|------------|
| Change colors / overall theme | Override **CSS variables** (best option — works almost everywhere) |
| Style both Canvas drawers the same | Use the class **`.sidebar-ux-shell`** |
| Style tab buttons when the drawer is closed (taskbar “strip”) | Also target **`.sidebar-ux-tab-list-pin-host`** |
| Style the normal Lumiverse main drawer (taskbar mode **off**) | Your old host selectors may still work; they do **not** style Canvas drawers |

**Important:** With **taskbar mode on**, the main drawer you see is built by Canvas. CSS written only for Lumiverse’s original drawer often **stops applying** to that chrome. Variables still work; class-based host hacks usually need the Canvas names below.

---

## 1. Easiest: change theme colors (recommended)

Lumiverse and Canvas share named colors called **CSS variables**. If you change those, both the original UI and Canvas drawers pick them up.

Paste something like this into your custom CSS:

```css
:root {
  --lumiverse-primary: #8b5cf6;   /* accent / active tab color */
  --lumiverse-bg-deep: #12121a;   /* drawer background */
  --lumiverse-text: #ececf1;      /* main text */
  --lumiverse-text-muted: #a1a1aa; /* quieter text */
}
```

### Useful color names

| Variable | What it mainly affects |
|----------|------------------------|
| `--lumiverse-primary` | Accent, active tab highlight |
| `--lumiverse-primary-015` (and similar `008`…`050`) | Soft fills, borders, hovers |
| `--lumiverse-bg-deep` | Drawer background |
| `--lumiverse-bg` / `--lumiverse-bg-hover` | Surfaces and hover |
| `--lumiverse-text` | Main text |
| `--lumiverse-text-muted` / `--lumiverse-text-dim` | Labels, quieter icons |
| `--lumiverse-border-hover` | Borders on the open/close edge control |
| `--lumiverse-shadow-xl` | Drawer shadow |

**Tip:** Prefer variables for color. You avoid most “it works on one drawer but not the other” problems.

---

## 2. Why two drawers can look different

Think of it this way:

1. **Taskbar mode off** — you see Lumiverse’s own main drawer (classes with long random-looking names).
2. **Taskbar mode on** — Canvas hides that chrome and shows its own drawer UI, shared with the **second drawer**.

So:

- Rules aimed only at Lumiverse classes → fine when taskbar is **off**, often **miss** Canvas chrome when taskbar is **on**.
- Rules using Canvas class names (below) → style **main-mirror + second drawer**.
- Color variables → work in **both** situations.

The **stuff inside** a tab (Settings, chat tools, extensions) is often still the original UI. Your CSS for that content may still work. It’s the **frame around** the tab (strip, header, open/close button) that Canvas owns.

---

## 3. Style both Canvas drawers with one name

Canvas puts a shared class on both drawers:

**`.sidebar-ux-shell`**

Example — same border radius on both open drawers:

```css
.sidebar-ux-shell .sidebar-ux-drawer {
  /* your styles here */
}
```

### Tab buttons (including when the drawer is closed)

With taskbar mode, tab icons can sit on a **pin strip** at the screen edge even when the panel is closed. That strip is **not always inside** `.sidebar-ux-shell`, so list it too:

```css
/* Tab buttons on open drawers AND on the closed pin strip */
.sidebar-ux-shell .sidebar-ux-tab-list button,
.sidebar-ux-tab-list-pin-host .sidebar-ux-tab-list button {
  border-radius: 10px;
}
```

### Only one drawer

```css
/* Second drawer only */
.sidebar-ux-secondary-wrapper .sidebar-ux-panel-header {
  /* … */
}

/* Main drawer (taskbar / main-mirror) only */
.sidebar-ux-main-mirror-wrapper .sidebar-ux-panel-header {
  /* … */
}
```

Or by owner:

```css
[data-drawer-owner="secondary"] .sidebar-ux-panel-header { /* … */ }
[data-drawer-owner="main"] .sidebar-ux-panel-header { /* … */ }
```

---

## 4. Common pieces (simple map)

You only need these if variables are not enough.

| Class / name | Plain English |
|--------------|---------------|
| `.sidebar-ux-shell` | **Both** Canvas drawers (prefer this) |
| `.sidebar-ux-main-mirror-wrapper` | Main drawer only (taskbar mode) |
| `.sidebar-ux-secondary-wrapper` | Second drawer only |
| `.sidebar-ux-drawer` | The drawer panel body |
| `.sidebar-ux-drawer-tab` | Edge button that opens/closes the drawer |
| `.sidebar-ux-tab-list` | Column (or strip) of tab buttons |
| `.sidebar-ux-panel-header` | Title bar at the top of the panel |
| `.sidebar-ux-panel-content` | Where tab content sits |
| `.sidebar-ux-tab-active` | The currently selected tab button |
| `.sidebar-ux-tab-label` | Short name under a tab icon |
| `.sidebar-ux-tab-list-pin-host` | Pin strip when the drawer is closed |

Left vs right edge: `.sidebar-ux-side-left` / `.sidebar-ux-side-right`.

---

## 5. If you use taskbar mode **and** keep it off sometimes

Color variables still cover both.

If you also target tab **buttons** by Lumiverse class names (for taskbar **off**), add Canvas names for taskbar **on**:

```css
/* Old-style Lumiverse main tabs (taskbar off) — names can change with app updates */
[class*="tabBtn"]:not(.sidebar-ux-main-tab-mirror-btn),

/* Canvas tabs (taskbar on / second drawer / pin strip) */
.sidebar-ux-shell .sidebar-ux-tab-list button,
.sidebar-ux-tab-list-pin-host .sidebar-ux-tab-list button {
  /* shared look */
}
```

If colors are all you need, skip this section.

---

## 6. Common gotchas

1. **“My CSS worked until I turned on taskbar mode.”**  
   You were styling Lumiverse’s drawer. Use variables and/or `.sidebar-ux-shell` (and the pin strip for closed tabs).

2. **“My rule works when the drawer is open but not when it’s closed.”**  
   Include `.sidebar-ux-tab-list-pin-host` for the pin strip.

3. **“Active tab color won’t change.”**  
   Canvas sometimes uses strong rules for the active tab. Prefer changing `--lumiverse-primary` (and related variables) instead of fighting the active class.

4. **“I styled the frame, but the content inside looks the same.”**  
   That’s expected: inside the panel is often still Lumiverse or an extension, not Canvas chrome.

5. **Canvas will not auto-translate your old host CSS** onto the new drawers. There is no “copy my old rules onto main-mirror” switch. Use variables + the names in this guide.

---

## 7. Optional: layout variables (advanced)

These change sizes/margins more than theme. Only touch them if you know you need them.

| Variable | Role |
|----------|------|
| `--sidebar-ux-secondary-w` | Second drawer width |
| `--sidebar-ux-main-mirror-w` | Main Canvas drawer width |
| `--sidebar-ux-chat-ml` / `--sidebar-ux-chat-mr` | Chat side margins (reflow) |

For how drawers are built, see [sidebar.md](sidebar.md). For chat margins, see [chat-reflow.md](chat-reflow.md).

---

## Quick checklist

1. Change colors with `--lumiverse-…` variables first.  
2. For both Canvas drawers: **`.sidebar-ux-shell`**.  
3. For tab buttons when closed: also **`.sidebar-ux-tab-list-pin-host`**.  
4. Don’t expect old Lumiverse-only class hacks to style taskbar-mode chrome.  
5. Prefer variables over complex selectors when something “won’t stick.”
