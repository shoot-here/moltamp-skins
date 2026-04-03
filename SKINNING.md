# Moltamp Skin Specification v2.0

The single source of truth for building Moltamp skins.

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

---

## The Rules

These are non-negotiable. The validator enforces them, and community skins that break them won't be accepted.

### 1. All colors live in `:root`

Every color value — hex, rgb, rgba, hsl — must be defined as a CSS variable in `:root`. Outside of `:root`, colors are referenced with `var()`. Never hardcode a color in a selector.

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

### 3. Never hide panels

Skins must not use `display: none` on any panel element (`.moltamp-vibes`, `.moltamp-panel-left`, `.moltamp-panel-right`, `.moltamp-panel-bottom`). All panels ship ON. Users toggle them via Settings > Layout. Skins can style panels freely — backgrounds, borders, overlays — but visibility is the user's choice.

### 4. No executable content

- No external URLs (`https:`, `http:`)
- No `@import`
- No `expression()`, `javascript:`, `behavior:`, `-moz-binding`
- No `data: text/*` URIs
- CSS only. Zero JS.

### 5. Don't hide the permission dialog

These properties trigger validator warnings on elements that could obscure permission prompts:
- `visibility: hidden`
- `opacity: 0`
- `pointer-events: none`

### 6. No backgrounds on `.moltamp-vibes`

The vibes panel is always transparent. Visual content (GIFs, images) must come from **GIF widget slots** in `skin.json`, not CSS `background` or `background-image`. Moltamp strips vibes backgrounds at load time with `!important`.

### 7. Pseudo-elements need `pointer-events: none`

Every `::before` and `::after` pseudo-element **must** include `pointer-events: none`. Without it, pseudo-elements intercept clicks and break right-click menus, drag handles, and widget interaction. Moltamp auto-fixes this at load time, but your skin should declare it explicitly.

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

### 8. Assets stay in `assets/`

- Reference with `url('./assets/filename.ext')`
- Supported images: PNG, JPG, WebP, GIF, SVG, AVIF
- Supported fonts: WOFF2, WOFF, TTF, OTF
- Max 5MB per file, 20MB per skin
- No nested directories in `assets/`
- No path traversal (`../`)

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

### Effects — Built-in + Custom

Built-in effects that Moltamp knows about:

| Variable | Type | Purpose | Default |
|----------|------|---------|---------|
| `--effect-scanlines` | `0`–`1` | Horizontal line overlay | `0` |
| `--effect-glow` | `0`–`1` | Inset phosphor glow around terminal | `0` |
| `--effect-crt` | `0`–`1` | Barrel distortion + edge shadow | `0` |
| `--effect-vignette` | `0`–`1` | Darkened edge overlay | `0` |
| `--effect-noise` | `0`–`1` | Static noise / film grain | `0` |
| `--effect-flicker` | `0`–`1` | CRT refresh animation | `0` |
| `--effect-grid` | `0`–`1` | Background grid lines | `0` |
| `--effect-text-glow` | `0`–`1` | Phosphor blur on text | `0` |

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
  --effect-radar: 1;        /* Shows as "Radar" toggle */
  --effect-heat-haze: 0.5;  /* Shows as "Heat Haze" at 50% */
}
```

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

When the user toggles the effect off, `--effect-radar` becomes `0`, and `opacity: 0` hides it. When they adjust the slider, it scales between 0 and 1.

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

### Layout

| Class | What it is |
|-------|-----------|
| `.moltamp-shell` | Root container |
| `.moltamp-titlebar` | Top title bar |
| `.moltamp-vibes` | Top banner — your canvas |
| `.moltamp-deck` | Holds left + terminal + right |
| `.moltamp-panel-left` | Left panel (Intel tabs) |
| `.moltamp-panel-right` | Right panel (Signal) |
| `.moltamp-panel-bottom` | Bottom bar (Telemetry) |
| `.moltamp-statusbar` | Bottom status bar |

### Terminal

| Class | What it is |
|-------|-----------|
| `.moltamp-terminal` | Terminal container |
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

### Media Widgets

| Class | Element |
|-------|---------|
| `.moltamp-widget-music` | Now playing widget |
| `.moltamp-widget-visualizer` | Audio visualizer widget |
| `.moltamp-widget-equalizer` | EQ widget |
| `.moltamp-equalizer` | EQ inner container |

All widgets get `.moltamp-widget-{id}` on their container automatically.

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

Bind animations to data attributes. Use `var()` for all colors.

```css
:root {
  --skin-glow-idle:    rgba(255, 204, 0, 0.05);
  --skin-glow-active:  rgba(255, 204, 0, 0.4);
  --skin-error-color:  #cc2200;
}

[data-activity="idle"] .moltamp-vibes {
  filter: brightness(0.5);
  transition: filter 1s ease;
}
[data-activity="high"] .moltamp-vibes {
  filter: brightness(1.2) saturate(1.1);
  animation: skin-glow 2s ease-in-out infinite;
}

[data-shell-state="thinking"] .moltamp-vibes {
  animation: skin-pulse 1.5s ease-in-out infinite;
}

[data-shell-state="error"] .moltamp-vibes {
  box-shadow: inset 0 0 20px var(--skin-error-color);
}

[data-shell-state="permission"] .moltamp-titlebar {
  border-bottom: 2px solid var(--t-yellow);
}

@keyframes skin-glow {
  0%, 100% { box-shadow: inset 0 0 10px var(--skin-glow-idle); }
  50%      { box-shadow: inset 0 0 20px var(--skin-glow-active); }
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

**Built-in skins are protected** — they cannot be deleted. To customize one, clone it first.

---

## Validation

Skin CSS is validated at load time. The validator returns **errors** (fatal — skin won't load) and **warnings** (logged but skin loads anyway).

### Errors (block loading)

- Missing required manifest fields (`id`, `name`, `version`, `engine`)
- `theme.css` exceeds 100KB
- Invalid skin ID (must match `^[a-zA-Z0-9_-]+$`)
- Asset exceeds 5MB, or total assets exceed 20MB
- CSS contains external URLs, `@import`, `javascript:`, or `expression()`

### Warnings (logged, skin still loads)

- `background` or `background-image` on `.moltamp-vibes`
- `::before`/`::after` without `pointer-events: none`
- `visibility: hidden` or `opacity: 0` on elements that could obscure permission prompts

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

EFFECTS:
- Declare --effect-myeffect: 1; in :root to auto-register a toggle
- Use opacity: var(--effect-myeffect) to make it toggleable
- Optional: add labels in skin.json under "effects" key

VIBES IMAGES:
- Use skin.json vibes slots, not CSS background:
  "vibes": { "slots": [{ "widgetId": "gif", "gifSrc": "assets/vibes.gif", "gifFit": "cover" }] }

TARGETABLE CLASSES:
- .moltamp-shell, .moltamp-titlebar, .moltamp-vibes
- .moltamp-panel-left, .moltamp-panel-right, .moltamp-panel-bottom
- .moltamp-terminal, .moltamp-statusbar
- .moltamp-message-user, .moltamp-message-assistant
- .moltamp-tool-call, .moltamp-thinking, .moltamp-code-block

REACTIVE ATTRIBUTES (on .moltamp-shell):
- data-activity="idle|low|high"
- data-shell-state="idle|thinking|streaming|tool-use|permission|error|complete"
- Live CSS vars: --data-context-pct, --data-cost-cents, --data-tokens-in, etc.

WHAT YOU CAN DO (unlimited):
- Custom animations, transitions, gradients, filters, transforms
- Reactive styles bound to data-activity and data-shell-state
- ::before/::after overlays with effects (always include pointer-events: none)
- Custom --effect-* variables for user-toggleable effects
- Sprite sheets, blend modes, backdrop-filter, clip-path, anything CSS can do
```

---

## Checklist

Before sharing your skin:

- [ ] All colors defined as variables in `:root`
- [ ] Zero hardcoded hex/rgb/rgba outside `:root`
- [ ] Contract variables (`--c-*`, `--t-*`) overridden for your palette
- [ ] Custom variables use `--skin-*` prefix
- [ ] No `background` or `background-image` on `.moltamp-vibes` (use GIF widget slots instead)
- [ ] Every `::before`/`::after` has `pointer-events: none`
- [ ] No external URLs or `@import`
- [ ] `theme.css` under 100KB
- [ ] Assets under 5MB each, 20MB total
- [ ] Tested at different panel sizes (resize handles)
- [ ] Right-click context menus work on all panels
- [ ] Permission dialog still visible and clickable
