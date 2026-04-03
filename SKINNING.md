# Moltamp Skin Specification v2.0

The single source of truth for building Moltamp skins.

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

## Skin Format

```
skins/my-skin/
  skin.json          <- Required: manifest
  theme.css          <- Required: all styles
  assets/            <- Optional: images, GIFs, SVGs
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

All fields required. `id` must match `^[a-zA-Z0-9_-]+$`.

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
| `--effect-scanlines` | `0`–`1` | Scanline overlay | `0` |
| `--effect-glow` | `0`–`1` | Inner glow | `0` |
| `--effect-crt` | `0`–`1` | CRT curvature | `0` |
| `--effect-vignette` | `0`–`1` | Darkened edge overlay | `0` |
| `--effect-noise` | `0`–`1` | Film grain texture | `0` |
| `--effect-flicker` | `0`–`1` | CRT refresh animation | `0` |
| `--effect-grid` | `0`–`1` | Background grid lines | `0` |
| `--effect-text-glow` | `0`–`1` | Phosphor blur on text | `0` |
| `--scanline-opacity` | `0`–`1` | Scanline intensity | `0.04` |
| `--glow-intensity` | pixels | Glow blur radius | `12px` |
| `--c-glow` | color | Glow color | `rgba(212,160,54,0.15)` |

**Custom effects:** Declare any `--effect-*` variable in `:root` and it automatically appears in the Effects panel as a user toggle with an intensity slider. Name it anything:

```css
:root {
  --effect-radar: 1;        /* Shows as "Radar" toggle */
  --effect-heat-haze: 0.5;  /* Shows as "Heat Haze" at 50% */
}
```

For nicer labels, add metadata in `skin.json`:

```json
{
  "effects": {
    "radar": { "label": "Sonar Radar", "description": "Rotating sweep overlay" },
    "heat-haze": { "label": "Heat Haze", "description": "Thermal distortion" }
  }
}
```

Then use the variable in your CSS to control the effect:

```css
.moltamp-vibes::before {
  opacity: var(--effect-radar);
  animation: radar-sweep 4s linear infinite;
}
```

When the user toggles the effect off, `--effect-radar` becomes `0`, and `opacity: 0` hides it. When they adjust the slider, it scales between 0 and 1.

### Typography (4 vars)

| Variable | Default |
|----------|---------|
| `--font-terminal` | `'SF Mono', 'JetBrains Mono', monospace` |
| `--font-chrome` | `'Inter', 'SF Pro Display', sans-serif` |
| `--font-size` | `14px` |
| `--font-line-height` | `1.35` |

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
  background: var(--skin-overlay);
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

| Class | What it is | Default |
|-------|-----------|---------|
| `.moltamp-shell` | Root container | Visible |
| `.moltamp-titlebar` | Top title bar | Visible |
| `.moltamp-vibes` | Top banner — your canvas | Visible |
| `.moltamp-deck` | Holds left + terminal + right | Visible |
| `.moltamp-panel-left` | Left panel (Intel tabs) | Visible |
| `.moltamp-panel-right` | Right panel (Signal) | Visible |
| `.moltamp-panel-bottom` | Bottom bar (Telemetry) | Visible |
| `.moltamp-statusbar` | Bottom status bar | Visible |

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
| `.moltamp-token-chart` | Token usage bar chart |
| `.moltamp-stats` | Data readout container |
| `.moltamp-stat-row` | Individual stat row |
| `.moltamp-stat-cost` | Cost row |
| `.moltamp-stat-time` | Time row |

---

## The Vibes Panel

The `.moltamp-vibes` div is your hero canvas. All panels are visible by default — just style them.

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
```

**Example: Context gauge as pie chart**
```css
.moltamp-context-gauge {
  width: 80px; height: 80px;
  border-radius: 50%;
  background: conic-gradient(
    var(--c-chrome-accent) calc(var(--data-context-pct) * 1%),
    var(--c-chrome-border) 0
  );
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

@keyframes skin-glow {
  0%, 100% { box-shadow: inset 0 0 10px var(--skin-glow-idle); }
  50%      { box-shadow: inset 0 0 20px var(--skin-glow-active); }
}
```

**Performance:** Prefer `transform` and `opacity` for animations. Use `will-change` sparingly. Keep durations under 5s for responsiveness.

---

## Sprite Sheets

For frame-by-frame animations, use a dedicated element inside a panel (not `.moltamp-vibes` — vibes backgrounds are blocked). Sprite sheets work well in `::before`/`::after` pseudo-elements on other panels:

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

## Sharing

**Export:** Settings > Skins > Export — saves as `.zip`
**Import:** Settings > Skins > Import — select `.zip`

The `.zip` file contains `skin.json`, `theme.css`, and `assets/`.

---

## Quick Start

```bash
cp -r skins/_template skins/my-skin
```

1. Edit `skin.json` with your info
2. Edit `theme.css` — set your palette in `:root`, style elements with `var()`
3. Drop art in `assets/`, reference as `url('./assets/file.ext')`
4. Launch Moltamp, open Settings > Skins, select your skin

---

## Preflight (Auto-Fixes at Load Time)

Moltamp applies these overrides **after** your skin CSS loads. They protect interaction while giving skins full creative freedom.

| What | Override | Why |
|------|----------|-----|
| `.moltamp-vibes` background | `background: transparent !important` | Vibes visuals come from GIF widget slots, not CSS |
| All `::before` / `::after` | `pointer-events: none !important` | Pseudo-elements must never block clicks |
| Buttons, inputs, links, draggables | `pointer-events: auto !important` | Interactive elements always stay clickable |
| Inputs, textareas | `cursor: text !important` | Text fields keep text cursor |
| `.moltamp-shell` | `cursor: default !important` | Cursor never hidden globally |

**Your skin can style anything visually.** Colors, gradients, shadows, animations, transforms, filters, blend modes — all unrestricted. The preflight only protects interaction: clicks, hovers, cursor, and drag.

### For AI-generated skins

If you're using ChatGPT, Claude, Codex, or another AI to generate a skin, **paste this entire block** with your prompt:

```
MOLTAMP SKIN RULES — follow these exactly:

FILE STRUCTURE:
- skin.json (manifest) + theme.css (all styles) + assets/ (images)
- skin.json requires: id, name, version, author, description, engine: "1.0"

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
- [ ] Assets under 5MB each, 20MB total
- [ ] Tested at different panel sizes (resize handles)
- [ ] Right-click context menus work on all panels
- [ ] Permission dialog still visible and clickable
