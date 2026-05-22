# Strength Identity — Claude Notes

## Orange top-line bleed (Focus Mode only)

The orange/red line at the very top of the screen occurs **only in Focus Mode** (`#siFocusEngine`). Do NOT change global Home styles.

### Root cause
`.si-focus-accent-bar` — `position:absolute;top:0;height:3px;background:#ff6b00` — sits at literal `top:0` of `#siFocusEngine` which is `position:fixed;inset:0`, so it bleeds into the iOS safe-area/notch region.

### Fix applied
- Moved accent bar to `top:env(safe-area-inset-top,0px)` so it sits below the notch.
- Added overflow:hidden guard on `.si-focus-engine`, `.si-focus-panel*`, and `::before`/`::after` safe-area anchoring.
- All rules scoped to `.si-focus-*` selectors only — zero impact on Home screen.

### Scoping rule
When fixing visual issues in Focus Mode, scope ALL CSS changes to:
- `.si-focus-engine`
- `.si-focus-panel`
- `.si-focus-*` child selectors
- `.focus-screen`, `.focus-mode`, `.session-screen`, `.focus-session-bg`
Never touch global selectors (`.home-*`, `body`, `:root`, etc.) for Focus Mode bugs.

### Elements to audit for top bleed
- Focus progress bar / accent bar
- Top border on any panel
- `box-shadow` on fixed/absolute elements
- Orange glow overlay or radial-gradient
- `::before` / `::after` pseudo-elements
- Fixed background layer
- Safe-area background fill

### Canonical CSS fix template
```css
/* Containers — clip and darken */
.focus-screen,
.focus-mode,
.session-screen,
.si-focus-engine {
  background-color: #000 !important;
  overflow: hidden !important;
}

/* Pseudo-elements — respect safe-area notch */
.focus-screen::before, .focus-screen::after,
.focus-mode::before,   .focus-mode::after,
.session-screen::before, .session-screen::after,
.si-focus-engine::before, .si-focus-engine::after {
  top: env(safe-area-inset-top) !important;
}
```

If any CTA/button glow extends upward through a fixed or absolute overlay, add `overflow:hidden` on its nearest positioned ancestor (`.si-focus-bottom`, `.si-focus-controls`, etc.).

---

## Architecture

- **Single-file PWA**: `index.html` (~9200+ lines). All CSS, HTML, JS in one file.
- **Capacitor iOS deploy**: `cp index.html www/index.html` → `npx cap sync ios` → Xcode clean build.
- **Active workout engine**: `#siFocusEngine` (z-index:700). `openFocusMode()` → `siFocusOpen()`.
- **Legacy dead code**: `#focusMode` / `fm-` prefix components — never routed to, ignore.
- **Rest timer overlay**: `#siFocusRestOverlay` — `position:absolute;inset:0;z-index:200` inside `#siFocusEngine`. Shown/hidden explicitly in `_sfStartRest`, `siFocusSkipRest`, `siFocusSetRest`.
- **Tiny orange top-right countdown**: `.si-focus-dur` (workout duration, styled orange by premium CSS) — NOT the rest countdown.
