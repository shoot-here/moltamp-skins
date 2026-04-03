# MOLTamp Skins

Community skins for [MOLTamp](https://moltamp.com) — the skinnable shell for Claude Code.

MOLTamp wraps Claude Code's terminal in a customizable cockpit UI. Think Winamp for AI terminals. Every panel, color, animation, and overlay is controlled by skins — pure CSS, zero JavaScript.

## What is MOLTamp?

MOLTamp is an Electron app that gives Claude Code a visual cockpit:

- **Vibes panel** — GIF banner at the top, resizable, multi-slot
- **Intel panel** — left sidebar with widgets (git, files, agents, events)
- **Signal panel** — right sidebar with data readouts
- **Telemetry bar** — bottom ticker with live stats
- **Terminal** — the actual Claude Code PTY, fully skinnable

Everything reacts to Claude's state — thinking, streaming, tool use, errors — through CSS data attributes. Skins can animate based on activity, change colors per model, pulse on errors, glow during streaming.

**Download:** [moltamp.com](https://moltamp.com)
**Main repo:** [github.com/shoot-here/Moltamp](https://github.com/shoot-here/Moltamp)
**Skinning guide:** [moltamp.com/skinning](https://moltamp.com/skinning/)

## This Repo

This is the community skin repo. Browse skins, fork them, submit your own.

```
skins/
  blade-runner/       <- Each skin is a folder
    skin.json         <- Manifest (id, name, author, etc.)
    theme.css         <- All styles
    assets/           <- GIFs, PNGs, SVGs (optional)
    preview.png       <- Screenshot for the gallery (optional)
```

### Browse Skins

| Skin | Author | Description |
|------|--------|-------------|
| [Obsidian](skins/obsidian/) | MOLTamp | Clean dark default |
| [Phosphor](skins/phosphor/) | MOLTamp | Green phosphor CRT terminal |
| [Blade Runner](skins/blade-runner/) | MOLTamp | Amber noir with rain vibes |
| [Deep Space](skins/deep-space/) | MOLTamp | Cyan deep-space station |
| [Kosmos](skins/kosmos/) | MOLTamp | Soviet space program green |
| [Biodiagnostic](skins/biodiagnostic/) | MOLTamp | Teal medical/biotech |
| [Ice Nine](skins/ice-nine/) | MOLTamp | Frozen blue crystalline |
| [LCARS](skins/lcars/) | MOLTamp | Star Trek computer interface |
| [Lunar](skins/lunar/) | MOLTamp | Moon phase observatory |
| [Neon Horizon](skins/neon-horizon/) | MOLTamp | Magenta/cyan synthwave |

## Install a Skin

1. Download the skin folder (or clone this repo)
2. Open MOLTamp > Settings > Skins > Import
3. Select the folder or `.zip`

Or copy directly:
```bash
cp -r skins/blade-runner ~/Moltamp/skins/
```

## Create a Skin

A skin is a folder with two required files:

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

### theme.css

All colors as CSS variables in `:root`. Style elements with `var()`. Never hardcode colors in selectors.

```css
:root {
  /* Required contract vars */
  --t-foreground: #e0e0e8;
  --t-background: #0a0a0f;
  --c-chrome-bg: #0a0a0f;
  --c-chrome-text: #9898a8;
  --c-chrome-border: #1a1a2e;
  --c-chrome-accent: #d4a036;
  --c-chrome-dim: #606070;
  --c-chrome-hover: #1e1e32;

  /* Your custom vars */
  --skin-panel-bg: #070b07;
  --skin-overlay: rgba(7, 11, 7, 0.15);

  /* Custom effects — auto-appear as toggles in Settings */
  --effect-scanlines: 1;
  --effect-my-custom-effect: 0.5;
}

/* Style everything with var() */
.moltamp-panel-left { background: var(--skin-panel-bg); }

/* Reactive — changes with Claude's state */
[data-activity="high"] .moltamp-vibes {
  filter: brightness(1.2) saturate(1.1);
}

[data-shell-state="thinking"] .moltamp-terminal {
  box-shadow: inset 0 0 20px rgba(100, 200, 255, 0.1);
}

/* Overlays — always include pointer-events: none */
.moltamp-vibes::before {
  content: '';
  position: absolute;
  inset: 0;
  background: var(--skin-overlay);
  mix-blend-mode: multiply;
  pointer-events: none;
}
```

### Full Specification

See [SKINNING.md](SKINNING.md) for the complete spec — every variable, every targetable class, reactive data attributes, custom effects, and the AI prompt block.

## The Rules

1. **All colors in `:root`** as CSS variables. `var()` everywhere else.
2. **Contract variables first** — override `--t-*` and `--c-chrome-*`.
3. **Never hide panels** — users control visibility, skins control style.
4. **No executable content** — no external URLs, `@import`, or JS.
5. **No backgrounds on `.moltamp-vibes`** — use GIF widget slots in `skin.json`.
6. **`pointer-events: none` on every `::before`/`::after`**.
7. **Assets in `assets/`** — PNG, JPG, WebP, GIF, SVG, AVIF. Max 5MB/file.

MOLTamp enforces these at runtime with preflight overrides — skins literally cannot break clicks, hovers, or drag handles.

## Custom Effects

Any `--effect-*` variable in `:root` automatically appears in the Effects settings panel as a user toggle with an intensity slider.

```css
:root {
  --effect-radar: 1;       /* "Radar" toggle at 100% */
  --effect-heat-haze: 0;   /* "Heat Haze" toggle, off by default */
}
```

Use the variable to control visibility:
```css
.moltamp-vibes::before {
  opacity: var(--effect-radar);
  animation: radar-sweep 4s linear infinite;
  pointer-events: none;
}
```

Add nice labels in `skin.json`:
```json
{
  "effects": {
    "radar": { "label": "Sonar Radar", "description": "Rotating sweep overlay" }
  }
}
```

## Reactive Data

Skins can react to Claude's live state via data attributes on `.moltamp-shell`:

| Attribute | Values |
|-----------|--------|
| `data-activity` | `idle`, `low`, `high` |
| `data-shell-state` | `idle`, `thinking`, `streaming`, `tool-use`, `permission`, `error`, `complete` |
| `data-model` | Model name string |

And live CSS custom properties:

| Property | Description |
|----------|-------------|
| `--data-context-pct` | Context window used (0-100) |
| `--data-cost-cents` | Session cost in cents |
| `--data-tokens-in` | Input tokens (thousands) |
| `--data-tokens-out` | Output tokens (thousands) |
| `--data-rate-5h` | 5-hour rate limit (0-100) |
| `--data-agents` | Active subagent count |
| `--data-git-changed` | Changed file count |

## Using AI to Generate Skins

Point ChatGPT, Claude, Codex, or any AI at the [SKINNING.md](SKINNING.md) spec. It contains a ready-to-paste prompt block in the "For AI-generated skins" section.

MOLTamp's preflight system auto-fixes common AI mistakes (missing `pointer-events: none`, vibes backgrounds, etc.), so AI-generated skins work safely even when the AI doesn't follow every rule perfectly.

## Contributing

1. Fork this repo
2. Create your skin in `skins/your-skin-name/`
3. Include `skin.json`, `theme.css`, and optionally `assets/` + `preview.png`
4. Open a PR

### PR Checklist

- [ ] `skin.json` has all required fields
- [ ] All colors are CSS variables in `:root`
- [ ] Custom variables use `--skin-*` prefix
- [ ] No hardcoded colors in selectors
- [ ] Every `::before`/`::after` has `pointer-events: none`
- [ ] No `background` on `.moltamp-vibes`
- [ ] No external URLs or `@import`
- [ ] Assets under 5MB each, 20MB total
- [ ] `preview.png` screenshot included
- [ ] Right-click menus work on all panels
- [ ] Tested at different panel sizes

## License

MIT
