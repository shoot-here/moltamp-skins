# Moltamp Skin Specification v3.0

The single source of truth for building Moltamp skins.

> **Spec ↔ Validator contract:** This document is the spec. Two validators implement it and **must stay in lockstep**:
>
> - **App validator:** `moltamp/electron/skins/skin-validator.js` — runs at import time in the desktop app
> - **Site validator:** `moltamp-site/functions/api/_shared/skin-validator.ts` — runs at upload time on moltamp.com
>
> Any rule added or changed below must land in both validators in the same change. Any validator change must be reflected here. A skin that passes one surface and fails the other is a broken contract — if you find drift, fix it.

## Philosophy

Moltamp is the shell. Your skin is the experience.

Our job is to say **yes** to whatever you create. Every rule in this doc exists to protect user interaction — clicks, permissions, accessibility. Everything else is yours. Colors, animations, GIFs, glows, sprite sheets, data-driven visuals, reactive state machines — if CSS can do it, your skin can do it.

The rules below protect interaction, permissions, and licensing. Everything else is yours. Go wild.

## Layout Overview

Here's how every panel sits in the Moltamp window:

```
┌─────────────────────────────────────┐
│  TITLEBAR              [Skins] [⚙]  │
├─────────────────────────────────────┤
│  VIBES (.moltamp-vibes)             │  ← GIF art, overlays, mood
├────────┬────────────────┬───────────┤
│  LEFT  │   TERMINAL     │  RIGHT    │
│ Intel  │   Claude PTY   │  Signal   │
│ Panel  │  (.moltamp-    │  Panel    │
│        │   terminal)    │           │
├────────┴────────────────┴───────────┤
│  BOTTOM (.moltamp-panel-bottom)     │  ← Telemetry ticker
├─────────────────────────────────────┤
│  STATUSBAR                          │
└─────────────────────────────────────┘
```

- **Titlebar** — App controls: logo, zoom, panel toggles, settings gear. Style freely but don't break interactive elements.
- **Vibes** (`.moltamp-vibes`) — Your hero canvas. GIF artwork, animated overlays, mood-setting visuals. Backgrounds are injected via `skin.json` slots, not CSS. Use `::before`/`::after` for overlay effects.
- **Left Panel** (`.moltamp-panel-left`) — "Intel" panel. Hosts tabbed widgets like Events, Files, Agents, Git, and any custom tabs you define.
- **Terminal** (`.moltamp-terminal`) — The Claude PTY. Where the conversation happens. Style message bubbles, code blocks, and the input field.
- **Right Panel** (`.moltamp-panel-right`) — "Signal" panel. Hosts custom tabs with stats, gauges, media widgets, and companions.
- **Bottom Panel** (`.moltamp-panel-bottom`) — Telemetry ticker. A narrow bar for scrolling data, status readouts, or decorative tickers.
- **Statusbar** (`.moltamp-statusbar`) — Skin selector, file path, layout controls. Keep it functional.

All panels are user-resizable and togglable. Your skin sets the defaults; the user decides what stays visible.

## Quick Start

```bash
# 1. Copy the template
cp -r skins/_template skins/my-skin

# 2. Edit the manifest
$EDITOR skins/my-skin/skin.json

# 3. Design your theme
$EDITOR skins/my-skin/theme.css

# 4. Add art (optional)
cp ~/Downloads/cool-banner.gif skins/my-skin/assets/vibes.gif

# 5. Launch and select your skin
npm run electron:dev
```

Click the **Skins** button in the title bar, select your skin, and iterate.

## Your First Skin

This walkthrough takes you from the template to a working skin in five minutes. You'll change colors, add a vibes GIF, and create a custom effect.

### Step 1: Copy the template

```bash
cp -r skins/_template skins/neon-mint
```

Open `skins/neon-mint/skin.json` and set your identity:

```json
{
  "id": "neon-mint",
  "name": "Neon Mint",
  "version": "1.0.0",
  "author": "Your Name",
  "description": "Cool mint tones with a neon glow.",
  "engine": "1.0"
}
```

### Step 2: Change the contract colors

Open `skins/neon-mint/theme.css`. Replace the `:root` block with your palette. Start with just three variables — background, accent, and border — and you'll immediately see a different skin:

```css
:root {
  /* Contract — the minimum viable skin */
  --t-foreground: #d0f0e0;
  --t-background: #0a1210;
  --c-chrome-bg: #0a1210;
  --c-chrome-accent: #00e68a;
  --c-chrome-border: #1a3a2e;
  --c-chrome-text: #88b8a0;
  --c-chrome-dim: #4a6a5a;
  --c-chrome-hover: #152a22;
}
```

Save, switch to your skin in Moltamp, and you'll see a mint-green workspace. That's it for a basic color theme.

### Step 3: Add a vibes GIF

Drop a GIF file into `skins/neon-mint/assets/` — call it `vibes.gif`. Then tell `skin.json` about it:

```json
{
  "id": "neon-mint",
  "name": "Neon Mint",
  "version": "1.0.0",
  "author": "Your Name",
  "description": "Cool mint tones with a neon glow.",
  "engine": "1.0",
  "vibes": {
    "slots": [
      { "widgetId": "gif", "gifSrc": "assets/vibes.gif", "gifFit": "cover" }
    ]
  }
}
```

Your GIF now fills the vibes panel. The user can resize it, swap it, or add more slots.

### Step 4: Add a custom effect

Back in `theme.css`, add an effect variable and wire it to a glow:

```css
:root {
  /* ... your contract colors from Step 2 ... */

  /* Custom palette */
  --skin-glow-color: rgba(0, 230, 138, 0.3);

  /* Custom effect — auto-appears in Settings > Effects */
  --effect-mint-glow: 1;
}

/* Gated glow on the terminal */
.moltamp-terminal {
  box-shadow: inset 0 0 calc(15px * var(--effect-mint-glow)) var(--skin-glow-color);
}
```

The user now has a "Mint Glow" toggle in Settings > Effects with an intensity slider. At 0% (`--effect-mint-glow: 0`), the glow disappears. At 200% (`--effect-mint-glow: 2`), the blur radius doubles to 30px.

Add a label in `skin.json` for a nicer name:

```json
{
  "effects": {
    "mint-glow": { "label": "Mint Glow", "description": "Soft inset glow on the terminal" }
  }
}
```

### Step 5: Export and share

Click **Skins** in the title bar, then **Export** to save a `.zip`. Share it directly, or submit it to this repo as a PR (see [Submitting Your Skin](#submitting-your-skin)).

---

## The Rules

These guidelines keep skins safe and usable. The validator checks them automatically, and community skins need to pass before merging — but we'll help you get there.

### 1. All colors live in `:root`

Every color value — hex, rgb, rgba, hsl — should be defined as a CSS variable in `:root`. Outside of `:root`, reference colors with `var()` instead of hardcoding them.

```css
/* WRONG */
.moltamp-panel-left { background: #070b07; }

/* RIGHT */
:root { --skin-panel-bg: #070b07; }
.moltamp-panel-left { background: var(--skin-panel-bg); }
```

**Why:** Color overrides inject a `:root` block on top of your skin. If you hardcode `#070b07` in a selector, the override can't reach it. The user's customization breaks.

### 2. Contract variables come first

The 6 chrome variables and 21 terminal variables are the **contract**. Override them in `:root`. Everything else in your skin should derive from them or extend them with `--skin-*` prefixed variables.

### 3. Keep panels visible

Skins must not use `display: none` on any panel element (`.moltamp-vibes`, `.moltamp-panel-left`, `.moltamp-panel-right`, `.moltamp-panel-bottom`). All panels ship ON. Users toggle them via Settings > Layout. Skins can style panels freely — backgrounds, borders, overlays — but visibility is the user's choice.

### 4. No executable content

- No external URLs (`https:`, `http:`)
- No `@import`
- No `expression()`, `javascript:`, `behavior:`, `-moz-binding`
- No `data: text/*` URIs
- CSS only. Zero JS.

### 5. Avoid global hide patterns

Skin CSS that uses `visibility: hidden`, `opacity: 0`, or `pointer-events: none` on global selectors (`*`, `body > *`, `body *`) will trigger validator warnings. Be specific with your selectors — target the elements you actually want to style, not everything on the page.

### 6. No targeting Moltamp internal UI

Skins must not target Moltamp's internal license, billing, or permission UI. The validator hard-blocks any skin that does — you'll see a clear error on import. If you're not sure whether a class name belongs to Moltamp's internals or to the skinnable shell, stick to the documented `.moltamp-*` classes listed in this guide; everything else is off-limits.

### 7. No backgrounds on `.moltamp-vibes`

The vibes panel is always transparent. Visual content (GIFs, images) must come from **GIF widget slots** in `skin.json`, not CSS `background` or `background-image`. Moltamp strips vibes backgrounds at load time with `!important`.

### 8. Pseudo-elements need `pointer-events: none`

Include `pointer-events: none` on every `::before` and `::after` pseudo-element. Without it, overlays can interfere with right-click menus, drag handles, and widget interaction. Moltamp auto-fixes missed cases at load time, but declaring it explicitly avoids validator warnings.

```css
/* WRONG — blocks all clicks behind the overlay */
.moltamp-panel-right::before {
  content: '';
  position: absolute;
  inset: 0;
  background: var(--skin-overlay);
}

/* RIGHT */
.moltamp-panel-right::before {
  content: '';
  position: absolute;
  inset: 0;
  background: var(--skin-overlay);
  pointer-events: none;  /* always include this */
}
```

### 9. Assets stay in `assets/`

- Reference with `url('./assets/filename.ext')`
- Supported images: PNG, JPG, WebP, GIF, SVG, AVIF
- Supported fonts: WOFF2, WOFF, TTF, OTF
- Max 5MB per file, 20MB per skin
- No nested directories in `assets/`
- No path traversal (`../`)

### 10. All visual effects must be gated behind `--effect-*` variables

Every visual flourish must be controllable by the user through the Effects panel. The validator scans `theme.css` for **six families** of effect and warns on import when any of them aren't gated behind an `--effect-*` variable:

| Family | What it catches | Examples |
|---|---|---|
| **Motion** | CSS animations | `animation`, `animation-name`, `@keyframes` applications |
| **Glow** | Luminous shadows & drop-shadow | `box-shadow`/`text-shadow` with blur ≥ 12px on pseudo-elements, `box-shadow` with blur ≥ 20px on regular elements, `filter: drop-shadow()` with large blur |
| **Frost** | Blur / glass effects | `backdrop-filter: blur()`, `filter: blur()`, `backdrop-filter: saturate/brightness/contrast/hue-rotate` |
| **Overlay** | Decorative layers & texture | `mix-blend-mode`, `repeating-linear-gradient`, `repeating-radial-gradient`, animated `.gif`/`.webp`/`.apng` backgrounds |
| **Distortion** | Geometric warping | `transform: perspective/skew/rotate3d/matrix3d`, `mask`, `mask-image`, `-webkit-mask`, `clip-path` |
| **Color shift** | Decorative filter chains | `filter: brightness/saturate/contrast/hue-rotate/invert/sepia/grayscale` |

This means for every flaggable rule in your CSS:
- Declare an `--effect-*` variable in `:root`
- Gate the effect so that when the variable is `0`, the effect is invisible or neutral
- Name it descriptively (e.g. `--effect-frosted-glass`, not `--effect-fx-1`)

The validator runs on import and emits one warning per ungated family per selector. Users see these in a preview popup before the skin is committed to their library.

#### Effect Slider Range

All `--effect-*` variables use a numeric range of `0` to `2`:

| Variable Value | UI Display | Meaning |
|---------------|------------|---------|
| `0` | 0% (Off) | Effect is completely invisible |
| `1` | 100% (Default) | The skin author's intended intensity |
| `2` | 200% (Max Boost) | Double the author's default — for users who want more |

The skin author's value in `:root` is the default. When the user sets the slider in Settings > Effects, the variable interpolates between `0` and `2`. Design your default (`1`) as the intended look. Values above `1` let users push harder if they want.

This range is consistent across all effects — built-in and custom. When you see `--effect-*` anywhere in this document, the `0`-to-`2` range applies.

#### Gating patterns

**Pseudo-element effects** — use `opacity: var(--effect-*)` directly:

```css
/* WRONG — always-on animation, user can't disable */
.moltamp-vibes::after {
  animation: orbit-rotate 8s linear infinite;
  box-shadow: 0 0 12px var(--c-chrome-accent);
}

/* RIGHT — gated behind an effect variable */
:root {
  --effect-orbit: 1;
}
.moltamp-vibes::after {
  animation: orbit-rotate 8s linear infinite;
  box-shadow: 0 0 12px var(--c-chrome-accent);
  opacity: var(--effect-orbit);
}
```

**Box-shadow on elements** — use `calc()` to scale the blur/spread to zero:

```css
:root {
  --effect-heartbeat: 1;
}
@keyframes heartbeat {
  0%, 100% { box-shadow: inset 0 0 0 0 transparent; }
  50% { box-shadow: inset 0 0 calc(30px * var(--effect-heartbeat)) var(--skin-glow); }
}
```

**Filter effects on elements** — use `calc()` to collapse filter values to neutral:

```css
:root {
  --effect-vibes-pulse: 1;
}
@keyframes pulse {
  0%, 100% { filter: brightness(calc(1 - 0.3 * var(--effect-vibes-pulse))); }
  50%      { filter: brightness(calc(1 + 0.1 * var(--effect-vibes-pulse))); }
}
```

When `--effect-vibes-pulse` is `0`, brightness stays at `1` (no visible change).

**GIF/image backgrounds on panels** — use `calc()` on opacity:

```css
:root {
  --effect-panel-art: 1;
}
.moltamp-panel-left::before {
  background: url('./assets/art.gif') center / contain no-repeat;
  opacity: calc(0.25 * var(--effect-panel-art));
  pointer-events: none;
}
```

**Keyframes that use `opacity`** — if the animation itself varies opacity (e.g., a fade-in sweep), use `filter: opacity()` in the keyframes so the element-level `opacity` gate isn't overridden:

```css
.moltamp-panel-right::after {
  opacity: var(--effect-scan-sweep);  /* gate */
  animation: sweep 3s ease-in-out infinite;
}
@keyframes sweep {
  0%   { top: 0; filter: opacity(0.3); }   /* NOT opacity: 0.3 */
  50%  { filter: opacity(1); }
  100% { top: 100%; filter: opacity(0.3); }
}
```

**Frost / backdrop-filter** — multiply the blur radius by the effect variable. When the user sets the effect to `0`, blur becomes `0px` (no effect):

```css
:root {
  --effect-frosted-glass: 1;
}
.moltamp-titlebar,
.moltamp-panel-left,
.moltamp-panel-right {
  backdrop-filter: blur(calc(8px * var(--effect-frosted-glass)));
}
```

**Overlays / mix-blend-mode** — use `opacity` on the same element, since `mix-blend-mode` itself can't be disabled by a variable. When `opacity` is `0`, the blended layer disappears:

```css
:root {
  --effect-screen-blend: 1;
}
.moltamp-vibes::before {
  background: linear-gradient(135deg, rgba(255, 95, 135, 0.16), transparent);
  mix-blend-mode: screen;
  opacity: var(--effect-screen-blend);
}
```

**Overlays / repeating gradients** — treat them like any other pseudo-element overlay: gate on `opacity`:

```css
:root {
  --effect-scanlines: 1;
}
.moltamp-shell::after {
  content: "";
  position: absolute;
  inset: 0;
  pointer-events: none;
  background: repeating-linear-gradient(
    0deg,
    transparent 0,
    transparent 2px,
    rgba(0, 0, 0, 0.05) 3px
  );
  opacity: var(--effect-scanlines);
}
```

**Distortion / transform** — multiply the distortion magnitude by the effect variable. For `perspective()`, divide by the variable so `0` collapses to no perspective:

```css
:root {
  --effect-crt-curve: 1;
}
.moltamp-terminal {
  /* At effect=0, skew is 0deg; at effect=1, skew is 0.5deg */
  transform: skewX(calc(0.5deg * var(--effect-crt-curve)));
}
```

**Distortion / mask & clip-path** — gate the whole element on `opacity`, or provide a `none` fallback behind a CSS custom property:

```css
:root {
  --effect-vignette: 1;
}
.moltamp-vibes {
  -webkit-mask: radial-gradient(ellipse at center, black 60%, transparent 100%);
          mask: radial-gradient(ellipse at center, black 60%, transparent 100%);
  opacity: calc(0.4 + 0.6 * var(--effect-vignette));
}
```

**Color shift / filter chains** — for any `filter: hue-rotate / saturate / contrast / brightness / invert / sepia / grayscale`, collapse to the identity value (`0deg`, `1`, etc.) when the effect is `0`:

```css
:root {
  --effect-hue-cycle: 1;
}
[data-activity="high"] .moltamp-vibes {
  filter:
    hue-rotate(calc(15deg * var(--effect-hue-cycle)))
    saturate(calc(1 + 0.3 * var(--effect-hue-cycle)));
}
```

#### What doesn't need gating

- Static styling: backgrounds, borders, gradients (non-repeating), border-radius
- Subtle `transition` properties (hover states, active states)
- Layout: padding, margins, font sizes, colors
- Activity-based transitions (`[data-activity]`) that are purely visual feedback
- Drop shadows under ~12px blur on pseudo-elements, or under ~20px blur on regular elements — these read as panel chrome, not glow

**Why:** Users should have full control over their workspace. Every visual flourish should be disableable for accessibility and preference. The validator detects ungated effects on import and flags them in a preview popup before the skin is committed.

---

### 11. Don't declare dead toggles

Every `--effect-X` variable you declare in `:root` must be read by at least one CSS rule via `var(--effect-X)`. If you declare it and never use it, you've created a **dead toggle**: the Effects panel shows a switch for it (because the panel discovers toggles by scanning `:root`), but flipping the switch does nothing.

```css
/* WRONG — --effect-glow appears in the panel but has no wire */
:root {
  --effect-glow: 1;
}
/* ...no rule anywhere reads var(--effect-glow)... */
```

```css
/* RIGHT — declared AND wired */
:root {
  --effect-glow: 1;
}
.moltamp-terminal {
  box-shadow: inset 0 0 calc(18px * var(--effect-glow)) var(--c-chrome-accent);
}
```

The validator detects this pattern (declared in `:root`, never referenced in any non-`:root` rule) and warns on import. Two ways to fix a dead toggle:

1. **Remove it from `:root`** if you don't want the toggle at all.
2. **Add a rule that reads it** if you want the toggle to actually control something.

**Why:** Dead toggles create silently broken UX — users toggle them and nothing happens. The Effects panel's entire job is to give users control over what the skin is doing, so a toggle that does nothing is worse than no toggle at all. Always wire what you declare.

---

## Skin File Structure

```
skins/my-skin/
├── skin.json        ← Required: manifest
├── theme.css        ← Required: all styles (100KB max)
└── assets/          ← Optional: images, GIFs, fonts
    ├── vibes.gif    ← Top banner art
    ├── overlay.png  ← Texture overlays
    └── icon.svg     ← Anything you want
```

### skin.json

```json
{
  "id": "my-skin",
  "name": "My Skin",
  "version": "1.0.0",
  "author": "Your Name",
  "description": "A short description.",
  "engine": "1.0"
}
```

Required fields: `id`, `name`, `version`, `engine`. The `author` and `description` fields are recommended but not enforced by the validator.

`id` must match `^[a-zA-Z0-9_-]+$`.

#### Optional manifest fields

| Field | Type | Purpose |
|-------|------|---------|
| `preview` | `string` | Path to a preview/thumbnail image |
| `effects` | `object` | Labels and descriptions for custom effects (see [Custom Effects](#custom-effects)) |
| `vibes` | `object` | GIF widget slot configuration (see [The Vibes Panel](#the-vibes-panel)) |
| `layout` | `object` | Default panel layout shipped with the skin |

### Engine Versioning

The `engine` field in `skin.json` declares which version of the Moltamp Skin Engine your skin targets.

```json
{
  "engine": "1.0"
}
```

- **Always set `engine` to the version you built and tested against.** Currently, that's `"1.0"`.
- If Moltamp ships Engine v2.0 with breaking changes (new contract variables, changed class names, removed features), skins targeting `"1.0"` will still load but may show a compatibility warning to the user.
- Moltamp maintains **backward compatibility within major versions**. A skin built for Engine `1.0` will work with Engine `1.1`, `1.2`, etc. without changes. Breaking changes only happen at major version bumps.
- When a new major version ships, you can update your skin by adjusting for any breaking changes and bumping `engine` in your manifest.

---

## The Variable Contract

These variables are the API between Moltamp and your skin. Override them in `:root`.

### Terminal Colors (21 vars)

| Variable | Purpose | Default |
|----------|---------|---------|
| `--t-foreground` | Main text | `#e0e0e8` |
| `--t-background` | Terminal background | `#0a0a0f` |
| `--t-cursor` | Cursor color | `#d4a036` |
| `--t-cursor-accent` | Cursor text/bg when selected | `#0a0a0f` |
| `--t-selection` | Selection highlight | `rgba(212,160,54,0.25)` |
| `--t-black` | ANSI 0 | `#1a1a2e` |
| `--t-red` | ANSI 1 | `#d43636` |
| `--t-green` | ANSI 2 | `#36d480` |
| `--t-yellow` | ANSI 3 | `#d4a036` |
| `--t-blue` | ANSI 4 | `#3678d4` |
| `--t-magenta` | ANSI 5 | `#a036d4` |
| `--t-cyan` | ANSI 6 | `#36b5d4` |
| `--t-white` | ANSI 7 | `#e0e0e8` |
| `--t-bright-black` | ANSI 8 | `#3a3a52` |
| `--t-bright-red` | ANSI 9 | `#ff5555` |
| `--t-bright-green` | ANSI 10 | `#50fa7b` |
| `--t-bright-yellow` | ANSI 11 | `#f1c232` |
| `--t-bright-blue` | ANSI 12 | `#6299e6` |
| `--t-bright-magenta` | ANSI 13 | `#c678dd` |
| `--t-bright-cyan` | ANSI 14 | `#56d4ef` |
| `--t-bright-white` | ANSI 15 | `#ffffff` |

### Chrome Colors (6 vars)

| Variable | Purpose | Default |
|----------|---------|---------|
| `--c-chrome-bg` | Panel backgrounds | `#0a0a0f` |
| `--c-chrome-text` | Secondary/UI text | `#9898a8` |
| `--c-chrome-border` | Borders, dividers | `#1a1a2e` |
| `--c-chrome-accent` | Highlights, active states | `#d4a036` |
| `--c-chrome-dim` | Muted/disabled text | `#606070` |
| `--c-chrome-hover` | Hover state backgrounds | `#1e1e32` |

## Color Overrides

Moltamp lets users customize any skin's colors without editing CSS. This is the **Color Overrides** system.

### How it works

When a user adjusts colors in Settings > Skins > Colors, Moltamp injects a `:root` block **on top of** your skin's CSS. This override block contains only the variables the user changed — everything else falls through to your skin's original values.

```css
/* Your skin defines: */
:root {
  --c-chrome-accent: #00e68a;
  --skin-panel-bg: #0a1210;
}

/* User changes the accent to blue — Moltamp injects: */
:root {
  --c-chrome-accent: #4488ff;
}
```

Because CSS cascading applies, the user's `:root` wins over yours for any variable they've changed.

### Why Rule 1 matters

This is why all colors must be CSS variables in `:root`. If you hardcode a color value directly in a selector:

```css
/* This color can NEVER be overridden by the user */
.moltamp-panel-left { background: #0a1210; }
```

...the override system can't reach it. The user changes `--skin-panel-bg` in Settings, but nothing happens because your selector uses a raw hex value instead of `var(--skin-panel-bg)`.

### Per-skin persistence

Color overrides are stored per skin. If a user customizes the colors on "Neon Mint," those overrides only apply when "Neon Mint" is active. Switching to another skin uses that skin's defaults (or its own saved overrides).

Click **Reset to Defaults** in the color override panel to clear all customizations and return to the skin author's original palette.

### Effects — Built-in + Custom

Built-in effects that Moltamp knows about:

| Variable | Type | Purpose | Default |
|----------|------|---------|---------|
| `--effect-scanlines` | `0`–`2` | Horizontal line overlay | `0` |
| `--effect-glow` | `0`–`2` | Inset phosphor glow around terminal | `0` |
| `--effect-crt` | `0`–`2` | Barrel distortion + edge shadow | `0` |
| `--effect-vignette` | `0`–`2` | Darkened edge overlay | `0` |
| `--effect-noise` | `0`–`2` | Static noise / film grain | `0` |
| `--effect-flicker` | `0`–`2` | CRT refresh animation | `0` |
| `--effect-grid` | `0`–`2` | Background grid lines | `0` |
| `--effect-text-glow` | `0`–`2` | Phosphor blur on text | `0` |

Effect parameters:

| Variable | Type | Purpose | Default |
|----------|------|---------|---------|
| `--scanline-color` | color | Scanline overlay color | `rgba(255, 255, 255, 0.04)` |
| `--glow-intensity` | pixels | Glow blur radius | `20px` |
| `--c-glow` | color | Glow color | `rgba(212, 160, 54, 0.3)` |
| `--vignette-color` | color | Vignette overlay color | `rgba(0, 0, 0, 0.5)` |
| `--grid-color` | color | Grid line color | `rgba(255, 255, 255, 0.03)` |
| `--grid-size` | pixels | Grid cell size | `20px` |
| `--text-glow-color` | color | Text glow color | `var(--c-chrome-accent)` |

#### Custom Effects

Declare any `--effect-*` variable in `:root` and it **automatically appears** in the Effects panel as a user toggle with an intensity slider. No registration needed — Moltamp auto-discovers them.

```css
:root {
  --effect-radar: 1;        /* Shows as "Radar" toggle at 100% */
  --effect-heat-haze: 0.5;  /* Shows as "Heat Haze" at 50% */
}
```

The skin author's value in `:root` is the default (100% = `1`). Users can **dim below** that or **boost above** it — up to 200% (`2`). This means:
- `opacity: var(--effect-radar)` — clamps at 1 visually, but `calc()` expressions scale beyond
- `box-shadow: inset 0 0 calc(30px * var(--effect-heartbeat))` — at 200%, blur doubles to 60px
- `opacity: calc(0.25 * var(--effect-panel-art))` — at 200%, opacity becomes 0.5

Design your default (`1`) as the intended look. Values above 1 let users push harder if they want.

Auto-labeling: the variable name is converted from kebab-case to title case (`heat-haze` → "Heat Haze"). The 8 built-in effects have hardcoded labels (e.g., `scanlines` → "Scanlines", `noise` → "Film Grain").

For nicer labels, add metadata in `skin.json`. Values can be objects with `label`/`description`, or simply `true` for auto-labeling:

```json
{
  "effects": {
    "radar": { "label": "Sonar Radar", "description": "Rotating sweep overlay" },
    "heat-haze": true
  }
}
```

Then use the variable in your CSS to control the effect:

```css
.moltamp-panel-left::after {
  content: '';
  position: absolute;
  inset: 0;
  opacity: var(--effect-radar);
  animation: radar-sweep 4s linear infinite;
  pointer-events: none;
}
```

When the user toggles the effect off, `--effect-radar` becomes `0`, and `opacity: 0` hides it. When they adjust the slider, it scales between `0` and `2` (see [Effect Slider Range](#effect-slider-range)).

### Typography (6 vars)

| Variable | Purpose | Default |
|----------|---------|---------|
| `--font-terminal` | Terminal font stack | `'SF Mono', 'JetBrains Mono', monospace` |
| `--font-chrome` | Chrome/UI font stack | `'Inter', 'SF Pro Display', sans-serif` |
| `--font-size` | Base font size | `14px` |
| `--font-line-height` | Line height multiplier | `1.35` |
| `--terminal-scale` | Terminal font size multiplier (1 = 100%) | `1` |
| `--panel-scale` | Panel/chrome text multiplier (1 = 100%) | `1` |

**Custom fonts:** Bundle font files in `assets/` and declare with `@font-face`:

```css
@font-face {
  font-family: 'MyTermFont';
  src: url('./assets/my-font.woff2') format('woff2');
  font-weight: 400;
  font-style: normal;
}

:root {
  --font-terminal: 'MyTermFont', 'SF Mono', monospace;
}
```

Supported formats: `.woff2` (preferred), `.woff`, `.ttf`, `.otf`. Same size limits as images (5MB/file).

**Settings panel integration:** Fonts declared via `@font-face` in your skin's CSS are automatically discovered and appear as selectable options under "Bundled with skin" in **Settings > Skins > Fonts**. The panel parses your `@font-face` declarations and extracts the `font-family` names — no manifest registration needed.

Users can also override your skin's fonts with any installed system font without editing CSS. Their override persists alongside color and effect overrides and resets when they click "Reset to Defaults."

**Export:** When a user exports a skin with font overrides, the overrides are baked into the exported `theme.css` so the recipient gets the exact same fonts. Bundled font files in `assets/` are included automatically.

### Layout (3 vars)

| Variable | Default |
|----------|---------|
| `--radius` | `4px` |
| `--chrome-height` | `38px` |
| `--statusbar-height` | `24px` |

---

## Extending the Contract

Need colors the contract doesn't provide? Define **skin-local variables** in `:root` with the `--skin-` prefix.

```css
:root {
  /* Contract */
  --c-chrome-bg:       #080c08;
  --c-chrome-border:   #1a2a14;
  --c-chrome-accent:   #ffcc00;

  /* Skin-local — derived shades for visual depth */
  --skin-panel-bg:       #070b07;              /* darker than chrome-bg */
  --skin-border-subtle:  #1a2a14;              /* tinted border */
  --skin-overlay:        rgba(7, 11, 7, 0.15); /* content tint */
  --skin-glow-color:     rgba(255, 204, 0, 0.4);
  --skin-ticker-dim:     #665511;              /* muted accent */
  --skin-ticker-active:  #ccaa00;              /* bright accent */
}

/* Now use var() everywhere */
.moltamp-panel-left {
  background: var(--skin-panel-bg);
  border-right: 1px solid var(--skin-border-subtle);
}

.moltamp-vibes::before {
  content: '';
  position: absolute;
  inset: 0;
  background: var(--skin-overlay);
  pointer-events: none;
}
```

**Benefits:**
- All colors are discoverable in `:root` (one place to look)
- The skin builder can expose `--skin-*` variables as advanced controls
- Hand-editing is easy — change one value, it propagates
- Community members can fork your skin and tweak the palette without reading 300 lines of CSS

---

## Targetable Elements

### Layout Structure

| Class | What it is | Safe to restyle |
|-------|-----------|----------------|
| `.moltamp-shell` | Root container | Colors, borders. **Never** change `flex-direction`, `overflow`, `pointer-events`, or `position`. |
| `.moltamp-titlebar` | Top title bar — logo, zoom, panel toggles, settings | Background, borders, border-radius. See [Title Bar](#the-title-bar) section. |
| `.moltamp-vibes` | Top banner — your canvas | No `background` (enforced). Use GIF widget slots. `::before`/`::after` overlays OK. |
| `.moltamp-deck` | Holds left + terminal + right | Background, gap, padding. **Never** change `flex-direction` (breaks layout). |
| `.moltamp-panel-left` | Left panel (Intel tabs) | Background, borders, overlays. |
| `.moltamp-panel-right` | Right panel (Signal) | Background, borders, overlays. |
| `.moltamp-panel-bottom` | Bottom bar (Telemetry ticker) | Background, borders. |
| `.moltamp-statusbar` | Bottom status bar — skin selector, path, layout controls | Background, borders. **Never** set `overflow: hidden` (clips dropdown menus). |

### Terminal

| Class | What it is |
|-------|-----------|
| `.moltamp-terminal` | Terminal container (xterm.js lives here) |
| `.moltamp-message-user` | User message bubbles |
| `.moltamp-message-assistant` | Assistant message bubbles |
| `.moltamp-tool-call` | Tool call blocks |
| `.moltamp-thinking` | Thinking/reasoning blocks |
| `.moltamp-code-block` | Code blocks |
| `.moltamp-input` | Message input field |

### Stats & Gauges

| Class | Element |
|-------|---------|
| `.moltamp-model-badge` | Model name display |
| `.moltamp-context-gauge` | Context window gauge container |
| `.moltamp-context-track` | Gauge background track |
| `.moltamp-context-fill` | Gauge fill (width = context %) |
| `.moltamp-context-label` | CONTEXT / % text row |
| `.moltamp-context-value` | The percentage number |
| `.moltamp-token-chart` | Token usage bar chart |
| `.moltamp-stats` | Data readout container |
| `.moltamp-stat-row` | Individual stat row |
| `.moltamp-stat-label` | Label text in a stat row |
| `.moltamp-stat-value` | Value text in a stat row |

Stat rows get a second class based on their data key: `.moltamp-stat-cost`, `.moltamp-stat-time`, `.moltamp-stat-tokens-in`, `.moltamp-stat-tokens-out`, `.moltamp-stat-diff`, `.moltamp-stat-cache-read`, `.moltamp-stat-cache-write`, `.moltamp-stat-api-time`, `.moltamp-stat-agent`, `.moltamp-stat-rate-5h`, `.moltamp-stat-rate-7d`.

### Widgets

| Class | Element |
|-------|---------|
| `.moltamp-widget` | Base class on every widget container |
| `.moltamp-widget-{id}` | Per-widget class (e.g., `.moltamp-widget-clock`) |
| `.widget-resize-handle` | Bottom resize drag bar on widgets |
| `.moltamp-widget-music` | Now playing widget |
| `.moltamp-music-art` | Album art container |
| `.moltamp-music-controls` | Playback buttons |
| `.moltamp-music-progress` | Progress bar |
| `.moltamp-widget-visualizer` | Audio visualizer widget |
| `.moltamp-visualizer` | Visualizer inner container |
| `.moltamp-widget-equalizer` | EQ widget |
| `.moltamp-equalizer` | EQ inner container |
| `.moltamp-calendar` | Session activity calendar |

All widgets get `.moltamp-widget-{id}` on their container automatically.

### Tab Bars

| Class | Element |
|-------|---------|
| `.moltamp-tabbar` | Tab bar container |
| `.moltamp-tab` | Individual tab button |
| `.moltamp-tab-active` | Currently active tab |

### Vibes Internals

| Class | Element |
|-------|---------|
| `.moltamp-vibes-deck` | Slot container (position: absolute, inset: 0) |
| `.moltamp-vibes-slot` | Individual slot |
| `.moltamp-vibes-widget-fill` | Widget fill within a slot |

---

## The Title Bar

The `.moltamp-titlebar` contains the logo, zoom controls, panel toggle buttons, and the settings gear. Skins can restyle the title bar's background, borders, and shape freely — but **must not break the interactive elements inside**.

### What's safe

```css
.moltamp-titlebar {
  background: var(--skin-titlebar-bg);
  border-bottom: 2px solid var(--skin-titlebar-border);
  border-radius: 20px 20px 0 0;
}
```

### What breaks things

```css
/* WRONG — kills all buttons inside the title bar */
.moltamp-titlebar * {
  color: var(--skin-deep-space) !important;
  font-weight: 700 !important;
  text-transform: uppercase !important;
}

/* WRONG — clips the font scale popup (position: absolute child) */
.moltamp-titlebar {
  overflow: hidden;
}

/* WRONG — buttons become unclickable */
.moltamp-titlebar {
  pointer-events: none;
}
```

### How to style title bar text without breaking controls

Target the logo specifically, not `*`:

```css
/* RIGHT — styles the logo text only */
.moltamp-titlebar > span:first-child {
  color: var(--skin-logo-color);
  font-weight: 700;
  text-transform: uppercase;
  letter-spacing: 0.1em;
}
```

The logo text uses `--font-brand` (defaults to `'Syne', 'SF Pro Display', sans-serif`). You can override this in `:root`:

```css
:root {
  --font-brand: 'MyCustomFont', sans-serif;
}
```

### Title bar contents (for reference)

The title bar contains these interactive elements that must remain functional:
- **Logo** (text + optional image) — non-interactive
- **Home button** — returns to launch screen
- **Zoom controls** (-, o, +) — scale the UI
- **Font scale popup** (Aa) — panel/terminal scale sliders (renders as `position: absolute` dropdown)
- **Panel toggle pills** — show/hide panels
- **Settings gear** — opens settings window

---

## Properties That Break Things

These CSS properties will break Moltamp if applied carelessly. Avoid them on core layout containers.

### Never use on `.moltamp-shell` or `.moltamp-deck`

| Property | Why it breaks |
|----------|--------------|
| `flex-direction: column` (on `.moltamp-deck`) | Rotates the entire layout — panels stack vertically instead of side-by-side |
| `pointer-events: none` | Disables all click/drag interaction |
| `overflow: hidden` | Clips context menus, popovers, and dropdowns that render outside bounds |
| `position: relative` + `z-index` (on `.moltamp-deck`) | Creates stacking context that can hide fixed-position context menus |
| `transform` (any value on `.moltamp-shell`) | Creates a containing block that breaks `position: fixed` children |

### Never use with `*` (universal selector)

```css
/* ALL OF THESE WILL BREAK THE APP */
* { pointer-events: none; }
* { display: none; }
* { opacity: 0; }
* { position: fixed; }
* { overflow: hidden; }
* { transform: scale(0); }
```

If you need to style "everything" in a section, use specific selectors:

```css
/* WRONG */
.moltamp-panel-right * { color: var(--skin-color); }

/* RIGHT — targets text containers, not buttons/inputs */
.moltamp-panel-right .moltamp-stat-label,
.moltamp-panel-right .moltamp-stat-value {
  color: var(--skin-color);
}
```

### Overflow and menus

Context menus and popovers use `position: fixed` and render at the viewport level. If any ancestor has `overflow: hidden`, `transform`, or creates a new stacking context, these menus can be clipped or hidden.

**Safe rule:** Only set `overflow: hidden` on elements you fully control (like `::before`/`::after` pseudo-elements). Never on `.moltamp-shell`, `.moltamp-deck`, `.moltamp-titlebar`, `.moltamp-statusbar`, or any panel container.

---

## The Vibes Panel

The `.moltamp-vibes` div is your hero canvas.

**Backgrounds are not allowed** on `.moltamp-vibes`. The vibes panel is always transparent — all visual content (GIFs, images) must come from **GIF widget slots** defined in `skin.json`, not from CSS `background` or `background-image`. Moltamp enforces this with a `!important` override at load time.

```json
{
  "vibes": {
    "slots": [
      { "widgetId": "gif", "gifSrc": "assets/vibes.gif", "gifFit": "cover" }
    ]
  }
}
```

The vibes panel supports **multiple slots** — each slot is a horizontal section with its own widget. Slots are resizable by the user via drag handles, and width ratios persist between sessions.

Use `::before` and `::after` for overlays and effects — these are still allowed:

```css
.moltamp-vibes::before {
  content: '';
  position: absolute;
  inset: 0;
  background: var(--skin-overlay);
  mix-blend-mode: multiply;
  pointer-events: none;
}
```

---

## Layout Configuration

Skins can ship a `defaultLayout` in `skin.json` that sets up panels, widgets, tabs, and dimensions to match the skin's personality. This is applied when the user first selects your skin.

```json
{
  "defaultLayout": {
    "tabAssignments": {
      "intel": ["events", "files", "agents", "git"],
      "signal": ["custom-1", "custom-2", "custom-3", "custom-4"]
    },
    "tabVisibility": {
      "events": true, "files": true, "agents": true, "git": true,
      "custom-1": true, "custom-2": true, "custom-3": true, "custom-4": true
    },
    "customTabs": {
      "custom-1": { "label": "SENSORS", "widgets": ["system", "weather", "world-clock"] },
      "custom-2": { "label": "COMMS", "widgets": ["music", "visualizer", "equalizer"] },
      "custom-3": { "label": "CREW", "widgets": ["live2d"] },
      "custom-4": { "label": "TELEMETRY", "widgets": ["usage-model", "usage-context", "usage-tokens", "usage-session", "usage-rates", "usage-lifetime"] }
    },
    "dimensions": {
      "leftWidth": 260,
      "rightWidth": 180,
      "vibesHeight": 180,
      "bottomHeight": 26,
      "panelScale": 1,
      "terminalScale": 1
    },
    "panelVisibility": {
      "showVibes": true,
      "showLeft": true,
      "showRight": true,
      "showBottom": true
    }
  }
}
```

### Make it yours

Each layout is an opportunity to shape the experience. Consider how your tab names, widgets, and dimensions tell your skin's story.

- **Tab names should match your theme.** A Star Trek skin uses SENSORS, COMMS, HOLODECK. A medical HUD uses VITALS, IMAGING, DIAGNOSTICS. A synthwave skin uses ARCADE, MIXTAPE, AVATAR.
- **Widget selection should be intentional.** A data-heavy skin puts `system` and `weather` front and center. A minimal skin drops everything but `clock` and `session-timer`. A media skin leads with `visualizer` and `equalizer`.
- **Dimensions set the vibe.** Tall vibes (200px) for cinematic skins with big GIF banners. Narrow panels (160px) for terminal-focused skins. Wide panels (290px) for data-dense skins like LCARS.
- **Panel visibility defaults matter.** Some skins work best with vibes hidden by default. Some want maximum chrome. It's your call — the user can always override in Settings > Layout.

### Widget Catalog

These are the widgets available for your `customTabs`:

| ID | Name | Best for |
|----|------|----------|
| `clock` | Clock | Universal — every skin benefits |
| `session-timer` | Session Timer | Work-focused skins |
| `calendar` | Activity Calendar | Data/analytics skins |
| `world-clock` | World Clock | Multi-timezone / mission control skins |
| `event-calendar` | Event Calendar | Productivity skins |
| `gif` | GIF Viewer | Vibes / art-focused skins |
| `visualizer` | Audio Visualizer | Media / entertainment skins |
| `music` | Now Playing | Any skin with audio |
| `equalizer` | Equalizer | Audio-focused skins |
| `system` | System Monitor | Sci-fi / monitoring skins |
| `weather` | Weather | Ambient / dashboard skins |
| `notes` | Notes | Productivity / professional skins |
| `live2d` | Live2D Companion | Character / companion skins |
| `usage-model` | Model Badge | AI-focused skins |
| `usage-context` | Context Gauge | AI-focused skins |
| `usage-tokens` | Token Count | AI-focused skins |
| `usage-session` | Session Cost | AI-focused skins |
| `usage-rates` | Rate Limits | AI-focused skins |
| `usage-lifetime` | Lifetime Stats | AI-focused skins |

Mix and match. Put `system` next to `weather` for a monitoring dashboard. Put `visualizer` next to `live2d` for a media experience. There are no wrong combinations.

---

## Reactive Data Attributes

The shell sets data attributes on `.moltamp-shell` that change with Claude's state.

### Activity Level (`data-activity`)

| Value | Meaning | Use for |
|-------|---------|---------|
| `idle` | No PTY output | Dim, calm |
| `low` | Some output | Moderate energy |
| `high` | Fast output (streaming) | Bright, intense |

### Shell State (`data-shell-state`)

| Value | Meaning |
|-------|---------|
| `idle` | Waiting for input |
| `thinking` | Claude is reasoning |
| `streaming` | Claude is outputting text |
| `tool-use` | Claude is calling a tool |
| `permission` | Waiting for user permission |
| `error` | Something went wrong |
| `complete` | Response finished |

### Additional Attributes

| Attribute | Description |
|-----------|-------------|
| `data-session-count` | Number of active terminal sessions |
| `data-split-active` | Whether split view is active |

### Live CSS Custom Properties

Set on `.moltamp-shell` — use for data-driven visuals.

| Property | Type | Description |
|----------|------|-------------|
| `--data-context-pct` | 0-100 | Context window used % |
| `--data-cost-cents` | int | Session cost in cents |
| `--data-tokens-in` | int | Input tokens (thousands) |
| `--data-tokens-out` | int | Output tokens (thousands) |
| `--data-rate-5h` | 0-100 | 5-hour rate limit % |
| `--data-rate-7d` | 0-100 | 7-day rate limit % |
| `--data-duration-sec` | int | Session duration in seconds |
| `--data-lines-added` | int | Lines added this session |
| `--data-lines-removed` | int | Lines removed this session |
| `--data-cache-pct` | 0-100 | Cache hit ratio % |
| `--data-agents` | int | Active subagent count |
| `--data-git-changed` | int | Changed file count |
| `--data-events` | int | Event count |

### String Attributes

| Attribute | Description |
|-----------|-------------|
| `data-model` | Current model name |
| `data-agent-name` | Active agent display name |
| `data-git-branch` | Current git branch |

**Example: Style by model**
```css
[data-model*="Opus"] .moltamp-model-badge { color: var(--t-magenta); }
[data-model*="gpt"] .moltamp-model-badge { color: var(--t-green); }
[data-model*="Gemini"] .moltamp-model-badge { color: var(--t-blue); }
```

**Example: Context gauge as ring**
```css
.moltamp-context-gauge {
  width: 60px; height: 60px;
  border-radius: 50%;
  background: conic-gradient(
    var(--c-chrome-accent) calc(var(--data-context-pct) * 1%),
    transparent 0
  );
  display: flex;
  align-items: center;
  justify-content: center;
}
.moltamp-context-gauge::after {
  content: '';
  width: 44px; height: 44px;
  border-radius: 50%;
  background: var(--c-chrome-bg);
  pointer-events: none;
}
.moltamp-context-track { display: none; }
```

**Example: Cost-reactive vibes glow**
```css
.moltamp-shell { --cost-glow: calc(var(--data-cost-cents) * 0.5px); }
.moltamp-vibes {
  box-shadow: inset 0 0 var(--cost-glow) rgba(255, 100, 100, 0.3);
}
```

**Example: Git change indicator dot**
```css
.moltamp-vibes::after {
  content: '';
  position: absolute;
  bottom: 4px; right: 4px;
  width: calc(var(--data-git-changed) * 4px + 8px);
  height: 8px;
  background: var(--t-yellow);
  border-radius: 4px;
  opacity: calc(min(var(--data-git-changed), 1));
  pointer-events: none;
}
```

---

## Reactive Animations

Bind animations to data attributes. Use `var()` for all colors. **Gate reactive effects** with `--effect-*` variables so users can disable them (see [Rule 10](#10-all-visual-effects-must-be-gated-behind---effect--variables)).

```css
:root {
  --skin-glow-idle:    rgba(255, 204, 0, 0.05);
  --skin-glow-active:  rgba(255, 204, 0, 0.4);
  --skin-error-color:  #cc2200;

  --effect-vibes-glow:  1;   /* user-toggleable */
  --effect-vibes-pulse: 1;   /* user-toggleable */
}

/* Transitions don't need gating */
[data-activity="idle"] .moltamp-vibes {
  filter: brightness(0.5);
  transition: filter 1s ease;
}

/* Animations DO need gating */
[data-activity="high"] .moltamp-vibes {
  filter: brightness(calc(1 + 0.2 * var(--effect-vibes-glow))) saturate(calc(1 + 0.1 * var(--effect-vibes-glow)));
  animation: skin-glow 2s ease-in-out infinite;
}

[data-shell-state="thinking"] .moltamp-vibes {
  animation: skin-pulse 1.5s ease-in-out infinite;
}

/* Error flash is exempt — it's a system signal */
[data-shell-state="error"] .moltamp-vibes {
  box-shadow: inset 0 0 20px var(--skin-error-color);
}

[data-shell-state="permission"] .moltamp-titlebar {
  border-bottom: 2px solid var(--t-yellow);
}

@keyframes skin-glow {
  0%, 100% { box-shadow: inset 0 0 calc(10px * var(--effect-vibes-glow)) var(--skin-glow-idle); }
  50%      { box-shadow: inset 0 0 calc(20px * var(--effect-vibes-glow)) var(--skin-glow-active); }
}

@keyframes skin-pulse {
  0%, 100% { filter: brightness(calc(1 - 0.3 * var(--effect-vibes-pulse))); }
  50%      { filter: brightness(calc(1 + 0.1 * var(--effect-vibes-pulse))); }
}
```

**Performance:** Prefer `transform` and `opacity` for animations. Use `will-change` sparingly. Keep durations under 5s for responsiveness.

---

## Sprite Sheets

For frame-by-frame animations, use `::before`/`::after` pseudo-elements on panels:

```css
.moltamp-panel-left::after {
  content: '';
  position: absolute;
  inset: 0;
  background-image: url('./assets/sprites.png');
  background-size: 500% 100%;
  pointer-events: none;
}
[data-activity="idle"] .moltamp-panel-left::after {
  animation: sprite-idle 4s steps(5) infinite;
}
[data-activity="high"] .moltamp-panel-left::after {
  animation: sprite-active 0.5s steps(5) infinite;
}
@keyframes sprite-idle {
  from { background-position: 0 0; }
  to   { background-position: -500% 0; }
}
```

---

## Using Assets

Drop images in your skin's `assets/` folder. Reference with relative paths — the skin loader resolves them to absolute `file://` paths at runtime.

```css
.moltamp-panel-left {
  background-image: url('./assets/sidebar-texture.png');
  background-repeat: repeat;
}
```

**Supported formats:** PNG, JPG, WebP, GIF, SVG, AVIF
**Font formats:** WOFF2 (preferred), WOFF, TTF, OTF
**Limits:** 5MB per file, 20MB total per skin

Path traversal (`../`) is blocked — any `../` path is replaced with `url(about:blank)`.

---

## Sharing & Distribution

**Export:** Click Skins in the title bar, then Export — saves as `my-skin.zip`.
**Import:** Click Skins, then Import — select any `.zip` file.

The `.zip` file contains `skin.json`, `theme.css`, and `assets/`. You can also share the skin folder directly — drop it into `skins/` and restart.

When exporting, user-selected vibes GIFs from the global asset pool (`~/.moltamp/assets/`) are bundled into a `user-assets/` directory in the zip for portability. On import, these are extracted back to the global pool automatically.

**Clone a skin:** Use "Save As" from the skin browser to duplicate an existing skin with a new name and ID for customization.

Built-in skins stay intact so you always have clean defaults. Clone one via "Save As" if you want to customize it.

---

## Validation

Skin CSS is validated at load time. The validator returns **errors** (fatal — skin won't load) and **warnings** (logged but skin loads anyway).

### Errors (block loading)

- Missing required manifest fields (`id`, `name`, `version`, `engine`)
- `theme.css` exceeds 100KB
- Invalid skin ID (must match `^[a-zA-Z0-9_-]+$`)
- Asset exceeds 5MB, or total assets exceed 20MB
- CSS contains external URLs, `@import`, `javascript:`, or `expression()`
- CSS references any of the reserved internal chrome variable namespaces (see [Reserved internal variables](#reserved-internal-variables) below). The validator will emit a clear error naming the offending line.

### How the validator reads your CSS

Before the validator runs its pattern checks, it pre-processes `theme.css` so authors can't accidentally (or intentionally) sneak forbidden tokens past it:

1. **Comments are stripped.** Both `/* block comments */` and `// line comments` are removed before any pattern test. This means you can safely mention reserved names inside a doc comment without triggering a false positive — e.g. `/* Do not style the internal license UI */` is fine.
2. **CSS escape sequences are normalized.** The validator unescapes both hex escapes (`\6e`, `\6E `) and identity escapes (`\n`) to their underlying characters before pattern matching. Trying to hide a forbidden token behind `\` escapes will not get you past the validator — it sees through them. Don't waste your time.

The practical upshot: write your CSS normally, keep your comments honest, and if the validator rejects something the message will tell you what it found.

### Reserved internal variables

Moltamp reserves several CSS custom-property namespaces for internal chrome (think: the surfaces the app uses to tell the user something important that must not be obscured). **Skins cannot read, set, or reference them.** Any `var(--x)` reference or declaration targeting a reserved namespace is a hard error at import.

We intentionally do not list the reserved prefixes here — there is no legitimate reason for a skin to touch them, so listing them would just turn this doc into a cheat sheet. If you try, the validator will reject the skin on import with a clear error pointing at the offending declaration. When you see that error, rename your variable to the `--skin-*` prefix (or one of the documented `--t-*`, `--c-*`, `--effect-*` contract namespaces) and re-import.

### Warnings (shown in the import preview popup, skin can still be published)

- `background` or `background-image` on `.moltamp-vibes`
- `::before`/`::after` without `pointer-events: none`
- `visibility: hidden`, `opacity: 0`, or `pointer-events: none` on security-critical selectors (permission dialogs, auth prompts)
- **Ungated effects** — the validator scans for six families (see [Rule 10](#10-all-visual-effects-must-be-gated-behind---effect--variables)) and emits one warning per ungated family per selector:
  - **Motion** — `animation` / `animation-name` on a rule whose body has no `var(--effect-*)` reference and doesn't use an animation name whose `@keyframes` body is internally gated
  - **Glow** — `box-shadow` / `text-shadow` on pseudo-elements (any non-zero blur), or on regular elements with blur ≥ 20px, or `filter: drop-shadow()` with large blur — none gated by `calc(... * var(--effect-*))`
  - **Frost** — `backdrop-filter` or `filter: blur()` without a `calc(... * var(--effect-*))` multiplier
  - **Overlay** — `mix-blend-mode`, `repeating-linear-gradient`, `repeating-radial-gradient`, or animated GIF/WebP/APNG backgrounds without an `opacity: var(--effect-*)` gate
  - **Distortion** — `transform: perspective/skew/skewX/skewY/rotate3d/matrix3d`, `mask`, `mask-image`, `-webkit-mask`, `clip-path` without `calc()` gating or `opacity` gating on the element
  - **Color shift** — `filter: brightness/saturate/contrast/hue-rotate/invert/sepia/grayscale` without a `calc()` neutralizer
- **Dead toggles** ([Rule 11](#11-dont-declare-dead-toggles)) — `--effect-X` declared in `:root` but no rule in the CSS reads it via `var(--effect-X)`
- **Selectors exempt from ungated-effect checks** (these never trigger family warnings, so you can use them freely): `:hover`, `:focus`, `:active`, `.active`, `.glow-active`, `.crt-active`, `[data-shell-state="error"]`, settings UI classes (`.moltamp-skin-card`, `.moltamp-skin-panel`, `.moltamp-skin-overlay`, `.moltamp-input`)
- **Keyframe propagation**: if an `@keyframes` body uses `var(--effect-*)` internally, any `animation:` reference to that keyframe is considered gated and won't trigger a motion warning

All warnings appear in the **import preview popup**. Each warning has a "Copy AI fix prompt" button that copies a self-contained remediation prompt — paste it into Claude/ChatGPT/Cursor/etc. and you'll get a patched theme.css back. Users see all warnings grouped by family and can choose *Publish anyway* (the skin still loads) or *Cancel*.

### Import validation

- Zip-slip defense: extracted files verified to stay within the temp directory
- Asset path containment: no directory traversal

---

## Preflight (Auto-Fixes at Load Time)

Moltamp applies these overrides **after** your skin CSS loads. They protect interaction while giving skins full creative freedom. The `!important` flag ensures preflight always wins.

| What | Override | Why |
|------|----------|-----|
| `.moltamp-vibes` background | `background: transparent !important` | Vibes visuals come from GIF widget slots, not CSS |
| All `::before` / `::after` on `.moltamp-shell *` | `pointer-events: none !important` | Pseudo-elements must never block clicks |
| Buttons, inputs, select, textarea, `[role="button"]`, anchors, `[draggable="true"]` | `pointer-events: auto !important` | Interactive elements always stay clickable |
| Inputs, textareas | `cursor: text !important` | Text fields keep text cursor |
| `.moltamp-shell` | `cursor: default !important` | Cursor never hidden globally |

**Your skin can style anything visually.** Colors, gradients, shadows, animations, transforms, filters, blend modes — all unrestricted. The preflight only protects interaction: clicks, hovers, cursor, and drag.

---

## Cookbook: Common Patterns

### Minimal dark theme (just colors)

```css
:root {
  --t-foreground: #c0c0c0;
  --t-background: #111111;
  --c-chrome-bg: #111111;
  --c-chrome-accent: #ff6600;
  --c-chrome-border: #333333;
}
```

### Activate CRT effect

```css
:root {
  --effect-scanlines: 1;
  --effect-glow: 1;
  --effect-crt: 1;
  --scanline-color: rgba(255, 255, 255, 0.06);
  --glow-intensity: 12px;
}
```

### Style panels differently per side

```css
:root {
  --skin-left-bg: linear-gradient(180deg, #0a0a0a 0%, #0f0f1a 100%);
}

.moltamp-panel-left {
  background: var(--skin-left-bg);
  border-right: 2px solid var(--c-chrome-accent);
}
.moltamp-panel-right {
  background: var(--c-chrome-bg);
}
```

### Transparent / shaped window

Moltamp's window supports transparency — your desktop shows through any transparent areas of the skin. This lets you create floating-pod UIs, shaped panels, and see-through gaps between elements.

To use transparency, make the shell and html backgrounds transparent, then use opaque backgrounds only on the panels you want visible:

```css
/* Make the window see-through */
html, body {
  background: transparent !important;
}

.moltamp-shell {
  background: transparent !important;
  gap: 6px !important;      /* creates visible gaps between panels */
  padding: 6px !important;  /* space around the edges */
}

/* Each panel is an opaque "pod" floating over the desktop */
.moltamp-titlebar {
  background: var(--skin-pod-bg) !important;
  border: 1px solid var(--skin-pod-border) !important;
  border-radius: 20px !important;   /* pill shape */
  margin: 0 40px !important;        /* narrower than the window */
}

.moltamp-panel-left,
.moltamp-panel-right {
  background: var(--skin-pod-bg) !important;
  border: 1px solid var(--skin-pod-border) !important;
  border-radius: 16px !important;   /* rounded pods */
}

.moltamp-terminal {
  background: var(--t-background) !important;
  border-radius: 16px !important;
  border: 1px solid var(--skin-pod-border) !important;
}

.moltamp-panel-bottom {
  border-radius: 14px !important;
  margin: 0 60px !important;        /* tapered silhouette */
}

.moltamp-statusbar {
  border-radius: 14px !important;
  margin: 0 80px !important;        /* even narrower */
}
```

**Why `!important`?** The base layout CSS sets backgrounds, borders, and dimensions on these elements. Transparent skins need to override them. This is the one case where `!important` is expected and necessary.

**What you can do:**
- `border-radius` on any panel for rounded/organic shapes
- `margin` to make panels narrower than the window, creating a tapered silhouette
- `gap` and `padding` on the shell to space panels apart
- `clip-path: polygon(...)` for angular/diamond/hexagonal cuts
- `backdrop-filter: blur(10px)` for frosted glass over the desktop
- Combine opaque and transparent panels — e.g., solid terminal with floating side pods

**Performance note:** Transparent windows composite with the desktop on every frame. Heavy animations + transparency can stress the GPU. Test on your target hardware.

---

## Tips

- **Performance:** Use `transform` and `opacity` for animations. Use `will-change` sparingly.
- **Go bold:** Skins are art. Scanlines, glitch effects, parallax, color shifts — all encouraged.
- **Test at sizes:** Vibes panel and side panels are resizable. Check your art at all sizes.
- **Monochrome wins:** Pick one accent color and vary brightness. The built-in skins are good examples.
- **Start from a built-in:** Clone an existing skin instead of the template if you want a head start.
- **Use the data properties:** `--data-context-pct` as a gradient stop, `--data-cost-cents` as a glow radius — pure CSS, no JavaScript needed.

---

## For AI-Generated Skins

If you're using ChatGPT, Claude, Codex, or another AI to generate a skin, **paste this entire block** with your prompt:

```
MOLTAMP SKIN RULES — follow these exactly:

FILE STRUCTURE:
- skin.json (manifest) + theme.css (all styles) + assets/ (images)
- skin.json requires: id, name, version, engine: "1.0"
- theme.css max 100KB

CSS RULES:
- ALL colors must be CSS variables in :root. Never hardcode hex/rgb in selectors.
- Contract vars: --t-* (terminal), --c-chrome-* (chrome), --effect-* (effects)
- Custom vars: use --skin-* prefix (e.g., --skin-panel-bg, --skin-overlay)
- NEVER set background or background-image on .moltamp-vibes
- Every ::before and ::after MUST include pointer-events: none
- No external URLs, no @import, no expression(), no javascript:
- NEVER target Moltamp internal UI elements (validator hard-blocks)

EFFECTS — CRITICAL:
- The validator scans SIX effect families and warns when any are ungated:
    1. MOTION        — animation, animation-name, @keyframes applications
    2. GLOW          — box-shadow/text-shadow with large blur, filter: drop-shadow
    3. FROST         — backdrop-filter, filter: blur()
    4. OVERLAY       — mix-blend-mode, repeating-linear/radial-gradient, animated gif/webp/apng bg
    5. DISTORTION    — transform: perspective/skew/rotate3d/matrix3d, mask, clip-path
    6. COLOR SHIFT   — filter: brightness/saturate/contrast/hue-rotate/invert/sepia/grayscale
- EVERY ungated effect in any of those six families will trigger a warning on import
- Declare --effect-myeffect: 1; in :root — it auto-registers as a toggle (0%-200%)
- Effect variable range: 0 (off) to 2 (max boost), where 1 = 100% (default)
- Gate pseudo-elements:       opacity: var(--effect-myeffect)
- Gate box-shadows:            box-shadow: inset 0 0 calc(18px * var(--effect-myeffect)) <color>
- Gate backdrop-filter:        backdrop-filter: blur(calc(8px * var(--effect-myeffect)))
- Gate mix-blend-mode:         opacity: var(--effect-myeffect) (on the same element)
- Gate repeating gradients:    opacity: var(--effect-myeffect) (on the overlay element)
- Gate transform distortion:   transform: skewX(calc(0.5deg * var(--effect-myeffect)))
- Gate color-shift filters:    filter: hue-rotate(calc(15deg * var(--effect-myeffect)))
- Gate animated backgrounds:   opacity: calc(base * var(--effect-myeffect))
- Gate filters in keyframes:   brightness(calc(1 + delta * var(--effect-myeffect))) — neutralizes to 1 at 0
- If keyframes use opacity, use filter: opacity() in keyframes instead
- DEAD TOGGLES: every --effect-X declared in :root must be READ by at least one rule via var(--effect-X).
  Declaring it without reading it creates a silently broken toggle in the Effects panel.
- Exemptions (never trigger warnings): :hover, :focus, :active, .active, [data-shell-state="error"]
- Optional: add labels in skin.json under "effects" key

VIBES IMAGES:
- Use skin.json vibes slots, not CSS background:
  "vibes": { "slots": [{ "widgetId": "gif", "gifSrc": "assets/vibes.gif", "gifFit": "cover" }] }

LAYOUT — give the skin a personality:
- Ship a defaultLayout in skin.json with themed tab names and curated widgets
- Tab names should match the skin theme (e.g., SENSORS, COMMS, HOLODECK for sci-fi)
- Available widgets: clock, session-timer, calendar, world-clock, event-calendar,
  visualizer, music, equalizer, system, weather, notes, live2d,
  usage-model, usage-context, usage-tokens, usage-session, usage-rates, usage-lifetime
- Vary dimensions per personality: taller vibes for cinematic, narrower panels for minimal

TARGETABLE CLASSES:
- .moltamp-shell, .moltamp-titlebar, .moltamp-vibes
- .moltamp-panel-left, .moltamp-panel-right, .moltamp-panel-bottom
- .moltamp-terminal, .moltamp-statusbar
- .moltamp-message-user, .moltamp-message-assistant
- .moltamp-tool-call, .moltamp-thinking, .moltamp-code-block

REACTIVE ATTRIBUTES (on .moltamp-shell):
- data-activity="idle|low|high"
- data-session-count, data-split-active
- Live CSS vars: --data-context-pct, --data-cost-cents, --data-tokens-in, etc.

PROPERTIES THAT BREAK THINGS — NEVER DO THESE:
- NEVER use * (universal selector) with pointer-events, display, opacity, or position
- NEVER use .moltamp-titlebar * { } — breaks buttons, zoom controls, settings gear
  Use .moltamp-titlebar > span:first-child { } to target the logo text instead
- NEVER set overflow: hidden on .moltamp-shell, .moltamp-deck, .moltamp-titlebar, .moltamp-statusbar
  (clips context menus and popovers that use position: fixed)
- NEVER change flex-direction on .moltamp-deck (rotates the entire layout)
- NEVER set pointer-events: none on any container (disables all interaction inside)
- NEVER apply transform to .moltamp-shell (breaks position: fixed children)
- NEVER set position: relative + z-index on .moltamp-deck (hides fixed menus)

WHAT YOU CAN DO (unlimited):
- Custom animations, transitions, gradients, filters, transforms on CONTENT elements
- Reactive styles bound to data-activity (gate with --effect-*)
- ::before/::after overlays with effects (always include pointer-events: none)
- Custom --effect-* variables for user-toggleable effects (0%-200% range)
- Sprite sheets, blend modes, backdrop-filter, clip-path, anything CSS can do
- Data-driven visuals: --data-context-pct as a gradient stop, --data-cost-cents as glow radius
- Custom fonts bundled in assets/ via @font-face
- GIF artwork on panels via ::before pseudo-elements (gated with --effect-*)
- Title bar restyling: background, borders, border-radius (but not overflow or pointer-events)
- Status bar restyling: background, borders, border-radius, text color on spans
```

---

## Debugging & Dev Tools

### Opening DevTools

Open Chrome DevTools in Moltamp with **Cmd+Shift+I** (macOS) or from the application menu. This gives you the same inspector you'd use in Chrome — Elements, Console, Network, and all the rest.

### Inspecting your skin

- In the **Elements** panel, look for your `:root` variables. They'll appear on the `html` element. Expand the computed styles to see which variables are active and what values they resolve to.
- Search for `.moltamp-` classes to find the elements you're targeting. The Styles pane shows which rules are winning, including preflight overrides marked with `!important`.

### Console messages

The validator logs warnings and errors to the **Console** on skin load. Look for messages like:
- `[Skin Validator] Warning: ungated effect detected...`
- `[Skin Validator] Warning: missing pointer-events: none on ::before...`
- `[Skin Preflight] Auto-fixed: background on .moltamp-vibes`

These messages tell you exactly what the validator found and what preflight auto-corrected.

### Hot-reload

Skins reload when you switch away and back, or click the skin name in the skin browser. No restart needed for CSS changes. This makes iteration fast — edit `theme.css`, save, click your skin name, and see the result immediately.

### Resetting overrides

Use the **Reset to Defaults** button in Settings > Skins to clear all user color, font, and effect overrides. This returns the skin to exactly what your `theme.css` and `skin.json` define — useful when testing to make sure your defaults look right without residual user customizations.

---

## Troubleshooting

**"My skin loads but nothing changes"**
Check that you're overriding the contract variables (`--t-*`, `--c-chrome-*`), not defining custom names that Moltamp doesn't recognize. Open DevTools (Cmd+Shift+I) and inspect the `:root` element — your variables should appear there. If they don't, your CSS may have a syntax error preventing the file from loading. Also check for CSS specificity conflicts where a more specific selector is winning over yours.

**"Validator says ungated effect"**
Your animation, glow, or filter isn't controlled by an `--effect-*` variable. Every visual flourish needs to be gated so users can disable it. See [Rule 10](#10-all-visual-effects-must-be-gated-behind---effect--variables) for gating patterns. The most common fix: add `opacity: var(--effect-your-effect)` to the element with the animation.

**"Colors look wrong after export"**
Make sure all colors are defined as CSS variables in `:root`. Hardcoded hex values in selectors won't survive color override export — the exported skin bakes in the user's overrides, but only for values that are `:root` variables.

**"Right-click menu is clipped"**
You have `overflow: hidden` on a panel container. Remove it. Context menus use `position: fixed` and need the viewport to be unclipped. See [Properties That Break Things](#properties-that-break-things) for the full list of containers you should never restrict.

**"Vibes GIF doesn't show"**
GIFs go in `skin.json` vibes slots, not CSS `background`. The vibes panel background is forcibly set to `transparent !important` by preflight. Add a `vibes` block to your manifest:
```json
{ "vibes": { "slots": [{ "widgetId": "gif", "gifSrc": "assets/vibes.gif", "gifFit": "cover" }] } }
```

**"My skin works but the validator warns about pointer-events"**
Add `pointer-events: none` to all `::before`/`::after` pseudo-elements in your CSS. Moltamp auto-fixes this at load time via preflight, so your skin still works — but the warning means your CSS isn't spec-compliant. Declaring it explicitly silences the warning and makes your skin cleaner.

---

## Accessibility

Skins are visual art, but they're also someone's workspace. Keep these guidelines in mind so your skin works well for everyone.

### Contrast ratios

Aim for WCAG AA minimum contrast ratios:
- **4.5:1** for normal text (body copy, stat labels, tab names)
- **3:1** for large text (headings, model badge, logo)

You don't need to measure every element — focus on the primary reading surfaces: terminal text against its background, stat labels against panel backgrounds, and tab names against the tab bar. Use your contract variables (`--t-foreground` against `--t-background`, `--c-chrome-text` against `--c-chrome-bg`) as your guide.

### Reduced motion

Users who prefer reduced motion set `prefers-reduced-motion: reduce` in their OS. When possible, respect this in your skin:

```css
@media (prefers-reduced-motion: reduce) {
  .moltamp-vibes::after {
    animation: none;
  }
}
```

Note that effect gating already covers the most important case here — users can disable all animations individually through Settings > Effects by setting each slider to 0%. The `prefers-reduced-motion` media query is an additional courtesy for users who have it enabled system-wide.

### Color blindness

Don't rely on color alone to convey state. The reactive data attributes (`data-activity`, `data-shell-state`) are great for adding visual energy, but if your skin changes only the hue to indicate a state change, some users won't see it. Use brightness and contrast as backup signals — a brighter glow is more universally visible than a color shift from green to red.

### Effect controls

The effect gating system (Rule 10) is itself an accessibility feature. Every visual effect has a slider, and users can disable anything that's distracting or uncomfortable. Design your default intensity (`1`) to be comfortable for extended use, and let users who want more push toward `2`.

---

## Submitting Your Skin

Ready to share your skin with the community? Here's the process.

1. **Fork** the [moltamp-skins](https://github.com/shoot-here/moltamp-skins) repository.
2. **Create** your skin folder at `skins/your-skin-id/` with `skin.json`, `theme.css`, and optionally `assets/`.
3. **Add a preview screenshot** — `preview.png` in your skin folder, recommended 800x500 pixels. Show your skin in action with panels open and some terminal content visible.
4. **Run through the validation checklist** at the bottom of this document. Every check should pass.
5. **Open a PR** using the skin submission template. Describe your skin's theme, list its custom effects, and mention any special techniques you used.
6. **Community review** — maintainers review submissions for: validation pass, no security issues, visual quality, and properly gated effects. We'll work with you if anything needs adjustment.
7. **Accepted skins ship** in the next Moltamp release and appear in the built-in skin browser.

---

## Gallery & Inspiration

- **Browse community skins** at [moltamp.com/skins](https://moltamp.com/skins) — filter by style, color palette, or effect type.
- **Built-in skins in this repo** serve as reference implementations. Each one demonstrates different techniques:
  - **LCARS** — shaped panels with `border-radius` and `clip-path`, custom tab names
  - **Blade Runner** — SVG overlays, rain GIF in vibes, amber noir palette
  - **Biodiagnostic** — multiple GIF assets across panels, medical HUD styling
  - **Phosphor** — CRT effects (scanlines, glow, flicker), green monochrome
  - **Deep Space** — data-driven gauges, conic gradients for context rings
  - **Kosmos** — retro-futurist typography with bundled custom fonts
- Study how they structure their `:root` variables, gate their effects, and configure their `defaultLayout` — then make something entirely your own.

---

## Checklist

Before sharing your skin:

**Colors & Variables**
- [ ] All colors defined as variables in `:root`
- [ ] Zero hardcoded hex/rgb/rgba outside `:root`
- [ ] Contract variables (`--c-*`, `--t-*`) overridden for your palette
- [ ] Custom variables use `--skin-*` prefix

**Effects & Animations**
- [ ] Every **motion** effect gated (`animation`, `@keyframes` applications)
- [ ] Every **glow** gated (box-shadow / text-shadow blur, `filter: drop-shadow`)
- [ ] Every **frost** gated (`backdrop-filter`, `filter: blur()`)
- [ ] Every **overlay** gated (`mix-blend-mode`, `repeating-*-gradient`, animated GIF/WebP backgrounds)
- [ ] Every **distortion** gated (`transform: perspective/skew`, `mask`, `clip-path`)
- [ ] Every **color shift** gated (`filter: brightness/saturate/contrast/hue-rotate/invert/sepia/grayscale`)
- [ ] No **dead toggles** — every `--effect-X` in `:root` is read by at least one rule
- [ ] Effect labels added to `skin.json` for custom effects
- [ ] Slider range tested — effects look good from 0% to 200%

**Layout & Identity**
- [ ] `defaultLayout` in `skin.json` with themed tab names
- [ ] Widget selection curated for the skin's personality
- [ ] Dimensions tuned (vibes height, panel widths)
- [ ] Vibes slots configured in `skin.json` (if using GIF art)

**Accessibility**
- [ ] Text contrast meets WCAG AA (4.5:1 for body, 3:1 for large text)
- [ ] `prefers-reduced-motion` respected where practical
- [ ] State changes don't rely on color alone

**Safety & Interaction**
- [ ] No `background` or `background-image` on `.moltamp-vibes`
- [ ] Every `::before`/`::after` has `pointer-events: none`
- [ ] No external URLs or `@import`
- [ ] No selectors targeting Moltamp internal UI
- [ ] No references to reserved internal variable namespaces (stick to `--skin-*`, `--t-*`, `--c-*`, `--effect-*`)
- [ ] `theme.css` under 100KB
- [ ] Assets under 5MB each, 20MB total
- [ ] Tested at different panel sizes (resize handles)
- [ ] Right-click context menus work on all panels
- [ ] Permission dialog still visible and clickable

**Submission**
- [ ] `preview.png` screenshot included (800x500 recommended)
- [ ] `engine: "1.0"` set in `skin.json`
- [ ] Skin ID is lowercase-kebab-case
